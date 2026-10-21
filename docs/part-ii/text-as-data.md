<p class="chapter-kicker">Part II · Chapter 17 · <span class="week">Week 6</span></p>

# Text as data, and how models are judged

!!! guiding "The question this chapter answers"
    How does a machine read text — and when it labels what you wrote, what did it cost to get that wrong?

Most of what machine learning systems act on is text: reviews, posts, messages, support tickets, search queries. A model cannot do anything with raw words. Every technique in this part works on numbers, so first language has to become numbers, and every step of that translation throws something away.

This chapter is the synthesis of Part II. Text becomes vectors, as in Chapter 16. The vectors are classified by their neighbours, as in Chapter 14, with similarity measured by cosine, as in Chapter 16. And then the result is judged properly — because text classification is where the cost of being wrong becomes impossible to ignore. A moderation system that is 95% accurate is wrongly removing, or wrongly leaving up, one post in twenty.

!!! matters "Why this matters for designers"
    Text is where a huge share of consequential machine learning lives: moderation, sentiment, triage, search, autocomplete, the AI summaries above product reviews. Every step of turning text into numbers is a design decision with a victim — whose dialect gets treated as noise, whose complaint gets filed as low priority, whose post gets flagged. And when a system labels a person's words, being wrong is not a metric. It is an experience, and you are the one designing what that experience looks like.

!!! newsroom "Why this matters for journalists"
    Newsrooms increasingly use text classifiers themselves: to sort reader comments, to search leaked documents, to tag archives, to flag likely misinformation. Each one is making the two kinds of error this chapter is about, and the choice between them is editorial. A comment filter tuned to catch every insult will silence some legitimate anger; one tuned to never silence anyone will let insults through. Somebody chose. When you report on a platform's moderation, *which error did they choose to make more of?* is usually the story.

!!! outcomes "By the end of this chapter you can"
    - Turn raw text into vectors with bag-of-words and TF-IDF, and explain what the numbers mean.
    - Explain what each step of that translation removes, and from whom.
    - Train a text classifier end to end, and explain one prediction by its nearest examples.
    - Read a confusion matrix as consequences, not counts.
    - Choose between precision and recall by what each error costs, and move the threshold on purpose.
    - Catch a result that looks good and is not: imbalance, leakage, a cherry-picked metric.

!!! origins "Origins — Karen Spärck Jones and the rare word"
    In 1972 the British computer scientist Karen Spärck Jones published a short paper, *A Statistical Interpretation of Term Specificity and Its Application in Retrieval*. Its idea was simple: a word that appears in nearly every document tells you almost nothing about any of them, while a word that appears in only a few is a strong clue to what those few are about. So weight each word by how *rare* it is across the collection. That weighting, now called inverse document frequency, became half of TF-IDF and is still inside search engines fifty years later. She spent her career at Cambridge and is also remembered for a line about her field that applies well beyond it: *computing is too important to be left to men*.

## The teardown first

This chapter's teardown sets up the problem, so it comes before the code.

!!! case "Teardown — AI-written review summaries"
    Open a large webshop and find a product with hundreds of reviews. Many shops now show a generated summary above them: *"Customers say the fit runs small and the fabric is soft."*

    **Predicting.** Several things at once: which themes appear across the reviews, and whether each is positive or negative.

    **Collecting.** The text of every review, the star ratings, and your behaviour on the page — did you scroll past the summary to read the reviews yourself?

    **Teaching.** Very little, visibly. There is usually no way to say "this summary is wrong".

    **Choosing differently.** Text goes in, a confident claim comes out, and you have no way to check it. How many reviews actually said the fit runs small? Which ones? Were they all from one week, or one size? What would an interface have to show for that claim to be checkable?

    Hold on to that. By the end of the chapter you will have built the kind of classifier that sits under "customers say the fit runs small", and you will know exactly what it gets wrong.

## Text into numbers: bag-of-words

The simplest way to turn text into a vector is to count words. Take every word that appears anywhere in your collection; that list is the **vocabulary**. Each document becomes a row with one column per vocabulary word, holding how many times that word appears in it.

```python
import pandas as pd
from sklearn.feature_extraction.text import CountVectorizer

docs = ["Runs small, lovely colour.",
        "Lovely fabric, but it runs small. Really small."]
counter = CountVectorizer()
counts = counter.fit_transform(docs)
pd.DataFrame(counts.toarray(), columns=counter.get_feature_names_out(),
             index=["review 1", "review 2"])
#           but  colour  fabric  it  lovely  really  runs  small
# review 1    0       1       0   0       1       0     1      1
# review 2    1       0       1   1       1       1     1      2
```

