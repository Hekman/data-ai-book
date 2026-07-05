<p class="chapter-kicker">Part I · Chapter 3</p>

# The dataframe — your core instrument

If this book gives you one tool you keep for the rest of your career, it will be the dataframe. A dataframe is simply a table you can *compute with*: rows and columns, like a spreadsheet, but manipulated through small, composable commands rather than by hand. Almost everything in Part I — filtering, grouping, cleaning, summarising, feeding a chart — happens inside a dataframe. Learn its handful of core moves and you can do eighty per cent of practical data work; the rest is variations.

The idea is older than the software. The spreadsheet — VisiCalc in 1979, then Excel — already gave the world a live, recalculating grid, and it remains the most widely used data tool on Earth. The dataframe keeps the grid but trades direct manipulation for *reproducibility*: instead of clicking cells, you write a short sequence of instructions, which means you can run it again tomorrow, on new data, and see exactly what you did. For a designer that trade is worth understanding as a design decision — you give up the immediacy of touch to gain a record of your reasoning.

!!! origins "Origins — From double-entry ledgers to VisiCalc to pandas"
    The dataframe has a long pedigree. Double-entry bookkeeping, codified by Luca Pacioli in 1494, was already a structured data model with built-in error-checking. VisiCalc (Dan Bricklin and Bob Frankston, 1979) made interactive numerical modelling a mass activity and, famously, sold the personal computer. The lineage reaches the modern era through **pandas**, which Wes McKinney released in 2008: it brought the R-style DataFrame to Python and, paired with notebooks, assembled the workbench that defined the data-science profession of the 2010s. When you type a dataframe command, you're standing at the end of a five-hundred-year line of people structuring numbers so they can be trusted.

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

**1 · Select** — choose the columns you care about.

```python
orders[["city", "price"]]     # a smaller table, just these two columns
```

**2 · Filter** — keep only the rows meeting a condition.

```python
orders[orders["price"] > 150]           # only orders over €150
orders[orders["city"] == "Utrecht"]     # only Utrecht
```

The bit inside the brackets, `orders["price"] > 150`, produces a column of `True`/`False`; wrapping it in `orders[...]` keeps the `True` rows. This "boolean mask" is the single most useful pattern in pandas.

**3 · Sort** — order rows by a column.

```python
orders.sort_values("price", ascending=False)   # most expensive first
```

**4 · Group** and **5 · Aggregate** — split into buckets, then summarise each bucket.

```python
orders.groupby("category")["price"].mean()     # average price per category
orders.groupby("city")["order_id"].count()     # number of orders per city
```

**How it works.** `groupby("category")` splits the table into one group per distinct category; `["price"]` picks the column to summarise; `.mean()` collapses each group to a single number. This *group-and-aggregate* is the quiet workhorse of data analysis. "Average order value per country," "complaints per week," "median response time per team" — nearly every real question you'll be asked is a group-and-aggregate in disguise. Once you can hear that pattern in a stakeholder's question, half the work is done before you touch the keyboard.

You can summarise several ways at once with `.agg`:

```python
orders.groupby("category").agg(
    avg_price=("price", "mean"),
    orders=("order_id", "count"),
    top_rating=("rating", "max"),
)
```

!!! practice "In Practice — The same question, two tools"
    Asked "which product category has the highest average rating?", in a spreadsheet you'd drag fields into a pivot table — fast, visual, and gone the moment the data changes. In a dataframe you'd write one line: `orders.groupby("category")["rating"].mean().sort_values(ascending=False)`. It's less immediate, but it's a durable artefact: hand it a fresh export next month and it answers again, identically, and anyone can see exactly how you got the number. Neither is "better." The spreadsheet is a conversation; the dataframe is a record.

## Making new columns

Real analysis rarely uses only the columns you were given. Constantly you need a variable that isn't there yet — revenue including tax, price-per-unit, a flag for "expensive" — and you make it by **deriving a new column** from existing ones. Assigning to a column name that doesn't exist yet creates it:

