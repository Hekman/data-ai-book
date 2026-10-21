<p class="chapter-kicker">Part II · Chapter 16 · <span class="week">Week 5</span></p>

# Similarity and recommendation: cosine similarity

!!! guiding "The question this chapter answers"
    How does a system decide two things are alike — and what does "more like this" do to the person receiving it?

This part of the book has been circling one idea for three chapters. KNN let neighbours vote. k-means grouped things around a centre. Both measured similarity as *distance*: how far apart two rows sit. This chapter introduces a different measure, **cosine similarity**, which asks not how far apart two things are but whether they *point the same way*. It turns out to be the right question for comparing tastes, documents and profiles, and it is the measure inside a very large share of the recommenders you use.

By the end of the chapter you will have built one: vectors in, nearest neighbours out, a list of suggestions you can argue with. And because it is the last new technique before Chapter 17 brings everything together, you will have met every tool this part teaches.

!!! matters "Why this matters for designers"
    "Similar to what you liked" is one of the most common and most invisible patterns online. It feels neutral. It is not: it narrows what you see toward what you have already seen, and "similar" is somebody's choice of vector and measure. A recommender is a feedback loop with a friendly face — what it shows you shapes what you do, which shapes what it shows you next. Designers decide whether the person can see that happening, and whether they can get out.

!!! newsroom "Why this matters for journalists"
    Recommendation is now a large part of how journalism reaches people, through news apps, search and social feeds. "Readers who read this also read" decides what a story sits next to, and an engine that optimises for clicks will happily put careful reporting next to its sensational cousin. Two questions are worth asking any outlet or platform: what is the recommender optimising for, and what does it never recommend? The second is usually more revealing than the first.

!!! outcomes "By the end of this chapter you can"
    - Represent things — people, films, documents — as vectors of numbers.
    - Explain cosine similarity in plain language, and why it is used instead of straight-line distance.
    - Build a small content-based recommender with scikit-learn.
    - Connect this to KNN and clustering as one idea seen three ways.
    - Critique a "more like this" feature: what it amplifies, what it hides, and how it closes a loop.
    - Recognise cosine similarity when you meet it again under another name.

!!! origins "Origins — Salton's vector space, and the Netflix Prize"
    Cosine similarity came into computing through information retrieval. In the 1960s and 1970s Gerard Salton and his group at Cornell built SMART, a search system that represented every document and every query as a vector of word counts and ranked documents by the cosine of the angle between them. The 1975 paper by Salton, Wong and Yang, *A Vector Space Model for Automatic Indexing*, set out the idea that still sits under most search: text becomes geometry, and relevance becomes an angle.

    Recommendation arrived later. Amazon described its "customers who bought this item also bought" approach in 2003, comparing *items* by the people who bought them. In 2006 Netflix offered a million dollars to anyone who could improve its rating predictions by ten percent; a team won in 2009, and Netflix never fully deployed the winning method, partly because by then it was streaming and cared less about predicting star ratings than about what people actually watched. The most famous competition in recommender history ended by discovering that it had been optimising the wrong thing.

## Things as vectors

A **vector** is just a row of numbers. You have been using them all along: every row in a dataframe is one. What changes in this chapter is how we think about them. Treat each number as a direction — one axis per column — and every row becomes an arrow pointing somewhere in that space.

Here are three listeners, described by how many hours they spent on three genres last month.

```python
import pandas as pd
from sklearn.metrics.pairwise import cosine_similarity, euclidean_distances

plays = pd.DataFrame(
    {"jazz": [40, 4, 2], "hiphop": [10, 1, 30], "techno": [0, 0, 25]},
    index=["heavy_jazz_fan", "light_jazz_fan", "club_kid"])

euclidean_distances(plays).round(1)
# [[ 0.  37.1 49.7]
#  [37.1  0.  38.3]
#  [49.7 38.3  0. ]]
```

**How it works.** `euclidean_distances` computes the straight-line distance between every pair of rows — the same distance KNN used. Read the middle row. The light jazz fan is 37.1 from the heavy jazz fan and 38.3 from the club kid: almost equally far from both. By distance, the light jazz fan has no particular taste. They are just a small arrow near the origin, close to everything that is also small.

That is clearly wrong. The light jazz fan listens to exactly the same mix as the heavy jazz fan — four parts jazz to one part hip-hop, no techno — just ten times less of it. What they share is the *direction* of their arrow, not its length.

## Measuring direction

Cosine similarity measures exactly that: the angle between two arrows, ignoring how long they are. Two arrows pointing the same way score 1. Two at right angles, sharing nothing, score 0. With counts, which cannot be negative, it never goes below 0.

