<p class="chapter-kicker">Part II · Chapter 13 · <span class="week">Week 2</span></p>

# Classification by rules: decision trees

!!! guiding "The question this chapter answers"
    How does a model decide which category something belongs to, when the rule is one it wrote itself?

Last chapter the answer was a number. This chapter it is a category: spam or not, clickbait or not, eligible or not. Predicting a category is called **classification**, and it is behind more consequential decisions than any other kind of model, because so many decisions about people come down to a yes or a no.

You meet the most readable classifier there is: the **decision tree**. It is a flowchart, written by the model, by looking at examples. You can print it out and follow it with your finger. That makes it the right place to meet classification — and it makes it a trap, because a rule you can read feels like a rule you can trust.

!!! matters "Why this matters for designers"
    Classifiers drive the calls that decide what happens to people: fraud flags, content labels, moderation, eligibility. Trees power a lot of them precisely because their rules can be shown to a review board, which makes them easy to over-trust. And every classifier has two ways to be wrong, which almost never cost the same. Designing for a classifier means deciding, on purpose, which error you would rather make — and then designing the screen for the person on the receiving end of it.

!!! newsroom "Why this matters for journalists"
    The classifier stories that matter are about errors, and about who gets which kind. In 2016 ProPublica's *Machine Bias* investigation of the COMPAS risk score argued that Black defendants who did not reoffend were far more often labelled high-risk than white defendants who did not. The company replied that the score was equally accurate for both groups. Both claims were arithmetically defensible, and the argument that followed showed that a classifier often *cannot* be fair in every sense at once. You cannot report that kind of story without knowing the two ways to be wrong.

!!! outcomes "By the end of this chapter you can"
    - Train a decision tree with scikit-learn and print its rules.
    - Explain one prediction by following its splits out loud.
    - Explain how a tree chooses a split, and why a deeper tree is not a better tree.
    - Read a confusion matrix, and explain why accuracy alone misleads on imbalanced data.
    - Name the two ways a classifier can be wrong, and say which one costs more in a given context.
    - Spot a feature that is acting as a proxy for something you did not intend to use.

!!! origins "Origins — trees that grew themselves"
    People have drawn decision trees by hand for as long as there have been procedures: a doctor's triage chart is one, and so is the flowchart on the back of a tax form. What changed in the 1980s was that methods appeared for *learning* the tree from data. Leo Breiman, Jerome Friedman, Richard Olshen and Charles Stone published *Classification and Regression Trees* in 1984 — the CART method, still the basis of scikit-learn's trees — and Ross Quinlan published ID3 in 1986. Breiman went on to write one of the field's most quoted papers, *Statistical Modeling: The Two Cultures* (2001), arguing that statisticians cared too much about readable models and too little about prediction. Fifteen years later the field had swung so far the other way that readable models needed defending again. This part of the book is, in a small way, part of that defence.

## The spam folder

Two thousand emails, one in nine of them spam. For each we know four things: how many links it contains, whether the sender is in your contacts, how many exclamation marks it uses, and whether it is written in a non-Latin script — Arabic, Cyrillic, Chinese and so on. And we know the label: spam or not.

```python
import numpy as np
import pandas as pd

rng = np.random.default_rng(3)
n = 2000
spam = rng.random(n) < 0.10
emails = pd.DataFrame({
    "links":            np.where(spam, rng.poisson(3, n), rng.poisson(1.5, n)),
    "known_sender":     np.where(spam, rng.random(n) < 0.15, rng.random(n) < 0.60).astype(int),
    "exclamations":     np.where(spam, rng.poisson(2, n), rng.poisson(0.8, n)),
    "non_latin_script": np.where(spam, rng.random(n) < 0.50, rng.random(n) < 0.04).astype(int),
    "spam":             spam.astype(int),
})
emails["spam"].mean()      # 0.11  about one email in nine is spam
```

**How it works.** As in the last chapter the data is synthetic, so we know what we built. Spam has more links and exclamation marks, rarely comes from a known sender — and, in this inbox, half of it arrives in a non-Latin script, because a large spam campaign happened to be written that way. Only 4% of legitimate mail is. Hold on to that last fact.

## Growing a tree

