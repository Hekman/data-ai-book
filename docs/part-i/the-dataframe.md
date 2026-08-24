<p class="chapter-kicker">Part I · Chapter 2 · <span class="week">Week 1</span></p>

# The dataframe, your core instrument

!!! guiding "The question this chapter answers"
    How do I load a table and shape it — select, filter, sort, group, aggregate — until it answers the question I was asked?

If this book gives you one tool you keep for the rest of your career, it will be the dataframe. A dataframe is simply a table you can *compute with*: rows and columns, like a spreadsheet, but manipulated through small, composable commands rather than by hand. Almost everything in Part I (filtering, grouping, cleaning, summarising, feeding a chart) happens inside a dataframe. Learn its handful of core moves and you can do eighty per cent of practical data work; the rest is variations.

The idea is older than the software. The spreadsheet, VisiCalc in 1979, then Excel, already gave the world a live, recalculating grid, and it remains the most widely used data tool on Earth. The dataframe keeps the grid but trades direct manipulation for *reproducibility*: instead of clicking cells, you write a short sequence of instructions, which means you can run it again tomorrow, on new data, and see exactly what you did. For a designer that trade is worth understanding as a design decision: you give up the immediacy of touch to gain a record of your reasoning.

!!! matters "Why this matters for designers"
    Every column you keep, every row you filter out, and every level you group by is a design decision that shapes what someone downstream can know. Aggregation is where those decisions hide most effectively: what you choose to group by determines the story the data is able to tell. Fluency here isn't about memorising pandas syntax — it's about being able to translate a stakeholder's question into a sequence of table operations, and to say out loud what each one assumed.

!!! newsroom "Why this matters for journalists"
    Most data stories are a group-and-count at heart: complaints by borough, deaths by year, contracts by supplier. What you choose to group by decides which story the data is able to tell, and which one stays invisible — group by year and the spike disappears, group by month and it's the whole piece. Fluency here is less about pandas than about being able to state, in a sentence an editor can check, what each step assumed.

!!! outcomes "By the end of this chapter you can"
    - Load a dataset into a pandas DataFrame and articulate what it does and does not contain.
    - Interrogate a new table with `.shape`, `.head()`, `.info()`, `.describe()` and `.value_counts()`, and spot missing values and inconsistent categories.
    - Select, filter, sort, and use `groupby` + aggregation at a granularity you chose deliberately.
    - Pick rows and columns together with `.loc`, and explain when `.iloc` is the wrong tool.
    - Recognise when your grouping variable has become the index, and use `reset_index()`.
    - Avoid `SettingWithCopyWarning` by deciding whether you're editing your data or a copy of it.
    - Derive a new column that the raw data doesn't contain, and say what it assumes.
    - Hear the group-and-aggregate hiding inside an ordinary stakeholder question.
    - Keep a notebook that still produces the same answers after *Restart & Run All*.

!!! origins "Origins — From double-entry ledgers to VisiCalc to pandas"
    The dataframe has a long pedigree. Double-entry bookkeeping, codified by Luca Pacioli in 1494, was already a structured data model with built-in error-checking. VisiCalc (Dan Bricklin and Bob Frankston, 1979) made interactive numerical modelling a mass activity and, famously, sold the personal computer. The lineage reaches the modern era through **pandas**, which Wes McKinney released in 2008: it brought the R-style DataFrame to Python and, paired with notebooks, assembled the workbench that defined the data-science profession of the 2010s. When you type a dataframe command, you're standing at the end of a five-hundred-year line of people structuring numbers so they can be trusted.

## Loading a dataset

In practice a dataset arrives as a file, most often a **CSV** (comma-separated values: plain text, one row per line). Here we build a small CSV in memory so the example runs anywhere, then read it exactly as you'd read a real file.

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

## First contact: what does this table contain?

Never start analysing a table you haven't interrogated. Five one-liners tell you most of what you need:

```python
df.shape            # (rows, columns) — how big is it?           -> (5, 5)
df.head()           # the first rows — what does it look like?
df.info()           # columns, types, and how many values are missing
df.describe()       # quick stats for the numeric columns
df["city"].value_counts()   # what categories exist, and how often
```

