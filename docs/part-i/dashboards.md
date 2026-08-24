<p class="chapter-kicker">Part I · Chapter 9 · <span class="week">Week 6</span></p>

# Dashboards and interactive views

!!! guiding "The question this chapter answers"
    How do I turn a set of charts into an interactive dashboard an audience can actually use?

A chart makes one argument. Sometimes what a stakeholder actually needs is not your single argument but a small instrument they can operate themselves (filter to their region, change the date range, pick a category) and see the data respond. That instrument is a **dashboard**, and building one is the step where your analysis stops being a document you hand over and becomes a *tool someone else uses*. That shift changes your job: a document you control; a tool you must design for someone who will use it without you in the room.

!!! matters "Why this matters for designers"
    A dashboard hands the reader the controls, which means it can mislead in states you never looked at. Designing the default view, the sensible ranges and the guardrails is the real work — interactivity is easy to add and just as easy to make dishonest. The most useful dashboards say one clear thing on open, and reward exploration without ever letting the user reach a misleading view.

!!! newsroom "Why this matters for journalists"
    A news app hands the reader the steering wheel, and every state it can reach is a claim you have published — including the ones you never looked at. Filter to a small town and a rate built on four cases becomes a headline in someone's screenshot. The defaults, the sensible ranges and the visible caveats are the editing; the interactivity is the easy part.

!!! outcomes "By the end of this chapter you can"
    - State who a dashboard is for, what decision it supports, and what must be true for it to be trusted.
    - Add interactivity to a chart with Plotly: hover, zoom, range sliders, dropdowns.
    - Build a small dashboard in Gradio or Streamlit, and explain why you chose that framework.
    - Lay out one headline, grouped controls, supporting detail and a provenance footer, in that order.
    - Design a deliberate, defensible default view — the argument you make before anyone touches a control.
    - Keep encodings honest across every interactive state a user can reach, including the empty one and the one-row one.
    - Prepare data once rather than on every interaction, and say why responsiveness is a design concern.
    - Judge when interactivity helps the reader reason and when it merely decorates, and defend what you left out.

The lineage runs straight back through [Chapter 8](visualisation.md). Playfair's claim, let the eye do the work, industrialised over two centuries into self-service analytics.

!!! origins "Origins — Self-service visualisation"
    Tableau, spun out of a Stanford research project in 2003, put grammar-of-graphics-style visual analysis into the hands of people who couldn't program, and normalised the expectation that anyone in an organisation could interrogate data visually rather than wait for a specialist. A dashboard is that expectation made concrete: Playfair's argument, handed to the reader with the controls attached.

## What a dashboard is for, and against

The danger of dashboards is that they're easy to make badly. A wall of twenty charts is not a dashboard; it's an abdication, pushing onto the viewer the very job of finding what matters that you were supposed to do. A good dashboard is *designed*, and the design questions are ones you already know from interface work. What is the one thing this should show at a glance, before anyone touches a control? Which few dimensions are worth making interactive, and which would only add noise? What does the reader see *first*, because the default view is the argument you're making before they change anything, and most people never change anything. Restraint is the whole craft. The best dashboards show less than they could and make the important thing impossible to miss.

### Start with who and what for

Before any code, answer three questions in one sentence each, because they determine everything downstream:

**Who opens this, and how often?** A colleague checking daily needs a different thing from a client seeing it once in a meeting. The daily user wants change since last time and will learn the controls; the one-time viewer needs the finding stated on arrival and will touch nothing.

**What decision does it support?** "To understand the data" is not a decision. "To decide which three stations get more docks next quarter" is, and it tells you what to put at the top, what to make filterable, and what to leave out. A dashboard that supports no decision is a report with extra steps.

**What must be true for it to be trusted?** Usually: the numbers are current, the source is visible, and the caveats you learned in Chapters 3 and 5 survive into the interface. A dashboard strips away your careful prose, which is precisely why the honesty has to be built into the layout instead.

If you can't answer these, build a chart instead. Interactivity is a cost you pay in complexity and in new ways to mislead, and it should buy you something specific.

### Layout is hierarchy

A dashboard is read, not scanned uniformly, and the reading order is a design decision you make whether or not you notice. Three principles carry most of the weight:

**One headline, top-left.** In left-to-right reading cultures the eye lands top-left first, so that's where the single most important number or chart goes. If everything is the same size, nothing is important.

**Controls next to what they change, or all together at the edge.** Scattered controls make the reader hunt for cause and effect. A sidebar of filters is a convention worth following precisely because it's a convention.

