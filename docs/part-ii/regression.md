<p class="chapter-kicker">Part II · Chapter 12 · <span class="week">Week 1</span></p>

# Learning from data: correlation and regression

!!! guiding "The question this chapter answers"
    How does a model learn to predict a number — and how do I read what it learned?

The last chapter was about where examples come from. This one is about what a model does with them. We start with the oldest learning machine in the field and still one of the most useful: a straight line, fitted through a cloud of points, that predicts a number. It is called **linear regression**, and its great virtue for this part of the book is that you can read it. Every number it learns has a meaning you can say out loud.

You have met half of it already. In Chapter 7 you measured **correlation**: whether two columns move together. Regression takes that one step further. It does not just say that two things move together. It says by *how much*, and it uses that to make a guess.

!!! matters "Why this matters for designers"
    A predicted number usually ends up on a screen as a promise: "ready in 25 minutes", "arrives Thursday", "estimated price €340". Whoever designs that screen decides how confident the promise sounds. Regression is the cleanest place to learn what such a number is made of, how wrong it usually is, and — the part that matters most — whether it is wrong in the same way for everyone. A delivery estimate that is reliably late for one neighbourhood is a design problem, not just a modelling one.

!!! newsroom "Why this matters for journalists"
    Regression is how researchers say "controlling for". When a study reports that one thing is associated with another *after controlling for income and age*, there is a regression underneath, and its coefficients are the sentence you are about to write. Knowing how to read one tells you what the finding actually claims, what it leaves out, and when "is associated with" has been quietly upgraded to "causes" somewhere between the paper and the press release.

!!! outcomes "By the end of this chapter you can"
    - Measure and read a correlation, and say why it is not causation.
    - Explain what a regression line is: the line that makes the squared errors as small as possible.
    - Fit a linear regression with scikit-learn and read its intercept and coefficients in plain words.
    - Compute residuals and use them to ask where, and for whom, a model is wrong.
    - Read mean absolute error and R² without over-reading either.
    - Explain why adding a feature can change the other coefficients.

!!! origins "Origins — Galton, sweet peas and the word “regression”"
    The word comes from Francis Galton, Charles Darwin's half-cousin. In the 1870s and 1880s Galton measured sweet peas and then people, and noticed that very tall parents tended to have children who were tall but less extreme, and very short parents had children who were short but less so. He called it *regression towards mediocrity* (1886). We now call it regression to the mean, and the method he and Karl Pearson developed to describe it became the regression line. The arithmetic underneath, least squares, was older still: Legendre published it in 1805 and Gauss claimed he had used it for years.

    Galton also founded eugenics, and Pearson was one of its most prominent champions. The tool is not tainted by that history, but it is worth knowing that one of the first things people did with a method for predicting traits of children from their parents was to argue about which parents should have children. Methods arrive carrying the questions their makers wanted answered.

## Correlation, first

Start with something you will want to predict. Recipe websites promise a cooking time. Anyone who has cooked from one knows the promise. Here is a set of 300 recipes, each with the number of steps, the number of ingredients, whether it goes in the oven, the time the recipe *claims*, and the time it *actually* took someone to make it.

```python
import numpy as np
import pandas as pd

rng = np.random.default_rng(12)
n = 300
recipes = pd.DataFrame({
    "steps":       rng.integers(3, 15, n),
    "ingredients": rng.integers(4, 18, n),
    "baked":       rng.choice([0, 1], n, p=[0.7, 0.3]),     # 1 = goes in the oven
})
recipes["claimed_min"] = (5 + 2.5 * recipes["steps"] + 1.0 * recipes["ingredients"]
                          + 15 * recipes["baked"] + rng.normal(0, 4, n)).round()
recipes["actual_min"] = (8 + 4.0 * recipes["steps"] + 1.2 * recipes["ingredients"]
                         + 30 * recipes["baked"] + rng.normal(0, 6, n)).round()

recipes[["steps", "ingredients", "actual_min"]].corr().round(2)
#              steps  ingredients  actual_min
# steps         1.00        -0.03        0.70
# ingredients  -0.03         1.00        0.18
# actual_min    0.70         0.18        1.00
```

**How it works.** The data is synthetic, so we know the truth: we built it so that each step really costs about four minutes and the oven adds half an hour. `.corr()` gives the Pearson correlation between each pair of columns, a number between −1 and 1. Steps and actual time correlate at 0.70: recipes with more steps take longer, fairly reliably. Ingredients correlate at only 0.18. Steps and ingredients barely correlate with each other at all (−0.03), which will matter later.

A correlation tells you two columns move together and how tightly. It does not tell you how *much* one moves when the other does. A correlation of 0.70 could mean each step costs one minute or ten. For that you need a line.