**How it works.** `CountVectorizer` lowercases the text, splits it into words, builds the vocabulary from all the documents it is fitted on, and counts. The result is called a **bag of words**, because that is what it is: the words tipped into a bag, with their order thrown away. Punctuation is gone. Capitals are gone. *small* appears twice in review 2, so it scores 2.

Look at what was lost. "Runs small" and "small runs" are now the same. "Not lovely" would count as *lovely*. Sarcasm, emphasis, the difference between *it* and *It* in a sentence about a brand called It — gone. Every one of those losses is usually fine and occasionally decisive. The point is not that bag-of-words is bad. It is that **turning text into numbers is a series of decisions about what does not matter**, and those decisions are made long before any model sees the data.

## Weighting the rare word: TF-IDF

Counts have a problem Spärck Jones identified. Common words — *the*, *it*, *but*, *lovely* in a collection of reviews — score highly everywhere and distinguish nothing. **TF-IDF** fixes it by multiplying each count (the *term frequency*) by a weight that is high for rare words and low for common ones (the *inverse document frequency*). Words that are frequent in this document *and* rare in the collection end up with the highest scores. Those are the words that say what a document is about.

You will see it in action below. For now, the one thing to remember is what the numbers mean: **a high TF-IDF score means "this word is unusually important to this document, compared with the rest".**

## A sizing detector

Back to the webshop. Here are 1,200 short clothing reviews. About one in six mentions a sizing problem — *runs small*, *order a size up*, *sleeves are too short* — and the shop wants to detect them, so it can say *"customers say the fit runs small"*. A small number of sizing reviews are written differently: in Dutch (*valt klein*, *te klein*) or in internet shorthand (*fits smol lol*).

```python
import numpy as np

rng = np.random.default_rng(21)
sizing = ["runs small", "order a size up", "too tight across the shoulders",
          "fit is much smaller than expected", "had to return it for a bigger size",
          "sleeves are too short", "size chart is wrong"]
sizing_other = ["te klein", "valt klein", "fits smol lol"]
praise = ["lovely colour", "great fabric", "arrived quickly", "really soft",
          "looks just like the photo", "good value", "my new favourite jumper", "washes well"]
gripe = ["took ages to arrive", "colour faded after one wash", "packaging was ripped",
         "the zip broke", "smells of plastic", "customer service never replied"]

def review(is_sizing, other_phrasing=False):
    parts = list(rng.choice(praise + gripe, rng.integers(1, 3), replace=False))
    if is_sizing:
        parts.append(rng.choice(sizing_other if other_phrasing else sizing))
    rng.shuffle(parts)
    return ", ".join(parts).capitalize() + "."

rows = []
for i in range(1200):
    s = rng.random() < 0.15
    o = s and rng.random() < 0.15
    rows.append({"text": review(s, o), "sizing": int(s), "other_phrasing": int(o)})
reviews = pd.DataFrame(rows)
reviews["sizing"].mean()            # 0.164
```

**How it works.** Each review is one to two random phrases of praise or complaint, plus — for sizing reviews — one sizing phrase, shuffled into a sentence. Of the sizing reviews, about 15% use the Dutch or shorthand phrasings. Real reviews are far messier than this; the simplification keeps the mechanics visible.

Now the synthesis. TF-IDF turns each review into a vector. A KNN classifier from Chapter 14 labels a new review by its five nearest training reviews, and "nearest" is measured by cosine similarity from Chapter 16 — because, as with the listeners, a long review and a short one about the same thing should count as close.

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import train_test_split
from sklearn.pipeline import make_pipeline

X_train, X_test, y_train, y_test, other_train, other_test = train_test_split(
    reviews["text"], reviews["sizing"], reviews["other_phrasing"],
    test_size=0.3, random_state=0, stratify=reviews["sizing"])

model = make_pipeline(TfidfVectorizer(),
                      KNeighborsClassifier(n_neighbors=5, metric="cosine"))
model.fit(X_train, y_train)
```

**How it works.** As in Chapter 14, the pipeline means the vectoriser learns its vocabulary and its rarity weights from the training reviews only. Test reviews are vectorised with that same vocabulary; a word that never appeared in training simply does not exist for the model.

Look inside one vector:

```python
tfidf = model.named_steps["tfidfvectorizer"]
vector = tfidf.transform(["Really soft, runs small."]).toarray()[0]
words = tfidf.get_feature_names_out()
{words[i]: round(vector[i], 2) for i in np.nonzero(vector)[0]}
# {'really': 0.4, 'runs': 0.58, 'small': 0.58, 'soft': 0.4}
```

**How it works.** Four words, four weights. *runs* and *small* score higher than *really* and *soft*, because "really soft" appears in many reviews and "runs small" in fewer. The rare words carry the meaning. That is Spärck Jones's idea, doing its job.

## Judging it: accuracy first, then properly

```python
from sklearn.metrics import accuracy_score, confusion_matrix, precision_score, recall_score

