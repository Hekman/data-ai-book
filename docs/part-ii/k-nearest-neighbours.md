<p class="chapter-kicker">Part II · Chapter 14 · <span class="week">Week 3</span></p>

# Classification by similarity: k-nearest neighbours

!!! guiding "The question this chapter answers"
    What does it mean for two things to be alike — and what happens when a system decides you are like someone else?

The tree judged by rules. This chapter meets the other way to classify, and it has no rules at all. **k-nearest neighbours**, or KNN, finds the examples most like the new case and lets them vote. If most of your five nearest neighbours bought a ticket, the model predicts you will too. That is the entire algorithm.

It is also the beginning of the idea that holds the rest of this part together: *things that are close get treated alike.* Clustering (Chapter 15), recommenders (Chapter 16) and text search (Chapter 17) are all built on it. So it is worth getting the hard question right here, while the algorithm is simple enough to see through: **close in what sense?**

!!! matters "Why this matters for designers"
    KNN is the logic behind "people like you", and a recommendation and a stereotype wear exactly the same coat. The difference between them is which features someone decided counted as similarity. When a product treats you as your neighbours, someone chose the neighbourhood — and the person most likely to be misjudged is the one whose nearest neighbours are not very near at all.

!!! newsroom "Why this matters for journalists"
    "People like you also…" is a claim about a comparison group, and comparison groups are where reporting can do real work. Which people? Like you in what way? Lookalike audiences in advertising, risk scores that compare an applicant with past applicants, matching on dating apps: all of them stand or fall on a definition of similarity that is almost never published. Asking for it is a legitimate question, and often the one the company least wants to answer.

!!! outcomes "By the end of this chapter you can"
    - Train a KNN classifier and explain a prediction by naming its nearest neighbours.
    - Explain why scaling decides who your neighbours are, and scale features properly.
    - Choose *k* with reasoning, and say what a very small and a very large *k* each do.
    - Split data into training and test sets, and explain why testing on training data is meaningless.
    - Identify who a similarity-based system serves badly, and why.

!!! origins "Origins — a report for the US Air Force"
    The nearest-neighbour rule first appeared in 1951, in a technical report by Evelyn Fix and Joseph Hodges for the US Air Force School of Aviation Medicine. It was not published in a journal for decades. In 1967 Thomas Cover and Peter Hart proved its most surprising property: with enough data, the error of the one-nearest-neighbour rule is never worse than twice the best error any method could achieve. A rule with no training step, no parameters and no theory of the problem turned out to be, in the limit, almost as good as anything. The catch is in "with enough data", and in a question Cover and Hart did not have to answer: enough data about *whom*?

## Judging by company

Here is a small festival app. It knows three things about each of 600 users — age, how many concerts they go to a year, and how much they spent through the app last year — and whether they bought a ticket for a jazz night it promoted.

```python
import numpy as np
import pandas as pd

rng = np.random.default_rng(5)
n = 600
older = rng.random(n) < 0.08                                  # few users over 55
fans = pd.DataFrame({
    "age":           np.where(older, rng.integers(55, 75, n), rng.integers(18, 35, n)),
    "concerts_year": rng.poisson(5, n),
    "spend_eur":     rng.gamma(2, 150, n).round(),
})
p_buy = np.where(older, 0.75, 1 / (1 + np.exp(-(fans["concerts_year"] - 5))))
fans["bought"] = (rng.random(n) < p_buy).astype(int)
fans["older"] = older.astype(int)          # kept aside: we will NOT give this to the model
```

**How it works.** We built two kinds of user. Younger users buy if they go to a lot of concerts. Older users, only about one in twelve of the base, mostly buy regardless. The `older` flag is not a feature; we keep it to check the model's errors later.

## Train and test

Before fitting, split. This is the habit Chapter 13 asked you to trust, and here is why it matters.

```python
from sklearn.model_selection import train_test_split

X = fans[["age", "concerts_year", "spend_eur"]]
y = fans["bought"]
X_train, X_test, y_train, y_test, older_train, older_test = train_test_split(
    X, y, fans["older"], test_size=0.25, random_state=1, stratify=y)
```

**How it works.** `train_test_split` shuffles the rows and deals them into two piles: 450 to learn from, 150 to test on. Passing `fans["older"]` as a third argument splits it the same way, so each test user keeps their flag. The model will never see the test users during fitting.

Why bother? Because KNN makes the problem obvious. Ask a one-nearest-neighbour model to predict a user it has already seen, and the nearest neighbour is *that same user*, at distance zero, with the right answer attached. It will score 100% every time. That is not a model being good. That is a model being asked to remember. Every score you report from now on should come from data the model did not learn from. If someone hands you a score without saying how it was split — a colleague, a paper, an AI assistant — that is the first question to ask.

## Fitting, and the first surprise

```python
from sklearn.neighbors import KNeighborsClassifier

raw = KNeighborsClassifier(n_neighbors=5).fit(X_train, y_train)
round(raw.score(X_test, y_test), 3)        # 0.58
```