Everything Chapter 7 said about correlation still holds, and it holds harder now that a model is involved. Correlation is not causation. Two columns can move together because one causes the other, because the other causes the one, because a third thing drives both, or by chance at your sample size. A regression does not fix this. It builds on it. If the correlation is a coincidence, the regression is a very precise description of a coincidence.

## Fitting a line

Picture the recipes as dots: steps along the bottom, actual time up the side. A regression line is the straight line through that cloud that is, on average, closest to the dots. "Closest" has a specific meaning. For every dot, measure the vertical gap between the dot and the line. Square each gap, so that misses above and below both count and big misses count a lot. Add them up. The regression line is the line that makes that sum as small as it can be. That is *least squares*, and it is what scikit-learn's `LinearRegression` computes.

```python
from sklearn.linear_model import LinearRegression

X = recipes[["steps", "ingredients", "baked"]]    # the features
y = recipes["actual_min"]                         # the target

model = LinearRegression()
model.fit(X, y)                                   # (1)!

model.intercept_                                  # 6.9
dict(zip(X.columns, model.coef_.round(1)))        # {'steps': 4.0, 'ingredients': 1.3, 'baked': 29.9}
```

1.  `fit` is where the learning happens. Everything before it is you deciding what the model is allowed to see.

**How it works.** The pattern — create a model, call `.fit(X, y)`, then use it — is the same for almost every model in scikit-learn, and you will type it in every chapter of this part. `X` is a table of features, `y` is the column to predict. After fitting, the model stores what it learned. For a linear regression that is one number per feature, the **coefficients**, plus a starting value, the **intercept**.

## Reading what it learned

This is the moment that makes regression worth starting with. The model's entire reasoning fits in one sentence, and you can say it out loud:

> *Actual time ≈ 6.9 minutes, plus 4.0 minutes per step, plus 1.3 minutes per ingredient, plus 29.9 minutes if it goes in the oven.*

Each coefficient reads the same way: **holding the other features fixed**, one more unit of this feature adds this many minutes. One more step, four more minutes. Baked rather than not, half an hour more. The intercept is the prediction for a recipe with zero of everything, which is not a real recipe; treat it as the line's starting height rather than something to interpret.

Because we built the data, we can check: the true costs were 4.0, 1.2 and 30. The model recovered them almost exactly from nothing but examples. That is learning, in the sense this part of the book means it.

Now compare with what the recipes claim. Average the ratio of actual to claimed time and you find recipes in this set take about 55% longer than they say. The model has learned something the recipe authors were not telling you: they price each step at two and a half minutes, and it takes four.

```python
recipes["predicted"] = model.predict(X)
recipes["residual"]  = recipes["actual_min"] - recipes["predicted"]

recipes["residual"].abs().mean()     # 5.0   mean absolute error, in minutes
model.score(X, y)                    # 0.92  R²
```

**How it works.** `.predict(X)` runs the fitted line on every row. The **residual** is what actually happened minus what the model said: positive means the model guessed too low, negative too high. The average size of a residual, ignoring its sign, is the **mean absolute error (MAE)**. It is in the units of the target — five minutes here — which makes it the easiest error measure to explain to anybody. `.score()` returns **R²**, the share of the variation in actual time that the model accounts for. Here 0.92: the model explains most of why some recipes take longer than others.

Be careful with R². A high one feels like a grade, and it is not. It says the line fits the data it was fitted on. It does not say the line will work on recipes it has never seen; we have not tested that yet, and Chapter 14 is where that test becomes non-negotiable. It does not say the features *cause* the time. And it can be high while the model is badly wrong for a particular kind of row, which is the next section.

## Wrong for whom?

A model can be good on average and bad for a group. Averages hide that, so look at the residuals by group.

Suppose we had fitted the model without the `baked` column. Perhaps nobody thought to record it, or the recipe site does not store it.

```python
simple = LinearRegression().fit(recipes[["steps", "ingredients"]], y)
recipes["residual_simple"] = y - simple.predict(recipes[["steps", "ingredients"]])

simple.score(recipes[["steps", "ingredients"]], y)          # 0.53
recipes.groupby("baked")["residual_simple"].mean().round(1)
# baked
# 0    -8.8     not baked: the model guesses about 9 minutes too long
# 1    20.6     baked: the model guesses about 21 minutes too short
```

**How it works.** Without `baked`, the model has to draw one line for all recipes. It splits the difference: every stovetop recipe is overestimated by about nine minutes and every oven recipe underestimated by about twenty. The overall R² drops to 0.53, which tells you something got worse. The grouped residuals tell you *what*, and for *whom*. Put the oven back in and both groups' average residual returns to zero.

