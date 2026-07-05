<p class="chapter-kicker">Part I · Chapter 5</p>

# Combining datasets

One table is rarely enough. The interesting questions almost always live in the space *between* datasets: trips and the weather on the day they happened; sales and the region each store sits in; survey answers and the demographics of who answered. Each table on its own is thin; joined together, they can answer questions neither could alone. This chapter is about that move — combining two datasets into one richer table — and it is the skill your Part I project leans on when it asks you to *enrich* your data.

There are only two fundamentally different ways to combine tables, and keeping them straight saves endless confusion. You can **stack** tables that describe the same kind of thing — same columns, more rows (this January's trips on top of February's). Or you can **join** tables that describe *different* things linked by a shared key — more columns, matched row by row (attach each trip's weather by matching on the date). Stacking grows the table downward; joining grows it sideways.

!!! origins "Origins — The join is what the relational model was built for"
    In 1970 Edgar Codd, a researcher at IBM, published *A Relational Model of Data for Large Shared Data Banks*, proposing that data be held in simple tables linked by shared keys rather than tangled together in one rigid structure. The **join** — recombining those separate tables on demand — is the operation his model was designed around, and it's why databases keep customers, orders and products in *separate* tables and stitch them together only when asked. Every `merge` you write is Codd's idea in miniature: keep data normalised and apart, then bring the pieces together for the question at hand.

## Our running data

From here on, a single synthetic dataset runs through the book — a **city bike-share** in Utrecht. It arrives, realistically, as *three separate tables* that only become useful once combined:

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

Notice the shape of the problem: `trips` tells you *which station*, but not *which district* — that lives in `stations`. And it tells you *when*, but not *the weather* — that lives in `weather`. Answering "do people ride longer in the East on dry days?" requires all three tables joined into one.

## Stacking with `concat`

When two tables share the same columns — say trips from two months — you stack them:

```python
may_trips  = trips
june_trips = trips.copy()          # pretend this is June's data
all_trips  = pd.concat([may_trips, june_trips], ignore_index=True)
all_trips.shape                    # rows added together; ignore_index renumbers them
```

**How it works.** `pd.concat` lines the tables up by column name and puts one below the other. `ignore_index=True` throws away the old row numbers and renumbers cleanly, so you don't end up with two rows both labelled `0`. Stacking is the easy case — the only thing to watch is that the columns really do match; a stray renamed column will quietly become a half-empty column of `NaN`.

## Joining with `merge`

The real workhorse is `merge`, which matches rows between two tables on a shared **key** column. Attach each trip's district by matching `station`:

```python
enriched = trips.merge(stations, on="station", how="left")   # (1)!
enriched[["trip_id", "station", "district", "duration_min"]]
```

1.  `on="station"` is the key both tables share. `how="left"` keeps every row of the left table (`trips`) and pulls in matching columns from the right (`stations`).

**How it works.** For each trip, `merge` finds the row in `stations` with the same `station` value and copies its extra columns (`district`, `docks`) alongside. One trip, one matching station — so the trip table gains columns without gaining rows. This is exactly the enrichment pattern: a big "fact" table (trips) gains descriptive detail from a small "lookup" table (stations).

The `how` argument decides what happens to rows that *don't* find a match, and choosing it correctly is where most join bugs hide:

- `how="left"` keeps **all left rows**; unmatched ones get `NaN` in the new columns. The safe default for enrichment — you never silently lose data.
- `how="inner"` keeps **only rows that match in both**. Convenient, but it *drops* unmatched rows, which can quietly shrink your dataset.
- `how="outer"` keeps **everything from both**, filling gaps with `NaN`.
- `how="right"` keeps all right rows (rarely needed; usually you just swap the tables).

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

    If the row count *grew*, your key wasn't unique on the right (a station listed twice will duplicate every trip that matches it — a "fan-out" that inflates every later total). If new columns are full of `NaN`, your keys didn't match — often a trailing space or a capital letter, straight out of [Chapter 4](chapter-4.md). Check the count every single time; a join that runs without error is not a join that did what you meant.

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

    In three joins the thin `trips` table became one that knows *where* and *in what weather* each ride happened — the raw material for the rest of Part I. We'll keep building on `df`.

!!! critical "Critical Lens — Joining is also how people get re-identified"
    The same power that enriches a dataset can strip its anonymity. Individually harmless tables, joined on a shared key, can become deeply revealing — a famous result showed that roughly 87% of Americans are uniquely identifiable from just ZIP code, birth date and sex, so an "anonymous" medical dataset can be re-identified by joining it to a public voter roll. This is the *mosaic effect*: privacy lost not in any one dataset but in the *combination*. When you join datasets about people, you may be creating something far more sensitive than either input — a responsibility we return to in [Chapter 9](chapter-9.md).

## Going further

!!! further "Going Further — When keys don't line up cleanly"
    Real joins are rarely as tidy as `on="station"`. Three complications you'll meet: **many-to-many** joins, where a key repeats on *both* sides and the result is every combination (usually a sign you should aggregate one side first); **mismatched keys**, where "Utrecht CS" must be reconciled with "Centraal" before anything matches (a cleaning job, sometimes needing fuzzy matching); and **differently named key columns**, handled with `left_on`/`right_on` rather than `on`. When a join misbehaves, the cause is almost always the *key*, not the merge — inspect the distinct key values on each side first.

Combining datasets is where thin data becomes rich, and where the milestone's "enrich it yourself" requirement is met. With one well-built table in hand, we can finally start asking it questions — which is exactly what exploratory analysis, next, is for.

## Exercises

??? exercise "Exercise 5.1 · Enrich by joining"
    Using `trips`, `stations` and `weather` above: produce a single table with one row per trip that includes its `district`, `temp_c` and `rain_mm`. Confirm with a row-count check that you didn't gain or lose any trips, and report which columns (if any) contain `NaN` and why.

    ??? note "Show a solution"
        ```python
        before = len(trips)
        df = trips.merge(stations, on="station", how="left")
        df["date"] = df["start_time"].dt.floor("D")
        df = df.merge(weather, on="date", how="left")
        print(before, "->", len(df))     # should be unchanged: 6 -> 6
        df.isna().sum()                  # any NaN? which column, and is a key missing?
        ```
        If `district` were ever `NaN`, it would mean a trip started at a station missing from `stations` — a real gap worth a line in your provenance note, not something to paper over.

??? exercise "Exercise 5.2 · Break a join on purpose"
    Change one station name in `trips` to lowercase (e.g. `"centraal"`) and re-run the district join with `how="left"`. What happens to that trip's `district`, and why? Then fix it with a cleaning step from Chapter 4 and confirm the join now matches.

    ??? note "What good looks like"
        The mismatched row gets `NaN` for `district`, because `"centraal"` ≠ `"Centraal"` — the key didn't match. Normalising case (`trips["station"] = trips["station"].str.title()`) before the join fixes it. The lesson is the one this chapter keeps making: joins fail on the *key*, so clean your keys before you merge, and always check the result.
