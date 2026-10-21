<p class="chapter-kicker">Part II · Chapter 18 · <span class="week">Week 7</span></p>

# The interface around the model

!!! guiding "The question this chapter answers"
    The model works. Now — what does a person actually touch, and what happens when it is wrong?

Every chapter in this part has ended with a model that runs in a notebook. Nobody uses a notebook. People use a screen, and the screen is where a model's prediction turns into something that happens to someone: a label, a ranking, a promise, a decision. This chapter is about that screen. It is the part of machine learning that designers are actually responsible for, and the part almost nobody teaches.

Three questions carry it. What does the interface show when the model is **uncertain**? What does it do when the model is **wrong** — can the person notice, undo, appeal or route around it? And what does the interface **collect**, which is to say: what will this model become after a year of being used? That last question closes the loop Chapter 11 opened.

!!! matters "Why this matters for designers"
    This is the whole part arriving somewhere. You have spent six chapters reading other people's systems from the outside; now you build the outside of your own. A model with no interface is half a project, and a model whose interface hides its uncertainty is worse than half — it is a confident stranger. Designing for failure is not pessimism here. It is the only honest way to put a probabilistic thing in front of a person.

!!! newsroom "Why this matters for journalists"
    When an automated system harms someone, the story is rarely that the model was inaccurate. It is that the person had no way to know a model was involved, no way to see why it decided what it did, and no one to appeal to. Those are interface failures, and they are reportable: what did the person see, what could they do, and what happened when they tried? The interface is the part of the system an ordinary person meets, so it is the part an ordinary reader can understand.

!!! outcomes "By the end of this chapter you can"
    - Design the affordances around a model: what you invite the user to do, and what that teaches the system.
    - Show uncertainty honestly in an interface, rather than rounding it into a confident answer.
    - Design for error: how someone notices, corrects or escapes a wrong prediction.
    - Explain a prediction to a non-technical user without lying about how it was made.
    - Build a small working prototype in Streamlit that does all of the above.
    - Describe what your system would become after a year of its own output feeding back into it.

!!! origins "Origins — guidelines for working with a machine that guesses"
    In 1999 Eric Horvitz at Microsoft Research published *Principles of Mixed-Initiative User Interfaces*, one of the first serious attempts to say how an interface should behave when part of it is guessing what the user wants. Among his principles: consider uncertainty about the user's goals, let the user easily invoke or dismiss the system, and make it cheap to correct it when it is wrong. Twenty years later, in 2019, a Microsoft team led by Saleema Amershi distilled decades of research into eighteen *Guidelines for Human-AI Interaction*, and Google's PAIR team published the *People + AI Guidebook*. Read either today and you find Horvitz's concerns at the centre: make clear what the system can do and how well, support efficient correction, and remember that the system will change as it learns.

## A model with a face

Start with the sizing detector from Chapter 17. A shop's support team wants to paste a review and know whether it is a sizing complaint, so they can fix the size chart. Here is the simplest interface that would work, in Streamlit, the tool you used for dashboards in Part I.

