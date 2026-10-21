<p class="chapter-kicker">Part II · Chapter 15 · <span class="week">Week 4</span></p>

# Groups without labels: k-means clustering

!!! guiding "The question this chapter answers"
    How do systems group people without labels — and what does it mean to be sorted into a category you never chose?

Every model so far had a teacher. Someone had written down the right answer — the actual cooking time, spam or not, bought or not — and the model learned to reproduce it. This chapter takes the teacher away. **Clustering** looks at unlabelled data and proposes groups: listeners with similar habits, products bought together, neighbourhoods with similar profiles. Learning without answers is called **unsupervised** learning, and k-means is its most common tool.

It is the engine behind audience segments, "your year in review" and many of the neighbourhoods a recommender draws on. It is also, of everything in this part, the easiest to over-believe. With no right answer to check against, a clustering cannot be wrong in the way a classifier can. It can only be more or less useful, more or less stable, and more or less honest about being an arrangement rather than a discovery.

!!! matters "Why this matters for designers"
    Segmentation is one of the most consequential quiet acts in digital products. A cluster becomes a *type*, and a type gets a different price, a different feed, a different level of trust — usually without the person knowing they were sorted, and always without them being asked. Designers decide how clusters are formed, what they are called, and what happens to someone once they are in one. Naming a cluster is an act of authorship.

!!! newsroom "Why this matters for journalists"
    "Our analysis identified five types of voter" is a clustering result, and it is reported as though the five types were found in nature. They were not. Somebody chose the variables, chose the number five, and chose the names. The variables decide what kind of difference counts; the number decides how finely society is sliced; the names decide what readers think each group is. All three deserve the same scrutiny as a poll's question wording.

!!! outcomes "By the end of this chapter you can"
    - Explain what k-means does, step by step, without formulas.
    - Run k-means with scikit-learn on scaled data.
    - Choose *k* with reasoning — the elbow and the silhouette score — rather than by eye.
    - Profile clusters and name them, and say what a name adds that the numbers do not.
    - Test whether a clustering is stable, and recognise one that is arbitrary.
    - Reason about the ethics of segmentation: consent, stereotyping and differential treatment.

!!! origins "Origins — a Bell Labs memo and "you are where you live""
    The algorithm was written down by Stuart Lloyd at Bell Labs in 1957 as a method for compressing signals; the memo was not formally published until 1982. James MacQueen gave it the name *k-means* in 1967. Its commercial life began in marketing. In the 1970s the American firm Claritas built PRIZM, a system that clustered US neighbourhoods by census data into dozens of named segments with names like "Shotguns and Pickups" and "Blue Blood Estates". Its slogan, *you are where you live*, captures both the power and the problem of segmentation in four words: it is often right enough to sell things, and it treats a postcode as a personality.

## How k-means works

The algorithm is short enough to do by hand.

1. **Choose *k***, the number of groups you want. This is your decision, not the algorithm's.
2. **Drop *k* centre points** into the data at random.
3. **Assign** every row to its nearest centre.
4. **Move** each centre to the middle — the *mean* — of the rows assigned to it.
5. **Repeat** steps 3 and 4 until nothing moves.

That is all. Each cluster is the set of rows nearer to its centre than to any other. The *k* is how many centres; the *means* is where they end up. Notice step 3: "nearest" is the same distance as in Chapter 14, with the same problem. Unscaled, the feature with the biggest numbers decides everything. So we scale first.

## Clustering listeners

A streaming service has 700 listeners and four facts about each: hours listened per week, the share of listening done at night, the share that is new releases, and the average tempo of what they play.

```python
import numpy as np
import pandas as pd
from sklearn.preprocessing import StandardScaler

rng = np.random.default_rng(8)
def listeners_like(n, hours, night, new, bpm):
    return pd.DataFrame({
        "hours_week":  rng.normal(hours, 3, n).clip(0),
        "night_share": rng.normal(night, 0.07, n).clip(0, 1),
        "new_release": rng.normal(new, 0.08, n).clip(0, 1),
        "avg_bpm":     rng.normal(bpm, 8, n),
    })

listeners = pd.concat([
    listeners_like(300,  6, 0.15, 0.20,  95),
    listeners_like(250, 18, 0.55, 0.60, 125),
    listeners_like(150, 30, 0.30, 0.35, 105),
], ignore_index=True)

X = StandardScaler().fit_transform(listeners)
```

**How it works.** We built three kinds of listener on purpose, so we can check whether k-means finds them. In real data you would not know this, which is the whole difficulty. The model gets none of the group information — only the four columns, scaled.

