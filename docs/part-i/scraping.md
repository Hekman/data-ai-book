<p class="chapter-kicker">Part I · Chapter 10 · <span class="week">Week 7</span></p>

# Collecting data: Scraping the web

!!! guiding "The question this chapter answers"
    What do I do when the data I need has no API — how do I scrape it, and when am I allowed to?

Back in [Chapter 6](apis.md) you collected data the polite way, through the front door. This chapter is the fallback for when there is no door: the data exists only inside a web page, formatted for human eyes, and getting it means reading the page's underlying HTML and pulling out the parts you want. **Scraping** is more fragile than an API call — a redesign can break your code overnight — and it carries more obligations, because you're taking data the source never packaged for you.

So the skill here is double. Getting the data *and* knowing when you shouldn't.

!!! matters "Why this matters for designers"
    Scraping is where acquisition meets ethics most sharply. Data being visible in a browser does not make it yours to take at scale. A designer who scrapes responsibly — respecting consent, terms of service, rate limits, and personal data — avoids building work on a foundation that is fragile, unfair, or simply illegal. Knowing when *not* to scrape is as much a part of the craft as the code.

!!! newsroom "Why this matters for journalists"
    Scraping is a reporting method, and like any method it comes with questions you should be able to answer before publication rather than after: was the material genuinely public, does the public interest outweigh the terms you're stretching, and would you be comfortable describing exactly what you did in the methodology box? Personal data raises the bar sharply. Keep the raw pages you collected and the date you collected them — a website is a moving target, and the cache is your evidence when the page changes and someone says it never said that.

!!! outcomes "By the end of this chapter you can"
    - Reach for `pd.read_html` first, and recognise when the data is already in a table.
    - Read a page's HTML structure and select the elements you want with CSS selectors, choosing robust ones over generated class names.
    - Scrape a multi-page site into a clean dataframe with BeautifulSoup, bounding and pacing your requests.
    - Separate fetching from parsing, caching pages to disk so your scrape is fast, polite and reproducible.
    - Judge what is legal and ethical to scrape: `robots.txt`, terms of service, rate limits, personal data.
    - Choose deliberately between an API and scraping for a given data need, and say why.
    - Recognise a JavaScript-rendered page, and look for the underlying API rather than reaching for a browser.

!!! origins "Origins — The web as a structure of data"
    It helps to remember what the web *is*. Tim Berners-Lee released it in 1991 as a universal system of linked documents and gave it to the public domain, turning the internet into humanity's primary medium for publishing, and therefore the largest generator of data in history. A few years later Larry Page and Sergey Brin noticed that the *links between* pages were themselves data: PageRank treated each hyperlink as a vote and computed authority from the web's graph structure, and built an empire on it. Both facts matter to a collector: the web is a vast, semi-structured dataset, and its *structure*, not just its text, is often the thing worth harvesting.

## Try the one-liner first

Before writing any scraper, check whether the data is in an HTML `<table>`, because if it is, pandas will do the entire job for you:

```python
import pandas as pd

tables = pd.read_html("https://en.wikipedia.org/wiki/List_of_cities_in_the_Netherlands")   # (1)!
len(tables)          # a LIST of every table on the page
df = tables[0]       # pick the one you want
df.head()
```

1.  Needs `lxml` (`pip install lxml`) and, given a URL, an internet connection. It also accepts an HTML string or a local file — wrap a string as `pd.read_html(io.StringIO(html))` — which is how you'd use it on a page you cached earlier.

**How it works.** `read_html` finds every `<table>` element on the page and returns a *list* of dataframes, one per table — which is why the second line is `len(tables)` rather than an assumption. You inspect the list, find your table by index, and you're done: no selectors, no loop, no BeautifulSoup. Two arguments are worth knowing: `match="some text"` filters to tables containing a string, which saves guessing indices, and `header=0` fixes the common case where the header row isn't detected.

Try this every single time before you reach for the rest of the chapter. A surprising share of real scraping tasks — statistics pages, reference tables, published league tables — are one line, and students routinely write forty lines of BeautifulSoup for data `read_html` would have handed over. You still need everything below for pages whose data *isn't* in a table, which is most of the modern web.

## Reading a page's structure