```python
import os
import datetime
import pandas as pd
import streamlit as st
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.neighbors import KNeighborsClassifier
from sklearn.pipeline import make_pipeline

@st.cache_resource
def load_model():
    reviews = pd.read_csv("reviews.csv")          # the labelled reviews from Chapter 17
    model = make_pipeline(TfidfVectorizer(),
                          KNeighborsClassifier(n_neighbors=5, metric="cosine"))
    model.fit(reviews["text"], reviews["sizing"])
    return model, reviews

model, reviews = load_model()
tfidf = model.named_steps["tfidfvectorizer"]
knn = model.named_steps["kneighborsclassifier"]

st.title("Does this review mention a sizing problem?")
text = st.text_area("Paste a review")

if text:
    vector = tfidf.transform([text])
    if vector.nnz == 0:                                       # (1)!
        st.warning("I don't recognise any of these words, so I can't tell. "
                   "I was trained on English reviews of clothes.")
    else:
        distances, idx = knn.kneighbors(vector)
        votes = int(reviews["sizing"].iloc[idx[0]].sum())    # how many of the 5 were sizing
        closeness = 1 - distances.mean()                     # average cosine similarity
        if closeness < 0.3:                                  # (2)!
            st.info("Not sure. Nothing I've seen before is much like this review.")
        elif votes >= 4:
            st.error(f"Probably about sizing: {votes} of the 5 most similar reviews were.")
        elif votes <= 1:
            st.success(f"Probably not about sizing: {5 - votes} of the 5 most similar reviews weren't.")
        else:
            st.info(f"Could go either way: {votes} of the 5 most similar reviews were about sizing.")

        with st.expander("Why? The five reviews this was compared with"):   # (3)!
            for i, d in zip(idx[0], distances[0]):
                label = "sizing" if reviews["sizing"].iloc[i] else "not sizing"
                st.write(f"“{reviews['text'].iloc[i]}” — {label}, similarity {1 - d:.2f}")

        left, right = st.columns(2)                          # (4)!
        verdict = None
        if left.button("That's right"):
            verdict = "right"
        if right.button("That's wrong"):
            verdict = "wrong"
        if verdict:
            row = pd.DataFrame([{"when": datetime.datetime.now().isoformat(timespec="seconds"),
                                 "text": text, "votes": votes, "user_says": verdict}])
            row.to_csv("feedback.csv", mode="a", index=False,
                       header=not os.path.exists("feedback.csv"))
            st.write("Thanks — logged. I'll learn from this next time I'm retrained.")
```

1.  **The state where the model knows nothing.** If none of the words were in the training vocabulary, the TF-IDF vector is empty. Rather than return a meaningless guess, say so, and say what the model *was* trained on.
2.  **The state where the model is guessing.** If even the nearest reviews are not very similar, the vote is a vote among strangers — Chapter 14's "nobody's neighbour". Say that instead of reporting it.
3.  **The explanation.** Not a story about the model, but the actual evidence it used: the five reviews, their labels and how similar each one was.
4.  **The correction.** Two buttons, both logged. This is where the loop closes.

**How it works.** Save this as `app.py` next to a `reviews.csv` and run `streamlit run app.py` in a terminal, as in Part I. `@st.cache_resource` trains the model once and keeps it, so it is not retrained on every click. The rest of the script runs top to bottom every time the user types or clicks, which is how Streamlit works. The prediction itself is three lines. Everything else is the interface around it, and everything else is where the design is.

Read the code again, and count the states. There are not two — "sizing" and "not sizing" — but five: *I don't know these words*, *nothing is close to this*, *probably sizing*, *probably not*, *could go either way*. That is the single most important change this chapter asks of you.

## Uncertainty

A classifier's natural output is a confident label. The model behind it almost never has a confident answer. KNN has a vote; a tree has the share of training examples in a leaf; regression has residuals. Every one of them knows, in some sense, how sure it is. The interface decides whether the person ever finds out.

Most interfaces throw that information away, because a single answer looks cleaner and tests better in a demo. The cost arrives later, when someone trusts a coin-flip as though it were a certainty.

Three ways to show uncertainty honestly, from simplest to richest:

- **Words, not numbers.** "Probably", "could go either way", "not sure". Most people read *73% confident* as *right*, and *51% confident* as *right* too. A small vocabulary of hedges, used consistently, communicates better than a decimal.
- **Counts, not percentages.** "4 of the 5 most similar reviews" is a sentence a person can check and picture. "80% probability" invites the question *80% of what?*
- **A state for "I don't know".** The most honest thing a model can say is that a case is outside what it learned from. It is also the state most interfaces do not have. If your project has no screen for that, design one.

Where you set the thresholds between those states — `votes >= 4`, `closeness < 0.3` — is a design decision, not a modelling one, and it should be made the way Chapter 17 made the threshold decision: by what each kind of error costs, written down before you look.

## Error

Every model in this part is wrong some of the time, and you know how often, because you evaluated it. So the question is not whether the interface will show a wrong answer. It will. The question is what the person can do next.

The PAIR Guidebook frames this well: design for **graceful failure**. For each kind of error your model makes, walk through four steps.