**Then supporting detail, then provenance.** Below the headline: the two or three charts that qualify it. At the bottom, the thing almost every student forgets — a line saying where the data came from, when it was last updated, and what it excludes. That footer is the interface version of your provenance note, and it is what separates a dashboard someone can rely on from one they merely enjoy.

A useful discipline: sketch the layout on paper before you write code. Boxes and labels, five minutes. It's much easier to notice that you have four headline charts and no hierarchy in a sketch than after you've built them.

## First, make the charts themselves interactive

Before you wrap anything in a dashboard, note that a chart can gain interactivity on its own. **Plotly** produces charts that respond to the cursor — hover to read exact values, drag to zoom, a range slider on a time axis, buttons and dropdowns to switch series — and it returns a figure you can drop straight into either framework below.

```python
import plotly.express as px

fig = px.scatter(pts, x="visits", y="orders",
                 hover_data=["visits", "orders"],       # hover reveals the exact numbers
                 title="More visits, more orders")
fig.update_layout(yaxis_range=[0, pts["orders"].max() * 1.1])   # honest baseline, still
fig.show()
```

Note the last line. Every honesty rule from [Chapter 8](visualisation.md) still applies, and interactivity adds a new obligation: the axis has to stay truthful in every state the user can zoom or filter into, not just the one you screenshotted.

## Building one without becoming an engineer

You can build a working, interactive dashboard directly beside your analysis, without standing up a website. Two frameworks are worth knowing, and they do the same job here:

- **Gradio** wraps a Python function in inputs and outputs and runs *inside the notebook*, then shares over a link. Nothing to install beyond the package, nothing to run separately — which is why the examples below use it.
- **Streamlit** gives you a standalone app with more layout control, written as a top-to-bottom script and launched with `streamlit run app.py`. More power, but it needs its own process, so it doesn't live inside a hosted notebook.

Pick the one that fits how you want to share your data story. The design questions are identical either way; only the plumbing differs. Start with the smallest possible version — a function, one input, one output:

```python
import gradio as gr

def greet(name):
    return f"Hello, {name} — welcome to your first dashboard."

demo = gr.Interface(fn=greet, inputs="text", outputs="text")
demo.launch()      # renders an interactive panel right in the notebook
```

**How it works.** `gr.Interface` takes three things: a function `fn`, a description of its `inputs`, and of its `outputs`. Gradio builds the UI, calls your function whenever the input changes, and shows what it returns. That's the whole model: *your analysis is the function; Gradio is the interface around it.* Now make the function do something real: filter a dataframe and draw a chart.

```python
import pandas as pd
import matplotlib.pyplot as plt
import gradio as gr

orders = pd.DataFrame({
    "city":     ["Utrecht","Amsterdam","Utrecht","Rotterdam","Amsterdam","Rotterdam"],
    "category": ["chair","lamp","desk","chair","desk","lamp"],
    "revenue":  [149, 39, 259, 199, 240, 45],
})

def revenue_chart(city):                      # (1)!
    subset = orders if city == "All" else orders[orders["city"] == city]
    fig, ax = plt.subplots()
    subset.groupby("category")["revenue"].sum().plot.bar(ax=ax)
    ax.set_title(f"Revenue by category — {city}")
    ax.set_ylabel("revenue (€)")
    return fig

cities = ["All"] + sorted(orders["city"].unique())
demo = gr.Interface(
    fn=revenue_chart,
    inputs=gr.Dropdown(cities, value="All", label="City"),   # the one control
    outputs=gr.Plot(),
    title="Revenue explorer",
)
demo.launch()
```

1.  The whole dashboard is this one function. It takes the control's value, filters the data, and returns a chart. Everything you learned in Chapters 2 and 8 is doing the work; Gradio just adds the knob.

!!! practice "In Practice — The same dashboard in Streamlit"
    If you'd rather ship a standalone app, the same logic becomes a script. Save it as `app.py` and run `streamlit run app.py`:

    ```python
    import streamlit as st

    st.title("Revenue explorer")
    city = st.selectbox("City", ["All"] + sorted(orders["city"].unique()))   # the control
    st.pyplot(revenue_chart(city))                                          # the same function
    ```

    Note that `revenue_chart` is unchanged. That's the point: your analysis is the function, and the framework is only the interface around it. Choosing between them is a distribution decision (inline and shareable by link, or standalone with real layout control), not an analytical one.

## Every state must be honest, including the empty one

A static chart has one state, which you checked. A dashboard has one state per combination of control values, and you have looked at almost none of them. That's the whole added risk, and two states cause most of the trouble.

