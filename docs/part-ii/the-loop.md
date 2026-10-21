<p class="chapter-kicker">Part II · Chapter 11 · <span class="week">Week 1</span></p>

# The loop: interfaces that teach models

!!! guiding "The question this chapter answers"
    What is an interface teaching the model behind it — and what does it mean for a machine to "learn" at all?

Part I ended with you building an interface: a dashboard someone else could steer. Part II starts with what happens when the interface steers back. Open any feed, any shop, any music app, and the screen in front of you is not just showing you things. It is watching what you do with them, and what you do becomes the material the next version of the screen is made from.

That is the whole of Part II in one sentence. A model learns from examples. In most products you touch, the examples are you.

!!! matters "Why this matters for designers"
    A like button is not feedback. It is a labelling tool you handed the user without telling them. Most of the consequential decisions in a machine learning product are made in the interface, long before anyone tunes a model: what you make easy to do decides what the system learns, and therefore what it will do to the next person. Designers author that part of the loop, usually without realising that is what they are doing.

!!! newsroom "Why this matters for journalists"
    "The algorithm decided" is the most common sentence in technology reporting and almost never true as written. Somebody chose what the system would count as success, and somebody designed the screen that collects the evidence. When you report on a platform, the useful questions are the ones in this chapter: what is it predicting, what signal does it collect, and who made those choices? A story that names the loop is harder to dismiss than one that names a black box.

!!! outcomes "By the end of this chapter you can"
    - Explain supervised learning in plain language: examples in, a fitted rule out, predictions on new cases.
    - Tell apart the three kinds of thing a model can be asked to do in this part: predict a number, predict a category, find structure without labels.
    - Draw the feedback loop: interface → behaviour → training data → model → interface.
    - Read an interface for its algorithmic affordances: what behaviour it invites, and what that behaviour labels.
    - Distinguish explicit signals (a rating) from implicit ones (how long you stayed), and say which one a system trusts more.
    - Use the four teardown questions on any ML-powered screen.

## What "learning" means here

Strip away the vocabulary and machine learning is a modest idea. You have examples where you know the answer. You have a method that looks at those examples and produces a rule. You use the rule on cases where you do not know the answer yet.

The examples are rows. The columns you give the method are the **features**. The column you want it to predict is the **target**, or the **label**. The rule it produces is the **model**. Using it on a new row is a **prediction**. That is all *supervised learning* means: learning supervised by examples where somebody, at some point, wrote down the right answer.

| You would say | The field says |
|---|---|
| the columns I give it | features |
| the thing I want it to guess | target, label |
| the examples it learns from | training data |
| working out the rule | fitting, training |
| the rule it ends up with | the model |
| a guess about a new case | a prediction |

The difference from the code you wrote in Part I is who writes the rule. In Part I you wrote it: *filter where the price is above 100, group by city*. Here you write the method, and the method writes the rule. You decide the features, the target and what counts as a good rule. The machine decides the details.

That division of labour is the reason this part of the book exists. The machine is very good at details. It has no opinion about whether the target was worth predicting, whether the examples were fair, or whether the features quietly encode something you would never use on purpose. Those are your decisions, and they do not stop being yours because a method was involved.

Three kinds of question run through the next chapters, and every technique in this part answers one of them:

- **How much?** Predict a number: a price, a duration, a rating. That is *regression* (Chapter 12).
- **Which kind?** Predict a category: spam or not, will buy or will not. That is *classification*, done by rules (Chapter 13) or by similarity (Chapter 14).
- **What is in here?** Nobody wrote down any answers. Find groups and likeness anyway. That is *unsupervised* learning (Chapters 15 and 16).

!!! origins "Origins — a checkers player that got better than its maker"
    In 1959 Arthur Samuel, an engineer at IBM, published *Some Studies in Machine Learning Using the Game of Checkers*. His program played thousands of games against itself and adjusted how it valued board positions according to which positions led to wins. It ended up beating Samuel. The phrase "machine learning" is usually credited to that work, along with a definition often attributed to him and still quoted: a field of study that gives computers the ability to learn without being explicitly programmed. Notice what Samuel still had to decide. What a win was. Which features of a board were worth scoring. How many games were enough. The program learned the weights; the human chose what was worth weighing.

## The loop

Here is the part a textbook on machine learning usually leaves out, because it is not about the model. In a product, the training data does not arrive from nowhere. It arrives from people using an interface. And the interface they use is shaped by the model trained on the last batch of people.

1. **The interface** shows something: a ranked feed, three suggested songs, a "you might also like" row.
2. **People behave**: they tap, skip, linger, scroll past, buy, return.
3. **The behaviour is logged** and becomes training data. A tap becomes "relevant". A fast scroll becomes "not relevant".
4. **The model is retrained** on that data and gets better at predicting what will be tapped.
5. **The interface changes**, because it is now driven by the new model. Back to step 1.