**How it works.** `read_csv` parses the text into a dataframe, guessing a type for each column. `.shape` gives its size. `.head()` shows the top five rows so you can eyeball the structure. `.info()` is the one to linger on: it lists every column, its inferred type (`object` means text, `int64`/`float64` mean numbers), and its non-null count; the gap between that count and the total is your missing data. `.describe()` summarises numeric columns (count, mean, min, max, quartiles). `.value_counts()` tallies a categorical column.

Run `.info()` on our toy data and two problems appear immediately. The `age` column is `float64` with only 3 non-null values out of 5, so **two ages are missing**, and the reason matters (did those people decline to answer, or did a form skip the field?). And `city` contains both `"Utrecht"` and `"rotterdam"`, **the same kind of thing recorded inconsistently**, which `value_counts()` shows as separate categories until you fix it. You haven't done any "analysis" yet, and you've already found the two issues that would have wrecked it. Fixing them is [Chapter 3](cleaning-messy-data.md); *asking why they are there* is [Chapter 5](where-data-comes-from.md).

## A dataset to work with

Let's make a small orders table and keep it for the whole chapter.

```python
import pandas as pd

orders = pd.DataFrame({
    "order_id": [1, 2, 3, 4, 5, 6, 7, 8],
    "city":     ["Utrecht", "Amsterdam", "Utrecht", "Rotterdam",
                 "Amsterdam", "Utrecht", "Rotterdam", "Amsterdam"],
    "category": ["chair", "lamp", "lamp", "chair",
                 "desk", "lamp", "desk", "chair"],
    "price":    [149, 39, 45, 199, 259, 42, 240, 159],
    "rating":   [4.5, 4.0, 3.5, 5.0, 4.8, 4.2, 4.9, 4.1],
})
orders
```

## The five core moves

You can get remarkably far with five verbs. Learn them as *concepts* first; the syntax follows.

**1 · Select**: choose the columns you care about.

```python
orders[["city", "price"]]     # a smaller table, just these two columns
```

**2 · Filter**: keep only the rows meeting a condition.

```python
orders[orders["price"] > 150]           # only orders over €150
orders[orders["city"] == "Utrecht"]     # only Utrecht
```

The bit inside the brackets, `orders["price"] > 150`, produces a column of `True`/`False`; wrapping it in `orders[...]` keeps the `True` rows. This "boolean mask" is the single most useful pattern in pandas.

**3 · Sort**: order rows by a column.

```python
orders.sort_values("price", ascending=False)   # most expensive first
```

**4 · Group** and **5 · Aggregate**: split into buckets, then summarise each bucket.

```python
orders.groupby("category")["price"].mean()     # average price per category
orders.groupby("city")["order_id"].count()     # number of orders per city
```

**How it works.** `groupby("category")` splits the table into one group per distinct category; `["price"]` picks the column to summarise; `.mean()` collapses each group to a single number. This *group-and-aggregate* is the quiet workhorse of data analysis. "Average order value per country," "complaints per week," "median response time per team", nearly every real question you'll be asked is a group-and-aggregate in disguise. Once you can hear that pattern in a stakeholder's question, half the work is done before you touch the keyboard.

You can summarise several ways at once with `.agg`:

```python
orders.groupby("category").agg(
    avg_price=("price", "mean"),
    orders=("order_id", "count"),
    top_rating=("rating", "max"),
)
```

### The five verbs at a glance

| Verb | You want | pandas | You get back |
|---|---|---|---|
| Select | fewer columns | `df[["a", "b"]]` | a DataFrame |
| Filter | fewer rows | `df[df["a"] > 5]` | a DataFrame |
| Sort | rows in an order | `df.sort_values("a", ascending=False)` | a DataFrame |
| Group + aggregate | one number per group | `df.groupby("a")["b"].mean()` | a **Series**, indexed by group |
| Group + aggregate (several) | several numbers per group | `df.groupby("a").agg(...)` | a DataFrame |