pred = model.predict(X_test)
accuracy_score(y_test, pred)                  # 0.975
1 - y_test.mean()                             # 0.836   a "detector" that never says sizing
confusion_matrix(y_test, pred)
# [[301   0]       not sizing: 301 correct, 0 wrongly flagged
#  [  9  50]]      sizing:      9 missed,   50 caught
```

97.5% accuracy, against 83.6% for doing nothing. That looks excellent, and Chapter 13 taught you not to stop there. Read the matrix. It made no false positives at all: every review it flagged really was about sizing. And it missed nine sizing reviews out of 59.

Two numbers summarise those two facts, and you will use them for the rest of your career.

- **Precision**: of everything the model flagged, how much was right? Here, 50 of 50: **1.0**. *When it says "sizing", believe it.*
- **Recall**: of everything it *should* have flagged, how much did it find? Here, 50 of 59: **0.85**. *It finds most sizing complaints, not all.*

```python
precision_score(y_test, pred), recall_score(y_test, pred)    # (1.0, 0.85)
```

These are not two scores for one model. They are **two different promises to two different people**. Precision is a promise to the people who *see* the flag: when we say it, it is true. Recall is a promise to the people who *wrote* the reviews: if you said it, we heard you. You can almost never keep both perfectly, and the next section shows why.

## Moving the threshold

KNN with five neighbours gives each review a score: the share of its five nearest reviews that were sizing complaints — 0, 0.2, 0.4 and so on up to 1. By default it flags a review if the score is above one half, so three or more of five. That cut-off is a **threshold**, and you are allowed to move it.

```python
proba = model.predict_proba(X_test)[:, 1]
for threshold in [0.2, 0.4, 0.6, 0.8]:
    flagged = (proba >= threshold).astype(int)
    print(threshold, round(precision_score(y_test, flagged), 2), round(recall_score(y_test, flagged), 2),
          confusion_matrix(y_test, flagged).ravel())