## Choosing k

k-means will give you exactly as many groups as you ask for, whether or not there are any. So the first real decision is *k*, and there are two standard ways to reason about it.

```python
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

for k in range(2, 8):
    km = KMeans(n_clusters=k, n_init=10, random_state=0).fit(X)
    print(k, round(km.inertia_), round(silhouette_score(X, km.labels_), 3))
# k   inertia  silhouette
# 2   1078     0.553
# 3    473     0.605
# 4    422     0.475
# 5    377     0.29
# 6    350     0.227
# 7    325     0.224
```

**How it works.** `n_init=10` runs the algorithm ten times from different random starts and keeps the best, because a bad start can trap k-means in a poor answer. **Inertia** is the total squared distance from every listener to their cluster's centre: how tight the clusters are. It always falls as *k* rises — with 700 clusters it would be zero — so you look for the **elbow**, the point where adding a cluster stops buying much. Here inertia halves from *k* = 2 to 3 and then barely moves. The **silhouette score** asks, for each row, whether it is much closer to its own cluster than to the next nearest one, averaged and scaled from −1 to 1. Higher is better. It peaks clearly at *k* = 3.

Both point to three. In this data that is because we built three. In real data the elbow is often soft and the silhouette often flat, and the honest conclusion may be "there is no clear number of groups here". That is a finding, not a failure.

## Profiling and naming

A cluster is a list of row numbers. To make it mean anything, profile it: what is typical of the people in it?

```python
km = KMeans(n_clusters=3, n_init=10, random_state=0).fit(X)
listeners["cluster"] = km.labels_
listeners.groupby("cluster").mean().round(2)
#          hours_week  night_share  new_release  avg_bpm
# cluster
# 0             17.80         0.55         0.60   124.83
# 1              6.04         0.15         0.20    94.38
# 2             29.79         0.30         0.36   104.64
listeners["cluster"].value_counts()      # 1: 300   0: 250   2: 150
```

**How it works.** `groupby("cluster").mean()` gives each cluster's centre in the original units, which is the cluster profile. k-means found the three groups we built, with almost exactly the sizes we built them.

Now name them. Cluster 0 listens at night, to new music, at a fast tempo: *night owls chasing new releases*. Cluster 1 listens a little, by day, to familiar slower music: *commuters with old favourites*. Cluster 2 listens a lot, all day, to a bit of everything: *the always-on*.

Those names are not in the data. They are a story laid over three columns of averages, and every one of them claims more than the numbers do. "Commuters" assumes *why* people listen in the daytime. "Old favourites" assumes a low new-release share means loyalty rather than, say, a children's playlist on repeat. The names will be what everyone remembers. The numbers will be forgotten by the next meeting. That is the power of naming, and it is why naming a cluster deserves as much care as building it.

## Is it real?

With no labels, you cannot compute accuracy. So the question changes shape: **is this clustering stable?** If small, arbitrary changes give you different groups, the groups were never in the data. They were in the procedure.

```python
from sklearn.metrics import adjusted_rand_score

a = KMeans(n_clusters=3, n_init=10, random_state=1).fit_predict(X)
b = KMeans(n_clusters=3, n_init=10, random_state=2).fit_predict(X)
adjusted_rand_score(a, b)       # 1.0    the same three groups, every time

a = KMeans(n_clusters=6, n_init=1, random_state=1).fit_predict(X)
b = KMeans(n_clusters=6, n_init=1, random_state=2).fit_predict(X)
adjusted_rand_score(a, b)       # 0.72   six groups, and they move around
```

**How it works.** The **adjusted Rand index** compares two clusterings of the same rows: 1.0 means they put everyone in identical groups, around 0 means no more agreement than chance. The cluster numbers themselves do not matter — it checks who ends up *together*. Ask for three clusters and two different random starts agree completely. Ask for six and they disagree substantially: the extra three "groups" are just different ways of slicing the same three real ones.

Run the same check on data with no structure at all, and you see the deepest risk of clustering:

```python
noise = rng.random((700, 4))          # four columns of pure randomness
for k in [2, 3, 4]:
    labels = KMeans(n_clusters=k, n_init=10, random_state=0).fit_predict(noise)
    print(k, round(silhouette_score(noise, labels), 3), np.bincount(labels))
# 2  0.21   [351 349]
# 3  0.197  [285 206 209]
# 4  0.207  [186 184 138 192]
```

