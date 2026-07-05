<p class="chapter-kicker">Part I · Chapter 4</p>

# Cleaning, shaping, and the reality of messy data

Here is the thing no one tells you before your first real dataset: cleaning *is* the job. The tutorials show tidy tables where every date is a date and every category is spelled the same way twice. Real data is not like that. It has blank cells, duplicated rows, three spellings of "Netherlands," a date field where someone typed "last Tuesday," numbers stored as text, and a column whose meaning changed halfway through the year because a form got redesigned. The folklore figure, that data professionals spend eighty per cent of their time on cleaning and preparation, is not much of an exaggeration. It is most of the work, and where most of the judgement lives.

That word, *judgement*, is the point. Cleaning is not janitorial tidying before the real analysis. Every cleaning decision is an editorial decision about what the data will be allowed to say. Drop rows with missing values and you've decided those cases don't count. Merge "NL," "Netherlands" and "The Netherlands" into one category and you've made a reasonable call, but merge "Holland" in too and you may have erased a distinction someone cared about. Cleaning is editing, and editing is never neutral.

!!! critical "Critical Lens — 'Raw data' is an oxymoron"
    A slogan worth internalising: *data is always already cooked, never truly raw.* By the time a number reaches you it has been defined, measured, categorised, entered and stored, each step a human choice. The comfortable phrase "the data shows…" hides all of that. A more truthful phrase is "the data, as collected and cleaned by these people for these purposes, is consistent with…". It's clumsier, and it's true. The big-data era's seductive promise was that enough volume would let the numbers speak for themselves; they never do. They speak in the voice of whoever prepared them, and part of your job is to keep that voice audible.

## The common problems, and principled responses

You'll meet the same handful of messes over and over. Let's build a deliberately dirty table and fix it.

```python
import pandas as pd
import numpy as np

dirty = pd.DataFrame({
    "customer": ["A. Jansen", "B. de Vries", "A. Jansen", "C. Bakker", "D. Smit"],
    "country":  ["Netherlands", "netherlands", "Netherlands", "The Netherlands", "Belgium"],
    "signup":   ["2026-01-03", "2026-01-05", "2026-01-03", "03/01/2026", "2026-02-11"],
    "spend":    ["149", "39", "149", "unknown", "259"],
})
dirty
```

### Missing values

The first question is never "how do I fill these in" but "*why* are they missing?" A blank because a sensor failed at random is very different from a blank because a question was skipped by everyone it made uncomfortable; the second isn't missing at random, and quietly deleting or averaging over it will bias everything downstream.

```python
df = dirty.copy()

# "unknown" is really a missing value in disguise — mark it as one.
df["spend"] = df["spend"].replace("unknown", np.nan)

df.isna().sum()          # how many missing per column?
# Two defensible options, chosen deliberately:
# df = df.dropna(subset=["spend"])          # drop rows we can't use
# df["spend"] = df["spend"].fillna(0)       # or fill — but say why
```

### Numbers and dates stored as text

A column that looks numeric but is secretly text will sort "10" before "9" and refuse to add up; a misread date can land events in the wrong century.

```python
df["spend"]  = pd.to_numeric(df["spend"], errors="coerce")   # text -> numbers
df["signup"] = pd.to_datetime(df["signup"], errors="coerce", dayfirst=False)
df.dtypes
```

`errors="coerce"` turns anything unparseable into a missing value rather than crashing, which surfaces problems instead of hiding them.

### Inconsistent categories

The "Netherlands" problem needs a *documented mapping*, not a silent fix.

```python
mapping = {
    "netherlands": "Netherlands",
    "the netherlands": "Netherlands",
    "Netherlands": "Netherlands",
    "Belgium": "Belgium",
}
df["country"] = df["country"].str.strip().str.lower().map(
    {k.lower(): v for k, v in mapping.items()}
)
df["country"].value_counts()
```

### Duplicates

Duplicates inflate whatever they touch. Find out whether a repeated row is a genuine second event or the same event recorded twice.

```python
df.duplicated().sum()            # how many exact duplicate rows?
df = df.drop_duplicates()        # keep the first of each
```

**How it works.** Each step is small and reversible, and each makes a *decision visible*: `replace(...np.nan)` declares that `"unknown"` means "no data"; `to_numeric`/`to_datetime` with `errors="coerce"` convert types while turning bad values into clean blanks; the mapping dictionary records exactly how categories were merged; `drop_duplicates` collapses repeats. The right response to every mess is the same: **decide deliberately, and write down what you did.**

!!! practice "In Practice — The provenance note"
    From your very first dataset, keep a short **provenance note** beside your analysis, a running log in plain language: where the data came from and when you downloaded it; every transformation and why ("merged three spellings of Netherlands into one"; "dropped 214 rows missing a delivery date, about 3%, because we can't analyse timing without it"); and any judgement calls a reader should know. It takes minutes and does three things: it lets you retrace your steps a month later, it lets a colleague check your work, and it forces you to notice when a "small tidy-up" is actually a decision about what the data may say. You'll reuse this note in your Part I project; it's the single most professional habit in the book.

    ```markdown
    ## Provenance note — signups.csv
    - Source: export from Acme dashboard, downloaded 2026-03-01
    - "unknown" in `spend` treated as missing (5 rows)
    - `spend` converted text -> number; `signup` parsed to dates
    - country spellings merged: {netherlands, the netherlands} -> Netherlands
    - 1 exact-duplicate row removed
    - Open question: are missing spends random, or concentrated in free-tier?
    ```