Before fitting anything, we hold some emails back. A model should be judged on cases it did not learn from, so we split the data: three quarters to learn from, one quarter to test on. Chapter 14 explains this properly; for now, trust it.

```python
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier, export_text

X = emails.drop(columns="spam")
y = emails["spam"]
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=0, stratify=y)       # (1)!

tree = DecisionTreeClassifier(max_depth=3, random_state=0)
tree.fit(X_train, y_train)
print(export_text(tree, feature_names=list(X.columns)))
```

1.  `stratify=y` keeps the share of spam the same in both halves. With rare classes, an unlucky split can otherwise put nearly all the spam on one side.

```text
|--- non_latin_script <= 0.50
|   |--- exclamations <= 2.50
|   |   |--- links <= 3.50
|   |   |   |--- class: 0
|   |   |--- links >  3.50
|   |   |   |--- class: 0
|   |--- exclamations >  2.50
|   |   |--- known_sender <= 0.50
|   |   |   |--- class: 0
|   |   |--- known_sender >  0.50
|   |   |   |--- class: 0
|--- non_latin_script >  0.50
|   |--- known_sender <= 0.50
|   |   |--- exclamations <= 0.50
|   |   |   |--- class: 0
|   |   |--- exclamations >  0.50
|   |   |   |--- class: 1
|   |--- known_sender >  0.50
|   |   |--- links <= 5.00
|   |   |   |--- class: 0
|   |   |--- links >  5.00
|   |   |   |--- class: 1
```

**How it works.** `max_depth=3` allows at most three questions before an answer. `export_text` prints the tree as indented rules: each `|---` line is a question, and each `class:` line is an answer, 0 for not spam and 1 for spam. That printout is the whole model. There is nothing else hidden inside it.

## Following a prediction with your finger

Take one email: two links, an unknown sender, one exclamation mark, written in Arabic. Start at the top.

1. `non_latin_script <= 0.50`? No, it is 1. Go down the lower branch.
2. `known_sender <= 0.50`? Yes, the sender is unknown.
3. `exclamations <= 0.50`? No, there is one.
4. **Spam.**

Now take the *same* email in Dutch. Step 1 sends it down the upper branch, and every path on that side ends in `class: 0`. **Not spam.**

```python
email = {"links": 2, "known_sender": 0, "exclamations": 1}
tree.predict_proba(pd.DataFrame([{**email, "non_latin_script": 1}])).round(2)   # [[0.12 0.88]]
tree.predict_proba(pd.DataFrame([{**email, "non_latin_script": 0}])).round(2)   # [[0.97 0.03]]
```

**How it works.** `.predict_proba` returns, for each class, the share of training emails in the leaf where this email lands. The Arabic version lands in a leaf where 88% of training emails were spam. The Dutch version, identical in every other respect, lands in one where 3% were. The only thing that changed was the alphabet.

This is the double edge of a readable model. You could see the problem immediately, because it is in the first line of the printout. You would never have seen it in a model you could not read. But the readability did not prevent it. The tree did exactly what it was built to do: find the question that best separates spam from not-spam in *this* data. In this data, the alphabet was that question.

## How a tree chooses a question

At every step, the tree tries every feature and every threshold — `links <= 1.5`, `links <= 2.5`, and so on — and asks the same thing of each: if I split the emails here, how *pure* are the two groups I get? A pure group is all spam or all not-spam. A mixed group is the problem we started with. The tree picks the split that makes the groups purest, then does the same thing again inside each group, and again, until it reaches the depth you allowed.

scikit-learn measures purity with *Gini impurity* by default: the chance that two emails drawn at random from a group have different labels. Zero means pure. You do not need the formula. You need the idea: **a tree is a sequence of questions, each chosen because it sorts the training examples best.** Nothing in that procedure asks whether a question is fair, sensible or legal to ask.

```python
dict(zip(X.columns, tree.feature_importances_.round(2)))
# {'links': 0.07, 'known_sender': 0.28, 'exclamations': 0.14, 'non_latin_script': 0.51}
```

**How it works.** `feature_importances_` adds up how much each feature improved purity across all the splits that used it, scaled to sum to one. The alphabet does half the work. That number is the warning light you would have wanted on a dashboard before this filter went live.