58% on a problem where buyers and non-buyers are roughly balanced is barely better than a coin. Something is wrong, and the way to find out what is to do the thing that makes KNN worth teaching: look at the neighbours.

```python
fan = X_test.iloc[[0]]          # a 22-year-old, 5 concerts a year, spent €569
dist, idx = raw.kneighbors(fan)
X_train.iloc[idx[0]]
#      age  concerts_year  spend_eur
# 471   24              2      569.0
# 555   25              4      574.0
# 38    19              7      557.0
# 78    21              4      556.0
# 331   34              0      563.0
```

**How it works.** `.kneighbors()` returns the distance to, and position of, the *k* nearest training rows. Look at what they have in common: all of them spent between €556 and €574. Their concert-going ranges from zero to seven — the one thing that actually predicts buying — and the model has ignored it.

## Close in what sense?

KNN measures distance the way a ruler would: the straight-line distance between two rows, treating each column as an axis. A difference of 10 in age counts the same as a difference of 10 in concerts and the same as a difference of €10 in spending.

Now think about the ranges. Ages run over about 55 years. Concerts per year run from zero to about fifteen. Spending runs over a thousand euros. So spending dominates every distance calculation, simply because its numbers are bigger. Change spending from euros to cents and the model gets worse. Change it to thousands of euros and it gets better. Nothing about the users changed. The units did.

The fix is **scaling**: put every feature on a comparable footing before measuring distance. The standard choice is to subtract each column's mean and divide by its standard deviation, so every feature has mean 0 and a typical spread of 1.

```python
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline

scaled = make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=5))
scaled.fit(X_train, y_train)
round(scaled.score(X_test, y_test), 3)     # 0.753

scaler = scaled.named_steps["standardscaler"]
knn = scaled.named_steps["kneighborsclassifier"]
dist, idx = knn.kneighbors(scaler.transform(fan))
X_train.iloc[idx[0]]
#      age  concerts_year  spend_eur
# 228   20              5      598.0
# 435   22              5      626.0
# 380   20              5      617.0
# 429   26              5      503.0
# 78    21              4      556.0
```

**How it works.** `make_pipeline` chains the scaler and the classifier into one model. When you call `.fit`, the scaler learns each column's mean and spread *from the training data only*, and the classifier learns from the scaled result. When you call `.score` or `.predict`, new rows go through the same scaling first. Accuracy jumps from 58% to 75%, and the neighbours now make sense: young, about five concerts, similar spend.

Two lessons, and the second matters more than the first.

The technical one: **scale before you measure distance**, and do it inside a pipeline so the scaler only ever learns from training data. Scaling on all the data first and *then* splitting quietly leaks information about the test set into training. It is one of the errors AI coding assistants make most often, because the code runs and the score looks fine.

The design one: **"similar" is a decision.** You chose which columns go into the distance and how much each one counts. Scaling is one way of making that choice — "every feature counts equally" — but it is still a choice. If you wanted concert-going to matter twice as much as spending, you could make it so. Whoever builds a similarity system is deciding what it means for two people to be alike.

## How many neighbours?

The *k* in KNN is how many neighbours vote. It is yours to choose, and it pulls in two directions.

```python
for k in [1, 3, 5, 11, 25, 51]:
    m = make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=k)).fit(X_train, y_train)
    print(k, round(m.score(X_train, y_train), 3), round(m.score(X_test, y_test), 3))
# k    train  test
# 1    1.0    0.727
# 3    0.84   0.72
# 5    0.818  0.753
# 11   0.802  0.773
# 25   0.796  0.78
# 51   0.793  0.767
```

**How it works.** With *k* = 1, training accuracy is a perfect 1.0, for the reason above: every training user is their own nearest neighbour. That number means nothing. On the test set, small *k* is jumpy — one odd neighbour can flip a prediction — and very large *k* starts to blur, because the "neighbourhood" grows until it is most of the dataset. Here the test score peaks around *k* = 25. It is the same trade-off as the tree's depth in Chapter 13: too flexible and you memorise noise, too rigid and you miss the pattern.

A note of honesty about that last table. We have now used the test set to choose *k*. Strictly, that makes the test set part of the training process, and the 0.78 is a little optimistic. The clean method is a third split, or *cross-validation*, which repeats the split several times and averages. For a project, choose *k* with cross-validation on the training data and touch the test set once, at the end.

## Who is nobody's neighbour?

Accuracy is 75%. Split it by the group we kept aside.

```python
pred = scaled.predict(X_test)
for flag, label in [(0, "younger"), (1, "older")]:
    mask = (older_test == flag).values
    print(label, mask.sum(), round((pred[mask] == y_test.values[mask]).mean(), 2))
# younger 134 0.79
# older    16 0.44

dist, _ = knn.kneighbors(scaler.transform(X_test), n_neighbors=5)
pd.Series(dist.mean(axis=1), index=X_test.index).groupby(older_test.values).mean().round(2)
# 0    0.33     younger users: neighbours close by
# 1    0.68     older users:   neighbours twice as far away
```