# threshold  precision  recall   [TN  FP  FN  TP]
# 0.2        0.97       0.98     [299   2   1  58]
# 0.4        1.0        0.93     [301   0   4  55]
# 0.6        1.0        0.85     [301   0   9  50]
# 0.8        1.0        0.81     [301   0  11  48]
```

**How it works.** At a low threshold — flag a review if even one of its five neighbours was about sizing — the model catches 58 of 59 sizing complaints, at the cost of two false alarms. At a high threshold it never raises a false alarm and misses eleven. Lowering the threshold trades precision for recall; raising it trades recall for precision. This is the single most important dial in classification, and it is set by a person, not learned by the model.

So which threshold? There is no technical answer. It depends on what each error costs.

- If the flag drives a public summary — *"customers say the fit runs small"* — a false positive puts a false claim on the product page. You want precision.
- If the flag routes reviews to a person who checks sizing problems and fixes the size chart, a missed complaint is a customer nobody listened to, and a false alarm costs a minute of someone's time. You want recall.

Same model, same data, opposite settings. **Choose the threshold by writing down what each error costs and who pays it, before you look at the scores.** If you look first, you will choose the threshold that makes the model look best, and that is a different question.

## Who did it not hear?

Recall was 0.85 overall. Split it.

```python
standard = (y_test.values == 1) & (other_test.values == 0)
other    = other_test.values == 1
pred[standard].mean(), standard.sum()      # (0.9, 49)
pred[other].mean(), other.sum()            # (0.6, 10)
```

**How it works.** For sizing reviews written in standard English phrasing, the model finds 90%. For the ones in Dutch or shorthand, 60% — six out of ten. Ten reviews is a small group, so treat the exact figure with care. The mechanism is not in doubt: *valt klein* shares no words with *runs small*, so in TF-IDF space the two are unrelated. The Dutch review's nearest neighbours are whatever reviews share its *other* words. Look:

```python
knn = model.named_steps["kneighborsclassifier"]
_, idx = knn.kneighbors(tfidf.transform(["Valt klein, great fabric."]))
list(zip(X_train.iloc[idx[0]], y_train.iloc[idx[0]]))
# [('Valt klein, smells of plastic.', 1),
#  ('Great fabric.', 0), ('Great fabric.', 0), ('Great fabric.', 0), ('Great fabric.', 0)]
```

One neighbour is another Dutch sizing review. The other four are reviews that also said *great fabric*. They outvote it, and the complaint disappears.

This is what "every step of turning text into numbers has a victim" means in practice. Nobody decided to ignore Dutch speakers. The vocabulary was built from the reviews that were there, most of them were in standard English, and the model learned that language. **A text model hears best the people who write like its training data.** The summary at the top of the product page will say "customers say" — and mean the customers who wrote like most customers.

## Results that look good and are not

Text classification is where AI assistants and tutorials most often hand you a convincing number that does not mean what it seems to. Three traps to check for every time.

1. **Imbalance.** If 95% of comments are fine, 95% accuracy means nothing. Report the baseline, and report precision and recall for the class you actually care about.
2. **Leakage.** If a word in the text gives away the label for reasons that will not hold in the real world, the model will find it. A spam dataset where every spam email was collected in one month will teach the model that month's news. A support-ticket dataset where urgent tickets were all written by one team will teach it that team's sign-off. Look at the words with the highest weights and ask whether each one *should* predict the label.
3. **The cherry-picked metric.** If someone reports only precision, ask for recall. If they report only recall, ask for precision. If they report one threshold, ask what happens at others. Any single number can be made to look good by choosing the right threshold after the fact.

!!! critical "Critical Lens — Being wrong about someone's words"
    When a classifier labels a product, a wrong label is an inconvenience. When it labels a *person's words* — as hateful, as spam, as misinformation, as low priority — a wrong label is something that happens to them. Their post disappears. Their complaint goes unanswered. Their account is restricted.

    Those errors do not fall evenly. Research on moderation systems has repeatedly found that text in some dialects is flagged as offensive more often than the same sentiment in others, for exactly the reason the Dutch review was missed above: the model learned what "normal" text looks like from whichever text dominated its training data. People who write differently are, by construction, further from normal.

    The design response is not only a better model. It is an interface that treats a label as a claim, not a verdict: show the person what was flagged and why, let them appeal, and route appeals to a human who can read the dialect. A system that cannot be told it is wrong will keep being wrong in the same direction.

## How would we know this is any good?

This chapter the question *is* the chapter. To summarise the method:

1. **Compute the baseline** and say how far above it the model is.
2. **Read the confusion matrix as people**: who is in each cell, and what happened to them.
3. **Choose precision or recall in writing, before you look at scores**, by what each error costs and who pays.
4. **Move the threshold on purpose** to the point that matches that choice, and report the result at that threshold.
5. **Split the errors by group** — language, region, length, anything you can identify — and say where the model hears worst.
6. **Check for leakage** by reading the words the model relies on most.

Precision and recall are not two numbers. They are two promises to two different people — the one whose words were wrongly flagged, and the one whose words were never heard. Pick which promise you are making before the scores tempt you.

!!! further "Going Further — Beyond bags of words"
    Bag-of-words and TF-IDF cannot tell that *valt klein* and *runs small* mean the same thing, because they share no words. Embeddings, which you met at the end of Chapter 16, can: they place phrases with similar meanings near each other even when the words differ, including across languages. That is why modern search and moderation systems mostly use them — and why they are harder to read, since you can no longer list the words a decision rested on. MLU-Explain's [Precision & Recall](https://mlu-explain.github.io/precision-recall/) and [ROC & AUC](https://mlu-explain.github.io/roc-auc/) pieces show the threshold trade-off interactively, and their [Equality of Odds](https://mlu-explain.github.io/equality-of-odds/) article picks up where this chapter's critical lens leaves off.

You now have every technique this part teaches and a way to judge any of them. What you do not yet have is a way for a person to *use* one. The last chapter is about the part of a machine learning system that designers are actually responsible for: the interface around the model.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. This is the heaviest week of practice in the part, so start early: one chapter cleans text, one turns it into numbers, and one judges the result.

    - [Introduction to Natural Language Processing in Python](https://app.datacamp.com/learn/courses/natural-language-processing-fundamentals-in-python) — **Ch. 1 Regular Expressions & Word Tokenization.**
    - [Introduction to Natural Language Processing in Python](https://app.datacamp.com/learn/courses/natural-language-processing-fundamentals-in-python) — **Ch. 2 Simple Topic Identification** *(bag-of-words and TF-IDF live here)*.
    - [Supervised Learning with scikit-learn](https://app.datacamp.com/learn/courses/supervised-learning-with-scikit-learn) — **Ch. 3 Fine-Tuning Your Model** *(precision, recall and ROC)*.

    **On your own** *(optional)*:

    - [Project: Reveal Categories Found in Data](https://www.datacamp.com/projects/2472) — about an hour; cluster negative app-store reviews into themes.
    - [Tutorial: Sentiment Analysis in Python](https://www.datacamp.com/tutorial/simplifying-sentiment-analysis-python) — it stops at accuracy; add the confusion matrix yourself.
