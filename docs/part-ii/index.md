<p class="chapter-kicker">Part II</p>

# Reading the Systems That Learn

**Goal:** understand machine learning well enough to read it critically — what a model is, how it learns, how it fails, and how to tell a good one from a dangerous one — without needing to build state-of-the-art systems.

> **The question running through this part:** *What is this system learning, from whose behaviour, and how would we know it is any good?*

Part I taught you to read data. Part II teaches you to read the systems that learn from it: the feeds, rankings, filters and suggestions you touch every day. You are not here to become an ML engineer. You are here to understand the loop between an interface and a model well enough to stand in front of any ML-powered product and say what it is learning, from whose behaviour, and where the design choices live.

## The path through Part II

Six techniques, chosen deliberately: each is the engine under a service you used this morning, together they cover the kinds of thing a model can do, and every one of them is a model you can **read** — coefficients you can interpret, splits you can follow, neighbours you can name. We start with the **loop** and the design side of it, because that is the side you are responsible for, then predict a **number**, then a **category** two ways — by rules and by similarity — then find groups with **no labels at all**, turn similarity into a **recommender**, and finally read **text**, where the cost of being wrong stops being a metric and becomes an experience. The part closes on the **interface around your own model**.

One question closes every chapter: *how would we know this is any good, and what does being wrong cost?*

## Chapters, week by week

The chapters follow the teaching weeks of Block B exactly. If you're taking the course, each week's reading is listed below; if you're reading independently, the weeks are simply a suggested pace.

### Week 1 · The loop, and predicting a number

<div class="grid cards" markdown>

-   __11 · The loop: interfaces that teach models__

    Algorithmic affordances, the interface → behaviour → training data → model loop, and what "learning" means.

    [:octicons-arrow-right-24: Read](the-loop.md)

-   __12 · Learning from data: correlation and regression__

    Correlation, causation, and the first model whose reasoning you can actually read.

    [:octicons-arrow-right-24: Read](regression.md)

</div>

### Week 2 · Classification by rules

<div class="grid cards" markdown>

-   __13 · Decision trees__

    A flowchart the model wrote itself: splits you can follow, proxies you didn't intend, and the two ways to be wrong.

    [:octicons-arrow-right-24: Read](decision-trees.md)

</div>

### Week 3 · Classification by similarity

<div class="grid cards" markdown>

-   __14 · k-nearest neighbours__

    Judging by company. Scaling, distance, train/test — and who a similarity-based system serves badly.

    [:octicons-arrow-right-24: Read](k-nearest-neighbours.md)

</div>

### Week 4 · Groups without labels

<div class="grid cards" markdown>

-   __15 · k-means clustering__

    Segmentation, choosing k with reasoning, naming a cluster — and the ethics of sorting people who never asked.

    [:octicons-arrow-right-24: Read](clustering.md)

</div>

### Week 5 · Similarity as a tool

<div class="grid cards" markdown>

-   __16 · Cosine similarity and a small recommender__

    "More like this": vectors in, neighbours out, and a feedback loop with a friendly face.

    [:octicons-arrow-right-24: Read](similarity-and-recommenders.md)

</div>

### Week 6 · Text, and judging a model properly

<div class="grid cards" markdown>

-   __17 · Text as data, and how models are judged__

    Bag-of-words, TF-IDF, a classifier on text — and a confusion matrix read as consequences, not counts.

    [:octicons-arrow-right-24: Read](text-as-data.md)

</div>

### Week 7 · After the break — the intervention

<div class="grid cards" markdown>

-   __18 · The interface around the model__

    Uncertainty, error, appeal and drift: putting a probabilistic thing in front of a person, honestly.

    [:octicons-arrow-right-24: Read](the-interface-around-the-model.md)

</div>
