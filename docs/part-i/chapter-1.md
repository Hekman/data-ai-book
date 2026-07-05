<p class="chapter-kicker">Part I · Chapter 1</p>

# Data, variables, and data structures

Before we ask where a dataset comes from or how to clean it, we need to know what data is actually *made of*. A dataset looks like one thing, a table, a file, but it is built from smaller parts, and every one of those parts is a decision. A single value is a decision about how to record one observation. A variable is a decision about what to measure and how finely. A data structure is a decision about how to hold it all together so a computer can work with it. Learn these building blocks and the rest of Part I becomes much less mysterious; skip them and you'll spend later chapters confused about why a chart won't draw or a number won't add up.

This is the most "technical" chapter in the part, but it stays close to the material. Think of it as learning the properties of your clay, what it's made of, and what each kind can and can't do, before you throw a pot.

## Values and variables

Start with the two smallest ideas. A **value** is a single recorded observation: the number `31`, the word `"Utrecht"`, the answer `True`. A **variable** is the *attribute* those values measure, age, city, whether someone subscribed, and in a table, a variable is usually a **column**. One column, one variable; one row, one observation; one cell, one value. That simple correspondence (championed as "tidy data") is the mental model to carry everywhere: **columns are variables, rows are observations.**

The word "variable" is borrowed from statistics, not programming, and it's worth keeping the statistical meaning in mind: a variable is something that *varies* across the things you observe. If a column never changes (every row says `"active"`), it isn't telling you anything, however much space it takes up.

## The levels of a variable, the grain of your material

Not all variables are the same *kind*, and the kind determines what you're allowed to do with them. This is the single most useful distinction in the chapter. Variables come in two broad families, each with two levels.

**Categorical** variables sort observations into named groups. They come as **nominal**, pure labels with no order (`city`, `plan`, `colour`), or **ordinal**, categories with a meaningful order but no fixed spacing (`small/medium/large`, a 1–5 satisfaction rating). You can count categories and find the most common, but "the average city" is nonsense.

**Numeric** variables measure quantity. They come as **interval**, numbers where differences are meaningful but there's no true zero (temperature in °C, calendar years), or **ratio**, numbers with a true zero, so ratios make sense (price, age, distance; €200 really is twice €100). With numeric variables you can add, average, and measure spread.

Why labour this? Because the level of a variable silently decides which statistics and which charts are legitimate. You can take the **mean** of a ratio variable but not of a nominal one. You compare categories with a **bar chart** and show the distribution of a numeric variable with a **histogram**, and mixing them up is a common way to draw a misleading picture. A number stored in your data isn't automatically a *quantity*: a postal code and a phone number are digits that you must treat as labels, because averaging them is meaningless.

!!! origins "Origins — Stevens names the scales of measurement"
    The four-level scheme, nominal, ordinal, interval, ratio, was set out by the psychologist S. S. Stevens in a 1946 paper, *On the Theory of Scales of Measurement*. Stevens was arguing about how to quantify human sensation, and needed a principled account of when a number was "really" a number. His taxonomy has been debated and refined ever since, but it endures because it answers a practical question every analyst faces: *what operations is this variable entitled to?* When a tool lets you compute the average of a category code and get a confident, meaningless answer, it's Stevens you needed and didn't have.

!!! critical "Critical Lens — Choosing a level is choosing a worldview"
    Deciding a variable's level is not a neutral technical act; it encodes a claim about the thing being measured. Record `gender` as two nominal categories and you've built a world with exactly two, invisibly excluding anyone who doesn't fit. Turn a rich, ordered experience like pain or satisfaction into an ordinal 1–5 and you've decided that the distance from "1" to "2" is comparable to "4" to "5", which it may not be. These choices, made once when the variable is defined, propagate into every later average and chart. This is the same lesson as [Chapter 2](chapter-2.md)'s missing datasets, one level down: what you *can't* measure faithfully is decided before you collect a single value.

## Python's building blocks: data types

When data lands in Python, each value has a **type** that governs what you can do with it. You'll meet four constantly:

```python
age      = 31          # int    — a whole number
price    = 149.95      # float  — a number with a decimal point
city     = "Utrecht"   # str    — text ("string"), always in quotes
active   = True        # bool   — True or False
missing  = None        # the absence of a value

type(age), type(price), type(city), type(active)
# -> (int, float, str, bool)
```

**How it works.** `type(x)` reports how Python is storing a value, and that storage decides its behaviour. `"31" + "31"` gives `"3131"` (text joined end to end) while `31 + 31` gives `62` (same-looking characters, completely different results), because one is a `str` and the other an `int`. This is exactly the "numbers stored as text" trap you'll meet again in [Chapter 4](chapter-4.md): the *type* and the *level of measurement* have to agree with your intent, or the data will quietly misbehave.

## Data structures: holding many values together

A single value is rarely useful; you need containers. Python gives you a few, and two of them do almost all the work in data science.

A **list** is an ordered sequence of values; think of one column, or one row, of your data:

```python
prices = [149, 39, 259, 199]     # a list of numbers
prices[0]        # 149  — lists are indexed from 0
prices.append(42)                 # add a value
len(prices)      # 5   — how many items
sum(prices) / len(prices)         # the average price
```