**The empty state.** Some filter combination will match no rows, and by default your chart will render blank or raise an error — inviting the reader to invent an explanation ("sales collapsed?") for what is really an absence of data. Say so explicitly:

```python
import matplotlib.pyplot as plt

def revenue_chart(city):
    subset = orders if city == "All" else orders[orders["city"] == city]

    if subset.empty:                                   # (1)!
        fig, ax = plt.subplots()
        ax.text(0.5, 0.5, "No orders match this filter",
                ha="center", va="center")
        ax.axis("off")
        return fig

    fig, ax = plt.subplots()
    subset.groupby("category")["revenue"].sum().plot.bar(ax=ax)
    ax.set_title(f"Revenue by category — {city}")
    ax.set_ylabel("revenue (€)")
    ax.set_ylim(0, orders.groupby("category")["revenue"].sum().max() * 1.1)   # (2)!
    return fig
```

1.  Check for emptiness *before* you try to draw. Three lines, and it removes a whole class of misreading.
2.  A fixed y-limit taken from the **full** dataset, not the filtered subset. This is the subtle one — see below.

**The incomparable state.** That second annotation is the failure almost nobody catches. If each filtered view scales its own axis, then filtering to a small city produces bars just as tall as the whole country, and a reader flipping between cities sees no difference where there is a tenfold one. It's the same error as unshared axes in small multiples, except here the reader generates the panels themselves, one click at a time, and never sees two at once to compare. Pin your scales to the full data whenever the point is comparison.

A third habit worth adopting: **show the n**. Put the row count of the current selection somewhere visible, so a reader who has filtered down to eleven rows can see that they have, and discount accordingly.

!!! practice "In Practice — Your dataset, three controls"
    Build the smallest dashboard that answers a real question about your Part I dataset, in either framework, with **no more than three controls**. Pick the single chart that carries the main finding, and add only the filters a genuine user would reach for. Then interrogate your defaults: when the dashboard first loads, before anyone clicks, does it already tell the truth? (Above, the default `"All"` view shows the true overall picture.) If the opening view is misleading or blank, you've mistaken a pile of controls for a tool. The three-control limit does the same work restraint always does in design: it forces you to decide what actually matters.

!!! practice "In Practice — Load once, not once per click"
    Your chart function runs again on **every** interaction. Put the data loading inside it and you re-read the file, re-clean it and re-join it each time someone moves a slider — fine for six rows, unusable for sixty thousand. Load and prepare once, outside the function; let the function do only the filtering and drawing.

    ```python
    # Once, at import time — the expensive part
    orders = pd.read_csv("data/orders.csv")      # load
    orders = clean(orders)                        # clean

    def revenue_chart(city):                      # cheap: filter + draw only
        subset = orders if city == "All" else orders[orders["city"] == city]
        ...
    ```

    When the preparation genuinely is slow, both frameworks will memoise it for you: Streamlit with `@st.cache_data` above the loading function, Gradio by simply computing it at module level as above. The design lesson underneath the performance one: a dashboard that takes three seconds to respond stops being explorable, and an interface nobody explores may as well have been a static chart.

!!! further "Going Further — A dashboard is a product, and it foreshadows Part III"
    The moment your dashboard has a user who isn't you, ordinary product questions arrive: what happens when the numbers update and your caveats don't; who is responsible when someone makes a real decision from a state you never looked at; how you version it so a screenshot from March can still be explained in June. For a more built-up layout, Gradio's `Blocks` API lets you arrange several controls and panels (`with gr.Blocks() as demo: ...`), and Streamlit gives you `st.columns` and `st.sidebar` for the hierarchy described above. And the same tool reappears in Part III as the way you'll ship an *AI-driven* app to a hosted space: the skill of wrapping something powerful in an interface a human can trust is identical, whether the thing inside is a filter or a language model.

A dashboard is your analysis, handed over with the controls attached and the responsibility that comes with them. In [Chapter 6](apis.md) you collected data by asking a service for it. The last chapter of Part I is about the harder case: when the data you need exists only inside a web page, and nobody built you a door.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp — one course for the interactive chart underneath, then a tutorial for whichever framework you build in.

    - [Introduction to Data Visualization with Plotly in Python](https://app.datacamp.com/learn/courses/introduction-to-data-visualization-with-plotly-in-python) — **Ch. 1 Introduction to Plotly** and **Ch. 4 Advanced Interactivity.**
    - [Building user interfaces with Gradio](https://www.datacamp.com/tutorial/gradio-python-tutorial) — if you build in Gradio.
    - [Streamlit tutorial](https://www.datacamp.com/tutorial/streamlit) — if you build in Streamlit.