Every recommender, feed, search ranking, autocomplete and spam filter you use is some version of this. And each step hides a design decision. What is shown first decides what can be tapped at all. What is logged decides what the model can see. What counts as success decides what "better" means. None of those is a modelling choice. All of them shape the model.

The loop has a property that makes it treacherous: **it grades its own homework.** The model is trained on behaviour it caused. If the feed never shows you a kind of video, you never skip it or watch it, so the model never learns whether you would have liked it. The absence of evidence becomes evidence of absence, and the next version of the feed shows it even less.

## Algorithmic affordances

An *affordance* is what an object invites you to do. The word comes from the psychologist James Gibson, and Don Norman brought it into design in *The Design of Everyday Things* (1988): a flat plate on a door says push, a handle says pull. Designers think about affordances constantly. A button should look pressable. A slider should look draggable.

In an ML product every control has a second life. It invites a behaviour from the person *and* it produces a label for the model. Call that an **algorithmic affordance**: a control whose real job, from the system's point of view, is to turn what you do into training data.

| Control | What it invites | What it tells the model | What it cannot say |
|---|---|---|---|
| ♥ heart | approval, in one tap | "more like this" | *why* — the song, the mood, the moment |
| skip | impatience | "less like this" | "not now", "already heard it", "my friend is in the room" |
| autoplay | doing nothing | "kept listening" counts as approval | the difference between enjoying and not bothering |
| dwell (no control at all) | nothing — you cannot opt out | "this held attention" | shock, disgust, confusion, a phone left on the table |
| "not interested" | rejection, but buried in a menu | a strong negative | anything, mostly, because few people find it |
| star rating | considered judgement | a graded score | it is rare, so the model learns from the few who rate |

Two kinds of signal are mixed in that table. **Explicit** signals are things people do on purpose to express a view: a rating, a thumbs down. **Implicit** signals are side effects of using the product: how long you watched, whether you finished, whether you came back tomorrow. Explicit signals are honest but scarce. Implicit signals are noisy but endless, and they cannot be withheld, because you produce them simply by being there.

Most large platforms lean on implicit signals for exactly that reason. It is worth sitting with what that means. The behaviour a system trusts most is the behaviour you did not choose to give it.

The design consequence is direct. If you put a heart on the screen, you decide that approval is one-dimensional. If you make autoplay the default, you decide that passivity counts as a vote. If you bury "not interested" three taps deep, you decide that the model will rarely hear the word no. These are interface decisions, made by designers, and they set the ceiling on what the model can ever learn.

## A loop you can run

You do not need a real platform to see the loop work. Here is a music shelf with fifty songs, and every song is exactly as good as every other: each has the same 10% chance of being played when shown. The shelf shows five songs a day, ranked by how often each has been played so far. A hundred visitors a day, two hundred days.

```python
import numpy as np

def run_shelf(explore_slots=0, days=200, seed=1):
    rng = np.random.default_rng(seed)
    n_songs = 50
    appeal = np.full(n_songs, 0.10)        # every song is exactly as good as every other
    plays = np.zeros(n_songs)
    for day in range(days):
        ranked = np.argsort(-(plays + rng.random(n_songs)))   # most-played first, ties broken at random
        shelf = list(ranked[:5 - explore_slots])
        rest = [s for s in range(n_songs) if s not in shelf]
        shelf += list(rng.choice(rest, explore_slots, replace=False))   # (1)!
        for visitor in range(100):
            played = rng.random(5) < appeal[shelf]
            plays[shelf] += played
    top5_share = np.sort(plays)[::-1][:5].sum() / plays.sum()
    never_played = int((plays == 0).sum())
    return round(top5_share * 100), never_played

run_shelf(explore_slots=0)   # (100, 45)  five songs get every play; 45 are never heard
run_shelf(explore_slots=1)   # (81, 0)    one random slot, and every song gets heard
```

1.  The *explore* slot: instead of filling the whole shelf by popularity, give one place to a song picked at random.

**How it works.** Each day the shelf is sorted by `plays`, so whatever got played early stays on top, gets played more, and stays on top. With pure ranking, five songs take 100% of the plays and 45 of the fifty are never heard at all — not because they are worse, since every song is identical, but because they were never shown. Give up one slot of five to a random pick and every song gets heard, while the top five still take most of the plays.

That is a feedback loop in twenty lines. The model here is the crudest one possible — "rank by past plays" — and it already manufactures a hierarchy out of nothing. A real recommender is more sophisticated and the effect is subtler, but the shape is the same: what the system shows determines what it learns, and what it learns determines what it shows. The explore slot is a design decision, not a modelling one. It costs something: a fifth of the shelf now shows songs the system has no evidence for. It buys a system that can still find out it was wrong.

