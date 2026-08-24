<p class="chapter-kicker">Part I · Chapter 3 · <span class="week">Week 2</span></p>

# Cleaning, shaping, and the reality of messy data

!!! guiding "The question this chapter answers"
    Real data is messy — what do I fix, what do I drop, what do I leave, and how do I stay honest about it?

Here is the thing no one tells you before your first real dataset: cleaning *is* the job. The tutorials show tidy tables where every date is a date and every category is spelled the same way twice. Real data is not like that. It has blank cells, duplicated rows, three spellings of "Netherlands," a date field where someone typed "last Tuesday," numbers stored as text, and a column whose meaning changed halfway through the year because a form got redesigned. The folklore figure, that data professionals spend eighty per cent of their time on cleaning and preparation, is not much of an exaggeration. It is most of the work, and where most of the judgement lives.

That word, *judgement*, is the point. Cleaning is not janitorial tidying before the real analysis. Every cleaning decision is an editorial decision about what the data will be allowed to say. Drop rows with missing values and you've decided those cases don't count. Merge "NL," "Netherlands" and "The Netherlands" into one category and you've made a reasonable call, but merge "Holland" in too and you may have erased a distinction someone cared about. Cleaning is editing, and editing is never neutral.

!!! matters "Why this matters for designers"
    A chart built on silently mishandled data is a beautiful lie. The credibility of everything you design downstream rests on choices made here. Professionals are distinguished not by having clean data but by *defensible cleaning*: being able to say exactly what they did to a missing value and why. The deliverable habit of this chapter is the cleaning log — one line per transformation — so your work stays reproducible and defensible.

!!! newsroom "Why this matters for journalists"
    Cleaning is where a data story acquires its methodology — or loses it. When an editor, a lawyer, or the agency that published the figures asks *how do you know*, the cleaning log is the answer: every decision you made, in order, with a reason. It is also what makes a correction survivable, because you can retrace exactly where a number came from. Reporters who delete the awkward rows and report one clean figure are the ones who end up retracting.

!!! outcomes "By the end of this chapter you can"
    - Diagnose the common data-quality problems: wrong types, missing values, duplicates, inconsistent categories, impossible values.
    - Choose between drop, impute and flag for a missing value, and justify the choice rather than defaulting to one.
    - Convert text to numbers and dates safely with `errors="coerce"`, and avoid the ambiguous-date trap.
    - Normalise messy text with the `.str` accessor, and tell real category variants from whitespace and capitals.
    - Define duplicates on the columns that identify a thing, not just whole identical rows.
    - Separate impossible values from merely extreme ones, and flag rather than silently delete.
    - Write `assert` checks that make your assumptions fail loudly when the data changes.
    - Reshape data between wide and long form with `melt` and `pivot`, and extract usable variables from a timestamp with `.dt` and `resample`.
    - Keep a cleaning log / provenance note that makes every transformation reproducible and defensible.

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

### Text that only looks consistent

Before you map categories, normalise the text, because most "inconsistent categories" are the same word wearing different whitespace and capitals. Three fixes handle the overwhelming majority:

```python
s = pd.Series([" Utrecht", "utrecht ", "UTRECHT", "Utre  cht", "Utrecht\n"])

s.str.strip()                      # remove leading/trailing whitespace — including \n and \t
s.str.strip().str.lower()          # case-fold, so Utrecht == utrecht == UTRECHT
s.str.strip().str.replace(r"\s+", " ", regex=True)   # collapse runs of inner whitespace

s.nunique()                                        # 5 — five "different" cities
s.str.strip().str.lower().nunique()                # 2 — after normalising
```

**How it works.** `.str` is the accessor that applies a string operation down a whole column, the text equivalent of the `.dt` accessor for dates. `strip()` deals with the invisible characters that sneak in from copy-paste and CSV exports — a trailing space is genuinely undetectable by eye and will absolutely break a join in [Chapter 4](combining-datasets.md). `lower()` collapses capitalisation. The regex replace fixes doubled inner spaces, which `strip` doesn't touch.

That `nunique()` pair is the diagnostic worth running on every text column you inherit: if the count drops sharply after normalising, you have found silent duplicates that were inflating your categories and splitting your groups. Note the direction of the judgement, though — `lower()` is safe for city names and *wrong* for anything case-sensitive like a product code or an ID.

### Inconsistent categories

Once the text is normalised, merging the genuine variants needs a *documented mapping*, not a silent fix.

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

# More often, "duplicate" means duplicate on the columns that identify a thing:
df.duplicated(subset=["customer", "signup"]).sum()   # same person, same day
df.drop_duplicates(subset=["customer"], keep="last")  # keep the most recent record
```

The `subset` argument is the one you'll actually need. Exact whole-row duplicates are rare in real data, because some irrelevant column usually differs; the meaningful question is "is this the same *thing* recorded twice", and only you know which columns identify a thing. `keep="first"`, `"last"` or `False` (drop all copies) is a third decision — and if two records of the same customer disagree, choosing which to keep is an editorial act that belongs in the log.

### Impossible values

Missing data announces itself. Impossible data doesn't: it sits in the column looking like a number and quietly poisons every average. An age of `999`, a `duration_min` of `-4`, a signup dated 1900, a percentage of `140` — these are usually sentinels, unit mix-ups, or data-entry slips, and the only way to find them is to state what *possible* means and check.

```python
import pandas as pd