```python
orders["price_incl_vat"] = (orders["price"] * 1.21).round(2)   # arithmetic on a whole column
orders["is_premium"]     = orders["price"] > 150               # a True/False flag
orders["value_score"]    = orders["rating"] / orders["price"]  # combine two columns
orders.head()
```

**How it works.** The key idea is that operations apply to the *entire column at once* — `orders["price"] * 1.21` multiplies every price in one go, no loop required (this is called *vectorised* computation, and it's both faster and easier to read than looping). A comparison like `orders["price"] > 150` produces a whole column of `True`/`False` you can then filter or count with. Derived columns are where analysis turns raw records into the *variables your question is actually about* — and each one you add is, quietly, a new [level-of-measurement decision](chapter-1.md) you should be able to name.

!!! case "Case study — a derived flag on the bike-share trips"
    Using the `trips` table from [Chapter 2](chapter-2.md), add a variable the raw data doesn't contain — whether a ride was "long" — then compare it across membership type:

    ```python
    trips["long_ride"] = trips["duration_min"] >= 15        # derived boolean
    trips.groupby("membership")["long_ride"].mean()          # share of long rides per group
    ```

    In two lines you've turned a raw duration into a question ("do casual riders take longer trips than members?") and answered it. That move — invent the variable your question needs, then group by it — recurs constantly for the rest of the book.

## Where you'll work: the notebook

Most of your dataframe work happens in a **notebook** — an environment that interleaves code, its output, and your written explanation in one scrolling document. For a designer this is a gift, because it makes analysis *narratable*: you can show the data, the transformation, the resulting chart, and a sentence saying what it means, all in sequence. A notebook is closer to a lab journal than to a program. You don't need to become an engineer to use one well; you need to become a careful note-taker who happens to run code between the notes.

!!! further "Going Further — When to reach past the dataframe for SQL"
    A dataframe lives in your computer's memory — fine for millions of rows, hopeless for billions. When data is too big to fit, or lives in a shared database, the tool is **SQL**, a language designed at IBM in the 1970s to query Edgar Codd's relational model, in which you *describe the result you want* rather than the steps to get it. SQL is the most successful data language ever made; fifty years on, most systems either speak it or apologise for not doing so. You don't need it for Part I, but the day your dataset won't fit on your laptop, SQL is where you'll go — and pleasingly, its `GROUP BY` looks a lot like the dataframe's.

Get comfortable here. The dataframe is the bench you'll stand at for the rest of the book; the next chapter is about the least glamorous and most important thing you'll do at it — cleaning.

## Exercises

??? exercise "Exercise 3.1 · Read a stakeholder question as a group-and-aggregate"
    Using the `orders` table, answer each in one line of pandas: (a) Which city has the highest *total* revenue? (b) What is the average rating *per city*? (c) How many orders cost more than €100?

    ??? note "Show a solution"
        ```python
        # (a) total revenue per city, highest first
        orders.groupby("city")["price"].sum().sort_values(ascending=False)

        # (b) average rating per city
        orders.groupby("city")["rating"].mean()

        # (c) count of orders over €100
        (orders["price"] > 100).sum()   # True counts as 1
        ```
        Notice (c): a boolean mask summed up *is* a count of matching rows. Small idioms like this are most of fluency.

??? exercise "Exercise 3.2 · Chain the verbs"
    Produce a table of the **top two most expensive orders in Amsterdam**, showing only `category` and `price`. You'll need to filter, sort, select and limit — in any order that works.

    ??? note "Show a solution"
        ```python
        (orders[orders["city"] == "Amsterdam"]   # filter
               .sort_values("price", ascending=False)  # sort
               [["category", "price"]]           # select
               .head(2))                         # limit
        ```
        There's no single correct order of operations — filtering first is just faster because everything after works on fewer rows. Being able to *explain* your order is the skill, not memorising one.