```python
cosine_similarity(plays).round(2)
# [[1.   1.   0.24]
#  [1.   1.   0.24]
#  [0.24 0.24 1.  ]]
```

**How it works.** The heavy and light jazz fans now score 1.0 — identical taste — and both score 0.24 against the club kid. Cosine similarity has done what distance could not: separated *how much* someone listens from *what* they listen to.

That separation is why cosine similarity dominates recommendation and search. In most collections, the size of a vector mostly reflects activity or length — a heavy user, a long document — while its direction reflects taste or topic. You almost always want to compare taste and topic. A heavy user and a light user with the same taste should get the same suggestions; a long article and a short one on the same subject should match the same search.

The formula, if you want it: multiply the two vectors position by position, add up, and divide by the lengths of both. You will not need to compute it by hand. You do need to know that it ignores length, because sometimes length is exactly what you care about, and then it is the wrong measure.

## A small recommender

Now the real thing. Eight films, each described by how strongly it has six qualities, scored 0 to 3 by a person who watched them. This is a **content-based** recommender: it recommends by what things *are*, not by who watched them.

```python
films = pd.DataFrame({
    "slow":    [3, 2, 0, 0, 1, 3, 0, 2],
    "funny":   [0, 1, 3, 2, 0, 0, 3, 0],
    "dark":    [2, 0, 0, 1, 3, 3, 0, 1],
    "space":   [0, 0, 0, 3, 3, 0, 1, 0],
    "music":   [1, 3, 1, 0, 0, 0, 2, 3],
    "romance": [1, 2, 2, 0, 0, 1, 1, 2],
}, index=["Harbour Light", "Second Verse", "The Wrong Wedding", "Orbit Club",
          "Cold Signal", "The Long Winter", "Moon Karaoke", "Late Set"])

similarity = pd.DataFrame(cosine_similarity(films), index=films.index, columns=films.index)

def more_like(title, n=3):
    return similarity[title].drop(title).sort_values(ascending=False).head(n).round(2)

more_like("Second Verse")
# Late Set         0.94
# Harbour Light    0.67
# Moon Karaoke     0.67

more_like("Cold Signal")
# Orbit Club         0.74
# The Long Winter    0.63
# Harbour Light      0.53
```

**How it works.** `cosine_similarity(films)` compares every film with every other and returns an 8 × 8 table of scores. `more_like` takes one film's column, drops the film itself (which always scores 1 against itself), and returns the highest-scoring others. *Second Verse*, a slow musical romance, is most like *Late Set*, another one. *Cold Signal*, dark and set in space, is most like *Orbit Club*, the other space film.

That is a working recommender, and the whole of it is readable. Ask why it suggested *Late Set* and you can answer exactly: both are strong on music and romance, mildly slow, not funny, not in space. Compare that with the explanations real products give you, and notice how much more this one can say.

Swap the films for songs and the qualities for genres, or for the listeners who played them, and you have the core of most "more like this" features. Swap them for documents and the qualities for words, and you have search — which is next chapter.

## Three views of one idea

Step back, because this is the moment the last three chapters connect.

| Chapter | Technique | The question it asks |
|---|---|---|
| 14 | k-nearest neighbours | Who is closest to this case, and how did they turn out? |
| 15 | k-means | Which centre is each case closest to? |
| 16 | cosine similarity | Which cases point in the same direction as this one? |

All three rest on the same assumption: **things that are close get treated alike.** All three need someone to decide what "close" means — which features, scaled how, measured with which formula. And all three inherit the same weakness: if you sit in a sparse part of the space, with few others near you, the system knows very little about you and will not say so.

A recommender is KNN without a label. Instead of letting the neighbours vote on a category, it hands you the neighbours themselves.

## You will meet this again

One more connection, because it is the bridge to Part III. The film vectors above have six numbers, chosen by a person: *slow*, *funny*, *dark* and so on. Every one of them is readable.

Modern AI systems use exactly the same geometry, at a different scale. They turn words, sentences, images and songs into vectors of several hundred or several thousand numbers. Nobody chooses those numbers; a model learns them, and no individual number means anything you could name. These vectors are called **embeddings**. Search engines, chatbots and recommender systems compare them with — very often — cosine similarity. When you read that a system "retrieves the most relevant documents", there is a good chance it is doing what `more_like` does above, with arrows you cannot read.

So the idea you learned this week is not a teaching simplification. It is the real thing. What you lose, going from your six columns to a learned embedding, is the ability to say *why* two things were judged similar. That loss is the subject of Part III.

## The loop, with a friendly face

Recommenders are where Chapter 11's loop is most visible. A system recommends what is similar to what you engaged with. You engage with some of it, because it is what you were shown. Your history now contains more of that kind of thing, so the next recommendations are even more similar. Over time the arrow representing you gets longer in the same direction, and the recommendations narrow.

