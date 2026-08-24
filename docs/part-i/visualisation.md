<p class="chapter-kicker">Part I · Chapter 8 · <span class="week">Week 5</span></p>

# Visualisation as argument

!!! guiding "The question this chapter answers"
    How do I choose the visualisation that answers a question?

A chart is not a decoration you add once the analysis is done. A chart is an argument, a claim about what matters in the data, made in a form the eye grasps faster than any table. This is the chapter where a designer's existing skills become a superpower, because visualisation sits exactly at the join between data work and design, and most people are good at one and weak at the other. You can be good at both. But that power comes with an ethics, because a chart persuasive enough to change a decision is also persuasive enough to mislead, and often the same chart does both.

!!! matters "Why this matters for designers"
    This is the most visibly "designerly" chapter in Part I, and the temptation to over-design is strongest here. Good data visualisation is an act of *editing*: removing everything that doesn't help the reader reason, and arranging what remains so the insight is unmissable. Sequence, framing and emphasis matter as much as the chart type. And honesty is a constraint, not an afterthought — a truncated axis or a hidden subgroup misleads even when every number is correct.

!!! newsroom "Why this matters for journalists"
    A published chart is a claim with your name on it, and readers who never finish the article will still take a number away from it. That makes the graphics decisions editorial ones: which comparison you draw, what you annotate, and what you leave out. It also makes the familiar sins — a truncated axis, a cherry-picked start year, a subgroup quietly folded into the total — corrections waiting to happen, because every figure in them is accurate.

!!! outcomes "By the end of this chapter you can"
    - Match a chart type to the question being asked, and name the visual channel each variable is encoded in.
    - Build and customise static charts in matplotlib and Seaborn for a defined audience.
    - Spot and avoid misleading encodings: truncated axes, poor channels, chartjunk, hidden subgroups.
    - Draw small multiples on shared axes to show a subgroup difference instead of averaging it away.
    - Write a title that states your claim, and annotate the feature you're arguing about.
    - Use colour to carry meaning, pick the right palette family, and make a chart that survives in greyscale.
    - Sequence a few charts into a short, honest narrative, and defend each choice in terms of the reader's decision.

The field was born persuasive. The agenda came first.

!!! origins "Origins — Visualisation was invented to change minds"
    William Playfair drew the first line and bar charts in his 1786 *Commercial and Political Atlas*, arguing that the eye grasps a shape faster than a column of figures, the founding claim of the whole field. What followed were not neutral diagrams but arguments. In 1854 John Snow mapped cholera deaths around a Soho water pump and used the picture to overturn the reigning theory of disease and get the pump handle removed. In 1858 Florence Nightingale designed her polar-area "rose" diagrams so politicians could not ignore that most Crimean War deaths came from preventable disease, not wounds. And Charles Minard's 1869 flow-map of Napoleon's march packed six variables into one image so eloquent that Edward Tufte later called it perhaps the best statistical graphic ever drawn. Every one was visualisation as rhetoric. That is the tradition you are joining.

## The grammar underneath every chart

Beneath the zoo of chart types is one simpler idea: a chart is a *mapping* from data to visual properties. You take a variable and assign it to a visual channel: position along an axis, length, area, colour, shape. Choose well and the reader sees the truth almost effortlessly; choose badly and you bury it. And this isn't only taste: some channels are simply more accurate than others. The eye reads *position* and *length* precisely and judges *area*, *angle* and *colour intensity* poorly, which is the real reason a bar chart usually beats a pie chart, and why encoding an important quantity as the *area* of a bubble makes it harder to read.

```python
import pandas as pd
import matplotlib.pyplot as plt

sales = pd.DataFrame({
    "category": ["chair", "lamp", "desk", "sofa"],
    "revenue":  [4200, 1800, 5100, 3300],
})

# Bar chart: length encodes revenue — easy to compare precisely.
sales.sort_values("revenue").plot.barh(x="category", y="revenue", legend=False)
plt.xlabel("revenue (€)"); plt.title("Revenue by category")
plt.tight_layout(); plt.show()
```

**How it works.** `plot.barh` maps each category to a horizontal bar whose *length* is its revenue, the channel the eye reads best, and sorting makes the ranking instant. Compare that with the same numbers as a pie: the wedges encode the values as *angles and areas*, which people compare poorly, so the reader has to squint to tell desk from chair. Same data, weaker argument. Reach for the pie only for a few parts of an obvious whole, and even then know you've chosen a harder-to-read channel.