## Why a deeper tree is not a better tree

If three questions are good, why not thirty? Because with enough questions a tree can carve out a leaf for every oddity in the training data, including the ones that are just noise. It stops learning the pattern and starts memorising the examples.

```python
for depth in [1, 2, 3, 5, 10, None]:
    t = DecisionTreeClassifier(max_depth=depth, random_state=0).fit(X_train, y_train)
    print(depth, round(t.score(X_train, y_train), 3), round(t.score(X_test, y_test), 3), t.get_n_leaves())
# depth  train  test   leaves
# 1      0.901  0.902   2
# 2      0.928  0.926   4
# 3      0.931  0.924   8
# 5      0.942  0.930  26
# 10     0.946  0.930  80
# None   0.946  0.930  80
```

**How it works.** `.score()` on a classifier is accuracy: the share of emails labelled correctly. On the training emails, accuracy keeps creeping up as the tree deepens. On the test emails it stops improving at depth 5, while the tree has grown from 8 leaves to 80. Those extra 72 leaves bought nothing on new emails. They are a description of the training set's accidents, and a tree with 80 leaves is no longer one you can follow with your finger.

The gap between training and test performance has a name, **overfitting**, and it is the most common way a model looks better in a notebook than in the world. The general shape — more flexibility fits the training data better and new data no better, or worse — is called the *bias–variance trade-off*, and it applies to every model in this part.

## Two ways to be wrong

Accuracy says the depth-3 tree gets 92.4% of test emails right. That sounds good until you compute what a filter that does nothing would score.

```python
from sklearn.metrics import accuracy_score, confusion_matrix

pred = tree.predict(X_test)
accuracy_score(y_test, pred)                    # 0.924
accuracy_score(y_test, np.zeros_like(y_test))   # 0.89   a "filter" that never says spam
confusion_matrix(y_test, pred)
# [[441   4]      real not-spam: 441 let through correctly, 4 wrongly binned
#  [ 34  21]]     real spam:      34 let through wrongly,  21 caught
```

**How it works.** A filter that never flags anything scores 89%, because 89% of email is not spam. On imbalanced data, accuracy is mostly a measure of how imbalanced the data is. The **confusion matrix** tells you what actually happened. Rows are the truth, columns are the prediction. The top-right cell is **false positives**: real mail sent to the spam folder. The bottom-left cell is **false negatives**: spam that reached the inbox.

Those two cells are not two flavours of the same mistake. They happen to different people and they cost different things.

- A **false negative** here is a spam email in your inbox. Annoying. You delete it.
- A **false positive** is a real email you never see. A job offer. A message from a relative abroad. The confirmation of a flight.

Most people would accept a lot of the first kind to avoid one of the second, which is why spam filters are tuned to be cautious. Now look at who gets the false positives in our tree:

```python
legit = y_test == 0
pred_s = pd.Series(pred, index=X_test.index)
pred_s[legit & (X_test["non_latin_script"] == 1)].mean()   # 0.20   one in five of their real emails binned
pred_s[legit & (X_test["non_latin_script"] == 0)].mean()   # 0.00   none
```

There are only twenty legitimate non-Latin emails in the test set, so treat the exact figure with care. But the direction is unmistakable, and in a real inbox it would be the person who corresponds with family in Arabic or Russian whose real email disappears. The tree's overall error rate looks fine. Its errors are not evenly shared.

## Proxies

`non_latin_script` is a **proxy**: a feature that stands in for something else. Here it stands in, partly, for "this is from that spam campaign". It also stands in for the language of your family, your country of origin, your community. The tree has no way of knowing which of those it is using. It only knows the feature sorts the training data well.

Proxies are rarely this obvious. A postcode stands in for income and often for ethnicity. A first name stands in for gender and age. Device type stands in for wealth. The time someone applies stands in for whether they work shifts. Deleting the sensitive column does not delete the information, because it is spread across the others. A model that never sees "nationality" can still learn it from the right combination of postcode, language and name.

So the defence is not to remove columns and hope. It is to ask, of every feature that does real work in your model, *what else could this be standing in for?* — and then to check, as we did above, whether the model's errors fall unevenly on any group you can identify.