1. **Can the person notice?** If the interface shows a confident answer and no evidence, a wrong answer looks exactly like a right one. Showing the evidence — the five similar reviews — is what lets a person say *those aren't sizing complaints at all*.
2. **Can they undo it?** Is a wrong prediction a suggestion they can ignore, or an action that already happened? A spam filter that moves an email to a folder is undoable. One that deletes it is not.
3. **Can they correct it?** Is there a way to say *that's wrong* that is visible, quick, and does something? The "not spam" button from Chapter 13 is an example. A "report a problem" link three menus deep is not.
4. **Can they route around it?** If the model will not budge — it is wrong and the person cannot fix it — is there a human, a manual option, a way to proceed anyway?

For most of your projects the honest answer to at least one of these is *no*. That is fine, as long as you know which one, and say so on your poster. A system that only works when it works is a system nobody should depend on.

## Explanation

The *Why?* panel in the prototype does something most real explanations do not: it shows the actual evidence. The five reviews the decision was based on, their labels, how similar each was. A person can read them and judge whether the comparison was fair.

That is possible because every model in this part is one you can read. Regression gives you coefficients, a tree gives you a path, KNN and cosine give you neighbours, clusters give you a profile. Each one supports a different honest explanation:

| Model | Honest explanation | What it lets the person check |
|---|---|---|
| Regression | "Each extra step added about 4 minutes" | whether the inputs were right |
| Decision tree | "Unknown sender, more than five links" | whether the rule makes sense |
| KNN / cosine | "These five reviews were most similar" | whether the comparison was fair |
| k-means | "You're in the group that listens late at night" | whether the label fits |

Notice what all four have in common: they describe *what the model actually did*, in terms the person can verify. The dishonest kind of explanation is the one that sounds plausible but has no connection to the computation — a friendly sentence generated after the fact. "We think you'll love this!" is not an explanation. Neither, strictly, is "Because you watched…" if the real reason was that the film was being promoted this week.

Part III is where this becomes hard, because the models there cannot be read and their explanations have to be built separately. Here, you have no excuse: your model can tell the truth about itself. Let it.

## What the interface collects

The two buttons at the bottom of the prototype write a line to `feedback.csv` every time someone presses one. That file is training data. The next time the model is retrained, the corrections can go in.

This is Chapter 11's loop, and now you are the one designing it. So ask the questions you asked in every teardown, about your own system.

- **What does each button teach?** *That's wrong* on a "sizing" prediction is a labelled negative example. But who presses it? If only annoyed users bother, the feedback will over-represent one kind of error.
- **What does silence mean?** Most people will press nothing. Does that count as *right*? If it does, the model learns that it is always right whenever nobody complains.
- **What will it become?** If the support team uses the tool mostly on reviews that already looked like sizing complaints, the feedback will mostly be about those. After a year, the model may be excellent on obvious cases and no better on the hard ones.

## Drift

There is one more way a model fails, and it is the quietest: the world changes and the model does not. This is called **drift**. A model trained on last year's reviews meets this year's products, this year's slang, this year's customers.

The shop starts selling trousers.

```python
def unknown_share(texts):
    read = tfidf.build_analyzer()                 # the same word-splitting the model uses
    words = [w for t in texts for w in read(t)]
    return round(sum(w not in tfidf.vocabulary_ for w in words) / len(words), 2)

unknown_share(X_test)          # 0.0    reviews like the ones it learned from
trousers = ["Waist is too loose, great fabric.", "Legs are far too long, arrived quickly.",
            "Lovely colour, the waistband digs in.", "Inseam is short, good value.",
            "Really soft, pockets are too shallow."]
unknown_share(trousers)        # 0.37   more than a third of the words are new to it
model.predict(trousers)        # [0 1 0 0 0]
```

**How it works.** `build_analyzer()` returns the function the vectoriser uses to split text into words, and `vocabulary_` is every word it learned during training. On reviews like its training data, every word is known. On the trousers reviews, more than a third are not: *waist*, *inseam*, *legs*, *waistband*. Four of those five reviews are sizing complaints. The model finds one.

Nothing crashed. No error appeared. Accuracy, if anyone measured it, would look the same as last month on the jumper reviews. The model simply stopped hearing a new kind of customer, and the summary at the top of the trousers page will say *customers are happy with the fit*.