That last column catches people out, so it's worth dwelling on. `groupby("category")["price"].mean()` hands you a **Series**: one column of numbers whose *labels* are the categories. That's convenient to read and awkward to keep working with, because the categories are now the index rather than a column. `reset_index()` turns them back into an ordinary column:

```python
avg = orders.groupby("category")["price"].mean()
avg                                  # a Series: category is the index
avg.reset_index()                    # a DataFrame with columns: category, price
avg.reset_index(name="avg_price")    # ...and give the value column a decent name
```

**How it works.** Every dataframe has an **index**: the labels down its left-hand side, which pandas uses to line rows up. Most of the time it's just `0, 1, 2, …` and you can ignore it, but `groupby` puts your grouping variable there, and so does `pivot`, and `value_counts`. When a later step complains that it can't find a column you can plainly see, it is almost always because that "column" is currently the index. `reset_index()` is the fix, and reaching for it early saves a lot of confusion.

### Picking rows and columns at the same time

`df[...]` handles one axis at a time. To ask for both at once — and to say clearly whether you mean *labels* or *positions* — use `.loc` and `.iloc`:

```python
orders.loc[orders["city"] == "Utrecht", ["category", "price"]]   # (1)!
orders.loc[0, "price"]        # 149 — one cell, by label
orders.iloc[0:3]              # the first three rows, by position
orders.iloc[0, 3]             # first row, fourth column, by position
```

1.  Read this as `.loc[which rows, which columns]`. The condition picks the rows; the list picks the columns.

**How it works.** `.loc` works with *labels* (index values and column names); `.iloc` works with *integer positions*. The practical rule: reach for `.loc` almost always, because names survive sorting and filtering while positions don't — `iloc[0]` means "whatever happens to be first right now", which changes the moment you sort. `.loc` is also the form you need when *assigning* to a subset, which is the subject of the warning below.

!!! practice "In Practice — The warning everyone meets: `SettingWithCopyWarning`"
    Sooner or later you'll filter a table, change a value in the result, and get a wall of pink text about setting a value on a copy of a slice. It is telling you something real: pandas isn't sure whether you meant to edit the original table or a throwaway extract, so it may have edited neither in the way you expected.

    ```python
    # The problem: chained brackets. Did this change `orders`, or a temporary copy?
    utrecht = orders[orders["city"] == "Utrecht"]
    utrecht["price"] = 0          # ⚠️ SettingWithCopyWarning

    # Fix 1 — you meant a separate table. Say so.
    utrecht = orders[orders["city"] == "Utrecht"].copy()
    utrecht["price"] = 0          # unambiguous: edits the copy

    # Fix 2 — you meant to edit the original. Say that, in one .loc.
    orders.loc[orders["city"] == "Utrecht", "price"] = 0
    ```

    The rule that avoids it for good: **never assign through two sets of brackets.** If you want a working subset, `.copy()` it; if you want to change the original in place, do it in a single `.loc[rows, column] = value`. Deciding which you meant is the useful part — half the time the warning is pointing at a genuine confusion about whether you're editing your data or a view of it.

!!! practice "In Practice — The same question, two tools"
    Asked "which product category has the highest average rating?", in a spreadsheet you'd drag fields into a pivot table, fast, visual, and gone the moment the data changes. In a dataframe you'd write one line: `orders.groupby("category")["rating"].mean().sort_values(ascending=False)`. It's less immediate, but it's a durable artefact: hand it a fresh export next month and it answers again, identically, and anyone can see exactly how you got the number. Neither is "better." The spreadsheet is a conversation; the dataframe is a record.

## Making new columns

Real analysis rarely uses only the columns you were given. Constantly you need a variable that isn't there yet (revenue including tax, price-per-unit, a flag for "expensive"), and you make it by **deriving a new column** from existing ones. Assigning to a column name that doesn't exist yet creates it:

```python
orders["price_incl_vat"] = (orders["price"] * 1.21).round(2)   # arithmetic on a whole column
orders["is_premium"]     = orders["price"] > 150               # a True/False flag
orders["value_score"]    = orders["rating"] / orders["price"]  # combine two columns
orders.head()
```

