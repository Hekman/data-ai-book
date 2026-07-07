---
hide:
  - navigation
  - toc
---

<div class="hero" markdown>
<p class="kicker">An open textbook for designers and journalists</p>
<h1 class="hero-title">Data & AI for Designers and Journalists</h1>
<p class="hero-sub">Learn to read data and models like a material, with a grain, a history and a politics; to weigh them like evidence; and then to build sound things and tell true stories with them. A hands-on companion to the HU <a href="https://www.hu.nl/voltijd-opleidingen/master-data-driven-design">Master Data-driven Design</a> and the <a href="https://husite.nl/minors/minors/big-data-and-design/">Big Data and Design</a> minor.</p>
<p class="byline">Written by Erik Hekman · HU University of Applied Sciences Utrecht</p>
</div>

A designer who has never written a line of code, and a reporter who has never opened a terminal, still work with data every day: a heat-map of taps, an A/B test that says version B "won," a leaked spreadsheet, a public budget in a badly-formatted CSV. You already make decisions, and claims, from these things. The only real question is whether you can *interrogate* the material or can only receive its conclusions.

This book teaches the first. It treats data, machine learning and AI two ways at once: the way a ceramicist treats clay (a **material** you learn well enough to work with its grain) and the way a reporter treats a source (**evidence** you test before you trust). Either way the discipline is the same: learn the stuff well enough to say plainly what it can and cannot support. No prior programming is assumed. Every chapter pairs plain-language ideas with real Python you can run, and ends with exercises.

<div class="grid cards" markdown>

-   :material-flag-outline:{ .lg .middle } __Start here__

    ---

    The maker's stance toward data: what this book means by treating data as a *material* and as *evidence*, and how to read the book.

    [:octicons-arrow-right-24: Chapter 0](chapter-0.md)

-   :material-table-large:{ .lg .middle } __Part I · Data as Material__

    ---

    Read, clean, explore, visualise and collect real, messy data, and stay clear-eyed about what it represents.

    [:octicons-arrow-right-24: Part I overview](part-i/index.md)

-   :material-brain:{ .lg .middle } __Part II · Reading the Systems That Learn__

    ---

    Understand machine learning well enough to read it critically. *(In progress.)*

-   :material-robot-outline:{ .lg .middle } __Part III · Building With AI__

    ---

    Build and ship interactive, AI-driven tools, from products to newsroom apps. *(In progress.)*

</div>

## How this book is built

The book runs on **two layers** so it works both for a student following the courses and for a designer picking it up cold. The **main text** is the core path. **Going Further** boxes push past the courses, and are skippable on a first read.

Woven through every chapter are four kinds of **sidebar**, so history and ethics live *beside* the practice rather than in a separate silo:

!!! origins "Origins — where an idea came from"
    Drawn from a timeline of some 260 events across nine threads. Knowing where an idea came from is often the fastest way to understand what it is *for*.

!!! practice "In Practice — a small, concrete example"
    A designerly example, a gotcha, or a habit worth keeping.

!!! critical "Critical Lens — who a method serves, and who it can harm"
    Responsibility is not a final chapter you skip to. It is a property of every decision.

!!! further "Going Further — beyond the courses"
    Harder techniques, deeper theory, further reading, for when you want more.

And each chapter carries **runnable code** with a *how it works* explanation, plus **exercises** with collapsible solutions:

??? exercise "Try it — how exercises look"
    You'll find tasks like this at the end of every chapter. Open the solution only after you've had a real go.

    ??? note "Show a solution"
        Solutions appear here, one reasonable approach among several. The point is rarely a single right answer; it's whether you can explain your choices.

---

*A note on the code.* Examples use **Python** with **pandas** and friends, the toolkit of the courses and of both professions. They're written to run in a hosted notebook with no setup, and most build their own small datasets so they work offline. You do not need to become an engineer to use them; you need to become a careful note-taker who happens to run code between the notes.