## Working with dates and time

Dates deserve their own section, because temporal data is everywhere (events, transactions, signups, sensor readings), and it's where sloppy handling does the most silent damage. The first rule you already met above: get dates out of text and into a real datetime with `pd.to_datetime`. Once a column is a genuine datetime, a whole toolbox opens up through the `.dt` accessor, which pulls the *parts* out of a timestamp:

```python
import pandas as pd
trips = pd.DataFrame({
    "trip_id":    [1, 2, 3, 4, 5, 6],
    "start_time": pd.to_datetime([
        "2026-05-01 08:12", "2026-05-01 08:40", "2026-05-01 17:30",
        "2026-05-02 09:05", "2026-05-02 18:15", "2026-05-03 12:20"]),
})

trips["hour"]    = trips["start_time"].dt.hour         # 8, 8, 17, 9, 18, 12
trips["weekday"] = trips["start_time"].dt.day_name()   # 'Friday', ...
trips["date"]    = trips["start_time"].dt.floor("D")   # midnight of that day
```

**How it works.** The `.dt` accessor turns one timestamp into many usable variables (`hour`, `day_name()`, `month`, `date`), each a new column you can group by or filter on. This is how a single "when" becomes analysable: "rides by hour of day," "signups by weekday." And to see a trend *over* time, **resample**, the time-aware cousin of `groupby`, which buckets rows into calendar periods:

```python
# how many trips per calendar day?
daily = trips.set_index("start_time").resample("D")["trip_id"].count()
daily      # one number per day — the raw material of a trend line
```

`resample("D")` groups by day; `"W"` by week, `"M"` by month, `"h"` by hour. Set the datetime as the index first, and pandas handles the calendar arithmetic (leap years, month lengths) for you.

!!! practice "In Practice — The ambiguous-date trap"
    `03/01/2026` is the 3rd of January in Europe and the 1st of March in the US, and software guesses, often wrong, and often silently. When parsing, be explicit: `pd.to_datetime(col, dayfirst=True)` for European data, and always spot-check a few rows after parsing. A date misread once at intake will quietly place events in the wrong month for the rest of your analysis, and nothing will flag it. Note the assumption in your provenance note.

!!! case "Case study — the bike-share's daily rhythm"
    Using the running `trips` data, find when the city rides. Extracting the hour turns a pile of timestamps into a clear daily pattern:

    ```python
    trips["hour"] = trips["start_time"].dt.hour
    trips.groupby("hour")["trip_id"].count()   # counts cluster around commute peaks
    ```

    Even in six rows you can see the shape of a commuter system: mornings around 8–9, evenings around 17–18. Time turned a flat list of trips into a story about how the city moves, which is exactly what the visualisation chapter will draw.

## Shaping: getting data into the form the question needs

Beyond cleaning sits *shaping*, reorganising well-formed data into the layout your question needs. The most common version is the move between "wide" and "long": one column per month versus one row per month. Neither is more correct; each suits different questions and tools, and a surprising amount of practical work is just reshaping fluently between them.

```python
# wide -> long
wide = pd.DataFrame({"city": ["Utrecht", "Amsterdam"],
                     "jan": [120, 90], "feb": [135, 110]})
long = wide.melt(id_vars="city", var_name="month", value_name="signups")

# long -> wide again
back = long.pivot(index="city", columns="month", values="signups")
```

The underlying skill is holding both shapes in your head (what the data looks like now, and what your next step needs), and knowing the gap is usually one reshaping move.

!!! further "Going Further — From personal habit to engineered pipeline"
    The provenance note is the human-scale version of what industry does at machine scale. When cleaning must run daily on new data, teams encode it as a **pipeline** with automated **validation**, rules that reject a batch if a supposedly-unique ID repeats or a percentage lands outside 0–100. This descends from a very old idea: double-entry bookkeeping's built-in balancing check, and later the database world's **ACID** guarantees, which insist a transaction completes wholly or not at all. You don't need pipelines for coursework, but knowing they exist shows where the discipline of your provenance note eventually leads: trustworthy data is *engineered*, not found.

Cleaning is unglamorous, and it is where integrity is won or lost. Do it deliberately, write it down, and you've already separated yourself from most people who "work with data." Now that the data is trustworthy, we can finally look at it.

## Exercises

??? exercise "Exercise 4.1 · Clean the dirty table end to end"
    Starting from the `dirty` dataframe above, produce a clean version where `spend` is numeric, `signup` is a real date, `country` has exactly two consistent values, and no duplicate rows remain. Then write a five-line provenance note describing what you did.

    ??? note "Show a solution"
        The code in this chapter, run in order, is a complete solution. The part that separates a good submission from a passing one is the note: it should state not just *what* you changed but *why*, and flag at least one open question (for example, whether the row you dropped as a duplicate might have been a real second purchase). The grader is reading for judgement, not for clean code.

??? exercise "Exercise 4.2 · Missing, but why?"
    Take a real dataset with missing values. For one column, investigate whether the blanks are missing *at random* or *systematically*, e.g. compare rows with and without the value on some other column. Write one sentence on what you found and how it should change the way you handle those blanks.

    ??? note "A worked example"
        ```python
        # Are missing incomes concentrated in a particular age group?
        df["income_missing"] = df["income"].isna()
        df.groupby("income_missing")["age"].mean()
        ```
        If the mean age differs sharply between the two groups, the data is *not* missing at random; filling the blanks with the overall average would quietly distort every age-based conclusion. The finding changes the fix.