A **dictionary** (`dict`) stores **key → value** pairs, a label attached to each value, like a single record:

```python
order = {"city": "Utrecht", "category": "chair", "price": 149}
order["price"]   # 149  — look up by key, not position
order.keys()     # the field names
```

Two more you'll see occasionally: a **tuple** is a list that can't be changed after creation (useful for fixed pairs like coordinates), and a **set** is an unordered collection with no duplicates (useful for asking "what distinct categories appear?").

```python
point = (52.09, 5.12)                     # tuple — latitude, longitude
distinct_cities = set(["Utrecht", "Utrecht", "Amsterdam"])  # {'Utrecht', 'Amsterdam'}
```

## From building blocks to a table

Here's the payoff. A table is just these structures composed: a **dictionary whose keys are variable names and whose values are lists of observations**. That is *exactly* how you'll build a dataframe in the next chapters, so you've already seen the shape of it.

```python
import pandas as pd

# a dict of lists: keys = variables (columns), lists = the values down each column
data = {
    "city":     ["Utrecht", "Amsterdam", "Rotterdam"],   # nominal
    "plan":     ["pro", "free", "team"],                 # nominal
    "age":      [31, 27, 44],                            # ratio (numeric)
    "price":    [149.0, 0.0, 259.0],                     # ratio (numeric)
}

df = pd.DataFrame(data)   # the dict-of-lists becomes a table
df.dtypes                 # pandas' name for each column's type: object / int64 / float64
```

**How it works.** `pd.DataFrame(data)` reads each key as a column heading and each list as that column's values, snapping the loose Python structures into a proper table. `.dtypes` shows the type pandas inferred per column (`object` for text, `int64`/`float64` for numbers), the software's echo of the levels-of-measurement idea. Recognising that a dataframe is *made of* dictionaries and lists demystifies it: it isn't a magic object, it's the humble containers from this chapter, arranged in a grid.

!!! practice "In Practice — Name the level before you touch the data"
    A five-minute habit that pays off all year: for each column in a new dataset, jot its **level**, nominal, ordinal, interval, or ratio. It immediately tells you which summaries are legitimate (mean and standard deviation for ratio; counts and mode for nominal) and which chart to reach for (bars for categories, histograms for numbers). Most analysis mistakes you'll make early on are really a variable being treated as the wrong level, a category averaged, or a quantity chopped into arbitrary bins.

!!! further "Going Further — Why models will make you turn categories into numbers"
    Everything here returns with force in Part II. Machine-learning models are, underneath, arithmetic; they can only consume numbers. So categorical variables have to be **encoded** into numeric form before a model can use them (one column per category, a `1` or `0` in each, "one-hot encoding"), and doing this naïvely to an *ordinal* variable throws away its order, while doing it to a high-cardinality *nominal* one (say, thousands of postcodes) explodes your data. The level-of-measurement judgement you make here is what tells you which encoding is faithful. Keep it in mind; it's why this "basics" chapter is really groundwork for the whole book.

You now know what data is built from: values with types, variables with levels, and the structures that hold them. With the material understood, the next chapter asks the question that shapes everything you'll do with it, where a dataset actually comes from, and what its origins do to what it can tell you.

## Exercises

??? exercise "Exercise 1.1 · Classify the variables"
    Take any dataset (or invent a table of ten rows about something you know). For every column, write down its **level of measurement**, nominal, ordinal, interval, or ratio, and one sentence on a summary or chart that its level makes appropriate, and one it makes *inappropriate*.

    ??? note "What good looks like"
        A strong answer catches the traps: "`postal_code` looks numeric but is nominal; averaging it is meaningless, though counting how many customers share one is fine." "`satisfaction` (1–5) is ordinal, so the *median* is defensible but the *mean* is a slight fiction, because the gaps between ratings may not be equal." Spotting a number that isn't really a quantity is the whole point.

??? exercise "Exercise 1.2 · Build a table from the ground up"
    Using only plain Python (no pandas), build a small dataset as a **dictionary of lists** with at least one nominal, one ordinal, and one ratio variable and four rows. Then compute the average of the ratio column with `sum(...) / len(...)`, and the set of distinct values of the nominal column. Finally, turn your dict into a dataframe with `pd.DataFrame(...)` and check `.dtypes`.

    ??? note "Show a solution"
        ```python
        import pandas as pd

        data = {
            "product": ["chair", "lamp", "desk", "lamp"],   # nominal
            "size":    ["M", "S", "L", "S"],                # ordinal
            "price":   [149, 39, 259, 42],                  # ratio
        }

        avg_price = sum(data["price"]) / len(data["price"])   # 122.25
        distinct_products = set(data["product"])              # {'chair','lamp','desk'}

        df = pd.DataFrame(data)
        df.dtypes    # product: object, size: object, price: int64
        ```
        The learning is in the seams: `size` is ordinal but pandas stores it as plain `object` text; the software doesn't know "S < M < L", so preserving that order later is *your* job, not the tool's. That gap between what you mean and what the computer records is a theme for the rest of Part I.