## Reading a screen: the teardown

Every class in this block opens by taking a real interface apart, and every chapter in this part has one. The method is four questions, always the same.

1. **What is the model predicting?** Not what the feature is called. What number or category is being guessed. "Recommended for you" is a prediction that you will watch for more than some number of seconds.
2. **What signal is the interface collecting?** List everything it could be logging: taps, pauses, scroll speed, replays, shares, the time of day.
3. **What does a like, a swipe, a skip teach it?** Translate each action into the label it becomes. Notice which actions have no label for the thing you meant.
4. **Where could the designer have chosen differently?** One concrete alternative: a different control, a different default, a different thing to measure.

!!! case "Teardown — a “For You” feed"
    Open TikTok, Instagram Reels or YouTube Shorts and scroll for two minutes with the four questions next to you.

    **Predicting.** Roughly: how long you will keep watching this video, and whether you will interact with it. The platforms describe their systems in general terms, but watch time sits near the centre of all of them.

    **Collecting.** Almost everything. Whether you watched to the end, watched twice, paused, turned the sound on, opened the comments, followed, shared, or swiped away in under a second. Most of this you do without deciding to.

    **Teaching.** The most important signal is the one with no control at all: how long you *failed* to scroll away. Lingering on a video because it upset you looks identical to lingering because you loved it. There is a "not interested" option, a long press away, and almost nobody uses it.

    **Choosing differently.** A feed could ask, now and then, *was that worth your time?* It could weight a deliberate save above a passive watch. It could show you why a video appeared. Each of those changes what the model can learn, and each has a cost the platform would have to accept.

    The purest lesson of this teardown is in the third answer. The richest signal is the one nobody chose to give.

!!! critical "Critical Lens — The labour nobody was asked for"
    Every label has to come from somewhere. In research datasets, people are paid to write them. In products, users write them for free, as a side effect of using the service, and usually without knowing.

    The clearest case is one you have probably done yourself. reCAPTCHA, the "type these distorted words" test, was used in the late 2000s to help digitise old books and newspaper archives: one word was the test, the other was a word the scanner could not read, and you transcribed it. Later versions asked you to click every square containing a traffic light. Each click was a human judgement, given to prove you were human, and also useful as a label.

    This is not a scandal so much as a pattern, and the pattern is worth naming in any project you build. If your interface turns behaviour into training data, then the people using it are doing part of the work of building your model. The minimum respect you owe them is to know that this is happening. The design question is whether *they* get to know it too.

## How would we know this is any good?

Every chapter in this part ends on that question, and this one has to start it with an uncomfortable answer: in a loop, ordinary measurement is compromised.

Say the shelf above reports that its top five songs have a 10% play rate, and the songs it never shows have no measurable play rate at all. That looks like proof the ranking is working. It is not. The unshown songs have no play rate because they were never shown. The system is comparing what it chose against nothing.

So the first evaluation habit of this part is a suspicious one. Whenever a system reports that it is performing well, ask *on whose data?* If the data was produced by people using the system, the system helped write its own exam. The honest ways round this — holding some users back from the model, randomising part of what is shown, asking people directly instead of watching them — all cost something. That cost is the price of knowing.

The second habit is simpler. A model can be accurate and the loop can still be bad. A feed that predicts your watch time perfectly can still be a feed you would rather not have. "Is the model good?" and "is the system good for the person using it?" are two different questions, and you will be asked both, in every chapter that follows.

!!! further "Going Further — When the measure becomes the target"
    The economist Charles Goodhart gave his name to an observation usually phrased as: *when a measure becomes a target, it ceases to be a good measure.* Watch time was a decent signal of whether people liked videos, until systems were optimised to maximise it, at which point videos were made to be hard to stop watching. The loop does not just learn from behaviour. Given time, it reshapes the behaviour and the content to fit what it measures.

    Cathy O'Neil's *Weapons of Math Destruction* (2016) is still the most readable account of what feedback loops do when they leave entertainment and enter policing, hiring and credit. It is a short book and you can read it in an evening. Read it with this chapter's diagram in mind: in nearly every case she describes, the damage is done by the loop, not by the arithmetic.

You now have the frame that holds the rest of this part: a model learns from examples, and in a product the examples are made by an interface you can design. The next chapter opens the simplest learning machine there is, one that predicts a number and whose reasoning you can read line by line.

!!! exercise "Practice on DataCamp"
    This chapter's practice is code-free. It builds the vocabulary before next chapter's code.

    - [Understanding Machine Learning](https://app.datacamp.com/learn/courses/understanding-machine-learning) — **Ch. 1 What is Machine Learning?** and **Ch. 2 Machine Learning Models.**
