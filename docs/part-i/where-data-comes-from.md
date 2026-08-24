<p class="chapter-kicker">Part I · Chapter 5 · <span class="week">Week 3</span></p>

# Where data comes from

!!! guiding "The question this chapter answers"
    Where does data come from, what does its origin bake in, and what am I allowed to use?

Before you touch a single row, ask a question most tutorials skip: *why does this dataset exist at all?* Datasets are not found objects. Each one was produced by some institution, for some purpose, at some cost, and its shape still carries the fingerprints of that purpose. A dataset built to *tax* people looks different from one built to *help* them, even when it contains the same names. Knowing why a dataset was made is the first and cheapest form of analysis.

Consider the oldest large datasets we have: censuses. States have counted their populations for as long as there have been states, and they did it for concrete reasons: taxation, conscription, the apportionment of power. Those reasons decided what got recorded. A census built to conscript soldiers counts men of fighting age carefully and everyone else roughly. Two thousand years later, when a historian uses that data to estimate a population, the original purpose is still bending the numbers. This is the general lesson in miniature: *the reason a dataset was collected is a lens that never fully comes off.*

!!! matters "Why this matters for designers"
    The origin of data is invisible in the finished chart and determines everything about it. Survey data carries the framing of its questions; behavioural traces capture what was easy to log, not what mattered; scraped content reflects who posts, not who exists. A designer who understands provenance can anticipate bias *before* it becomes a published mistake — and knows which claims a dataset simply isn't entitled to support.

!!! newsroom "Why this matters for journalists"
    Provenance is sourcing by another name, and the questions are the ones you'd ask about any document: who created this record, for what purpose, who is missing from it, and what did they have an interest in counting? Administrative data measures enforcement rather than incidence, complaints data measures who feels able to complain, and platform data measures who posts. Establishing this before you report saves you from the story that falls apart under a single call to the agency that published the file.

!!! outcomes "By the end of this chapter you can"
    - Say why a dataset exists, and explain how its original purpose still bends what it can measure.
    - Name the main routes data arrives by (portals, APIs, surveys, sensors and logs, scraping) and the bias each one introduces.
    - Identify a dataset's unit of observation, its columns' silent category choices, and its conspicuous absences.
    - Ask whether a sample is fit for the claim being made, and recognise selection bias when you see it.
    - Find a dataset's licence, judge what it permits, and record source, licence and personal-data status in your provenance note.
    - Read formats beyond CSV (Excel, flat and nested JSON) into a dataframe, and say what each container makes easy or hard.
    - Load an awkward CSV deliberately — separator, decimal comma, encoding, missing-value codes, forced text columns.

