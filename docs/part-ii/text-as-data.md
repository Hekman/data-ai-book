<p class="chapter-kicker">Part II · Chapter 17 · <span class="week">Week 6</span></p>

# Text as data, and how models are judged

!!! guiding "The question this chapter answers"
    How does a machine read text — and when it labels what you wrote, what did it cost to get that wrong?

!!! note "🚧 Work in progress"
    This chapter is still being written. The teaching week it belongs to is planned; the prose isn't here yet. It lands before the week it is needed.

**What it will cover.** Turning language into numbers with bag-of-words and TF-IDF, and what every step of that translation throws away, and from whom. Then the synthesis: text becomes vectors, vectors get classified, and the result is judged properly — confusion matrices read as consequences rather than counts, precision, recall and F1 chosen by what being wrong costs, and the results that look good and are not (imbalance, leakage, a cherry-picked metric).

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. This is the heaviest practice load in Part II — start early. Ch. 1 is the cleaning, Ch. 2 is the turning-into-numbers, and the last one is how you judge the result.

    - [Introduction to Natural Language Processing in Python](https://app.datacamp.com/learn/courses/natural-language-processing-fundamentals-in-python) — **Ch. 1 Regular Expressions & Word Tokenization** and **Ch. 2 Simple Topic Identification** (bag-of-words and TF-IDF live here).
    - [Supervised Learning with scikit-learn](https://app.datacamp.com/learn/courses/supervised-learning-with-scikit-learn) — **Ch. 3 Fine-Tuning Your Model.** Precision, recall, F1 and ROC.