This is often called a *filter bubble*, and the honest position is that its size is debated. Studies disagree about how much recommenders narrow what people consume, and some find that people narrow their own choices more than any algorithm does. What is not debated is the mechanism. A system that only ever shows you what is similar to your past cannot discover that you would have liked something different, for the same reason the music shelf in Chapter 11 could not discover its unplayed songs. Its own recommendations are the only evidence it collects.

The design levers are the same as before. Reserve some slots for things that are deliberately *not* most similar. Show the person why something was recommended, so they can recognise a rut. Give them a way to say "less of this" that actually works. Each of these costs some short-term engagement. Each buys a system that can still learn something new about the person.

!!! case "Teardown — a recommendation surface"
    Open Spotify's *Discover Weekly*, YouTube's up-next column, or the autoplay queue on any streaming service.

    **Predicting.** What you are most likely to engage with next, and for how long.

    **Collecting.** Everything from Chapter 11, plus your history as a vector: what you played, finished, saved, skipped, and what people with similar vectors did.

    **Teaching.** Every recommendation you accept makes your history more like the recommendation. Every one you ignore says very little, because ignoring is ambiguous. Notice how hard it is to say "I liked this but do not want more of it".

    **Choosing differently.** Where does each suggestion come from — similar items, or similar people? What is it optimising: your enjoyment, your time on the app, or the catalogue the service would like you to discover? And what would it take to escape it? Try to make the surface recommend something genuinely unlike your usual taste. How many steps does it take?

!!! critical "Critical Lens — Similarity is somebody's choice"
    In the film example a person chose six qualities and scored each film. That person decided that *dark* is one dimension, that *funny* is another, and that there is no axis for, say, *made outside the United States* or *directed by a woman*. Whatever has no axis cannot influence a recommendation. A system can only find two things similar in ways it was given the vocabulary for.

    Real systems mostly learn their dimensions from behaviour instead, which sounds more neutral and is not. Learned from past viewing, similarity reflects past viewing — including whatever was promoted, what was easy to find, and who was in the audience. If a kind of film was rarely shown, it was rarely watched, so it will be similar to very little, so it will rarely be recommended. The geometry inherits the history.

## How would we know this is any good?

Recommenders are the hardest thing in this part to evaluate, and that is worth sitting with. There is no test set for "did they enjoy it". So ask the awkward questions instead.

1. **Is it better than the obvious?** Compare its suggestions with a simple popularity ranking — the most-played items for everyone. If a popularity list would have done the same job, the "personalisation" is decoration.
2. **Are the suggestions merely obvious?** Recommending the sequel to a film someone just watched is accurate and useless. Good recommendations are often the ones the person would not have found alone.
3. **What does it never recommend?** Run every item through `more_like` and count how often each one appears in anyone's top three. Items that never appear are invisible to the system, however good they are.
4. **What happens over time?** Simulate a user who always accepts the top suggestion, as the music shelf did. Does their history narrow? How fast?

None of these produces a single number you can put on a poster. All of them are what a competent evaluation of a recommender looks like.

!!! further "Going Further — Collaborative filtering"
    This chapter built a *content-based* recommender: items are similar if their descriptions are. The other main family, *collaborative filtering*, ignores descriptions entirely: items are similar if the same people liked them, and people are similar if they liked the same items. Amazon's 2003 approach was of this kind. It needs no hand-labelled qualities, which is why platforms prefer it, and it struggles with anything new that nobody has rated yet — the *cold start* problem. Most production systems mix the two. DataCamp's *Unsupervised Learning in Python*, Chapter 4, ends by recommending musical artists from listening data, which is collaborative filtering with cosine similarity in a few lines.

You have now met every technique this part teaches: a line, a tree, neighbours, clusters and cosine. Next chapter puts them to work on the messiest material there is — language — and asks the evaluation question as seriously as this part ever asks it.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. Chapter 4 finishes by recommending musical artists with cosine similarity, which is, more or less, this chapter.

    - [Unsupervised Learning in Python](https://app.datacamp.com/learn/courses/unsupervised-learning-in-python) — **Ch. 4 Discovering Interpretable Features.**
    - [Unsupervised Learning in Python](https://app.datacamp.com/learn/courses/unsupervised-learning-in-python) — **Ch. 3 Decorrelating Your Data and Dimension Reduction** *(optional — what a vector is, geometrically)*.

    **On your own** *(optional)*:

    - [Tutorial: Recommender Systems in Python](https://www.datacamp.com/tutorial/recommender-systems-python) — a film recommender built with TF-IDF and cosine similarity, which previews Chapter 17.
