<p class="chapter-kicker">Part I · Chapter 6</p>

# Exploratory data analysis, looking before modelling

There is a strong temptation, once your data is clean, to jump straight to a conclusion: run the test, fit the model, produce the number the stakeholder asked for. Resist it for a while. The most reliable move in all of data work is also the humblest: *look at the data first.* Plot it. Summarise it. Poke at its extremes. Find out what shape it actually has before you impose one on it. This practice has a name, exploratory data analysis, or EDA, and a patron saint.

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

!!! case "Case study — summarising ride length fairly"
    For the bike-share `trips`, don't just report the average ride:

    ```python
    d = trips["duration_min"]
    print("mean:", round(d.mean(), 1), " median:", d.median())
    print(d.quantile([0.25, 0.5, 0.75]).to_dict())
    ```

    If the mean sits well above the median, a handful of long joyrides are inflating it, and the median is the number you should put in front of a stakeholder. Same data, more truthful headline: the whole point of reading the summary numbers rather than just printing them.

## Exploration is a loop, not a checklist

EDA isn't a stage you finish and leave behind; it's a loop. You look, notice something odd, form a hunch, check it, and the check usually turns up something new. A gap in a distribution sends you back to the cleaning question of whether a category is missing. A surprising correlation sends you to ask whether a lurking variable explains both. The genuinely useful findings come from the "huh, that's strange" you only reach by wandering the data with your eyes open, not from the pre-planned test.

## It's only a sample

One habit of mind separates people who read data well from people who over-read it: remembering that your dataset is almost always a **sample**, and that samples wobble. If you split 60 trips into two groups of 30 and the averages differ by a minute, that difference might be real, or it might be the coin-flip luck of which trips landed in which group. The smaller the group, the more the numbers jump around from nothing but chance, and the easier it is to mistake **noise** for a finding.

You don't need formal statistics yet to build the instinct. Two guardrails go a long way. First, **be suspicious of differences between small groups**: a "huge gap" between two categories with a dozen rows each is often just sampling noise. Second, **look at how much a number moves when the data changes slightly**: if dropping a few rows swings your headline result, it was never solid. The question "could this just be chance?" is the doorway to Part II, where evaluating whether a pattern is real becomes the whole game. For now, simply refusing to over-claim from a thin sample already puts you ahead.

!!! further "Going Further — From looking to testing, carefully"
    When exploration turns up something you want to *claim*, you cross from EDA into inference, where the ghost of Ronald Fisher waits. Fisher's 1925 *Statistical Methods for Research Workers* gave science its toolbox (significance testing, the p-value, randomised experimental design), and today's A/B testing culture in product design is essentially Fisher's methods at web scale. The one caution to carry across the border: exploration is where you *generate* hypotheses, testing is where you *check* them, and if you generate and confirm a hypothesis on the very same data ("p-hacking"), you've proven nothing but your own persistence. Look freely; confirm with discipline.

Looking first is a posture of humility toward the material: you let the data show you its shape before telling it what to be. In the next chapter that looking becomes public: exploration turned outward, into visualisation designed to make an argument someone else can see.

## Exercises

??? exercise "Exercise 6.1 · A five-minute EDA"
    On any dataset, produce in under five minutes: one `.describe()`, one histogram of a numeric column, one `.value_counts()` of a category, and one group-and-aggregate comparing a number across a category. Write two sentences on the most surprising thing you saw.

    ??? note "What good looks like"
        A strong answer notices a *shape*, not just a number: "spend is heavily right-skewed, so the mean (€82) badly overstates the typical customer (median €54); I should report the median." Spotting that the summary statistic misleads is exactly what EDA is for.

??? exercise "Exercise 6.2 · Correlation is not causation"
    Find (or construct) two columns in a dataset that are correlated but where causation is doubtful or reversed. Write a short paragraph proposing at least two explanations for the correlation *other than* "the first causes the second."

    ??? note "A model answer"
        "Ice-cream sales and drowning deaths correlate strongly across the year. But neither causes the other: a lurking third variable, hot weather, drives both. In our product data, 'number of support tickets' correlates with 'revenue,' but that's because both track *number of active users*; concluding that support tickets cause revenue (or vice versa) would be a costly mistake." Naming the lurking variable is the whole skill.