!!! origins "Origins — From Bertin's grammar to the tools on your laptop"
    This isn't folklore; it's a worked-out theory. In 1967 Jacques Bertin catalogued the "retinal variables" (position, size, value, colour, texture, orientation, shape) and analysed which could faithfully express which kinds of data. In 1999 Leland Wilkinson generalised this into *The Grammar of Graphics*: a chart as a composition of data, mappings, scales and geometries, not a named type off a shelf. That grammar is the literal architecture of the tools you use, from Hadley Wickham's **ggplot2** (2007) to Tableau to the web's charting libraries. When you choose which variable goes on which channel, you are doing Bertin, whether or not you know his name.

## Matching the chart to the question

A few reliable pairings: a trend over time wants a **line**; a comparison across categories wants **bars**; a relationship between two numeric variables wants a **scatter**; a part-to-whole relationship *might* want a pie if there are few enough parts.

```python
# Trend over time -> line
months = pd.DataFrame({"month": range(1, 7), "signups": [90, 120, 115, 160, 175, 210]})
months.plot.line(x="month", y="signups", marker="o", legend=False)
plt.title("Signups are trending up"); plt.show()

# Two numeric variables -> scatter
import numpy as np
rng = np.random.default_rng(1)
pts = pd.DataFrame({"visits": rng.integers(1, 40, 80)})
pts["orders"] = (pts["visits"] * 0.3 + rng.normal(0, 2, 80)).round()
pts.plot.scatter(x="visits", y="orders")
plt.title("More visits, more orders"); plt.show()
```

**A word on tools.** Everything above uses **matplotlib**, the layer underneath almost every Python chart, which is worth knowing because it's where you go when you need to control something precisely. **Seaborn** sits on top of it and is usually the faster way to a good default: it thinks in terms of a dataframe plus which variable goes on which channel, which is the grammar of this chapter expressed as an API.

```python
import seaborn as sns
sns.barplot(data=sales, y="category", x="revenue", color="#1b6ca8")   # same chart, less code
sns.scatterplot(data=pts, x="visits", y="orders", hue=None)
```

Seaborn's `hue`, `size` and `style` arguments *are* Bertin's retinal variables with names, so reaching for one is a deliberate encoding choice rather than a styling tweak. Use Seaborn to get to a defensible chart quickly, then drop to matplotlib for the final edits.

### Question → chart → channel

| The question | Chart | What encodes the number | Watch out for |
|---|---|---|---|
| How does this change over time? | line | vertical position | truncated axis; too many lines at once |
| How do these categories compare? | bar | length | unsorted bars; a baseline that isn't zero |
| How is one variable distributed? | histogram | bar height (counts) | bin width changing the story |
| How does a distribution differ by group? | box plot, or small multiples | position and spread | hiding two clusters inside one box |
| Do these two numbers move together? | scatter | position on both axes | over-plotting; implying causation |
| What are the parts of a whole? | stacked bar (or pie, if very few parts) | length (or angle) | more than ~4 parts; parts that don't sum |
| Where is this happening? | map | position, colour | mapping raw counts instead of rates |

The right-hand column is the working part of this table. Every chart type has one characteristic way of misleading, and knowing yours in advance is faster than discovering it in a review.

## Showing subgroups instead of hiding them

Chapter 7 ended on a warning: an average can conceal a reversal inside its subgroups. Visualisation is where you fix that rather than just caveat it, and the tool is **small multiples** — the same chart repeated once per group, on shared axes.

```python
import seaborn as sns
import pandas as pd

promo = pd.DataFrame({
    "rider":    ["casual"]*4 + ["member"]*4,
    "promo":    ["no", "no", "no", "yes", "no", "yes", "yes", "yes"],
    "duration": [30, 34, 32, 38, 8, 12, 14, 13],
})

g = sns.catplot(data=promo, x="promo", y="duration", col="rider",   # (1)!
                kind="bar", errorbar=None, height=3.2, aspect=0.9)
g.set_axis_labels("promotion", "avg duration (min)")
g.figure.suptitle("The promotion lengthened trips — in both groups", y=1.03)
```

1.  `col="rider"` is the whole idea: one panel per rider type, drawn on **shared axes** so the panels are comparable at a glance.

**How it works.** `catplot` with `col=` splits the data into panels rather than overlaying it. Two panels side by side make the Simpson's paradox from the last chapter *visible*: the promotion bar is taller in both, and the two panels sit at obviously different heights — which is the composition effect that the single overall average concealed. Small multiples are the honest default whenever a comparison might differ by group. They cost you nothing but space, and space is cheaper than a wrong conclusion.

