<p class="chapter-kicker">Part I</p>

# Data as Material

**Goal:** get fluent with real, messy data: load it, question it, clean it, visualise it, collect your own, and stay clear-eyed about what it does and doesn't represent.

> **The question running through this part:** *What does this data actually let me know, and what stories can I tell?*

Everything here builds toward one project: a well-documented, trustworthy piece of work on a dataset you sourced yourself, a *data study* if you're coming at it as a designer, a *data story* if you're coming at it as a journalist, communicated so that a non-technical reader can follow it and trust it.

A single dataset, a synthetic **city bike-share**, runs through the whole part as a case study, so each skill lands on the same real-feeling data rather than a fresh toy each time.

## The path through Part I

We start with the **building blocks**, what data is actually made of, and pick up the **dataframe**, the single instrument you'll use for almost everything. We then spend real time on **cleaning**, because it is most of the work and nobody warns you, and on **combining datasets** into something richer. Only then do we ask **where a dataset comes from**, because by that point you've handled enough real data to feel why its origins matter, and learn to **ask a service for data** through an API. We learn to **summarise honestly**, to **argue with pictures**, and to wrap the analysis in a **small interactive tool**. Finally we learn to **scrape** what has no API. Staying truthful about the data is the thread running through all ten chapters.

## Chapters, week by week

The book follows the teaching weeks of Block A exactly. If you're taking the course, each week's reading is listed below; if you're reading independently, the weeks are simply a suggested pace.

### Week 1 · Building blocks & the dataframe

<div class="grid cards" markdown>

-   __1 · Data, variables & data structures__

    What data is made of: values, variable levels, and the structures that hold them.

    [:octicons-arrow-right-24: Read](data-variables-structures.md)

-   __2 · The dataframe__

    Your core instrument: loading a table, interrogating it, and five verbs that do 80% of data work.

    [:octicons-arrow-right-24: Read](the-dataframe.md)

</div>

### Week 2 · Cleaning & combining data

<div class="grid cards" markdown>

-   __3 · Cleaning & messy data__

    Most of the job. Missing values, types, dates, duplicates, and the cleaning log.

    [:octicons-arrow-right-24: Read](cleaning-messy-data.md)

-   __4 · Combining datasets__

    Joining tables on a key, stacking, and enriching one dataset with another.

    [:octicons-arrow-right-24: Read](combining-datasets.md)

</div>

### Week 3 · Where data comes from, and APIs

<div class="grid cards" markdown>

-   __5 · Where data comes from__

    Why a dataset exists shapes what it can say. Routes, samples, absences, formats.

    [:octicons-arrow-right-24: Read](where-data-comes-from.md)

-   __6 · Collecting data: APIs__

    Endpoints, JSON, a live language-model call, and the duties that come with personal data.

    [:octicons-arrow-right-24: Read](apis.md)

</div>

### Week 4 · Descriptive statistics

<div class="grid cards" markdown>

-   __7 · Descriptive statistics__

    Centre, spread, distributions, correlation ≠ causation, and where a summary misleads.

    [:octicons-arrow-right-24: Read](descriptive-statistics.md)

</div>

### Week 5 · Visualisation as argument

<div class="grid cards" markdown>

-   __8 · Visualisation as argument__

    A chart is a claim. The grammar of graphics, colour, and the ethics of the y-axis.

    [:octicons-arrow-right-24: Read](visualisation.md)

</div>

### Week 6 · Dashboards

<div class="grid cards" markdown>

-   __9 · Dashboards__

    From a chart to a small tool someone else can steer — and a default view that tells the truth.

    [:octicons-arrow-right-24: Read](dashboards.md)

</div>

### Week 7 · Scraping the web

<div class="grid cards" markdown>

-   __10 · Collecting data: Scraping the web__

    When there's no API: HTML structure, selectors, crawling politely, and when not to.

    [:octicons-arrow-right-24: Read](scraping.md)

</div>

## Set up your environment

The examples run in any Python notebook. In the courses, use the hosted notebooks, with nothing to install. If you'd rather run locally:

```bash
pip install pandas numpy matplotlib seaborn openpyxl
# chapters 6, 9 and 10 also use:
pip install requests plotly gradio beautifulsoup4 lxml
```

Every code block is written to be copy-paste runnable, and most build their own small dataset so they work with no internet connection. The chapters that do call the network (6 and 10) always include a fallback.
