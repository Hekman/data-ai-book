<p class="chapter-kicker">Part I · Chapter 7</p>

# Visualisation as argument

A chart is not a decoration you add once the analysis is done. A chart is an argument, a claim about what matters in the data, made in a form the eye grasps faster than any table. This is the chapter where a designer's existing skills become a superpower, because visualisation sits exactly at the join between data work and design, and most people are good at one and weak at the other. You can be good at both. But that power comes with an ethics, because a chart persuasive enough to change a decision is also persuasive enough to mislead, and often the same chart does both.

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

## Exercises

??? exercise "Exercise 7.1 · One dataset, three framings"
    Take one finding from your own data and draw it three ways: a truthful version, an *accidentally* misleading version (bad channel, or a truncated axis), and your best final version. Write a sentence on what changed the argument between them.

    ??? note "What good looks like"
        The learning is in articulating *why* the misleading one misleads ("the truncated axis exaggerates a 2% difference into an apparent doubling") and defending your final choice in terms of the *decision* the reader has to make, not in terms of which looks nicest.

??? exercise "Exercise 7.2 · Redraw a chart in the wild"
    Find a chart in a news article, report or app that you think misleads: a truncated axis, a 3-D pie, a rainbow of meaningless colour. Rebuild it faithfully in matplotlib and write two sentences on what the original's design choices were doing to the reader.

    ??? note "Guidance"
        Be specific and fair: name the exact channel or scale at fault, and grant that the original may not be malicious: many misleading charts come from default settings, not deceit. That fairness is part of reading like a professional.