The one rule that makes them work: **shared axes**. If each panel scales to its own data, a small difference and a large one look identical, and you've built a chart that actively misleads. Seaborn shares them by default; matplotlib needs `sharey=True`.

## Say the thing: titles and annotation

Two edits improve a chart more than any amount of restyling, and both involve writing words on it.

First, **make the title your claim, not your column names.** "Revenue by category" describes the axes, which the reader can already see. "Desks earn most, chairs sell most" tells them what you concluded. A descriptive title makes the reader do your interpretive work; an argumentative one puts your reasoning on record where it can be checked — which is the more honest of the two, not the less.

Second, **annotate the point you're arguing about.** A chart with one labelled feature is dramatically more useful than a clean chart with none:

```python
import matplotlib.pyplot as plt

months = pd.DataFrame({"month": range(1, 7), "signups": [90, 120, 115, 160, 175, 210]})
ax = months.plot.line(x="month", y="signups", marker="o", legend=False)

ax.annotate("campaign launched",                       # (1)!
            xy=(4, 160), xytext=(2.2, 200),
            arrowprops={"arrowstyle": "->", "lw": 1})
ax.set_title("Signups more than doubled over six months")   # (2)!
ax.set_ylabel("signups"); ax.set_ylim(0, 230)          # (3)!
plt.tight_layout(); plt.show()
```

1.  `xy` is the data point you're pointing at; `xytext` is where the label sits. The arrow does the connecting.
2.  Check the claim against the numbers before you write it: 90 → 210 is a rise of 133%, so "more than doubled" is exact. "Doubled" would have been sloppy and "tripled" would have been false.
3.  Still a true baseline. An argumentative title raises the bar for honest encoding, it doesn't lower it.

**How it works.** `annotate` draws text at `xytext` with an arrow to the data point at `xy`, both in data coordinates. The reason this matters so much: without the label, a reader sees a rising line and invents an explanation. With it, they see your explanation and can disagree with it. Annotation converts a chart from a picture into a claim — and a claim is something a reader can hold you to, which is exactly what you want.

Notice what the title deliberately does *not* say. The arrow marks the campaign, and the obvious next sentence — "the campaign doubled signups" — isn't supported: the line was already climbing in February and March, before the campaign existed. So the title reports the change and the annotation supplies the context, leaving the causal question open, which is where the data leaves it too. That gap between "these happened together" and "this caused that" is [Chapter 7](descriptive-statistics.md)'s warning arriving in visual form, and an annotation is the most persuasive way there is to blur it. Point at things; don't imply arrows of causation you can't defend.

!!! critical "Critical Lens — An argumentative title is a responsibility, not a licence"
    "Make the title your claim" can be misread as permission to editorialise. It isn't. The reason to state your conclusion on the chart is that it makes the conclusion *auditable*: a reader can check the claim against the marks and tell you you're wrong. A title that overstates what the data supports — "Signups doubled" when they rose 40%, or a causal claim from a correlation — is worse than a bland one, because the design has lent it authority. The test: could you defend this sentence, in front of the person who collected the data, using only what's visible in the chart? If not, weaken the sentence rather than strengthening the chart.

## Choosing fairly

Most visualisation ethics reduce to a few habits. **Don't lie with the axis:** truncating a bar chart's baseline to exaggerate a difference is the oldest trick in the book. **Don't add ink that carries no data:** Tufte's crusade against "chartjunk" (3-D effects, decorative gradients, gridlines shouting over the data) is a plea to respect the reader's attention. And **design for the decision, not for decoration**: ask what the viewer will *do* after seeing this, and make the chart answer exactly that.

```python
values = pd.Series([100, 103, 106], index=["Q1", "Q2", "Q3"])
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(9, 3.2))

values.plot.bar(ax=ax1, color="#b3261e"); ax1.set_ylim(98, 108)   # truncated!
ax1.set_title("Explosive growth?  (y starts at 98)")

values.plot.bar(ax=ax2, color="#0f766e"); ax2.set_ylim(0, 120)    # truthful
ax2.set_title("The same data, true baseline")
plt.tight_layout(); plt.show()
```

Run this and the point is undeniable: the left chart's truncated y-axis turns a modest 6% rise into a cliff face, while the right chart, same numbers, baseline at zero, shows what actually happened. The only difference is one `set_ylim`, and it's the difference between informing your reader and manipulating them.