Before you write any code, open the page and inspect it in your browser's developer tools. Scraping is mostly a *reading* task: finding the repeating structure that holds the data. Almost always there is one — a table, a list of cards, a set of rows — because the page was generated from a database in the first place, and your job is to work backwards to that shape.

The selectors you use to find it are the same CSS selectors you already know from styling: `#prices` for an id, `.card` for a class, `table tr td` for a descendant path.

A word on *which* selector to choose, because this decides whether your scraper still works next month. Modern sites are built by tools that generate class names like `.css-1x7bqm` or `.jsx-2847193`, and those change on every deployment. Prefer selectors anchored to something meaningful: an `id`, a semantic element (`<article>`, `<table>`, `<h2>`), a `data-` attribute, or a class whose name suggests a human chose it (`.product-price` will outlive `.sc-bdVaJa`). When you have no choice but a fragile selector, say so in a comment — the next person to run your code, probably you, needs to know it's the first thing to check when the scrape returns nothing.

```python
from bs4 import BeautifulSoup
import pandas as pd

html = """
<table id="prices">
  <tr><th>product</th><th>price</th></tr>
  <tr><td>chair</td><td>149</td></tr>
  <tr><td>lamp</td><td>39</td></tr>
  <tr><td>desk</td><td>259</td></tr>
</table>
"""

soup = BeautifulSoup(html, "html.parser")
rows = []
for tr in soup.select("#prices tr")[1:]:      # skip the header row
    cells = [td.get_text(strip=True) for td in tr.find_all("td")]
    rows.append(cells)

scraped = pd.DataFrame(rows, columns=["product", "price"])
scraped["price"] = pd.to_numeric(scraped["price"])   # text -> number
scraped
```

**How it works.** `BeautifulSoup` turns messy HTML into a searchable tree. `soup.select("#prices tr")` uses a CSS selector to grab the table's rows; `get_text(strip=True)` pulls the visible text out of each cell. The result is a list of lists, which becomes a dataframe. Notice the last line: scraped values arrive as *text*, so you're straight back into [Chapter 3](cleaning-messy-data.md)'s type conversion. Collection and cleaning are the same discipline, one step apart.

Here we parse an HTML string so the example is self-contained; in the wild you'd feed it `requests.get(page_url).text` — the same `requests` call from the API chapter, just asking for a page instead of JSON.

## Crawling across pages

One page is rarely the dataset. The data you want is usually spread across a paginated list, and collecting it means walking the pages and appending as you go — with the same courtesies as paginated API collection, and more need for them.

```python
import requests, time
from bs4 import BeautifulSoup
import pandas as pd

rows = []
for page in range(1, 6):                         # a hard bound, always
    r = requests.get(f"https://example.com/list?page={page}",
                     headers={"User-Agent": "HU student project (you@example.com)"},  # (1)!
                     timeout=10)
    if not r.ok:
        break
    soup = BeautifulSoup(r.text, "html.parser")
    cards = soup.select(".card")
    if not cards:                                # nothing left to collect
        break
    for c in cards:
        rows.append({"title": c.select_one(".title").get_text(strip=True),
                     "price": c.select_one(".price").get_text(strip=True)})
    time.sleep(1)                                # (2)!

df = pd.DataFrame(rows)
```

1.  Identify yourself. A `User-Agent` naming your project and a contact address is what separates a researcher from an anonymous bot, and it lets an administrator ask you to stop rather than simply blocking you.
2.  One second between requests. Slower than you'd like, kinder than you strictly need to be, and the single easiest way to stay welcome.

**How it works.** The loop is bounded (`range(1, 6)`), breaks when a request fails, and breaks again when a page yields no items — three independent stop conditions, because a runaway crawler is the classic beginner mistake. `select_one` grabs the first match inside each card, so each card contributes one dictionary, and a list of dictionaries becomes a dataframe directly.

## Collecting politely, and legally

Producing data from the web comes with a code of conduct, and it isn't optional.

