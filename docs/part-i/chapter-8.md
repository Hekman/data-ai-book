<p class="chapter-kicker">Part I · Chapter 8</p>

# Dashboards and interactive views

A chart makes one argument. Sometimes what a stakeholder actually needs is not your single argument but a small instrument they can operate themselves — filter to their region, change the date range, pick a category — and see the data respond. That instrument is a **dashboard**, and building one is the step where your analysis stops being a document you hand over and becomes a *tool someone else uses*. That shift changes your job: a document you control; a tool you must design for someone who will use it without you in the room.

The lineage runs straight back through Chapter 7. Playfair's claim — let the eye do the work — industrialised over two centuries into self-service analytics.

!!! origins "Origins — Self-service visualisation"
    Tableau, spun out of a Stanford research project in 2003, put grammar-of-graphics-style visual analysis into the hands of people who couldn't program, and normalised the expectation that anyone in an organisation could interrogate data visually rather than wait for a specialist. A dashboard is that expectation made concrete: Playfair's argument, handed to the reader with the controls attached.

## What a dashboard is for — and against

The danger of dashboards is that they're easy to make badly. A wall of twenty charts is not a dashboard; it's an abdication, pushing onto the viewer the very job of finding what matters that you were supposed to do. A good dashboard is *designed*, and the design questions are ones you already know from interface work. What is the one thing this should show at a glance, before anyone touches a control? Which few dimensions are worth making interactive, and which would only add noise? What does the reader see *first* — because the default view is the argument you're making before they change anything, and most people never change anything. Restraint is the whole craft. The best dashboards show less than they could and make the important thing impossible to miss.

## Building one without becoming an engineer

You can build a working, interactive dashboard directly beside your analysis, without standing up a website. In this course we use **Gradio**, which wraps a Python function in inputs and outputs and runs *inside the notebook*. Start with the smallest possible version — a function, one input, one output:

```python
import gradio as gr

def greet(name):
    return f"Hello, {name} — welcome to your first dashboard."

demo = gr.Interface(fn=greet, inputs="text", outputs="text")
demo.launch()      # renders an interactive panel right in the notebook
```

**How it works.** `gr.Interface` takes three things: a function `fn`, a description of its `inputs`, and of its `outputs`. Gradio builds the UI, calls your function whenever the input changes, and shows what it returns. That's the whole model — *your analysis is the function; Gradio is the interface around it.* Now make the function do something real: filter a dataframe and draw a chart.

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

1.  The whole dashboard is this one function. It takes the control's value, filters the data, and returns a chart. Everything you learned in Chapters 3 and 7 is doing the work — Gradio just adds the knob.

!!! practice "In Practice — Your dataset, three controls"
    Build the smallest dashboard that answers a real question about your Part I dataset, with **no more than three controls**. Pick the single chart that carries the main finding, and add only the filters a genuine user would reach for. Then interrogate your defaults: when the dashboard first loads, before anyone clicks, does it already tell the truth? (Above, the default `"All"` view shows the honest overall picture.) If the opening view is misleading or blank, you've mistaken a pile of controls for a tool. The three-control limit does the same work restraint always does in design — it forces you to decide what actually matters.

!!! further "Going Further — A dashboard is a product, and it foreshadows Part III"
    The moment your dashboard has a user who isn't you, ordinary product questions arrive: what happens when the filter returns nothing; how do you keep it honest when the numbers update; who is responsible when someone makes a real decision from it. For a more built-up layout, Gradio's `Blocks` API lets you arrange several controls and panels (`with gr.Blocks() as demo: ...`). And the same tool reappears in Part III as the way you'll ship an *AI-driven* app to a hosted space — the skill of wrapping something powerful in an interface a human can trust is identical, whether the thing inside is a filter or a language model.

A dashboard is your analysis, handed over with the controls attached and the responsibility that comes with them. So far we've worked with data someone gave us. The last chapter of Part I is about what to do when the data you need doesn't exist yet.

## Exercises

??? exercise "Exercise 8.1 · A three-control dashboard on your data"
    Wrap one finding from your Part I dataset in a Gradio dashboard with at most three controls. Make sure the default view (before any interaction) states the main finding honestly. Write two sentences justifying which controls you included and which you deliberately left out.

    ??? note "Guidance"
        The grade is in the *omissions*. A good answer explains a control you rejected — "I dropped the date filter because 90% of the data is from one month, so it would imply a richness that isn't there." Deciding what not to build is the design act.

??? exercise "Exercise 8.2 · Design the empty state"
    Extend your dashboard so that when a filter combination returns no rows, the user sees a clear message instead of a broken or blank chart. Why is the empty state a data-ethics issue as much as a UX one?

    ??? note "A hint and the point"
        ```python
        if subset.empty:
            fig, ax = plt.subplots()
            ax.text(0.5, 0.5, "No orders match this filter",
                    ha="center", va="center"); ax.axis("off")
            return fig
        ```
        It's an ethics issue because a blank or error-y chart invites the user to invent an explanation ("sales collapsed?") for what is really just missing data. Saying "no data here" plainly is the honest default — the same honesty Chapter 4 asked of your cleaning.
