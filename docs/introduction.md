<p class="chapter-kicker">Introduction</p>

# The maker's stance toward data

A designer who has never touched code still works with data every day. So does a reporter. You read a heat-map of where people tap, an A/B result that says version B "won," a spreadsheet of survey answers, a chart in a stakeholder's deck, a dataset attached to a press release, a leaked export nobody has cleaned. You make decisions from these things, and if you're a journalist, public claims. The only real question is whether you do it as a reader or as a bystander: whether you can interrogate the material or can only receive its conclusions.

This book is about becoming a reader, and then a maker. It asks you to hold two stances toward data at once, because designers and journalists come at it from two directions that meet in the same place. The first is the **designer's**: treat data, machine learning and AI the way a furniture designer treats wood, or a ceramicist treats clay, as a *material*. Materials have a grain. They resist you in some directions and give way in others. They carry a history: where they were sourced, who cut them, what was thrown away in the cutting. A good maker learns the material well enough to work *with* its grain instead of fighting it, and well enough to say plainly what it can and cannot do.

The second is the **journalist's**: treat data as a *source*, evidence to be tested before it is trusted and never quoted straight. A reporter asks of any source who it is, why it's talking, what it can actually witness, and what it has reason to leave out. A dataset deserves exactly the same interview. The two stances turn out to be one discipline seen from two trades: the ceramicist's respect for the grain and the reporter's refusal to print an unchecked claim are the same instinct, *know the material well enough to say truly what it shows*. Where the trades differ is only in what you make at the end. A designer ships an artefact, a journalist publishes a story, and both put their name to it.

That framing matters because the dominant way data gets talked about is the opposite of a material. Data is described as if it were a fact of nature: objective, given, simply "out there" to be collected. The word itself encourages this: *data*, from the Latin, means "the givens." But nothing about a dataset is given. Someone decided what to count, what to call it, which column to make and which to leave out, when to stop collecting, and what to do with the rows that didn't fit. Every one of those is a design decision, usually made by someone other than you, often invisibly. Learning to see those decisions is the whole game.

!!! critical "Critical Lens — 'Datafication,' and the worldview inside it"
    In 2013, Viktor Mayer-Schönberger and Kenneth Cukier popularised the term *datafication*: the rendering of everything (friendship into a social graph, a walk to work into a location trail, a mood into a sentiment score) as quantifiable data. Their book captured a genuine optimism, the belief that enough messy correlation at scale could rival careful causation. Much of the critical literature since is, in effect, a reply to that claim. Hold both in your head at once: datafication is powerful *and* it is a choice about how to see the world, not a discovery about how the world is.

## Two capabilities, growing across three parts

The course this book supports is built around two abilities, and so is the book.

The first is **analytical reasoning with data**: being able to take a real, messy dataset and get somewhere trustworthy with it. Load it, question it, clean it, visualise it, and know the difference between what it shows and what you wish it showed. That capability is the spine of Part I, and it deepens in Part II when the "data" becomes a learning system you have to evaluate.

The second is **building with machine intelligence**: designing and shipping things that use models (classic machine learning, and now large language models and agents), where the hard part is rarely the model and almost always the judgement around it. That capability is the spine of Part III, but it rests entirely on the first: you cannot responsibly build with a model you cannot read.

The three parts map onto the three blocks of the Master track, but they are organised by idea rather than by week, so you can also read the book straight through.

- **Part I · Data as material.** Work with data you're given and data you collect yourself: dataframes, cleaning, exploratory analysis, visualisation, dashboards, scraping and APIs. The recurring discipline is staying truthful about what the data can support.
- **Part II · Reading the systems that learn.** Enough machine learning to read it critically: what a model is, how it learns, how it's evaluated, how it fails, how bias enters.
- **Part III · Building with AI.** Design and ship interactive, AI-driven applications: choosing the paradigm, working with language models and agents, and keeping a human meaningfully in charge.

## Reading vs. building

It is worth naming the difference between the two modes early, because both trades are rushed toward output: designers toward shipping, journalists toward publishing. *Reading* a system means being able to say what it is doing, on what data, optimising for what, and at whose expense. *Building* a system means making those choices yourself. Reading is the older and more transferable skill. Products change and stories move on; the questions you ask of them do not. This is why the book front-loads reading and treats building as something you earn.

!!! origins "Origins — Writing was invented for accounting"
    The earliest cuneiform tablets from Uruk, around 3200 BCE, are not myths or poems. They are receipts: rations, livestock, taxes. Humanity's most consequential information technology, writing, was invented to keep structured records, and only later learned to tell stories. Data management is not a by-product of literacy; it is its origin. When you feel that working with data is somehow less creative than "real" design, remember that the spreadsheet is older than the sonnet.

## Literacy is the point, especially now

You might reasonably ask, given that this book is being written at a moment when an AI can turn a sentence into working pandas code, why learn any of this yourself. Why not just ask the model to clean the data, make the chart, train the classifier, and hand you the answer?

Because AI does not remove the need for literacy; it moves it. It shifts your work from *writing* to *judging*, and judging is the harder skill. A model will happily produce a chart, an analysis, or a confident paragraph explaining what the data "shows." What it cannot do is know whether that output is *right* for your data, your question, and your context: whether the average it computed is meaningless because the distribution is skewed, whether the column it treated as a number was secretly a category, whether the correlation it found is causation, coincidence, or an artefact of how the data was collected. Those judgements are exactly what the rest of this book teaches, and they are precisely what a fluent-sounding wrong answer is best at hiding. You cannot catch an error you don't have the literacy to see.

