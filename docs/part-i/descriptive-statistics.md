<p class="chapter-kicker">Part I · Chapter 7 · <span class="week">Week 4</span></p>

# Descriptive statistics, and looking before you conclude

!!! guiding "The question this chapter answers"
    How do I capture a dataset in a few numbers without distorting it?

There is a strong temptation, once your data is clean, to jump straight to a conclusion: run the test, fit the model, produce the number the stakeholder asked for. Resist it for a while. The most reliable move in all of data work is also the humblest: *look at the data first.* Plot it. Summarise it. Poke at its extremes. Find out what shape it actually has before you impose one on it. This practice has a name: exploratory data analysis, or EDA.

!!! matters "Why this matters for designers"
    A single statistic usually becomes the headline of a data story. "Average response time fell 20%" is a design decision about what to foreground. If the mean is being dragged by outliers, or the groups behind it diverge, your headline misleads even when the arithmetic is correct. This chapter is about interpretation rather than formulas: knowing which summary fits which question is what lets you choose an honest insight to build a design around.

!!! newsroom "Why this matters for journalists"
    One number usually becomes the lede, and the choice between mean and median is an editorial decision even when both are arithmetically correct. An average dragged upward by a handful of extreme cases, or a total that hides two groups moving in opposite directions, produces a headline that is wrong in the way that gets corrected. And correlation presented as cause is the single most common data-story error there is: the arithmetic checks out, the sentence doesn't.

!!! outcomes "By the end of this chapter you can"
    - Compute and interpret measures of centre (mean, median, mode) and spread (range, standard deviation, IQR).
    - Choose mean vs median appropriately for a skewed distribution, and say why.
    - Read a distribution from a histogram, and read centre, spread, skew and outliers from a box plot.
    - Read a correlation without inferring causation, and name a plausible lurking variable.
    - Identify where a summary misleads: skew, outliers, hidden subgroups, and small-sample noise.
    - Recognise Simpson's paradox, and check that two groups you're comparing are made of the same kinds of thing.
    - Spot when an average moved because its underlying population changed rather than because anything improved.
    - Look for a change in rules or measurement before reading a kink in a trend line as a change in behaviour.
    - Report every summary with its *n*, and refuse to over-read a difference between small groups.
    - Run a quick exploratory pass on a new dataset and report the most surprising *shape* you found, not just a number.

It also has a patron saint.

!!! origins "Origins — Tukey's argument for looking"
    In 1977 the statistician John Tukey published *Exploratory Data Analysis*, a quiet revolt against the culture of leaping to hypothesis tests. Tukey, who also coined the words "bit" and "software", argued for visual, iterative investigation of data *before* any formal modelling, and invented simple tools like the box plot to make it easy. His deeper claim, sketched as early as 1962, was that a discipline broader than statistics was needed to handle real data in the wild. The profession eventually grew into the space he described; every quick histogram you make before doing anything serious is Tukey's advice in action.

## What you're looking for

Exploration is not aimless. You're building a mental model of the data along a few dimensions. For each important column you want its **distribution**: bunched or spread, symmetric or lopsided, one peak or several? Its **extremes**, the max and min, and whether outliers are real signals or data-entry ghosts (an age of 999 is the database saying "unknown," not a very old customer). And where columns meet, their **relationships**: do two variables move together, how tightly, in what direction?

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

rng = np.random.default_rng(7)
data = pd.DataFrame({
    "age":   rng.integers(18, 70, 200),
    "spend": rng.gamma(2.0, 40, 200).round(2),   # a skewed, realistic spread
    "plan":  rng.choice(["free", "pro", "team"], 200, p=[0.6, 0.3, 0.1]),
})

data.describe()                 # numeric summary: count, mean, min, max, quartiles
data["plan"].value_counts()     # category counts

