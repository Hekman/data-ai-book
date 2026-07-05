<p class="chapter-kicker">Part I · Chapter 2</p>

# The origins of a dataset

Before you touch a single row, ask a question most tutorials skip: *why does this dataset exist at all?* Datasets are not found objects. Each one was produced by some institution, for some purpose, at some cost, and its shape still carries the fingerprints of that purpose. A dataset built to *tax* people looks different from one built to *help* them, even when it contains the same names. Knowing why a dataset was made is the first and cheapest form of analysis.

Consider the oldest large datasets we have: censuses. States have counted their populations for as long as there have been states, and they did it for concrete reasons — taxation, conscription, the apportionment of power. Those reasons decided what got recorded. A census built to conscript soldiers counts men of fighting age carefully and everyone else roughly. Two thousand years later, when a historian uses that data to estimate a population, the original purpose is still bending the numbers. This is the general lesson in miniature: *the reason a dataset was collected is a lens that never fully comes off.*

!!! origins "Origins — From counting people to counting them by machine"
    Ancient states ran periodic counts — Egypt's cattle-and-people censuses are documented from the Old Kingdom, and the Han dynasty census of 2 CE recorded some 57.7 million people, totals that still survive. But scale eventually broke the method. The 1880 US census took most of a decade to tabulate by hand; by the time it was finished it was nearly obsolete. Herman Hollerith's electric punched-card tabulators processed the 1890 count in a fraction of the time — and the company he founded to sell them became IBM. The census didn't just *use* data technology; its impossible scale is what *created* the data-processing industry.

## Tables, and the quiet decisions inside them

Most data you'll meet arrives as a table: rows and columns. It looks so natural that it's easy to forget it's a *model* — a compression of a messy world into a rectangle. Every table makes at least three decisions for you, and each is worth surfacing.

First, the table decides **what a row is** — the unit of observation. One row per person? Per transaction? Per person-per-day? The same reality can be tabled many ways, and the choice determines what questions are even askable. Second, the table decides **what a column is** — which attributes are deemed worth recording. Everything without a column is, for your analysis, invisible. Third, the table decides **the categories inside a column** — the fixed boxes a messy reality is sorted into. A "gender" column with two values has made a claim; a "country" column with no row for a stateless person has made another.

None of these decisions are wrong, exactly. They are *choices*, and as the designer working the material you should be able to name them, because they are the seams where the data will most often mislead you.

!!! critical "Critical Lens — The library of missing datasets"
    The artist-researcher Mimi Onuoha keeps a project she calls *The Library of Missing Datasets*: things conspicuously *not* collected — for years, comprehensive data on police killings in the United States, even as everything else was measured. Such gaps are rarely accidents. Data gets collected where there is power, profit or obligation to collect it, and not where there isn't. So when you inventory a dataset, inventory its *absences* too. The most important column is sometimes the one that was never made.

## Was this a good sample?

Almost every dataset is a **sample** — a subset of some larger population you actually care about — even when it feels like "all the data." A table of your app's users is not a sample of *people*; it's a sample of *people who found and installed your app*. The single most important question you can ask of a dataset's origins is therefore: **who or what is in here, and who or what is systematically left out?** When the answer is "the sample is skewed," no amount of clever analysis downstream can fix it — the bias is baked in at collection.

This failure has a name, **selection bias**: the process that put rows into your dataset is related to the very thing you're trying to measure. A satisfaction survey answered only by people with strong feelings. A sensor that fails in exactly the conditions you most want to study. A bike-share dataset that, by definition, contains only trips by people who live near a dock and use the app — so any claim it makes about "how the city cycles" is really a claim about a particular, unrepresentative slice of it.

!!! origins "Origins — Gallup and the lesson of the sample"
    In 1936 the *Literary Digest* polled over two million people and confidently predicted the wrong winner of the US presidential election; George Gallup polled a few thousand and got it right. The *Digest*'s huge sample was drawn from car and telephone owners — wealthier, and unrepresentative in a depression year — while Gallup's small sample was chosen to *mirror* the electorate. The lesson has echoed for ninety years and the big-data era had to relearn it: **how you sample matters more than how much you collect.** A biased million is worse than a representative thousand, because the million's size only lends false confidence to its skew.

