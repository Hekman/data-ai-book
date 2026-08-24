<p class="chapter-kicker">Part I · Chapter 4 · <span class="week">Week 2</span></p>

# Combining datasets

!!! guiding "The question this chapter answers"
    How do I bring two datasets together into one richer table — without silently gaining or losing rows?

One table is rarely enough. The interesting questions almost always live in the space *between* datasets: trips and the weather on the day they happened; sales and the region each store sits in; survey answers and the demographics of who answered. Each table on its own is thin; joined together, they can answer questions neither could alone. This chapter is about that move, combining two datasets into one richer table, and it is the skill your Part I project leans on when it asks you to *enrich* your data.

There are only two fundamentally different ways to combine tables, and keeping them straight saves endless confusion. You can **stack** tables that describe the same kind of thing: same columns, more rows (this January's trips on top of February's). Or you can **join** tables that describe *different* things linked by a shared key: more columns, matched row by row (attach each trip's weather by matching on the date). Stacking grows the table downward; joining grows it sideways.

!!! matters "Why this matters for designers"
    Enrichment is where a thin dataset becomes one worth designing from — and it is also where the most invisible errors enter. A join that runs without an error message is not a join that did what you meant: a duplicated key silently inflates every total downstream, and a mismatched key silently empties a column. Checking the row count every single time is a two-second habit that protects everything you build on top.

!!! newsroom "Why this matters for journalists"
    Joining records *is* the method behind a large share of investigative work: donors matched to contracts, inspection reports matched to licences, a payroll matched to an electoral roll. It is also where the story goes wrong quietly. A duplicated key inflates a count you then put in the lede, and a mismatched key empties a column so the pattern you were chasing disappears. Check the row count before and after every join, and treat any change you can't explain as a finding, not a nuisance.

!!! outcomes "By the end of this chapter you can"
    - Tell stacking from joining, and pick the right one for the shape of data you have.
    - Stack tables with `concat` and join them on a shared key with `merge`.
    - Choose `how="left"`, `"inner"`, `"outer"` or `"right"` deliberately, and say what each does to unmatched rows.
    - Verify a join with a row-count check, `indicator=True`, and `validate=` — and diagnose fan-out and failed key matches.
    - Name overlapping columns with `suffixes` instead of living with `_x` and `_y`.
    - Decide the grain of your result first, and aggregate a many-row table before joining it.
    - Enrich your own dataset with a second source, and explain what the join made answerable.
    - Recognise that joining data about people can re-identify them.

!!! origins "Origins — The join is what the relational model was built for"
    In 1970 Edgar Codd, a researcher at IBM, published *A Relational Model of Data for Large Shared Data Banks*, proposing that data be held in simple tables linked by shared keys rather than tangled together in one rigid structure. The **join**, recombining those separate tables on demand, is the operation his model was designed around, and it's why databases keep customers, orders and products in *separate* tables and stitch them together only when asked. Every `merge` you write is Codd's idea in miniature: keep data normalised and apart, then bring the pieces together for the question at hand.

## Our running data

You met the bike-share `trips` table in [Chapter 2](the-dataframe.md). Realistically, it doesn't arrive alone: the full dataset is *three separate tables* that only become useful once combined.

```python
import pandas as pd

# 1) trips — one row per bike trip
trips = pd.DataFrame({
    "trip_id":      [1, 2, 3, 4, 5, 6],
    "start_time":   pd.to_datetime([
        "2026-05-01 08:12", "2026-05-01 08:40", "2026-05-01 17:30",
        "2026-05-02 09:05", "2026-05-02 18:15", "2026-05-03 12:20"]),
    "station":      ["Centraal", "Uithof", "Centraal", "Lombok", "Uithof", "Centraal"],
    "membership":   ["member", "casual", "member", "member", "casual", "casual"],
    "duration_min": [7, 22, 9, 14, 25, 11],
})

# 2) stations — one row per station (a *lookup* table)
stations = pd.DataFrame({
    "station":  ["Centraal", "Uithof", "Lombok", "Zuilen"],
    "district": ["Centrum", "East", "West", "North"],
    "docks":    [40, 30, 18, 20],
})

# 3) weather — one row per day
weather = pd.DataFrame({
    "date":   pd.to_datetime(["2026-05-01", "2026-05-02", "2026-05-03"]),
    "temp_c": [17.5, 14.2, 21.0],
    "rain_mm": [0.0, 6.4, 0.0],
})
```

Notice the shape of the problem: `trips` tells you *which station*, but not *which district*; that lives in `stations`. And it tells you *when*, but not *the weather*; that lives in `weather`. Answering "do people ride longer in the East on dry days?" requires all three tables joined into one.

## Stacking with `concat`

When two tables share the same columns, say trips from two months, you stack them:

```python
may_trips  = trips
june_trips = trips.copy()          # pretend this is June's data
all_trips  = pd.concat([may_trips, june_trips], ignore_index=True)
all_trips.shape                    # rows added together; ignore_index renumbers them
```

**How it works.** `pd.concat` lines the tables up by column name and puts one below the other. `ignore_index=True` throws away the old row numbers and renumbers cleanly, so you don't end up with two rows both labelled `0`. Stacking is the easy case; the only thing to watch is that the columns really do match. A stray renamed column will quietly become a half-empty column of `NaN`.

## Joining with `merge`

The real workhorse is `merge`, which matches rows between two tables on a shared **key** column. Attach each trip's district by matching `station`:

```python
enriched = trips.merge(stations, on="station", how="left")   # (1)!
enriched[["trip_id", "station", "district", "duration_min"]]
```

1.  `on="station"` is the key both tables share. `how="left"` keeps every row of the left table (`trips`) and pulls in matching columns from the right (`stations`).

**How it works.** For each trip, `merge` finds the row in `stations` with the same `station` value and copies its extra columns (`district`, `docks`) alongside. One trip, one matching station, so the trip table gains columns without gaining rows. This is exactly the enrichment pattern: a big "fact" table (trips) gains descriptive detail from a small "lookup" table (stations).

The `how` argument decides what happens to rows that *don't* find a match, and choosing it correctly is where most join bugs hide:

| `how=` | Keeps | Unmatched rows | Use it when | The risk |
|---|---|---|---|---|
| `"left"` | every row of the left table | get `NaN` in the new columns | enriching a fact table from a lookup — **the safe default** | you may not notice the `NaN`s |
| `"inner"` | only rows matching in both | dropped from both sides | you genuinely need complete records | silently shrinks your dataset |
| `"outer"` | everything from both | kept, gaps filled with `NaN` | reconciling two partial sources | grows the table; lots of `NaN` |
| `"right"` | every row of the right table | get `NaN` | rarely — usually just swap the tables | confusing to read |

The habit worth forming: **start with `how="left"` and only change it when you can say why.** An `inner` join is a filter wearing a join's clothing, and it removes rows without telling you.

### Let pandas check the join for you

The row-count check below is essential, but pandas can also enforce your expectation *before* the damage happens. The `validate` argument states what kind of relationship you believe the key describes, and raises an error if you're wrong:

```python
# "many trips to one station" — the classic enrichment shape
enriched = trips.merge(stations, on="station", how="left", validate="many_to_one")   # (1)!

# If `stations` accidentally listed a station twice, this raises:
# MergeError: Merge keys are not unique in right dataset; not a many-to-one merge
```

1.  `validate` takes `"one_to_one"`, `"one_to_many"`, `"many_to_one"` or `"many_to_many"`. `"many_to_one"` is the one you want for nearly every enrichment: many fact rows, one lookup row each.

**How it works.** `validate="many_to_one"` asks pandas to confirm the key is unique on the *right* — which is precisely the condition that prevents fan-out. If a duplicate lurks in your lookup table, you get a loud `MergeError` instead of a table that has quietly grown and inflated every total downstream. This is the same move as the `assert` statements in [Chapter 3](cleaning-messy-data.md): take the thing you're assuming and make the computer check it. Add it to every `merge` you write and you eliminate the most expensive join bug there is.

### When column names collide

If both tables have a column of the same name that isn't the key, pandas keeps both and appends suffixes:

```python
left  = pd.DataFrame({"id": [1, 2], "name": ["a", "b"], "updated": ["2026-01", "2026-01"]})
right = pd.DataFrame({"id": [1, 2], "updated": ["2026-05", "2026-06"]})

left.merge(right, on="id")                                    # updated_x, updated_y  — unhelpful
left.merge(right, on="id", suffixes=("_trip", "_station"))     # updated_trip, updated_station
```

Default `_x` and `_y` are a trap: they're meaningless a week later, and it's genuinely easy to chart the wrong one. Name them, every time, after where they came from.

!!! practice "In Practice — Always check a join's row count"
    A join can silently change how much data you have, in both directions, and the failure is invisible unless you look. Two habits catch almost everything:

    ```python
    before = len(trips)
    enriched = trips.merge(stations, on="station", how="left")
    print(before, "->", len(enriched))          # did the row count change unexpectedly?

    # Which rows failed to find a match?
    check = trips.merge(stations, on="station", how="left", indicator=True)
    check[check["_merge"] == "left_only"]        # trips whose station isn't in `stations`
    ```

    If the row count *grew*, your key wasn't unique on the right (a station listed twice will duplicate every trip that matches it, a "fan-out" that inflates every later total). If new columns are full of `NaN`, your keys didn't match (often a trailing space or a capital letter), straight out of [Chapter 3](cleaning-messy-data.md). Check the count every single time; a join that runs without error is not a join that did what you meant.

!!! case "Case study — enrich the bike-share trips"
    Bring all three tables together and ask a question none could answer alone. First attach the district (join on `station`), then the weather (join on the trip's calendar date):

    ```python
    df = trips.merge(stations, on="station", how="left")   # + district, docks
    df["date"] = df["start_time"].dt.floor("D")            # date from the timestamp
    df = df.merge(weather, on="date", how="left")          # + temp_c, rain_mm

    # Now a real question: average trip length by district, wet vs dry days
    df["wet"] = df["rain_mm"] > 0
    df.groupby(["district", "wet"])["duration_min"].mean()
    ```

    In three joins the thin `trips` table became one that knows *where* and *in what weather* each ride happened, the raw material for the rest of Part I. We'll keep building on `df`.

!!! critical "Critical Lens — Joining is also how people get re-identified"
    The same power that enriches a dataset can strip its anonymity. Individually harmless tables, joined on a shared key, can become deeply revealing: a famous result showed that roughly 87% of Americans are uniquely identifiable from just ZIP code, birth date and sex, so an "anonymous" medical dataset can be re-identified by joining it to a public voter roll. This is the *mosaic effect*: privacy lost not in any one dataset but in the *combination*. When you join datasets about people, you may be creating something far more sensitive than either input — the duties set out in [Chapter 6](apis.md#handling-personal-data), and pressed harder in [Chapter 10](scraping.md).

## Going further

## Aggregate first, then join

The fan-out warning has a positive version worth knowing, because it turns a whole class of broken joins into easy ones. When the table you want to attach has *many* rows per key — several reviews per product, several trips per station — joining it directly multiplies your rows. Almost always what you actually wanted was a *summary* of those many rows, so summarise first, then join one-to-one:

```python
# Wrong instinct: attach every trip to every station row, then try to fix it later.
# Right move: reduce trips to one row per station, THEN join.

per_station = (trips.groupby("station")
                    .agg(n_trips=("trip_id", "count"),
                         avg_duration=("duration_min", "mean"))
                    .reset_index())            # (1)!

station_summary = stations.merge(per_station, on="station",
                                 how="left", validate="one_to_one")
station_summary
```

1.  `reset_index()` turns `station` from the index back into a column so it can serve as a join key — the [index gotcha from Chapter 2](the-dataframe.md), showing up exactly where you'd expect it.

**How it works.** `groupby().agg().reset_index()` collapses many trips into one row per station carrying whatever summaries you asked for. Now both sides have one row per station, the join is genuinely one-to-one, `validate` can prove it, and the row count cannot surprise you. The general principle: **decide the grain of your result first, then bring every table to that grain before joining.** Most join disasters are really an unnoticed disagreement about what one row means.

!!! further "Going Further — When keys don't line up cleanly"
    Real joins are rarely as tidy as `on="station"`. Four complications you'll meet. **Differently named key columns**, handled with `left_on`/`right_on` rather than `on`. **Mismatched key values**, where "Utrecht CS" must be reconciled with "Centraal" before anything matches — a cleaning job, sometimes needing fuzzy matching (`rapidfuzz` is the usual tool, and every fuzzy match is a judgement you should log). **Many-to-many** joins, where a key repeats on both sides and you get every combination; the fix is nearly always the aggregate-first move above. And **joining on time**, where rows don't share an exact timestamp but you want the nearest earlier one — `pd.merge_asof` does this, and it's how sensor readings get matched to events. When a join misbehaves, the cause is almost always the *key*, not the merge; inspect the distinct key values on each side first.

Combining datasets is where thin data becomes rich. With one well-built table in hand, the next chapter asks the question you've been able to postpone until now: where did this data come from in the first place, and what did its origins already decide for you?

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. Joins are easiest to trust once you have watched rows disappear in front of you.

    - [Joining Data with pandas](https://app.datacamp.com/learn/courses/joining-data-with-pandas) — **Ch. 1 Data Merging Basics** and **Ch. 2 Merging Tables With Different Join Types.**
    - [Examining the History of Lego Sets](https://app.datacamp.com/learn/projects/2378) — a guided project that is a join from start to finish *(optional)*.
