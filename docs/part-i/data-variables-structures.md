<p class="chapter-kicker">Part I · Chapter 1 · <span class="week">Week 1</span></p>

# Data, variables, and data structures

!!! guiding "The question this chapter answers"
    What is data actually made of, and what does each kind of variable let me legitimately do with it?

Before we ask where a dataset comes from or how to clean it, we need to know what data is actually *made of*. A dataset looks like one thing, a table, a file, but it is built from smaller parts, and every one of those parts is a decision. A single value is a decision about how to record one observation. A variable is a decision about what to measure and how finely. A data structure is a decision about how to hold it all together so a computer can work with it. Learn these building blocks and the rest of Part I becomes much less mysterious; skip them and you'll spend later chapters confused about why a chart won't draw or a number won't add up.

This is the most "technical" chapter in the part, but it stays close to the material. Think of it as learning the properties of your clay, what it's made of, and what each kind can and can't do, before you throw a pot.

!!! matters "Why this matters for designers"
    Most people consume charts and dashboards as finished objects. Your job starts one step earlier, at the raw material — and the material has a grain. Whether a column is a label or a quantity decides which summaries are honest and which charts are legible, and that decision was made by someone else before the file reached you. Learning to read the grain is how you stop inheriting other people's mistakes.

!!! newsroom "Why this matters for journalists"
    Data arrives in a newsroom as a file with no one to interview: a FOI release, a leaked export, a regulator's download. Whether a column is a label or a quantity, and whether its codes have an order, decides which claims the file can support — and that was settled by whoever designed the form long before it reached you. Reading the data dictionary is source assessment, and the answers are also your best list of questions for the agency that produced it.

!!! outcomes "By the end of this chapter you can"
    - Tell a value, a variable and a data structure apart, and say what one row and one column of a table represent.
    - Classify a variable as nominal, ordinal, interval or ratio, and name a summary and a chart its level makes appropriate — and one it rules out.
    - Spot the three kinds of column that look numeric but aren't quantities: identifiers, codes, and cyclical values.
    - Treat dates and missing values as first-class parts of your material rather than afterthoughts.
    - Use Python's core types (`int`, `float`, `str`, `bool`, `None`, `datetime`) and structures (list, dict, tuple, set) deliberately.
    - Build a table from a dictionary of lists, and check your assumptions with `.dtypes` and `.nunique()`.
    - Explain why choosing a variable's level is a claim about the world, not a technical formality.

## Values and variables

Start with the two smallest ideas. A **value** is a single recorded observation: the number `31`, the word `"Utrecht"`, the answer `True`. A **variable** is the *attribute* those values measure, age, city, whether someone subscribed, and in a table, a variable is usually a **column**. One column, one variable; one row, one observation; one cell, one value. That simple correspondence (championed as "tidy data") is the mental model to carry everywhere: **columns are variables, rows are observations.**

The word "variable" is borrowed from statistics, not programming, and it's worth keeping the statistical meaning in mind: a variable is something that *varies* across the things you observe. If a column never changes (every row says `"active"`), it isn't telling you anything, however much space it takes up.

That thought has a practical extension. Ask of every column *how many distinct values it holds* — its **cardinality** — because the answer tells you what kind of thing you're looking at before you know anything else about it. One distinct value: a constant, carrying no information. Two: a flag, effectively yes/no. A handful: categories you can group by, colour by, put on an axis. Dozens: usable, but a bar chart of them will be unreadable. As many distinct values as there are rows: not a variable at all in the statistical sense, but an **identifier** — a name for each row rather than a measurement of it. Cardinality is the cheapest diagnostic in data work, and you'll compute it in one line at the end of this chapter.

## The levels of a variable, the grain of your material

Not all variables are the same *kind*, and the kind determines what you're allowed to do with them. This is the single most useful distinction in the chapter. Variables come in two broad families, each with two levels.