## Loading and interrogating a dataset

Enough theory — let's open one. In practice a dataset arrives as a file, most often a **CSV** (comma-separated values: plain text, one row per line). Here we'll build a small CSV in memory so the example runs anywhere, then read it exactly as you'd read a real file.

```python
import pandas as pd
from io import StringIO

# Pretend this came from a file called "signups.csv".
raw = StringIO("""date,city,plan,age,referred_by
2026-01-03,Utrecht,pro,31,search
2026-01-03,Amsterdam,free,,search
2026-01-04,Utrecht,free,27,friend
2026-01-05,rotterdam,pro,44,search
2026-01-05,Amsterdam,team,,
""")

df = pd.read_csv(raw)   # (1)!
df                      # in a notebook, this displays the table
```

1.  In real work you'd write `pd.read_csv("signups.csv")` with a path or URL. Everything after this line is identical whether the data is faked or real.

Now apply **five questions** to it in code. Each is one line:

```python
df.shape            # (rows, columns) — how big is it?           -> (5, 5)
df.head()           # the first rows — what does it look like?
df.info()           # columns, types, and how many values are missing
df.describe()       # quick stats for the numeric columns
df["city"].value_counts()   # what categories exist, and how often
```

**How it works.** `read_csv` parses the text into a dataframe, guessing a type for each column. `.shape` gives its size. `.head()` shows the top five rows so you can eyeball the structure. `.info()` is the one to linger on: it lists every column, its inferred type (`object` means text, `int64`/`float64` mean numbers), and its non-null count — the gap between that count and the total is your missing data. `.describe()` summarises numeric columns (count, mean, min, max, quartiles). `.value_counts()` tallies a categorical column.

Run `.info()` on our toy data and two of Chapter 0's warnings appear immediately. The `age` column is `float64` with only 3 non-null values out of 5 — **two ages are missing**, and the reason matters (did those people decline to answer, or did a form skip the field?). And `city` contains both `"Utrecht"` and `"rotterdam"` — **the same kind of thing recorded inconsistently**, which `value_counts()` will show as separate categories until you fix it. You haven't done any "analysis" yet, and you've already found the two issues that would have wrecked it.

!!! practice "In Practice — The five questions to ask any new dataset"
    Before analysis, write down: (1) Who produced this, and for what purpose? (2) What is one row? (3) What's in the columns — and what obviously isn't? (4) When was it collected, and is that still current? (5) What's the format, and what might it have quietly mangled (dates, leading zeros, encodings)? Ten minutes on these five will save you hours and keep you from confidently analysing an artefact of *collection* rather than the world.