**How it works.** For younger users the model is right 79% of the time. For older users, 44% — worse than guessing. The second calculation shows why: the average distance to an older user's five nearest neighbours is twice that of a younger user's. There are only 45 older users in the training set. Many of an older user's "nearest" neighbours are younger users who happen to be the least far away, and they vote according to *their* habits.

Sixteen test users is a small group, and you should not quote 44% as a precise figure. But the mechanism is not a matter of sample size, and it is the most important idea in this chapter. **A similarity-based system is only as good for you as the people who resemble you.** If you belong to a group the training data barely contains, your neighbours are strangers, and the model will treat you as if you were them. Nothing in the overall accuracy will say so.

This is the quiet failure behind a great many products. A recommendation engine trained mostly on one kind of user recommends badly to everyone else. A face-matching system trained mostly on one kind of face matches badly on others. The fix is rarely a better algorithm. It is better coverage, an honest statement of who the system is for, and a design that admits when it is guessing — for example, by showing less confidence when the nearest neighbours are far away.

!!! case "Teardown — a matching interface"
    Open a dating app, a roommate finder, or LinkedIn's "people you may know". Four questions.

    **Predicting.** A category, usually: will these two people both be interested? Or a ranking built on that prediction.

    **Collecting.** Your profile, your photos, and above all your swipes: who you said yes to, who you skipped, how long you looked first. And theirs.

    **Teaching.** Each swipe is a vote about similarity. "People like you liked people like them." Over time the system learns whose profiles get swiped right *by people similar to you*, and shows you more of those. If people similar to you have skipped a kind of profile, you may simply stop seeing it.

    **Choosing differently.** What makes two people neighbours here: age, distance, photos, stated interests, past behaviour? Which of those did the user choose to share, and which were inferred? Who is nobody's neighbour — and what does the app show them? A matching interface is "people like you" with the stakes visible, which is why it is the clearest place to see that someone chose the neighbourhood.

!!! critical "Critical Lens — Similar to whom, measured how?"
    It is tempting to think of similarity as a fact about the world: two people either are alike or are not. KNN makes plain that it is nothing of the sort. Similarity is a formula, applied to whichever columns someone collected, weighted however someone decided. Change the columns or the weights and the same two people become close or distant.

    That matters most when the system is used on people who never chose the comparison. A "lookalike audience" in advertising decides who is like your existing customers. A risk model decides which past applicants a new applicant resembles. The person being compared does not see the formula, the neighbours or the distances. The design question worth asking of any such system is the one that makes companies uncomfortable: could a person see who they were compared with, and argue that the comparison was wrong?

## How would we know this is any good?

This is the week the train/test split becomes non-negotiable. A model that has seen the answers is not being tested; it is being asked to remember. So:

1. **Never report a score from training data.** Hold data back before you fit anything, and do your scaling inside a pipeline so the test set stays unseen.
2. **Be suspicious of any score that arrived without a split behind it**, including one an AI assistant hands you. Ask how it was split, and whether anything was fitted on the whole dataset first.
3. **Look at the neighbours.** For a handful of predictions, print the nearest training rows. If they do not look like sensible company, the distance is measuring the wrong thing.
4. **Check the distances by group.** If some users' neighbours are consistently far away, the model is guessing about them, whatever the overall accuracy says.

!!! further "Going Further — When distance stops meaning anything"
    KNN has a well-known weakness with many features. As the number of columns grows, the distances between all points start to look alike — everyone is roughly equally far from everyone — and "nearest" stops carrying information. This is called the *curse of dimensionality*, and it is one reason real recommender and search systems do not compute distance on raw columns but first compress them into a smaller number of meaningful dimensions. You will meet the idea again in Chapter 16, and its modern form, the embedding, in Part III. MLU-Explain's [Train, Test, and Validation Sets](https://mlu-explain.github.io/train-test-validation/) and [Cross-Validation](https://mlu-explain.github.io/cross-validation/) pieces are good visual companions to this chapter.

So far someone always told the model the right answer: spam or not, bought or not. Next week nobody does. The model has to find groups on its own, and you have to decide whether the groups it finds are real.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. The first chapter is the KNN chapter; the second is the one that decides who your neighbours are.

    - [Supervised Learning with scikit-learn](https://app.datacamp.com/learn/courses/supervised-learning-with-scikit-learn) — **Ch. 1 Classification.**
    - [Supervised Learning with scikit-learn](https://app.datacamp.com/learn/courses/supervised-learning-with-scikit-learn) — **Ch. 4 Preprocessing and Pipelines.**

    **On your own** *(optional)*:

    - [Tutorial: KNN Classification with scikit-learn](https://www.datacamp.com/tutorial/k-nearest-neighbor-classification-scikit-learn) — scaling inside the split, and *k* chosen by cross-validation.
    - [Project: Predictive Modeling for Agriculture](https://www.datacamp.com/projects/1772) — about an hour; which single measurement tells you the most?
    - [Project: Predicting Credit Card Approvals](https://www.datacamp.com/projects/1908) — about an hour; read it as an eligibility system and ask which error costs the applicant.