**Read the terms.** Many sites specify what may be collected and how, and ignoring that can breach a contract or worse. **Check `robots.txt`** (at `example.com/robots.txt`): it states which paths the site asks automated clients to leave alone. It isn't a law, but disregarding it is a deliberate choice you would have to defend. **Respect rate limits.** Space requests out, cache what you've fetched, and never re-scrape what you already have. **Take only what you need**, and think hard before collecting anything about identifiable people — the [personal-data duties from Chapter 6](apis.md#handling-personal-data) apply here with more force, not less, because nobody handed you these records deliberately. Data being *visible* on a public page does not make it *yours* to compile, and aggregating scattered public information into one dataset can create something far more sensitive than any of its parts.

!!! practice "In Practice — Fetch once, save the HTML, parse many times"
    You will get your selectors wrong four or five times before they work. Re-fetching the page on each attempt is slow, rude, and a good way to get blocked — so separate *fetching* from *parsing*, and cache the raw HTML on disk:

    ```python
    import pathlib, requests, time
    from bs4 import BeautifulSoup

    def get_page(url, cache_dir="cache"):
        path = pathlib.Path(cache_dir) / (url.replace("/", "_")[-100:] + ".html")
        path.parent.mkdir(exist_ok=True)
        if path.exists():
            return path.read_text(encoding="utf-8")     # (1)!
        r = requests.get(url, headers={"User-Agent": "HU student project (you@example.com)"},
                         timeout=10)
        r.raise_for_status()
        path.write_text(r.text, encoding="utf-8")
        time.sleep(1)
        return r.text

    soup = BeautifulSoup(get_page("https://example.com/list?page=1"), "html.parser")
    ```

    1.  Second and every later run: instant, offline, and zero requests to the server.

    Three payoffs, and the third is the one that matters most. You can iterate on your parsing as fast as you can type. You stay welcome on the site. And the cached HTML *is* your evidence — a web page is a moving target, so the only way anyone can check your scrape six months from now is if you kept what the page said on the day. Commit the cache, note the fetch date, and your scraping becomes as reproducible as reading a CSV.

!!! practice "In Practice — The four-question go/no-go check"
    Before you write a scraper, answer these in your provenance note: (1) Is there an API or a published dataset that would give me this legitimately? (2) What do the terms of service and `robots.txt` say about automated access? (3) Am I collecting anything about identifiable people, and could I do this without those fields? (4) What is my request budget — how many pages, how fast, and how will I cache so I fetch each one only once? If question 1 has a yes, stop and use it. Most "I need to scrape this" turns out to be "I didn't look for the API."

!!! critical "Critical Lens — From missing to made is a moral act"
    Recall Onuoha's *Library of Missing Datasets* from [Chapter 5](where-data-comes-from.md), the things that don't get counted. Collecting your own data is how a missing dataset becomes a made one, and that is genuine power: you can bring into being a record of something the world declined to measure. But making a dataset is *taking a position*. You decide what counts as an instance, what categories exist, who is represented and who is left out — the very decisions you spent Chapter 5 learning to see in *other people's* data are now yours to answer for. Collect with the same suspicion you'd bring to a dataset handed to you, because someone downstream will inherit yours and have to ask where it came from. Make sure the answer is on record.

!!! further "Going Further — XPath, Scrapy, and pages that build themselves"
    Three extensions you'll meet. **XPath** is a second selector language that expresses things CSS can't — "the row whose first cell contains 'total'", or walking *up* the tree to a parent — and `lxml` gives you it in Python. **Scrapy** is a full crawling framework: it manages a request queue, concurrency, retries, throttling and export pipelines, which is what you want the moment a scrape stops being one script and becomes a recurring job. And **client-side rendered pages** are the modern obstacle: if the HTML you fetch is nearly empty and the content appears only in the browser, the data is being loaded by JavaScript after the page arrives. The lazy answer is a headless browser (Playwright, Selenium); the better first move is to open the network tab and find the JSON API the page itself is calling — at which point you're back in [Chapter 6](apis.md), through the front door after all.

Collecting your own data closes the loop of Part I. You began by learning to read datasets other people made, seeing the decisions baked into their shape; now you make those decisions yourself, and answer for them. That symmetry — the same critical eye turned on your own production as on others' — is exactly the discipline this whole part has been building toward.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. Chapters 1–3 are what you need; Ch. 4 goes further into Scrapy than this book does.

    - [Web Scraping in Python](https://app.datacamp.com/learn/courses/web-scraping-with-python) — **Ch. 1 Introduction to HTML**, **Ch. 2 XPaths and Selectors**, **Ch. 3 CSS Locators, Chaining, and Responses.**