!!! case "Case study — meet the bike-share data"
    Throughout the rest of Part I we'll work a single running dataset: a synthetic **city bike-share** in Utrecht. Its main table has one row per trip.

    ```python
    import pandas as pd
    trips = pd.DataFrame({
        "trip_id":      [1, 2, 3, 4, 5, 6],
        "start_time":   pd.to_datetime([
            "2026-05-01 08:12", "2026-05-01 08:40", "2026-05-01 17:30",
            "2026-05-02 09:05", "2026-05-02 18:15", "2026-05-03 12:20"]),
        "station":      ["Centraal", "Uithof", "Centraal", "Lombok", "Uithof", "Centraal"],
        "membership":   ["member", "casual", "member", "member", "casual", "casual"],
        "duration_min": [7, 22, 9, 14, 25, 11],
    })
    trips.info()   # apply the five questions: one row = one trip; what's missing?
    ```

    Run the five questions on it and notice its *origins* already limit it: it records only trips that started and ended at a dock, by app users — the [selection bias](#was-this-a-good-sample) above, made concrete. Two companion tables (stations, weather) join on later, in [Chapter 5](chapter-5.md).

## File formats are not neutral either

Data arrives in a container — a CSV, an Excel file, a JSON blob, a database export — and the container shapes what's easy. A CSV is just text with commas; universal and dumb, which is both its virtue and its curse (it will happily turn a phone number into a giant integer and drop the leading zero). A spreadsheet mixes data with formatting and formulas, so the numbers and someone's interpretation of them often arrive fused. A JSON file can nest, carrying shapes a flat table can't, at the cost of being harder to reason about at a glance. You don't need to master all of these yet — you need the habit of asking, on first contact: *what form is this in, and what did that form make easy or hard for whoever produced it?*

## Reading beyond CSV

CSV is the most common container, but not the only one you'll meet. The same `pd.read_*` family opens the others, so switching format is usually a one-line change:

```python
# Excel — one sheet of a workbook (needs the openpyxl package installed)
xl = pd.read_excel("report.xlsx", sheet_name="signups")

# JSON — the format most web APIs return
j = pd.read_json("export.json")

# Nested JSON (an API response with objects inside objects) needs flattening:
import pandas as pd
payload = {"results": [
    {"id": 1, "city": {"name": "Utrecht"}, "plan": "pro"},
    {"id": 2, "city": {"name": "Amsterdam"}, "plan": "free"},
]}
flat = pd.json_normalize(payload["results"])   # -> columns: id, city.name, plan
flat
```

**How it works.** `read_excel` reads a named sheet from a workbook (remember that an Excel file often carries formatting and formulas *around* the data, so check you grabbed the right range). `read_json` maps a flat JSON array straight to a table. When JSON is *nested* — objects within objects, as almost every API returns — `json_normalize` flattens the nesting into dotted column names like `city.name`. You'll use exactly this in [Chapter 9](chapter-9.md) when you pull data from a live API.

!!! further "Going Further — Where data *lives* shapes what you can ask"
    Behind most large datasets sits a storage decision that constrains everything downstream. A **data warehouse** organises data in advance, structured for expected questions (schema-on-write); a **data lake** — a term James Dixon coined in the Hadoop era — stores everything raw and defers structure until you read it (schema-on-read). Warehouses keep you disciplined and can be rigid; lakes keep you flexible and can degenerate into "data swamps." You rarely choose this as a student, but recognising which world your data came from explains a lot about why it's clean-but-narrow or vast-but-feral. Ralph Kimball's dimensional modelling (the "star schema") is the classic account of shaping data around the questions humans actually ask.

Where data comes from is not throat-clearing before the real work. It *is* the real work, done early, when it's cheapest. Carry these questions into the next chapter, where we start moving data around in earnest.

## Exercises

??? exercise "Exercise 2.1 · The five questions, in code"
    Load any real CSV you can find (a public dataset, an export from an app, one of the course datasets) with `pd.read_csv`. Using only `.shape`, `.head()`, `.info()`, `.describe()` and `.value_counts()`, answer the five questions in a few written sentences. Flag at least one missing-data issue and one inconsistent category if they exist.

    ??? note "Show a solution sketch"
        ```python
        import pandas as pd
        df = pd.read_csv("your_file.csv")
        print(df.shape)
        df.info()                       # types + missing counts
        for col in df.select_dtypes("object"):
            print(df[col].value_counts()) # spot inconsistent categories
        df.describe()
        ```
        A strong write-up doesn't just report the numbers — it *interprets* them: "234 of 1,000 `income` values are missing, and they cluster in the youngest age band, so any average income here is biased upward." That sentence is worth more than any chart you'll make later.

??? exercise "Exercise 2.2 · Name the absences"
    For the same dataset, list three things it does **not** record but plausibly could, and for each write one sentence on how that absence could bias a conclusion someone might draw. This is the Onuoha exercise applied to your own data.

    ??? note "What good looks like"
        The trick is to connect each absence to a *decision it distorts*. "A food-delivery dataset has no column for orders that failed to check out, so 'most popular dish' really means 'most popular dish among completed orders' — it silently ignores everyone the interface lost." Naming the absence changes the claim you're allowed to make.