**How it works.** The key idea is that operations apply to the *entire column at once*: `orders["price"] * 1.21` multiplies every price in one go, no loop required (this is called *vectorised* computation, and it's both faster and easier to read than looping). A comparison like `orders["price"] > 150` produces a whole column of `True`/`False` you can then filter or count with. Derived columns are where analysis turns raw records into the *variables your question is actually about*, and each one you add is, quietly, a new [level-of-measurement decision](data-variables-structures.md) you should be able to name.

!!! case "Case study — put the bike-share data to work"
    You classified the columns of the `trips` table in [Chapter 1](data-variables-structures.md); now use it. Rebuild it if you need to:

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
    ```

    In Chapter 1 the derived `long_ride` flag was a *classification* exercise. Here it becomes a question, by grouping on the two-value `membership` column you identified as a flag:

    ```python
    trips["long_ride"] = trips["duration_min"] >= 15        # derived boolean
    trips.groupby("membership")["long_ride"].mean()          # share of long rides per group
    ```

    In two lines you've turned a raw duration into a question ("do casual riders take longer trips than members?") and answered it. That move — invent the variable your question needs, then group by it — recurs constantly for the rest of the book. Two companion tables (stations, weather) join on in [Chapter 4](combining-datasets.md).

## Where you'll work: the notebook

Most of your dataframe work happens in a **notebook**, an environment that interleaves code, its output, and your written explanation in one scrolling document. For a designer this is a gift, because it makes analysis *narratable*: you can show the data, the transformation, the resulting chart, and a sentence saying what it means, all in sequence. A notebook is closer to a lab journal than to a program. You don't need to become an engineer to use one well; you need to become a careful note-taker who happens to run code between the notes.

There is one trap in it, and it costs students more time than any syntax error. A notebook lets you run cells **in any order**, but it keeps a single, hidden memory of everything you've run. So the document you see and the state the computer is in can silently drift apart: you edit a cell and don't re-run it, or you run cell 8 before cell 5, and now `df` holds something the visible code doesn't describe. Worse, it usually still *works*, and gives you a plausible wrong number.

!!! practice "In Practice — Restart and run all, before you believe anything"
    Two habits keep a notebook honest. **Write cells so they run top to bottom**: load, clean, analyse, chart, in that order, with no backwards jumps. And **before you trust a result or hand the notebook in, restart the kernel and run every cell from the top** (in Jupyter and Colab: *Kernel → Restart & Run All*). If it fails, or the numbers change, your conclusion was resting on state you couldn't see. This is the notebook version of the reproducibility argument that made you choose a dataframe over a spreadsheet in the first place — a record of your reasoning is only a record if it still runs.

    A related courtesy: a notebook you share should include its own `import` statements and data loading, so a reader can reproduce it without asking you what to run first.

!!! further "Going Further — When to reach past the dataframe for SQL"
    A dataframe lives in your computer's memory, fine for millions of rows, hopeless for billions. When data is too big to fit, or lives in a shared database, the tool is **SQL**, a language designed at IBM in the 1970s to query Edgar Codd's relational model, in which you *describe the result you want* rather than the steps to get it. SQL is the most successful data language ever made; fifty years on, most systems either speak it or apologise for not doing so. You don't need it for Part I, but the day your dataset won't fit on your laptop, SQL is where you'll go, and pleasingly, its `GROUP BY` looks a lot like the dataframe's.

Get comfortable here. The dataframe is the bench you'll stand at for the rest of the book. The next chapter is where the bench gets dirty: real data arrives broken, and repairing it is not tidying up before the work — it *is* most of the work, and where most of the judgement lives.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. Do them with this chapter open beside you; the five verbs are muscle memory, not theory.

    - [Introduction to Data Science in Python](https://app.datacamp.com/learn/courses/introduction-to-data-science-in-python) — **Ch. 2 Loading Data in pandas.** Reading a CSV, inspecting a DataFrame, selecting and filtering.