data["spend"].hist(bins=20)     # the shape of one variable
plt.xlabel("spend (€)"); plt.ylabel("count"); plt.title("Distribution of spend")
plt.show()
```

**How it works.** `.describe()` gives five-number summaries of the numeric columns; `.value_counts()` does the same job for categories. `.hist()` draws a histogram, the single most useful EDA plot, by slicing the range into `bins` and counting how many values fall in each. Run it and you'll see `spend` is *right-skewed*: a long tail of big spenders pulling the mean above the median. That single picture already warns you that "average spend" is a misleading headline number here.

Two cautions belong here, because they are the errors exploration is meant to catch. First, **summary statistics can lie by omission**: an average is one number standing in for a whole distribution, and very different distributions can share one. Second, and larger, **correlation is not causation**: two things moving together may be cause and effect, effect and cause, both driven by a hidden third thing, or coincidence at your sample size. Exploration tells you *that* two variables relate; it doesn't, alone, tell you *why*.

```python
data.groupby("plan")["spend"].agg(["mean", "median", "count"])  # compare groups
data[["age", "spend"]].corr()                                   # a correlation
```

!!! practice "In Practice — Anscombe's quartet"
    In 1973 Francis Anscombe built four small datasets sharing almost every summary statistic (same means, variances, correlation and regression line), yet, plotted, looking nothing alike. Reproduce it and see for yourself:

    ```python
    import pandas as pd, matplotlib.pyplot as plt
    q = {
      "I":  ([10,8,13,9,11,14,6,4,12,7,5],
             [8.04,6.95,7.58,8.81,8.33,9.96,7.24,4.26,10.84,4.82,5.68]),
      "II": ([10,8,13,9,11,14,6,4,12,7,5],
             [9.14,8.14,8.74,8.77,9.26,8.10,6.13,3.10,9.13,7.26,4.74]),
      "III":([10,8,13,9,11,14,6,4,12,7,5],
             [7.46,6.77,12.74,7.11,7.81,8.84,6.08,5.39,8.15,6.42,5.73]),
      "IV": ([8,8,8,8,8,8,8,19,8,8,8],
             [6.58,5.76,7.71,8.84,8.47,7.04,5.25,12.50,5.56,7.91,6.89]),
    }
    for name,(x,y) in q.items():
        print(name, "mean y =", round(sum(y)/len(y), 2))   # ~7.5 for all four
    fig, axes = plt.subplots(1, 4, figsize=(12, 3))
    for ax,(name,(x,y)) in zip(axes, q.items()):
        ax.scatter(x, y); ax.set_title(name)
    plt.show()
    ```

    Identical numbers, four completely different stories: a clean trend, a curve, a line dragged askew by one outlier, and a vertical stack betrayed by a single point. Keep the quartet on your desk. It is the most economical proof that you must *look*.

## Reading the summary numbers

`.describe()` hands you a column of numbers, but they're only useful if you know what each one *means* and when it lies. There are two things you ever want to know about a numeric variable: where its **centre** is, and how **spread out** it is.

For the centre, three measures, and the difference between them matters:

- The **mean** (arithmetic average) uses every value, and is therefore dragged around by extremes. One billionaire in a room of teachers makes the *mean* income absurd.
- The **median** (the middle value when sorted) ignores how far away the extremes are, so it's *robust*; the billionaire barely moves it. For skewed data, the median is the truthful "typical."
- The **mode** (most common value) is the one that also works for categories.

For the spread:

- The **range** (max − min) is simple but hostage to a single outlier.
- The **standard deviation** is the typical distance of values from the mean: small means values huddle near the average, large means they're scattered.
- The **interquartile range (IQR)**, the span of the middle 50%, from the 25th to the 75th percentile, is the robust measure of spread, the median's natural partner.

```python
s = data["spend"]
s.mean(), s.median(), s.mode().iloc[0]       # three kinds of "centre"
s.std()                                       # typical distance from the mean
s.quantile([0.25, 0.5, 0.75])                 # quartiles: the shape in three numbers
```

**How it works.** `.quantile([0.25, 0.5, 0.75])` returns the values below which 25%, 50% and 75% of the data fall, the same quartiles `describe()` reports. The practical rule: when a distribution is roughly symmetric, **mean + standard deviation** describe it well; when it's skewed or has outliers (as `spend` is), reach for **median + IQR**, because the mean and standard deviation are being distorted by the tail. Quoting a mean on skewed data isn't wrong so much as *misleading*; it's the most common way a summary number tells a small lie.

### The box plot, and why Tukey drew it

Tukey didn't just argue for looking; he designed a picture to look *with*. A **box plot** draws the five numbers you just computed as a shape, which makes it the fastest way to compare a distribution across groups:

```python
import matplotlib.pyplot as plt