trips = pd.DataFrame({
    "trip_id":      [1, 2, 3, 4, 5],
    "duration_min": [7, 22, -4, 9, 1440],     # a negative, and a 24-hour "trip"
    "age":          [31, 27, 999, 44, 22],    # 999 is a sentinel, not an age
})

# 1. Look at the edges — the cheapest possible check
trips[["duration_min", "age"]].describe().loc[["min", "max"]]

# 2. State the rule, then find what breaks it
impossible = (trips["duration_min"] <= 0) | (trips["duration_min"] > 240) | (trips["age"] > 120)
trips[impossible]                             # inspect before you touch anything

# 3. Flag rather than silently delete
trips["duration_suspect"] = (trips["duration_min"] <= 0) | (trips["duration_min"] > 240)
trips.loc[trips["age"] > 120, "age"] = pd.NA  # a sentinel IS missing data — say so
```

**How it works.** `.describe()` on the numeric columns shows `min` and `max`, and an implausible extreme is visible in one glance — this is the single highest-yield check in the chapter and it costs one line. Then you write the rule *as code*, which forces you to decide what plausible means and leaves that judgement in the notebook where a reader can argue with it. Note the third step: the `999` becomes explicit missing data, because that's what it always was, while the odd durations get **flagged rather than removed**. Flagging is usually the better move — it keeps the row, keeps the count honest, and lets you run your analysis both with and without the suspects to see whether they change the conclusion.

Two traps. First, an outlier is not the same as an impossible value: a €50,000 order might be your most interesting row, and deleting it because it's far from the mean is not cleaning, it's editing your findings. Only remove what is genuinely *impossible*, and say so. Second, `1440` minutes is suspicious rather than impossible — a bike could sit un-docked for a day — which is exactly why it gets a flag and a sentence in the log rather than a decision pretending to be a fact.

!!! practice "In Practice — Assert what must be true"
    Once you've decided what your data may look like, write it down as a check that *fails loudly* if a future version of the file violates it:

    ```python
    assert df["trip_id"].is_unique,            "trip_id should identify one trip"
    assert df["duration_min"].between(1, 240).all(), "durations outside 1–240 min"
    assert df["membership"].isin(["member", "casual"]).all(), "unexpected membership value"
    assert df["spend"].notna().all(),          "spend has missing values after cleaning"
    ```

    Four lines, and they convert your assumptions from things in your head into things the notebook enforces. Their real value comes later: when you re-run the analysis on next month's export and an assertion fires, you've caught a data change *before* it reaches a chart. This is the human-scale version of the automated validation in the Going Further box below — same idea, four lines instead of a pipeline.

**How it works.** Each step is small and reversible, and each makes a *decision visible*: `replace(...np.nan)` declares that `"unknown"` means "no data"; `to_numeric`/`to_datetime` with `errors="coerce"` convert types while turning bad values into clean blanks; the mapping dictionary records exactly how categories were merged; `drop_duplicates` collapses repeats. The right response to every mess is the same: **decide deliberately, and write down what you did.**

Notice the very first line: `df = dirty.copy()`. Keep your raw data untouched and clean a copy, always. It costs nothing, it lets you re-run from the original when a decision turns out wrong, and it means you can show a reader the before and after. Overwriting your only copy of the raw file is the one cleaning mistake that can't be undone.

### The whole repertoire on one page

| Problem | How you spot it | Your options | What to record |
|---|---|---|---|
| Missing values | `df.isna().sum()` | drop rows · impute (mean/median/forward-fill) · flag and keep | how many, which column, why missing, which option and why |
| Disguised missing (`"unknown"`, `-`, `999`) | `value_counts()`, suspicious `min`/`max` | `replace(...)` to `NaN` | which strings you treated as missing |
| Numbers stored as text | `.dtypes` says `object` | `pd.to_numeric(errors="coerce")` | how many failed to convert |
| Dates as text, or ambiguous | `.dtypes`; spot-check a few rows | `pd.to_datetime(dayfirst=...)` | the day/month assumption you made |
| Messy text | `.nunique()` before/after normalising | `.str.strip().str.lower()` | that you case-folded, and where you didn't |
| Inconsistent categories | `value_counts()` | a documented mapping dict | the exact mapping, including anything you *didn't* merge |
| Duplicates | `df.duplicated(subset=...).sum()` | drop · keep first/last · investigate | which columns define a duplicate, how many went |
| Impossible values | `describe()` min/max, an explicit rule | set to `NaN` · flag · correct if you know the truth | the rule you used, and how many rows it caught |
| Outliers (possible but extreme) | `describe()`, a histogram | **usually keep** · flag · analyse twice | that they exist and whether they change the conclusion |

Read the last column down the page. The options change from row to row; the obligation never does.

!!! practice "In Practice — The provenance note (your cleaning log)"
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

Cleaning is unglamorous, and it is where integrity is won or lost. Do it deliberately, write it down, and you've already separated yourself from most people who "work with data." With one trustworthy table in hand, the next chapter makes it richer by joining it to another.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. This is the one course in Part I worth finishing completely — cleaning is most of the job.

    - [Cleaning Data in Python](https://app.datacamp.com/learn/courses/cleaning-data-in-python) — **Ch. 1 Common Data Problems**, **Ch. 2 Text and Categorical Data Problems**, **Ch. 3 Advanced Data Problems.**