## Colour, and designing for everyone

Colour is the channel designers reach for first and misuse most. Two principles keep it truthful. First, **colour should carry meaning, not decoration**: if a chart's colours don't *encode* something (a category, a value), they're noise competing with the data, and you should drop them. Second, **match the palette to the data type**: a *sequential* palette (light-to-dark of one hue) for ordered or numeric data; a *categorical* palette (distinct hues) for unordered groups; and a *diverging* palette (two hues from a neutral middle) for data with a meaningful centre like profit and loss. Using a rainbow for ordered data, or a light-to-dark ramp for unordered categories, quietly misleads.

Then there's the principle designers are best placed to champion and most often forget: **the chart has to work for everyone who reads it.** Around 8% of men have some red–green colour blindness, so a chart that distinguishes "good" from "bad" by red versus green alone is illegible to millions. The fix is not to avoid colour but to never let colour be the *only* cue, add direct labels, shapes, or position so the chart survives in greyscale too.

!!! case "Case study — a colour-safe comparison"
    Compare average ride length by membership, using a colourblind-safe pairing and labelling the bars directly so colour isn't load-bearing:

    ```python
    import matplotlib.pyplot as plt
    palette = {"member": "#1b6ca8", "casual": "#e08a1e"}   # blue/orange survives red-green CVD
    by_group = trips.groupby("membership")["duration_min"].mean()

    ax = by_group.plot.bar(color=[palette[m] for m in by_group.index])
    ax.set_ylabel("avg duration (min)")
    for i, v in enumerate(by_group):                        # direct labels = a second cue
        ax.text(i, v, f"{v:.0f}", ha="center", va="bottom")
    plt.title("Casual riders travel longer"); plt.tight_layout(); plt.show()
    ```

    Blue and orange stay distinct for colourblind readers where red and green would merge, and because each bar is labelled, the chart still reads with no colour at all, the accessibility test in miniature.

!!! practice "In Practice — The greyscale test"
    Before you ship a chart, screenshot it and desaturate it (or just imagine it printed in black and white). If you can still read every distinction it's making, it's robust; if two categories become the same grey, colour was doing work nothing else backed up. This is the spirit of the web's **WCAG** accessibility standards applied to data graphics: never encode essential meaning in colour alone, and keep enough contrast that the marks stand off the background.

!!! critical "Critical Lens — When 'beautiful' and 'true' pull apart"
    A designer's instinct for beauty is mostly an asset here and occasionally a trap. The trap is when polish lends false authority to a shaky claim, or a striking form flatters the data at the cost of accuracy. Two correctives. Tufte gave us *graphic integrity*: maximise the ink that carries information, delete the rest. And W. E. B. Du Bois, whose hand-drawn "data portraits" for the 1900 Paris Exposition argued against racism with rigorous, strikingly modern charts, shows the opposite balance done right: work unmistakably *designed* and scrupulously accurate, in the service of a moral argument. The goal is not to choose between beautiful and true, but to refuse any chart that buys one by spending the other.

!!! further "Going Further — Storytelling, motion, and the web as canvas"
    Two threads extend this chapter. One is *data storytelling*: Hans Rosling's 2006 Gapminder talk animated two centuries of global development into bouncing bubbles and reached millions, reviving Otto Neurath's century-old Isotype dream of statistics anyone could read. The other is the *interactive* turn: Mike Bostock's **D3.js** (2011) bound data directly to web-page elements and powered a golden age of data journalism at the *New York Times* and the *Guardian*, the newsroom becoming a leading employer of exactly the designer-analyst hybrid this course trains. If a static chart argues, an interactive one invites the reader to argue back. That's the bridge to the next chapter.

A chart is where your careful, truthful analysis meets another human being. Make the argument clearly, make it true, and take responsibility for how easy you've just made it to believe you. Next we let the reader touch the data themselves.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. Seaborn is the fastest route from a dataframe to a chart worth arguing with.

    - [Introduction to Data Visualization with Seaborn](https://app.datacamp.com/learn/courses/introduction-to-data-visualization-with-seaborn) — **Ch. 1 Introduction to Seaborn**, **Ch. 2 Visualizing Two Quantitative Variables**, **Ch. 3 Visualizing a Categorical and a Quantitative Variable.**
    - [Introduction to Data Visualization with Seaborn](https://app.datacamp.com/learn/courses/introduction-to-data-visualization-with-seaborn) — **Ch. 4 Customizing Seaborn Plots** *(optional — the typography and colour pass)*.