data.boxplot(column="spend", by="plan", grid=False)
plt.suptitle("");  plt.title("Spend by plan");  plt.xlabel("plan");  plt.ylabel("spend (€)")
plt.show()
```

**How to read it.** The **box** spans the interquartile range, from the 25th to the 75th percentile, so it holds the middle half of the data. The **line inside** is the median. The **whiskers** reach out to the furthest points within 1.5 × IQR of the box, and anything beyond them is drawn as an individual dot — Tukey's convention for "worth a second look". Four things are legible at a glance: where the group sits, how spread out it is, whether it's **skewed** (the median sits off-centre in the box, or one whisker is much longer), and which points are unusual.

That skew reading is why the box plot pairs so well with the mean-versus-median question. If the median hugs the bottom of the box and the upper whisker runs long, you are looking at a right-skewed distribution and the mean is being pulled upward — visible in one glance, no arithmetic required.

Its one weakness is worth knowing: a box plot hides *multimodality*. Two distinct clusters and one broad blob can produce identical boxes, because the five summary numbers don't distinguish them. When the box looks suspiciously wide, draw a histogram of that group as well. The lesson is Anscombe's again, one level up — even a good summary picture is a summary.

!!! case "Case study — summarising ride length fairly"
    For the bike-share `trips`, don't just report the average ride:

    ```python
    d = trips["duration_min"]
    print("mean:", round(d.mean(), 1), " median:", d.median())
    print(d.quantile([0.25, 0.5, 0.75]).to_dict())
    ```

    If the mean sits well above the median, a handful of long joyrides are inflating it, and the median is the number you should put in front of a stakeholder. Same data, more truthful headline: the whole point of reading the summary numbers rather than just printing them.

### Which summary, and when

| Statistic | Measures | Robust to outliers? | Reach for it when |
|---|---|---|---|
| Mean | centre | ✗ dragged by extremes | roughly symmetric data; you need to add things up |
| Median | centre | ✓ | skewed data, or any distribution with a long tail |
| Mode | most common | ✓ | categories, where mean and median are meaningless |
| Range (max − min) | spread | ✗ hostage to one point | a quick sanity check on plausible limits |
| Standard deviation | spread | ✗ | symmetric data; it pairs with the mean |
| IQR | spread | ✓ | skewed data; it pairs with the median |
| Count | how much you have | — | **always** — every summary needs its *n* beside it |

That last row is the one people skip. "Casual riders travel 40% further" is a very different claim when it rests on 4,000 trips than on 4, and a summary reported without its count is an invitation to over-read it. Put the *n* next to every number you publish.

## When the average hides the answer

The most consequential failure of a summary isn't imprecision, it's a genuinely correct number that points the wrong way. This happens when a group you didn't split by is doing the work, and it has a name: **Simpson's paradox**.

```python
import pandas as pd

# A bike-share promotion. Did it lengthen trips?
trips = pd.DataFrame({
    "rider":    ["casual", "casual", "casual", "casual", "member", "member", "member", "member"],
    "promo":    ["no",     "no",     "no",     "yes",    "no",     "yes",    "yes",    "yes"],
    "duration": [30,       34,       32,       38,       8,        12,       14,       13],
})

trips.groupby("promo")["duration"].mean()
# no  -> 26.00   yes -> 19.25      the promo looks WORSE

trips.groupby(["rider", "promo"])["duration"].mean()
# casual: no 32.0 -> yes 38.0      the promo looks BETTER
# member: no  8.0 -> yes 13.0      the promo looks BETTER

