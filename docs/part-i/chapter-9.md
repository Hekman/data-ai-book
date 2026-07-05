<p class="chapter-kicker">Part I · Chapter 9</p>

# Collecting your own data, scraping and APIs

Every dataset so far arrived pre-made. But the most interesting design questions often have no dataset waiting, the thing you want to study is scattered across a hundred web pages, or sitting behind a service that will hand it over if you ask correctly, or simply not collected by anyone yet. This chapter is about crossing from *consumer* of data to *producer* of it: pulling data from an **API**, gathering it by **scraping** web pages, and, the deeper theme, taking on the responsibilities that come the moment you make datasets instead of merely using them.

There are, broadly, two ways to collect from the web, differing mostly in whether the source *wants* to give you data. An **API** (application programming interface) is a front door: a service that offers its data in a clean, structured form, usually JSON, designed to be read by programs. When an API exists for what you need, use it; it's stable, sanctioned, and hands you tidy data. **Scraping** is the back door: extracting data from pages built for human eyes by parsing their underlying HTML. Scraping is more fragile, a redesign can break your code overnight, and carries more obligations, because you're taking data the source didn't package for you.

!!! origins "Origins — The web as a structure of data"
    It helps to remember what the web *is*. Tim Berners-Lee released it in 1991 as a universal system of linked documents and gave it to the public domain, turning the internet into humanity's primary medium for publishing, and therefore the largest generator of data in history. A few years later Larry Page and Sergey Brin noticed that the *links between* pages were themselves data: PageRank treated each hyperlink as a vote and computed authority from the web's graph structure, and built an empire on it. Both facts matter to a collector: the web is a vast, semi-structured dataset, and its *structure*, not just its text, is often the thing worth harvesting.

## Reading an API

An API call is just a web request that returns data instead of a page. You send a URL with some parameters; you get back JSON, which maps neatly onto Python dictionaries and then into a dataframe.

```python
import requests
import pandas as pd

url = "https://api.open-meteo.com/v1/forecast"
params = {"latitude": 52.09, "longitude": 5.12,      # Utrecht
          "hourly": "temperature_2m", "forecast_days": 1}

try:
    r = requests.get(url, params=params, timeout=10)   # (1)!
    r.raise_for_status()                               # fail loudly on errors
    payload = r.json()
    hourly = payload["hourly"]
    df = pd.DataFrame({"time": hourly["time"],
                       "temp_c": hourly["temperature_2m"]})
except Exception as e:
    print("Live call failed, using a fallback sample:", e)   # (2)!
    df = pd.DataFrame({"time": ["2026-07-05T00:00", "2026-07-05T01:00"],
                       "temp_c": [17.2, 16.8]})

df.head()
```

1.  `timeout=10` means "give up after 10 seconds", never let a network call hang forever.
2.  Always assume the network can fail. A fallback keeps your notebook runnable offline and during grading, and models the defensive habit real collection needs.

**How it works.** `requests.get` performs the call; `raise_for_status()` turns an HTTP error (a 404, a rate-limit 429) into a Python exception instead of silently returning junk. `r.json()` parses the response into nested dictionaries and lists, and from there it's ordinary pandas: pick the fields you want and build a dataframe. The `try/except` isn't boilerplate, it's the Chapter 4 cleaning mindset applied at the moment of *intake*: check that what came back is what you expected before you trust it.

Most APIs hand back results in **pages** rather than all at once, so collecting a full dataset means politely looping:

```python
# Sketch of paginated collection (pseudo-real):
all_rows = []
for page in range(1, 6):                     # first five pages only
    resp = requests.get(url, params={**params, "page": page}, timeout=10)
    if not resp.ok:
        break
    all_rows.extend(resp.json().get("results", []))
    # time.sleep(1)  # be kind: pause between requests
collected = pd.DataFrame(all_rows)
```

!!! case "Case study — replace the synthetic weather with the real thing"
    In [Chapter 5](chapter-5.md) the bike-share's `weather` table was hand-made. Now you can collect it for real and join it in, closing the loop of Part I. Pull the daily weather for the trip dates from a public API, then merge exactly as before:

    ```python
    import requests, pandas as pd
    try:
        r = requests.get("https://archive-api.open-meteo.com/v1/archive", timeout=10,
                         params={"latitude": 52.09, "longitude": 5.12,
                                 "start_date": "2026-05-01", "end_date": "2026-05-03",
                                 "daily": "temperature_2m_max,precipitation_sum"})
        r.raise_for_status()
        d = r.json()["daily"]
        weather = pd.DataFrame({"date": pd.to_datetime(d["time"]),
                                "temp_c": d["temperature_2m_max"],
                                "rain_mm": d["precipitation_sum"]})
    except Exception as e:
        print("Live call failed, using fallback:", e)
        weather = pd.DataFrame({"date": pd.to_datetime(["2026-05-01","2026-05-02","2026-05-03"]),
                                "temp_c": [17.5, 14.2, 21.0], "rain_mm": [0.0, 6.4, 0.0]})
    weather
    ```

    The enrichment you faked to learn joins is now a real, sourced column, the exact move your project milestone asks for.

## Scraping a page

When there's no API, you parse the HTML directly with **BeautifulSoup**. Here we parse an HTML string so the example is self-contained; in the wild you'd feed it `requests.get(page_url).text`.

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