**Categorical** variables sort observations into named groups. They come as **nominal**, pure labels with no order (`city`, `plan`, `colour`), or **ordinal**, categories with a meaningful order but no fixed spacing (`small/medium/large`, a 1–5 satisfaction rating). You can count categories and find the most common, but "the average city" is nonsense.

**Numeric** variables measure quantity. They come as **interval**, numbers where differences are meaningful but there's no true zero (temperature in °C, calendar years), or **ratio**, numbers with a true zero, so ratios make sense (price, age, distance; €200 really is twice €100). With numeric variables you can add, average, and measure spread.

Why labour this? Because the level of a variable silently decides which statistics and which charts are legitimate. You can take the **mean** of a ratio variable but not of a nominal one. You compare categories with a **bar chart** and show the distribution of a numeric variable with a **histogram**, and mixing them up is a common way to draw a misleading picture.

One honest caveat, so you don't over-invest in the taxonomy: in day-to-day work the line that constantly matters is **categorical versus numeric**, and within categorical, **ordered versus unordered**. The interval/ratio distinction is real but rarely changes what you do — you will almost never be stopped by the fact that Celsius has no true zero. Learn all four so you can read the literature and argue precisely when it counts, but the working question is the coarse one: *is this a label or a quantity, and if it's a label, does it have an order?*

### Numbers that aren't quantities

The trap the levels exist to catch is a column full of digits that isn't measuring anything. Three kinds recur, and all three are nominal despite looking numeric:

**Identifiers** name a row rather than describe it: `trip_id`, `order_id`, a customer number. Their cardinality equals the row count, their values are arbitrary, and the only legitimate operations are counting them, checking they're unique, and joining on them. An average `order_id` is a number your software will happily produce and that means nothing at all.

**Codes** are labels that happen to be written in digits: postal codes, phone numbers, country calling codes, a survey's `1 = strongly agree`. The digits are a naming convention, not a magnitude. These also break in a second way: a leading zero is information, and any tool that reads the column as a number will silently delete it — which is why a Dutch postcode should live in your data as text.

**Cyclical values** are the subtlest, because arithmetic on them half works. Hour of day, day of week, compass bearing, month number: 23:00 and 01:00 are two hours apart, but their numeric difference is 22, and the mean of hours 23 and 1 comes out as noon. Treat them as ordered categories, or handle the wrap-around deliberately.

### One table to keep

Most of this chapter collapses into a lookup you'll use all year:

| Level | Example | Typical pandas dtype | Honest summaries | Reach for |
|---|---|---|---|---|
| Nominal | `city`, `membership` | `object`, `category` | count, mode, proportions | bar chart |
| Nominal — identifier | `trip_id` | `int64`, `object` | count, uniqueness check | nothing; it's a key |
| Ordinal | `size` S/M/L, 1–5 rating | ordered `category` | median, quartiles, counts | bar chart in category order |
| Interval | temperature °C, calendar year | `float64`, `int64` | mean, standard deviation, differences | line, histogram |
| Ratio | price, age, duration | `float64`, `int64` | mean, median, standard deviation, ratios | histogram, scatter, line |
| Boolean | `long_ride` | `bool` | count, proportion (a mean *is* a proportion) | bar chart, or just a number |
| Date / time | `start_time` | `datetime64[ns]` | earliest, latest, range, counts per period | line over time |

The right-hand columns are the point. The level is not a label you file away; it's a decision procedure that tells you what you may compute and what you may draw.

!!! practice "In Practice — Telling pandas what you meant"
    Note the dtype for ordinal in that table. You don't have to leave your level judgements in a comment: pandas' `category` dtype can carry an explicit order, so the tool enforces what you know.

    ```python
    import pandas as pd
    sizes = pd.Categorical(["M", "S", "L", "S"],
                           categories=["S", "M", "L"], ordered=True)   # order declared
    sizes.min()                     # 'S' — comparisons now work as you'd expect
    pd.Series(sizes).sort_values()  # sorts S, S, M, L — not alphabetically
    ```

    Three payoffs: sorting and comparison respect your order rather than the alphabet, charts put categories in the sensible sequence instead of the one they happened to appear in, and a categorical column with few distinct values uses far less memory than repeated text. It's the difference between *knowing* a variable is ordinal and *recording* that you know.