**How it works.** There are no groups in random noise. k-means returns tidy, similar-sized clusters anyway, every time, for any *k* you ask. Each has a centre, so each would have a profile and could be given a name. The silhouette scores, around 0.2 compared with 0.6 for the real structure, are the only warning.

This is the result to remember from this chapter. **A clustering algorithm always produces clusters.** It cannot tell you there are none. You have to check — with the silhouette, with stability, and by asking whether the profiles would survive a different set of features.

!!! case "Teardown — Spotify Wrapped, next to an ad-preferences page"
    Open two things side by side: your Spotify Wrapped (or any "year in review"), and the ad-preferences or "your interests" page of a platform you use — Google's *My Ad Center*, Instagram's *Ad topics*, or similar.

    **Predicting.** Both are telling you which group you belong to. Wrapped gives you a listening personality or a top genre. The ad page lists the interest categories you have been placed in.

    **Collecting.** The same kind of thing in both cases: everything you did on the platform for a year.

    **Teaching.** Here the interesting question is not what you teach the model but how you react to being told. Wrapped is designed to be shared; people post it. The ad page is buried in settings; people are surprised by it, sometimes unpleasantly.

    **Choosing differently.** It is the same operation twice — you were clustered, and the cluster was named. One feels like a gift and the other like surveillance. Twenty minutes on *why* gets you most of the ethics of segmentation: who sees the label, whether you can change it, whether it is flattering, and what it is used for when you are not looking.

    From this week a fifth question joins the four: *what group has this system put me in, and was I told?*

!!! critical "Critical Lens — A cluster becomes a person"
    The trouble with segments is not that they are inaccurate. On average they are often quite accurate. The trouble is what happens when an average is used to make a decision about an individual.

    Once a cluster has a name, it acquires a personality, and everyone in it is treated as having that personality. A listener sorted into "night owls" may get different playlists, different ads and different prices from one sorted into "commuters". They did not choose the label, usually cannot see it, and cannot appeal it. They may sit right on the border between two clusters, where a slightly different random start would have put them in the other one — which, as the stability check showed, is not a hypothetical.

    Three design questions follow for any project that clusters people. Do the people know they were sorted? Can they see, and correct, which group they are in? And does being in a group change what they are offered, charged or shown? If the answer to the last question is yes and the first two are no, you have built something that should make you uncomfortable.

## How would we know this is any good?

There is no accuracy here, so ask different questions.

1. **Is the structure there at all?** Compare the silhouette with what you get on shuffled or random data. If they are close, you have found an arrangement, not a pattern.
2. **Is it stable?** Re-run with different random seeds, a different *k*, with and without one of the features, scaled and unscaled. Groups that survive those changes are worth naming. Groups that dissolve were never groups.
3. **Is it useful?** A clustering is good if it helps someone do something: design three playlists instead of one, understand a user base, notice a group they had been ignoring. If nobody would act differently knowing it, it is decoration.
4. **Would the names survive scrutiny?** Read each name next to its profile. Does the name claim anything the numbers do not show?

Silhouette gives you a number. Stability gives you the honest answer.

!!! further "Going Further — Other ways to find groups"
    k-means assumes clusters are round blobs of similar size, and it puts every row in exactly one. Real groups often are not like that. *Hierarchical clustering* builds a tree of groups within groups, so you do not have to choose *k* in advance; DBSCAN finds clusters of any shape and is allowed to say that some points belong to no cluster at all, which is sometimes the most honest answer. DataCamp's *Unsupervised Learning in Python*, Chapter 2, covers the hierarchical approach, and the StatQuest video on k-means clustering is a clear twenty minutes if the steps above still feel abstract.

You have now seen the idea of similarity three ways: as the votes of neighbours, as groups around a centre, and — next week — as the direction two things point in. That last version is the one inside almost every recommender you use.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp.

    - [Unsupervised Learning in Python](https://app.datacamp.com/learn/courses/unsupervised-learning-in-python) — **Ch. 1 Clustering for Dataset Exploration.**
    - [Unsupervised Learning in Python](https://app.datacamp.com/learn/courses/unsupervised-learning-in-python) — **Ch. 2 Visualization with Hierarchical Clustering and t-SNE** *(optional)*.

    **On your own** *(optional)*:

    - [Tutorial: k-Means Clustering with scikit-learn](https://www.datacamp.com/tutorial/k-means-clustering-python) — choosing *k* with the silhouette score.
    - [Project: Clustering Antarctic Penguin Species](https://www.datacamp.com/projects/1809) — about an hour; afterwards, check whether the clusters are the species.