!!! origins "Origins — From counting people to counting them by machine"
    Ancient states ran periodic counts (Egypt's cattle-and-people censuses are documented from the Old Kingdom, and the Han dynasty census of 2 CE recorded some 57.7 million people), totals that still survive. But scale eventually broke the method. The 1880 US census took most of a decade to tabulate by hand; by the time it was finished it was nearly obsolete. Herman Hollerith's electric punched-card tabulators processed the 1890 count in a fraction of the time, and the company he founded to sell them became IBM. The census didn't just *use* data technology; its impossible scale is what *created* the data-processing industry.

## Tables, and the quiet decisions inside them

Most data you'll meet arrives as a table: rows and columns. It looks so natural that it's easy to forget it's a *model*, a compression of a messy world into a rectangle. Every table makes at least three decisions for you, and each is worth surfacing.

First, the table decides **what a row is**, the unit of observation. One row per person? Per transaction? Per person-per-day? The same reality can be tabled many ways, and the choice determines what questions are even askable. Second, the table decides **what a column is**, which attributes are deemed worth recording. Everything without a column is, for your analysis, invisible. Third, the table decides **the categories inside a column**, the fixed boxes a messy reality is sorted into. A "gender" column with two values has made a claim; a "country" column with no row for a stateless person has made another.

None of these decisions are wrong, exactly. They are *choices*, and as the designer working the material you should be able to name them, because they are the seams where the data will most often mislead you.

!!! critical "Critical Lens — The library of missing datasets"
    The artist-researcher Mimi Onuoha keeps a project she calls *The Library of Missing Datasets*: things conspicuously *not* collected: for years, comprehensive data on police killings in the United States, even as everything else was measured. Such gaps are rarely accidents. Data gets collected where there is power, profit or obligation to collect it, and not where there isn't. So when you inventory a dataset, inventory its *absences* too. The most important column is sometimes the one that was never made.

## Was this a good sample?

Almost every dataset is a **sample**, a subset of some larger population you actually care about, even when it feels like "all the data." A table of your app's users is not a sample of *people*; it's a sample of *people who found and installed your app*. The single most important question you can ask of a dataset's origins is therefore: **who or what is in here, and who or what is systematically left out?** When the answer is "the sample is skewed," no amount of clever analysis downstream can fix it; the bias is baked in at collection.

This failure has a name, **selection bias**: the process that put rows into your dataset is related to the very thing you're trying to measure. A satisfaction survey answered only by people with strong feelings. A sensor that fails in exactly the conditions you most want to study. A bike-share dataset that, by definition, contains only trips by people who live near a dock and use the app, so any claim it makes about "how the city cycles" is really a claim about a particular, unrepresentative slice of it.

!!! origins "Origins — Gallup and the lesson of the sample"
    In 1936 the *Literary Digest* polled over two million people and confidently predicted the wrong winner of the US presidential election; George Gallup polled a few thousand and got it right. The *Digest*'s huge sample was drawn from car and telephone owners, wealthier, and unrepresentative in a depression year, while Gallup's small sample was chosen to *mirror* the electorate. The lesson has echoed for ninety years and the big-data era had to relearn it: **how you sample matters more than how much you collect.** A biased million is worse than a representative thousand, because the million's size only lends false confidence to its skew.

## The routes data arrives by

So far the data has been handed to you — tidy in [Chapter 2](the-dataframe.md), messy in [Chapter 3](cleaning-messy-data.md) — but always already collected by someone else. In real work you have to *get* it yourself, and there are only a handful of routes it comes by. Each one shapes what you receive, and you inherit that shape whether you notice or not.

**Open data portals** (national statistics offices, municipal portals, the EU's data portal) publish curated files for public reuse. They're the easiest route and the most institutionally framed: what's published is what an institution decided was publishable, in the categories it uses administratively. **APIs** hand over structured data on request, live and in a form built for programs; the next chapter is entirely about them. **Surveys** produce data that carries the framing of its own questions: the wording, the order, and the answer options you were offered are all baked into the numbers. **Sensors and logs** record whatever was cheap to instrument, continuously and at scale, which is not the same as whatever was important — and they fail, often in exactly the conditions you most want to study. **Scraping** extracts data from pages meant for human eyes; it's the last resort, and [Chapter 10](scraping.md) covers both the technique and its limits.

The useful habit is to name the route before you trust the table, because each route has a signature failure. Portals lag reality. APIs give you a snapshot of a moment and quietly change their schema. Surveys measure what people will say, not what they do. Logs over-represent the instrumented. Scrapes over-represent whoever posts.

!!! practice "In Practice — The five questions to ask any new dataset"
    Before analysis, write down: (1) Who produced this, and for what purpose? (2) What is one row? (3) What's in the columns, and what obviously isn't? (4) When was it collected, and is that still current? (5) What's the format, and what might it have quietly mangled (dates, leading zeros, encodings)? Ten minutes on these five will save you hours and keep you from confidently analysing an artefact of *collection* rather than the world.

!!! case "Case study — the origins of the bike-share data"
    Turn the five questions on the running `trips` table from [Chapter 2](the-dataframe.md) and its origins immediately narrow what it can say. One row is one *docked* trip, by an *app user*, that completed successfully. That means no undocked bikes, no failed unlocks, no one without a smartphone, and nothing at all about people who'd cycle if a dock existed near them. It is the [selection bias](#was-this-a-good-sample) above made concrete: any claim beginning "this shows how the city cycles" is really a claim about a particular, unrepresentative slice of it.

    Write that limitation into your provenance note now, in one sentence, and it will keep you honest through every chart you make from this data later.

## What am I allowed to use?

Provenance has a legal half, and it's the half students skip. "I found it online" is not a licence. Before you build work on a dataset, find out what you're permitted to do with it, because the answer occasionally rules out the project you had in mind — and it is much cheaper to learn that now than after you've made the charts.

Three questions cover most cases.

**What licence is it under?** Open datasets usually carry one, and it tells you the terms. **CC0** or public domain means do as you like. **CC BY** means you may reuse and republish if you credit the source. **CC BY-SA** adds that your result must carry the same licence. **CC BY-NC** forbids commercial use — which can quietly include a client project. Some government data is released under national open-data licences with their own wording. If you can't find a licence at all, the safe default is that you have *no* redistribution rights: you may be able to analyse the data privately, but publishing it, or a derivative of it, is a different act.

**Is it about people?** If so, data-protection law applies regardless of licence, and being publicly available changes nothing about that. Under the **GDPR** you need a lawful basis for processing personal data, you may only use it for the purpose you collected it for, you must collect no more than you need, and the people in it retain rights over it — including having it deleted. Chapter 6 goes into what that means in practice; the point here is that a permissive licence and lawful processing are two separate hurdles, and you have to clear both.

**What did the terms of service say?** A dataset behind a login or an API usually comes with a contract you accepted, and it may restrict redistribution, commercial use, or automated collection entirely, even for data that looks public.

!!! practice "In Practice — Three lines in your provenance note"
    You don't need a lawyer, you need a record. Add three lines to your provenance note the moment you acquire a dataset: **Source** (the URL, the organisation, the date you downloaded it), **Licence** (the exact name, or "none stated — analysis only"), and **Personal data** (yes/no, and which fields). Ten seconds of typing that answers the question your assessor, your client, or your future self will ask. And if the licence line reads "none stated", treat that as a finding: it's a limitation on what you may publish, and it belongs in the write-up.

!!! critical "Critical Lens — Open data is not automatically fair data"
    "Open" describes a licence, not a politics. A dataset can be perfectly open and still cause harm: published in categories the people it describes never agreed to, opened because an institution had the capacity to open it while others didn't, or scrubbed of the context that made it interpretable. Open data has also been re-identified more than once by joining it against something else, so a well-intentioned release can expose the very people it was meant to serve. The right reading of an open licence is that the *legal* question has been answered and the *ethical* one has not. You still have to ask who is in this data, whether they had any say, and whether your use of it is one they'd recognise.

## File formats are not neutral either

Data arrives in a container (a CSV, an Excel file, a JSON blob, a database export), and the container shapes what's easy. A CSV is just text with commas; universal and dumb, which is both its virtue and its curse (it will happily turn a phone number into a giant integer and drop the leading zero). A spreadsheet mixes data with formatting and formulas, so the numbers and someone's interpretation of them often arrive fused. A JSON file can nest, carrying shapes a flat table can't, at the cost of being harder to reason about at a glance. You don't need to master all of these yet; you need the habit of asking, on first contact: *what form is this in, and what did that form make easy or hard for whoever produced it?*

## Reading beyond CSV

CSV is the most common container, but not the only one you'll meet. The same `pd.read_*` family opens the others, so switching format is usually a one-line change:

```python
# Excel — one sheet of a workbook (needs the openpyxl package installed)
xl = pd.read_excel("report.xlsx", sheet_name="signups")

# JSON — the format most web APIs return
j = pd.read_json("export.json")

# Nested JSON (an API response with objects inside objects) needs flattening:
import pandas as pd
payload = {"results": [
    {"id": 1, "city": {"name": "Utrecht"}, "plan": "pro"},
    {"id": 2, "city": {"name": "Amsterdam"}, "plan": "free"},
]}
flat = pd.json_normalize(payload["results"])   # -> columns: id, city.name, plan
flat
```

**How it works.** `read_excel` reads a named sheet from a workbook (remember that an Excel file often carries formatting and formulas *around* the data, so check you grabbed the right range). `read_json` maps a flat JSON array straight to a table. When JSON is *nested*, objects within objects, as almost every API returns, `json_normalize` flattens the nesting into dotted column names like `city.name`. You'll use exactly this in [Chapter 6](apis.md) when you pull data from a live API.

## When a CSV fights back

`pd.read_csv("file.csv")` works perhaps three times out of four. The other quarter is where the "formats aren't neutral" argument stops being abstract, and a handful of arguments fixes almost all of it. This matters more than usual in the Netherlands, because a CSV exported from a Dutch-locale Excel is a different file from an English-locale one:

```python
import pandas as pd

df = pd.read_csv(
    "gemeente_export.csv",
    sep=";",                    # (1)!
    decimal=",",                # (2)!
    thousands=".",              # (3)!
    encoding="utf-8",           # (4)!
    dtype={"postcode": str},    # (5)!
    na_values=["", "n/a", "onbekend", "-", "999"],   # (6)!
)
```

1.  Dutch Excel writes CSVs with **semicolons**, because the comma is already the decimal separator. Open the file in a text editor and look before you guess. `sep=None, engine="python"` will try to detect it for you.
2.  `1,5` means one and a half, not fifteen. Without this the column arrives as text and every number in it is wrong or unusable.
3.  `1.234` means one thousand two hundred thirty-four. Without this, pandas reads it as `1.234`, off by a factor of a thousand and *silently plausible* — the most dangerous kind of error.
4.  If names come out as `Ren√©` or `Ren?`, the encoding is wrong. Try `encoding="latin-1"` or `encoding="cp1252"` for older Dutch exports.
5.  Forces the postcode to stay text, preserving leading zeros and keeping you from averaging a label — the identifier problem from [Chapter 1](data-variables-structures.md), fixed at the door.
6.  Declares which strings mean "missing", so they arrive as `NaN` instead of masquerading as categories. `999` is here because sentinel values are common in older datasets.

**How it works.** Every one of these arguments is you overriding a guess. `read_csv` inspects the file and infers separators, types and what counts as missing; the inference is good but it is an inference, and when it's wrong it usually fails *quietly* rather than loudly. The `thousands` case is the one to fear: nothing errors, the column is numeric, and your totals are a thousandfold out.

!!! practice "In Practice — Open the file before you load it"
    Look at the first three lines of any new file as plain text before you write a single line of pandas. In a notebook:

    ```python
    with open("gemeente_export.csv", encoding="utf-8", errors="replace") as f:
        for _ in range(3):
            print(f.readline().rstrip())
    ```

    In five seconds you can see the separator, whether there's a header, how numbers are punctuated, whether there are junk rows above the real header (`skiprows=` handles those), and whether accented characters survived. Then load it deliberately. Loading first and debugging afterwards is how a decimal-comma error reaches a published chart.

### Format cheat sheet

| Format | Reader | The thing that bites |
|---|---|---|
| CSV | `pd.read_csv` | separator, decimal comma, encoding, leading zeros stripped |
| Excel `.xlsx` | `pd.read_excel` | data mixed with formatting and formulas; multiple sheets; merged header cells |
| JSON | `pd.read_json` | nesting a flat table can't hold — use `pd.json_normalize` |
| Text with fixed columns | `pd.read_fwf` | column widths must be given or guessed |
| Parquet | `pd.read_parquet` | needs `pyarrow`; keeps types, unlike CSV — a good format to *save* to |
| HTML table | `pd.read_html` | returns a *list* of tables; see [Chapter 10](scraping.md) |
| Clipboard | `pd.read_clipboard` | convenient, unreproducible — never in final work |

The last column is the useful one. Choosing a format is choosing which class of error you'll be debugging later, and the reason CSV dominates despite being the most error-prone is simply that everything can read it.

!!! further "Going Further — Where data *lives* shapes what you can ask"
    Behind most large datasets sits a storage decision that constrains everything downstream. A **data warehouse** organises data in advance, structured for expected questions (schema-on-write); a **data lake**, a term James Dixon coined in the Hadoop era, stores everything raw and defers structure until you read it (schema-on-read). Warehouses keep you disciplined and can be rigid; lakes keep you flexible and can degenerate into "data swamps." You rarely choose this as a student, but recognising which world your data came from explains a lot about why it's clean-but-narrow or vast-but-feral. Ralph Kimball's dimensional modelling (the "star schema") is the classic account of shaping data around the questions humans actually ask.

Where data comes from is not throat-clearing before the real work. It *is* the real work, done early, when it's cheapest. Carry these questions into the next chapter, where you stop receiving data and start *asking* for it.

!!! exercise "Practice on DataCamp"
    The practice for this chapter is code-free — it is about provenance and consent, not syntax.

    - [Understanding Data Science](https://app.datacamp.com/learn/courses/understanding-data-science) — **Ch. 2 Data Collection and Storage.** Where datasets come from and how they are stored.
    - [Introduction to Data Ethics](https://app.datacamp.com/learn/courses/introduction-to-data-ethics) — **Ch. 2 Data Ethics: Now and Forever.** Consent, transparency and accountability across the data lifecycle *(optional)*.