!!! origins "Origins — Stevens names the scales of measurement"
    The four-level scheme, nominal, ordinal, interval, ratio, was set out by the psychologist S. S. Stevens in a 1946 paper, *On the Theory of Scales of Measurement*. Stevens was arguing about how to quantify human sensation, and needed a principled account of when a number was "really" a number. Statisticians have argued with him ever since — most sharply that his rules are too strict to be useful, since researchers compute means of Likert-scale ratings all the time and mostly get away with it, and that four levels are too few for the variables people actually collect (counts, proportions and cyclical values all sit awkwardly). The taxonomy endures anyway, because it answers a practical question every analyst faces: *what operations is this variable entitled to?* When a tool lets you compute the average of a category code and hands you a confident, meaningless answer, it's Stevens you needed and didn't have.

!!! critical "Critical Lens — Choosing a level is choosing a worldview"
    Deciding a variable's level is not a neutral technical act; it encodes a claim about the thing being measured. Record `gender` as two nominal categories and you've built a world with exactly two, invisibly excluding anyone who doesn't fit. Turn a rich, ordered experience like pain or satisfaction into an ordinal 1–5 and you've decided that the distance from "1" to "2" is comparable to "4" to "5", which it may not be. These choices, made once when the variable is defined, propagate into every later average and chart. This is the same lesson as [Chapter 5](where-data-comes-from.md)'s missing datasets, one level down: what you *can't* measure faithfully is decided before you collect a single value.

## Python's building blocks: data types

The level of a variable is what you *mean*. A **type** is what the computer has actually stored, and the two can disagree. Every value in Python carries a type that governs what you can do with it, and you'll meet six constantly:

```python
age      = 31          # int      — a whole number
price    = 149.95      # float    — a number with a decimal point
city     = "Utrecht"   # str      — text ("string"), always in quotes
active   = True        # bool     — True or False
missing  = None        # NoneType — the absence of a value
from datetime import datetime
start    = datetime(2026, 5, 1, 8, 12)   # datetime — a moment in time

type(age), type(price), type(city), type(active)
# -> (int, float, str, bool)
```

**How it works.** `type(x)` reports how Python is storing a value, and that storage decides its behaviour. `"31" + "31"` gives `"3131"` (text joined end to end) while `31 + 31` gives `62` — same-looking characters, completely different results, because one is a `str` and the other an `int`. This is exactly the "numbers stored as text" trap you'll meet again in [Chapter 3](cleaning-messy-data.md): the *type* and the *level of measurement* have to agree with your intent, or the data will quietly misbehave.

The last two deserve sections of their own, because they're the ones tutorials skip and real datasets are full of.

## Dates are a type, not a string

A date arrives from a file as text — `"2026-05-01 08:12"` — and until you convert it, that's all it is. Sorted as text it will put `"2026-10-01"` before `"2026-9-01"`; subtracted, it will raise an error. Converted to a real datetime, it becomes the richest column in most datasets, because a single timestamp silently contains a whole family of variables:

```python
import pandas as pd

start = pd.to_datetime("2026-05-01 08:12")   # text -> a real moment in time

start.year, start.month, start.day     # 2026, 5, 1     — interval-level numbers
start.hour                             # 8              — cyclical! (23 and 1 are close)
start.day_name()                       # 'Friday'       — ordinal category
start.date()                           # 2026-05-01     — the day it belongs to

later = pd.to_datetime("2026-05-01 08:40")
later - start                          # Timedelta('0 days 00:28:00') — a ratio quantity
```