So blindly delegating to AI is not a shortcut around this material; it is a way of inheriting mistakes you can't detect and can't defend. The tool can generate the work in seconds. It cannot take responsibility for it. That still falls to the designer whose name is on the deliverable, or the reporter whose byline is on the story, when someone asks "why does it say this?" A literate practitioner uses AI the way a skilled woodworker uses a power tool: it makes them faster at things they already understand, and it is dangerous in exactly the places they don't. Literacy is what turns the model from an oracle you must trust into an assistant you can supervise, and, for a journalist, into a source you would still fact-check before quoting.

None of this is an argument against using AI. Part III of this book is entirely about *building* with it, and doing so well. The argument is that you should build with it as someone who can read what it produces, not as someone outsourcing their judgement to it. Work with the grain of the tool, know where it lies, and keep your hand on the material. That is the whole reason to learn to read data and code yourself, even in an age when a machine will write both for you.

!!! critical "Critical Lens — Automation bias and the confident wrong answer"
    Decades of research on *automation bias* show a stubborn human tendency: once a machine produces an answer, people trust it more than they should and check it less than they would a colleague, and the more polished and confident the output, the stronger the effect. Modern AI is the most fluent, most confident answer-generator ever built, which makes it uniquely good at producing plausible mistakes that sail past an untrained eye. The defence is not suspicion of every output; it is the literacy to interrogate it: to ask the same five questions of an AI's analysis that [Chapter 2](part-i/chapter-2.md) asks of any dataset, the way a newsroom asks them of any tip. A maker who can do that is safe to work fast with AI. One who can't is being quietly steered by a tool that never says "I'm not sure."

## A first look

You don't need to write code to read this book, but you'll understand it better if you run the examples. Here is the smallest possible taste, not to teach syntax yet but to make one point concrete: even three rows of "data" are full of decisions.

```python
import pandas as pd  # (1)!

# A tiny, hand-made dataset. Notice: we chose the columns,
# the categories, and what counts as one row.
people = pd.DataFrame({
    "name":    ["Ada", "Alan", "Grace"],
    "country": ["UK", "UK", "USA"],
    "role":    ["mathematician", "logician", "programmer"],
})

print(people)          # look at the whole table
print(people.shape)    # (rows, columns) -> (3, 3)
print(people["country"].value_counts())  # how many per country
```

1.  `pandas` is the library that gives Python its "dataframe," a table you can compute with. We'll always import it as `pd`. More on this in [Chapter 3](part-i/chapter-3.md).

**How it works.** `pd.DataFrame({...})` builds a table from a dictionary, where each key becomes a column. `.shape` reports `(rows, columns)`. `.value_counts()` tallies how often each value appears in a column. Trivial as it is, this snippet already embodies Chapter 0's argument: *we* decided that a person's country is one of two clean labels ("UK," "USA"), that "role" gets a single word, that Ada, Alan and Grace each get exactly one row. A different maker (another designer, another reporter) would have built a different table of the same three people, and every later analysis, and every headline drawn from it, would inherit those choices.

!!! further "Going Further — 'Data science' is a young name for an old practice"
    The activities in this book are ancient, but the profession is not. John Tukey argued for a discipline broader than statistics as early as 1962; William Cleveland proposed the name *data science* in 2001; by 2012 the *Harvard Business Review* was calling "data scientist" the sexiest job of the century. The hype has cooled and the tools have consolidated into roughly the craft this book teaches. For the intellectual history in one sitting, read Tukey's *The Future of Data Analysis* and notice how much of today's field he sketched sixty years ago.

## How to read this book

The main text is the core path, and it assumes no prior programming. Alongside it run the four sidebars you met on the [home page](index.md): Origins, In Practice, Critical Lens, and Going Further, plus runnable code and exercises in every chapter. Students following the syllabus can pass the Going Further boxes by without losing the thread; everyone else can dig in.

A maker's stance toward data, then, is neither the technologist's reflex ("collect it all, we'll find a use") nor the sceptic's refusal ("it's all biased, ignore it"). It is the stance the material worker and the reporter share: respect the grain, know the provenance, check the claim before you repeat it, and take responsibility for the thing you make, whether that thing is a product or a published story. The rest of the book is practice in exactly that.

## Exercises

??? exercise "Exercise 0.1 · Interrogate a dataset you already use"
    Pick a dataset or dashboard you encounter in your own work or studies (a fitness app's stats, a course analytics page, a public dataset). Without any code, write three or four sentences answering: **Who made this, and why? What is one row? What's obviously missing?** Keep your notes; you'll do this formally, in code, in Chapter 2.

    ??? note "What a good answer looks like"
        A good answer names a *purpose* ("my bank's app tracks transactions to sell me products, not to help me budget"), identifies the *unit* ("one row = one transaction"), and spots at least one *absence* ("cash spending never appears, so the totals understate what I actually spend"). Noticing the absence is the sign you're reading like a maker, not a bystander: the same absence a reporter would chase before running the number.

??? exercise "Exercise 0.2 · Rebuild the table differently"
    Take the `people` dataframe above and, on paper or in code, redesign it to make a *different* set of choices: for example, one row per person-per-role, a `born` column, or countries as full names. Write one sentence on what your new design makes easy to ask, and one on what it now hides.

    ??? note "Show a solution"
        ```python
        people2 = pd.DataFrame({
            "name":    ["Ada", "Alan", "Grace"],
            "born":    [1815, 1912, 1906],
            "country": ["United Kingdom", "United Kingdom", "United States"],
        })
        ```
        Adding `born` makes questions about *age or era* possible that the original table couldn't answer; using full country names removes the ambiguity of "UK." But if you dropped the `role` column to make room, you've now hidden what each person *did*: the table got better at chronology and worse at biography. There's no neutral version, only a version fit for some questions and not others.