**How it works.** `BeautifulSoup` turns messy HTML into a searchable tree. `soup.select("#prices tr")` uses a CSS selector, the same selectors you know from styling, to grab the table's rows; `get_text(strip=True)` pulls the visible text out of each cell. The result is a list of lists, which becomes a dataframe. Notice the last line: scraped values arrive as *text*, so you're straight back into Chapter 4's type-conversion, collection and cleaning are the same discipline, one step apart.

## Collecting politely, and legally

Producing data from the web comes with a code of conduct, and it isn't optional. **Read the terms**, many sites and APIs specify what you may collect and how, and ignoring that can breach a contract or worse. **Respect rate limits**, hammering a server with rapid requests is at best rude and at worst indistinguishable from an attack, so space requests out and cache what you've fetched. **Take only what you need**, and think hard before collecting anything about identifiable people. Data being *visible* on a public page does not make it *yours* to compile, and aggregating scattered public information into one dataset can create something far more sensitive than any of its parts.

## Handling personal data

The moment your dataset is about *people*, a heavier set of duties applies, and "it was technically public" is not a defence. It helps to distinguish two dangers. **Personally identifiable information (PII)**, names, emails, phone numbers, precise location, national ID numbers, identifies someone directly. **Quasi-identifiers**, a postcode, a birth date, a job title, identify no one alone but pin down an individual *in combination*, the mosaic effect from [Chapter 5](chapter-5.md). Both need care, and the second is the one people forget.

A practical stance rests on three moves. **Minimise:** collect only the fields your question actually needs, every extra personal column is a liability to store and protect, not an asset. **Respect purpose and consent:** data given for one purpose shouldn't be silently repurposed, and scraping people's public posts into a research dataset is exactly such a repurposing, whatever the terms allow. **Pseudonymise or anonymise:** replace names with meaningless IDs (pseudonymisation), or aggregate and blur until individuals disappear (anonymisation), while remembering that true anonymisation is genuinely hard, because a determined join can re-identify what looked anonymous.

!!! origins "Origins — Why European data law starts from 'collect less'"
    Data-protection law was born of memory. Nazi Germany's censuses were tabulated on Hollerith punched-card machines, and meticulous population registries, the Netherlands' among the most complete, helped identify people for persecution. After the war, that history hardened into a principle: the safest data is the data you never collected. The German state of Hesse passed the first data-protection act in 1970, a lineage running straight to today's **GDPR**, with its insistence on data minimisation, purpose limitation and a right to erasure. When privacy law feels burdensome, it's worth remembering what it is a reaction to.

!!! practice "In Practice — A minimum-viable privacy check"
    Before collecting data about people, answer four questions in your provenance note: (1) Do I actually need each personal field, or am I collecting it because I can? (2) Would the people in this dataset be surprised or troubled to find themselves in it? (3) Can I pseudonymise now, swap names for IDs at the point of collection? (4) Could this dataset, joined to something public, re-identify someone? If any answer worries you, collect less. The most defensible personal data is the personal data you chose not to gather.

!!! critical "Critical Lens — From missing to made is a moral act"
    Recall Onuoha's *Library of Missing Datasets* from Chapter 2, the things that don't get counted. Collecting your own data is how a missing dataset becomes a made one, and that is genuine power: you can bring into being a record of something the world declined to measure. But making a dataset is *taking a position*. You decide what counts as an instance, what categories exist, who is represented and who is left out, the very decisions we spent Chapter 2 learning to see in *other people's* data are now yours to answer for. Collect with the same suspicion you'd bring to a dataset handed to you, because someone downstream will inherit yours and have to ask where it came from. Make sure the answer is on record.

!!! further "Going Further — The mechanics you'll actually hit"
    Three things separate a toy collector from a usable one. *Pagination:* loop politely through result pages rather than grabbing the first screen. *Shape validation:* an API response can change or arrive malformed, so check the fields you expect are present before trusting it. *Fallbacks:* assume the network will sometimes fail and degrade gracefully instead of crashing (the `try/except` above). None of this is advanced, but it's the difference between code that works in the demo and code that survives contact with the real web.

Collecting your own data closes the loop of Part I. You began by learning to read datasets other people made, seeing the decisions baked into their shape; now you make those decisions yourself, and answer for them. That symmetry, the same critical eye turned on your own production as on others', is exactly the discipline this whole part has been building toward.

## Exercises

??? exercise "Exercise 9.1 · Pull from a public API"
    Choose a free, no-auth API (weather, public transit, an open government portal) and pull a small dataset into a dataframe, with a `try/except` fallback so your notebook runs offline. Write down, in your provenance note, the exact endpoint, the parameters, and the date you fetched it.

    ??? note "Guidance"
        The code pattern is the Open-Meteo example above, swap the URL and the fields you extract. The mark is as much about the *provenance note* as the code: an API response is a snapshot in time, and a reader six months from now needs to know exactly what you asked for and when.

??? exercise "Exercise 9.2 · Enrich your Part I dataset"
    Add at least one new column to your main dataset by collecting complementary data (e.g. join weather onto event dates, or population onto city names). Write two sentences on a *new* question this enrichment makes answerable, and one sentence on a new bias or gap the collected data introduces.

    ??? note "What good looks like"
        Enrichment is where Part I comes together: you're now *producing* the missing column from Chapter 2's exercise. A strong answer is candid about the seam, "I joined city population, but my population figures are 2021 estimates while my events are from 2026, so any per-capita rate is slightly off." Naming the imperfection is the professional move.