**How it works.** `pd.to_datetime` parses text into a timestamp; from there the *parts* are attributes you can read off. Notice that each part lands at a different level: `year` is interval, `day_name()` is an ordered category, `hour` is cyclical, and the *difference* between two datetimes is a duration — a ratio quantity with a true zero, which is why you can meaningfully say one trip took twice as long as another.

So a datetime is best understood not as one variable but as a *source* of variables, and which one you extract is a design decision with consequences. Group your bike trips by `date` and you get a trend; group the same data by `hour` and you get a daily rhythm; group by `day_name()` and you get the difference between commuting and weekends. Same column, three different stories — and you choose. Chapter 3 turns this into working code with the `.dt` accessor and `resample`.

!!! practice "In Practice — Write dates the boring way"
    `03/01/2026` is the 3rd of January in Europe and the 1st of March in the US, and software guesses — often wrongly, always silently. Use **ISO 8601** (`2026-01-03`, year–month–day) whenever you record or export a date yourself. It sorts correctly as plain text, it's unambiguous in every country, and it's what `pd.to_datetime` reads without being told. One boring convention prevents a class of error you would otherwise never notice.

## Missing values are values too

The most consequential cell in a dataset is often an empty one, and Python distinguishes several kinds of empty that beginners tend to conflate:

```python
import numpy as np
import pandas as pd

None          # Python's "nothing here"
np.nan        # the *numeric* missing value — a special float
""            # an empty string: present, but blank. NOT missing.
0             # a real, measured zero. Definitely NOT missing.
"unknown"     # missing, disguised as a category

pd.isna([None, np.nan, "", 0, "unknown"])
# -> [True, True, False, False, False]
```

**How it works.** `pd.isna` recognises `None` and `np.nan` as missing and nothing else — so the last three slip through as ordinary values. That's the whole problem in one line: an empty string, a zero, and the word `"unknown"` all *mean* "no data" to a human reader and *count as data* to your code. A column with two hundred `"unknown"` entries will report zero missing values and cheerfully include them in `value_counts()` as the most common category.

Two habits follow. First, **make missingness explicit** as soon as you spot it, so the tooling can see it — a job you'll do properly in [Chapter 3](cleaning-messy-data.md). Second, and more important, **treat a blank as a finding, not a nuisance**. Missing data is *evidence about the collection process*: a question people declined to answer, a sensor that failed in a particular condition, a field a form only showed to some users. Ages missing at random are an inconvenience; ages missing only for one group are a bias that will survive every clean-up you perform downstream.

