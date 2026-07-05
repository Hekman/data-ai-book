<p class="chapter-kicker">Part I</p>

# Data as Material

**Goal:** get fluent with real, messy data: load it, question it, clean it, visualise it, collect your own, and stay clear-eyed about what it does and doesn't represent.

Everything in this part builds toward one project: a well-documented, trustworthy piece of work on a dataset you sourced yourself, a *data study* if you're coming at it as a designer, a *data story* if you're coming at it as a journalist, communicated so that a non-technical reader can follow it and trust it.

## The path through Part I

We start with the **building blocks**, what data is actually made of: values, variables and the structures that hold them. Then we ask **where a dataset comes from**, because its origins already encode decisions you'd otherwise inherit blind. We meet the **dataframe**, the single instrument you'll use for almost everything, and spend real time on **cleaning**, because it is most of the work and nobody warns you. We learn to **combine datasets** into something richer, to **look before we model**, to **argue with pictures**, and to wrap our analysis in a **small interactive tool**. Finally we learn to **collect our own data** when the dataset we need doesn't exist. Staying truthful about the data is the thread running through all nine chapters.

A single dataset, a synthetic **city bike-share**, runs through the whole part as a case study, so each skill lands on the same real-feeling data rather than a fresh toy each time.

<div class="grid cards" markdown>

-   __1 · Data, variables & data structures__

    What data is made of: values, variable levels, and the structures that hold them.

    [:octicons-arrow-right-24: Read](chapter-1.md)

-   __2 · The origins of a dataset__

    Why a dataset exists shapes what it can say. Loading and interrogating data.

    [:octicons-arrow-right-24: Read](chapter-2.md)

-   __3 · The dataframe__

    Your core instrument: five verbs that do 80% of data work.

    [:octicons-arrow-right-24: Read](chapter-3.md)

-   __4 · Cleaning & messy data__

    Most of the job. Missing values, duplicates, and the provenance note.

    [:octicons-arrow-right-24: Read](chapter-4.md)

-   __5 · Combining datasets__

    Joining tables on a key, stacking, and enriching one dataset with another.

    [:octicons-arrow-right-24: Read](chapter-5.md)

-   __6 · Exploratory analysis__

    Look before you model. Summary statistics, distributions, correlation ≠ causation.

    [:octicons-arrow-right-24: Read](chapter-6.md)

-   __7 · Visualisation as argument__

    A chart is a claim. The grammar of graphics, colour, and the ethics of the y-axis.

    [:octicons-arrow-right-24: Read](chapter-7.md)

-   __8 · Dashboards__

    From a chart to a small tool someone else can use.

    [:octicons-arrow-right-24: Read](chapter-8.md)

-   __9 · Collecting your own data__

    APIs and scraping, and the responsibilities of making datasets.

    [:octicons-arrow-right-24: Read](chapter-9.md)

-   :material-flag-checkered:{ .lg .middle } __Project milestone__

    The Part I deliverable and how it's assessed.

    [:octicons-arrow-right-24: Brief](milestone.md)

</div>

## Set up your environment

The examples run in any Python notebook. In the course, use the hosted notebooks, with nothing to install. If you'd rather run locally:

```bash
pip install pandas matplotlib openpyxl
# chapters 8 and 9 also use:
pip install gradio requests beautifulsoup4
```

Every code block is written to be copy-paste runnable, and most build their own small dataset so they work with no internet connection.