Drift is why a model is never finished. The defence is not a better model but a habit: measure something simple about incoming data every week — the share of unknown words, the spread of predictions, how often people press *that's wrong* — and look at it. If the number moves, retrain or investigate. Put that check in the interface for whoever maintains the system. Your future self is a user too.

!!! case "Teardown — Netflix's “Because you watched…”"
    Open Netflix (or any service with a "because you watched" or "recommended because" row) and use all six questions the block has built up.

    **Predicting.** What you are likely to start, and to keep watching.

    **Collecting.** Your history; what you hovered over; what you started and abandoned.

    **Teaching.** Starting a title from this row is a vote for the explanation as well as the film.

    **Choosing differently.** This is one of the few shipped interfaces that tries to explain its own model. So test the explanation. Does it actually explain — could you predict the recommendation from the reason? Is it true — is the named film plausibly the cause, or a label attached to a ranking that came from somewhere else? Can you do anything with it — remove the film from your history, say "not because of that"?

    **What group has it put me in?** Rows like "Because you watched" sit next to rows of genres and moods. Which of them describe you?

    **How would I know if it was wrong?** This is the question your own prototype has to answer. Copy what works here; it is the closest thing to a reference design for what you are building.

!!! critical "Critical Lens — The confident stranger"
    An interface that rounds a model's uncertainty into a single confident answer does something subtle: it moves responsibility. If the screen says *sizing complaint*, and it was wrong, the person who acted on it made a mistake. If the screen said *could go either way* and they acted anyway, they made a judgement. The difference matters to them, and it matters to everyone downstream.

    Hiding uncertainty is usually not malicious. It tests well. Users say they prefer clear answers, demos look cleaner, and "the system is 51% sure" is an awkward thing to put on a screen. But a probabilistic system presented as a certain one is a confident stranger: it gives directions with total assurance and no memory of whether it has ever been to the place. The design work of this chapter — the extra states, the evidence, the correction button, the drift check — is all in service of one goal. The person should always be able to tell how much to trust what they are looking at.

## How would we know this is any good?

Evaluate the interface, not the model. You already know the model's precision and recall. What you do not know is whether a person, using the screen, makes better decisions than they would without it.

1. **Hand it to someone who has never seen it**, and do not explain anything. Watch.
2. **Do they know what it is claiming?** Ask them to say, in their own words, what the screen just told them and how sure it was.
3. **Do they notice when it is wrong?** Give them a case you know the model gets wrong. Do they spot it? What gave it away, or what hid it?
4. **Can they do anything about it?** Watch whether they find the correction, and whether they believe it does something.
5. **What did it collect?** Look at the log afterwards. Is it the data you would want to retrain on?

If the answer to the fourth question is no, you have designed a system that only works when it works. That is the most common finding in this kind of test, and the most useful one to put on a poster.

!!! further "Going Further — Reference designs"
    The [People + AI Guidebook](https://pair.withgoogle.com/guidebook/) from Google PAIR is free, short and practical; the chapters on [Explainability + Trust](https://pair.withgoogle.com/chapter/explainability-trust/), [Feedback + Control](https://pair.withgoogle.com/chapter/feedback-controls/) and [Errors + Graceful Failure](https://pair.withgoogle.com/chapter/errors-failing/) map almost one to one onto the sections above. Amershi and colleagues' *Guidelines for Human-AI Interaction* (CHI 2019) is the research version: eighteen guidelines, each tested against real products, and a good checklist to run your prototype against before the presentation.

That is the end of Part II. You can now read a system that learns — what it predicts, what it learned from, where its judgement lives, how it fails — and build a small one honestly, interface included. Part III starts where this part deliberately stopped: with models you cannot read, and the question of how to work with a system that will never tell you why.

!!! exercise "Practice"
    There is no DataCamp course for this chapter. The practice is your own project.

    - Take your model from the last six weeks and wrap it in a Streamlit interface with at least one *I'm not sure* state and one way to say *that's wrong*.
    - The [Streamlit tutorial](https://www.datacamp.com/tutorial/streamlit) from Part I's dashboards chapter covers everything the prototype above uses.