trips.groupby(["rider", "promo"]).size()   # (1)!
```

1.  Always look at the group sizes too. They're the whole explanation.

Run it. Overall, trips with the promotion average nearly seven minutes *shorter*. Split by rider type, the promotion is associated with *longer* trips among casual riders **and** among members. Every one of those numbers is arithmetically correct.

**How it happens.** Look at the sizes. Casual riders take much longer trips than members (32 minutes against 8), and the promotion was taken up mostly by *members* — three of their four trips, against one of four for casual riders. So the "promo" group is disproportionately made of short-trip riders, and that composition, not the promotion, drags the overall average down. The lurking variable isn't hiding in the data; it's hiding in *who ended up in each group*. Unequal group sizes are the engine of the paradox, which is why `.size()` belongs beside every group comparison you make.

There is no formula that saves you here, which is exactly why this sits in a chapter about looking. The defence is a habit: **before you report a difference between two groups, check whether the groups are made of the same kinds of thing.** If they're not, either compare within a subgroup or say plainly that the comparison is confounded. Reporting the headline and hoping nobody splits it is how a defensible analysis becomes an indefensible one.

!!! critical "Critical Lens — Aggregation is a political act"
    Simpson's paradox is usually taught as a curiosity. It is better understood as the mathematical form of a question about power: *who gets grouped with whom?* Real cases are rarely innocent. A university's overall admission rate can look unbiased while every individual department is biased, or the reverse. A national pay-gap figure can shrink or grow depending on whether you control for occupation — and whether you *should* control for occupation is itself the argument, because occupational segregation may be part of the thing you're measuring rather than a nuisance to remove.

    So the choice of what to aggregate over is never purely technical. It decides which disparities become visible and which dissolve into an average. When someone hands you a single national or company-wide number, the useful question is not "is this correct?" but "what does splitting it reveal, and who benefits from it staying whole?"

## A worked case: what arithmetic did to basketball

Everything in this chapter — a summary that misleads, an average that moves because its population moved, a number that was sitting in plain sight until somebody read it properly — happened to a professional sport in public, over thirty years, and it is the clearest real example I know.

I grew up watching the Orlando Magic of Shaquille O'Neal and Anfernee "Penny" Hardaway, the team that reached the 1995 NBA Finals and was swept by the Houston Rockets. That basketball was built around a giant near the basket and the mid-range jumper: Penny coming off a screen and shooting from fifteen feet. Today that shot is close to extinct. Here is why, in four columns:

| Season | 3-pointers attempted per game | 3-point accuracy | Points per 3-point attempt | Points per 2-point attempt |
|---|---|---|---|---|
| 1992-93 | 9.0 | 33.3% | 1.00 | 0.98 |
| 1994-95 | 15.3 | **35.95%** | 1.08 | 0.98 |
| 2025-26 | 37.0 | **35.95%** | 1.08 | 1.10 |

**First reading: the accuracy column.** In 1994-95 the league shot 35.95% from three. In 2025-26 it shot 35.95% from three. Identical to four decimal places. Nobody got better at shooting threes. What changed is that a 36% three is worth 1.08 points per attempt, an average two back then was worth 0.98, and 1.08 is larger than 0.98. That is the entire insight, it required no statistics beyond multiplication, and it was printed in the newspaper every morning of the 1995 season while teams took four times as many twos as threes. **The data hadn't changed. The reading of it had.** Thirty years later, attempts have gone from 15 a game to 37.

**Second reading, and this is the subtler one: the last column.** Two-pointers today yield 1.10 points per attempt against 0.98 in the nineties. Did players get dramatically better at two-point shots? No. The *composition* changed. Once the mid-range was abandoned, the twos that remained were mostly layups and dunks, which go in at a far higher rate. The average rose because the population behind it moved — Simpson's paradox in the wild, and a reminder that "two-point shot" was never one thing.

Two habits from this chapter would have caught both readings. Comparing a rate across groups rather than trusting a total. And asking what a category actually contains before averaging over it.

!!! critical "Critical Lens — Being right is not the same as being harmless"
    I want to resist the tidy version of this story, in which data arrives and improves a sport. The analysis was right; you cannot argue with 1.08 against 0.98. But it did not only make teams efficient. It changed what the game looks like, which skills are worth developing, which players get contracts, and it retired a shot that people loved to watch. A method arrived carrying a taste, and the taste won.

    Expect this wherever you take your own analysis. Optimising for a measurable quantity reshapes the thing being measured, and the parts that don't appear in your metric are the parts that quietly go. Something true was found and something valuable was lost, and both were the same event. Your job is not to pretend the trade-off away, but to name it — so that whoever acts on your number knows what it is optimising and what it is willing to spend.

!!! further "Going Further — Check my numbers"
    Attempts, makes and accuracy are league averages per game from [Basketball-Reference](https://www.basketball-reference.com/leagues/NBA_stats_per_game.html). The points-per-attempt columns are derived: accuracy × points per shot, with two-point figures computed by subtracting three-point makes and attempts from the totals. Recomputing them from the source is a decent exercise in itself.

    One complication I left out of the table, which is a good lesson in reading a time series: the NBA shortened the three-point line to a uniform 22 feet from 1994-95 to 1996-97, which is part of why attempts jumped that season. When the line moved back in 1997-98, attempts *fell* — from 16.8 to 12.7. So a rule change moved behaviour temporarily and the arithmetic moved it permanently. Whenever a trend line has a kink in it, look for a change in the *measurement or the rules* before you reach for a change in behaviour.

## Exploration is a loop, not a checklist

EDA isn't a stage you finish and leave behind; it's a loop. You look, notice something odd, form a hunch, check it, and the check usually turns up something new. A gap in a distribution sends you back to the cleaning question of whether a category is missing. A surprising correlation sends you to ask whether a lurking variable explains both. The genuinely useful findings come from the "huh, that's strange" you only reach by wandering the data with your eyes open, not from the pre-planned test.

## It's only a sample

One habit of mind separates people who read data well from people who over-read it: remembering that your dataset is almost always a **sample**, and that samples wobble. If you split 60 trips into two groups of 30 and the averages differ by a minute, that difference might be real, or it might be the coin-flip luck of which trips landed in which group. The smaller the group, the more the numbers jump around from nothing but chance, and the easier it is to mistake **noise** for a finding.

You don't need formal statistics yet to build the instinct. Two guardrails go a long way. First, **be suspicious of differences between small groups**: a "huge gap" between two categories with a dozen rows each is often just sampling noise. Second, **look at how much a number moves when the data changes slightly**: if dropping a few rows swings your headline result, it was never solid. The question "could this just be chance?" is the doorway to Part II, where evaluating whether a pattern is real becomes the whole game. For now, simply refusing to over-claim from a thin sample already puts you ahead.

!!! further "Going Further — From looking to testing, carefully"
    When exploration turns up something you want to *claim*, you cross from EDA into inference, where the ghost of Ronald Fisher waits. Fisher's 1925 *Statistical Methods for Research Workers* gave science its toolbox (significance testing, the p-value, randomised experimental design), and today's A/B testing culture in product design is essentially Fisher's methods at web scale. The one caution to carry across the border: exploration is where you *generate* hypotheses, testing is where you *check* them, and if you generate and confirm a hypothesis on the very same data ("p-hacking"), you've proven nothing but your own persistence. Look freely; confirm with discipline.

Looking first is a posture of humility toward the material: you let the data show you its shape before telling it what to be. In the next chapter that looking becomes public: exploration turned outward, into visualisation designed to make an argument someone else can see.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. Two chapters, and you can skip the probability ones in between.

    - [Introduction to Statistics in Python](https://app.datacamp.com/learn/courses/introduction-to-statistics-in-python) — **Ch. 1 Summary Statistics** and **Ch. 4 Correlation and Experimental Design.**