That pattern generalises far beyond recipes, and it is the most important evaluation habit in this chapter. A model that is wrong *evenly* is imprecise. A model that is wrong *in a pattern* has learned something false about a group, and the group is often one that was missing from the features. If a delivery-time model does not know which streets have no lift, it will be late for everyone on the fourth floor, every time, and the average error will look fine.

The fix in the recipe case was a column. Often it is not that easy. The missing feature may not be collectable, or may be one you should not use. Then the honest move is to know where your model is wrong and design for it: widen the estimate, say "usually" rather than a single number, or tell people which recipes the estimate is least sure about.

## Adding a feature changes the others

One more property is worth knowing, because it is the source of endless misreadings. A coefficient means "holding the other features fixed", so it depends on what the other features are.

In this data, steps and ingredients are nearly uncorrelated, so adding or removing one barely moves the other's coefficient. In real data, features usually do correlate. Recipes with more steps tend to have more ingredients. Houses with more rooms tend to be larger. When two features carry overlapping information, the model has to decide how to share the credit between them, and the split can shift dramatically when you add a third. A coefficient can even change sign.

So never read one coefficient as *the* effect of a feature. Read it as the effect of that feature *in this model, given these other features*. That is exactly what researchers mean by "controlling for", and exactly the part that gets dropped when a finding travels from a paper to a headline.

!!! case "Teardown — the delivery estimate"
    Open any food-delivery or parcel app and look at the time estimate. Use the four questions from Chapter 11.

    **Predicting.** A number: minutes until arrival. Usually shown as a single time, sometimes as a range.

    **Collecting.** When you ordered, from where, to where, the courier's position, how long the restaurant took last time. And your behaviour: do you cancel when the estimate is long?

    **Teaching.** Every completed delivery is a labelled example: predicted time, actual time. Cancellations are interesting — an order cancelled because the estimate looked too long never produces an actual time, so the model never learns how long it would have taken.

    **Choosing differently.** A range instead of a single time. A range that is honest about being wider for some addresses. An apology that arrives *before* the estimate is missed, not after. The model's residuals already know which deliveries run late. The interface decides whether you get to.

!!! critical "Critical Lens — A line is an opinion about the world"
    Linear regression assumes that each feature adds a fixed amount, the same for everyone, however much of it there already is. One more step always costs four minutes, whether it is the third step or the thirteenth. That is a strong claim and often a false one. Sometimes effects grow, sometimes they flatten, and sometimes a feature matters only in combination with another.

    The model will not tell you when this assumption is wrong. It will draw the best straight line it can and report its coefficients with the same confidence either way. The residuals are where you find out: if they curve, fan out, or cluster by group, the world is not as straight as the model assumed. Reading residuals is not a technical nicety. It is how you check whether the model's opinion about the world matches the world.

## How would we know this is any good?

Look at the residuals: the gaps between what the model said and what actually happened. Three questions, in this order.

1. **How big are they, typically?** MAE, in the units people care about. "Usually within five minutes" is something you can design a screen around.
2. **Are they a pattern or noise?** Plot residuals against each feature and against the prediction. Noise looks like a shapeless cloud around zero. A curve, a funnel or a group sitting above the line is the model telling you it has misunderstood something.
3. **Who is standing in the pattern?** Group the residuals by anything you know about the rows, including things that are *not* features. A model that is wrong evenly is a different thing from a model that is wrong about one group.

And one question this chapter cannot answer yet: would it work on recipes it has never seen? Every number above was measured on the training data. That is like marking an exam using the answer sheet the student studied from. Chapter 14 fixes it.

!!! further "Going Further — From lines to probabilities"
    Linear regression predicts a number. Its close cousin *logistic regression* predicts a probability — the chance that something belongs to a category — and is one of the most widely used classifiers there is, partly because its coefficients are still readable. It is not one of the six techniques in this part, but if a project needs a readable classifier and a decision tree feels too blunt, it is the next thing to teach yourself. [MLU-Explain](https://mlu-explain.github.io/logistic-regression/) has an interactive introduction, and their [linear regression](https://mlu-explain.github.io/linear-regression/) piece is a good companion to this chapter.

You now have a model that predicts a number and tells you exactly how it got there. Next week the target stops being a number and becomes a category — spam or not — and the model becomes a flowchart it wrote itself.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp.

    - [Supervised Learning with scikit-learn](https://app.datacamp.com/learn/courses/supervised-learning-with-scikit-learn) — **Ch. 2 Regression.**

    **On your own** *(optional)*:

    - [Tutorial: Sklearn Linear Regression](https://www.datacamp.com/tutorial/sklearn-linear-regression) — the same steps on house prices, ending in a residual plot.