!!! critical "Critical Lens — Zero and blank are different claims"
    Confusing a zero with a blank is not a coding slip; it changes what your data asserts. "This shop sold 0 units" is a measurement. "We have no sales figure for this shop" is an absence of measurement. Fill the blanks with zeros — a one-keystroke default in every spreadsheet — and you have silently converted ignorance into a confident claim, one that will lower every average and flatten every chart it touches. The reverse error is quieter still: import a file badly so that genuine zeros read as blanks, and you erase real events. Whenever you meet an empty cell, ask which of the two it is, and write the answer down.

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
df.nunique()              # cardinality: how many distinct values per column
```

**How it works.** `pd.DataFrame(data)` reads each key as a column heading and each list as that column's values, snapping the loose Python structures into a proper table. `.dtypes` shows the type pandas inferred per column (`object` for text, `int64`/`float64` for numbers), the software's echo of the levels-of-measurement idea. `.nunique()` counts distinct values, so the two lines together give you a first read on every column: *what the computer thinks it is*, and *how many different things it holds*. Recognising that a dataframe is *made of* dictionaries and lists demystifies it: it isn't a magic object, it's the humble containers from this chapter, arranged in a grid.

Those two lines are also a *disagreement detector*. Pandas infers types from the characters in the file; it has no idea what you mean. A postcode column reads as `int64` when you meant a label. An ordinal `size` column reads as plain `object`, with S/M/L in whatever order they happened to appear. Cardinality equal to the row count says "this is an identifier" no matter what dtype it wears. Every one of those gaps is yours to close, and closing them is the actual work of the chapter.

!!! case "Case study — classify the bike-share columns"
    The dataset that runs through Part I is a synthetic Utrecht bike-share, one row per trip. Before you compute anything with it, name what each column is:

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

    trips.dtypes      # what pandas thinks each column is
    trips.nunique()   # how many distinct values each one holds
    ```

    Read the two outputs together against the table above:

    | Column | Level | Cardinality tells you | Legitimate |
    |---|---|---|---|
    | `trip_id` | nominal — **identifier** | 6 of 6 rows: a key | count, uniqueness check |
    | `start_time` | date/time | 6 of 6: near-unique, as timestamps are | range, and the variables you extract from it |
    | `station` | nominal | 3: a groupable category | counts per station, bar chart |
    | `membership` | nominal, binary | 2: a flag | proportions, group comparisons |
    | `duration_min` | ratio | 6: a genuine quantity | mean, median, spread, histogram |

    Two things worth noticing. `trip_id` and `duration_min` are both `int64`, and pandas will average either one for you — but only one of those averages means anything. And `membership` with two values is the column that makes the dataset *comparative*: nearly every question you'll ask of this data in later chapters is "…and does it differ between members and casual riders?" Naming a two-value category as a flag is how you spot the comparison the data is built to support.

    Now derive one:

    ```python
    trips["long_ride"] = trips["duration_min"] >= 15   # bool — a new variable, your decision
    trips["long_ride"].mean()                          # 0.33 — the mean of a bool IS a proportion
    ```

    You've just added a variable the data didn't contain, at a level you chose, using a threshold you picked. Fifteen minutes is not a fact about cycling; it's an editorial judgement that now shapes every finding downstream. Write it in your provenance note.

!!! practice "In Practice — Name the level before you touch the data"
    A five-minute habit that pays off all year. For each column in a new dataset, write down four things: its **level**, its **cardinality**, whether it's secretly an **identifier or a code**, and whether it has **missing values and why**. Then run `.dtypes` and `.nunique()` and look for the places where pandas disagrees with you. That's the whole diagnostic, and it takes less time than making a single chart. Most analysis mistakes you'll make early on are really a variable being treated at the wrong level: a category averaged, an ID plotted, a quantity chopped into arbitrary bins, or a blank counted as a zero.

!!! further "Going Further — Why models will make you turn categories into numbers"
    Everything here returns with force in Part II. Machine-learning models are, underneath, arithmetic; they can only consume numbers. So categorical variables have to be **encoded** into numeric form before a model can use them (one column per category, a `1` or `0` in each, "one-hot encoding"), and doing this naïvely to an *ordinal* variable throws away its order, while doing it to a high-cardinality *nominal* one (say, thousands of postcodes) explodes your data. The level-of-measurement judgement you make here is what tells you which encoding is faithful. Keep it in mind; it's why this "basics" chapter is really groundwork for the whole book.

You now know what data is built from: values with types, variables with levels and cardinalities, dates that unpack into whole families of variables, blanks that mean something, and the structures that hold it all together. None of it is neutral — every one of those was a decision someone made, and from here on you're the one making them. With the material understood, the next chapter hands you the instrument you'll shape it with: the dataframe, and the handful of moves that turn a question into a table operation.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. Work through them after reading — the concepts land properly only once you have typed them.

    - [Introduction to Data Science in Python](https://app.datacamp.com/learn/courses/introduction-to-data-science-in-python) — **Ch. 1 Getting Started in Python.** Variables, types and your first lines of Python.
    - [Introduction to Python](https://app.datacamp.com/learn/courses/intro-to-python-for-data-science) — **Ch. 2 Python Lists** *(if lists and indexing are new to you)*.