!!! case "Teardown — the spam folder and its “not spam” button"
    Open your email's spam folder and use the four questions.

    **Predicting.** A category: does this message belong in the inbox?

    **Collecting.** The content and sender of every message, and what you do with it: open, reply, delete unread, move to spam, rescue from spam.

    **Teaching.** Look at the two buttons. *Report spam* and *Not spam* are each a label, given by you, about one specific email. The second one is the interesting one. You press it to get one message back. You are also, without being told, correcting a false positive in the training data. And notice the asymmetry: the inbox has *report spam* on every message, but you only meet *not spam* if you go looking in a folder most people never open. The system hears about one kind of error far more often than the other.

    **Choosing differently.** A weekly "here is what we filtered, did we get it right?" would surface false positives. Showing *why* a message was filtered — "unknown sender, several links" — would let you judge the rule. Each is a design decision about which error the system gets to learn from.

!!! critical "Critical Lens — Readable is not the same as accountable"
    Trees are popular in high-stakes settings because a rule can be printed and shown to a regulator. That is a real virtue. It is also how bad rules get waved through: a printout looks like transparency, and a long one is rarely read past the first page.

    The Netherlands has its own case of what happens next. In the childcare-benefits scandal, the *toeslagenaffaire*, the Dutch tax authority's risk-classification model used Dutch nationality — yes or no — as one of its indicators for flagging applications, and tens of thousands of families were wrongly pursued for fraud. In 2021 the Dutch Data Protection Authority fined the tax administration for processing nationality in a discriminatory way, and the government resigned over the wider scandal. The lesson for this chapter is narrow and sharp: the problem was not that nobody could read the rules. The rules could be read. Nobody with the power to stop them asked what nationality was standing in for, or who would carry the false positives.

## How would we know this is any good?

Accuracy is the obvious answer and usually the wrong one. If 89% of email is not spam, a model that says "never spam" scores 89%. So ask three things instead.

1. **What would doing nothing score?** Always compute the baseline — predict the most common class every time — and compare. A model is only as good as its margin over that.
2. **What does it get wrong, in which direction?** Read the confusion matrix cell by cell, and name the person on the receiving end of each kind of error. Then decide, in writing, which error you would rather make, *before* you look at the score.
3. **Who gets the errors?** Split the false positives and false negatives by any group you can identify. A model that is wrong evenly is a different thing from a model that is wrong about one community.

Chapter 17 gives you the vocabulary for doing this precisely — precision, recall and the trade-off between them. For now, the confusion matrix and a sentence for each cell will carry you a long way.

!!! further "Going Further — Forests, and why we stop at one tree"
    A single tree is readable and somewhat unstable: change a few training rows and the top split can change. The standard fix is a *random forest* — hundreds of trees, each trained on a slightly different sample, voting together. Forests are more accurate and far less readable: you cannot follow five hundred trees with your finger. They sit just outside this part of the book on purpose. MLU-Explain's [Decision Trees](https://mlu-explain.github.io/decision-tree/) and [Random Forest](https://mlu-explain.github.io/random-forest/) pieces show both, interactively, and their [bias–variance](https://mlu-explain.github.io/bias-variance/) article is the best visual explanation of overfitting I know.

A tree judges by rules. Next week you meet the other way to classify: no rules at all, just looking at which examples you resemble and letting them vote.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp.

    - [Machine Learning with Tree-Based Models in Python](https://app.datacamp.com/learn/courses/machine-learning-with-tree-based-models-in-python) — **Ch. 1 Classification and Regression Trees.**
    - [Machine Learning with Tree-Based Models in Python](https://app.datacamp.com/learn/courses/machine-learning-with-tree-based-models-in-python) — **Ch. 2 The Bias-Variance Tradeoff** *(optional — why a deeper tree is not a better tree)*.

    **On your own** *(optional)*:

    - [Tutorial: Decision Tree Classification in Python](https://www.datacamp.com/tutorial/decision-tree-classification-python) — build, draw and prune a tree.
    - [Project: Predicting Movie Rental Durations](https://www.datacamp.com/projects/1796) — about an hour; trees predict numbers too, and you compare models.
