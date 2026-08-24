<p class="chapter-kicker">Part I · Chapter 6 · <span class="week">Week 3</span></p>

# Collecting data: APIs

!!! guiding "The question this chapter answers"
    How do I ask a service for data, read what it sends back, and stay inside what I'm allowed to collect?

The last chapter was about datasets other people made. This one is where you start crossing from *consumer* of data to *producer* of it, and the politest way to make that crossing is to ask. An **API** (application programming interface) is a front door: a service that offers its data in a clean, structured form, usually JSON, designed to be read by programs rather than people. When an API exists for what you need, use it. It's stable, sanctioned, and hands you tidy data. The alternative, extracting data from pages built for human eyes, is the back door, and it waits for you in [Chapter 10](scraping.md).

An API call is one of the smallest, highest-leverage things you'll learn all block. It's also the mechanism behind almost everything else in this track: the AI models you'll build with in Block C are reached exactly this way, by sending a request to an endpoint and reading a JSON reply.

!!! matters "Why this matters for designers"
    The moment you can call an API, the set of things you can design with expands enormously — live weather, transit times, prices, public statistics, a language model. But an API response is not neutral raw material either. Someone decided which fields to expose, what the rate limit is, and what the terms permit. Reading an API well means reading its *documentation* as a design artefact: what it makes easy is what it wants you to build.

!!! newsroom "Why this matters for journalists"
    An API is a rare kind of source: one that answers the same question the same way twice, which is what makes a story reproducible months later. But it is still a source with interests. The organisation decided which fields to publish, how far back the archive goes, and when to change the terms — and those decisions shape the story you are able to tell as surely as a press officer's framing does. Read the documentation the way you'd read a spokesperson: note what it volunteers, and note what it doesn't.

!!! outcomes "By the end of this chapter you can"
    - Explain what an API is, and identify the endpoint, parameters, credentials and response in a set of API docs.
    - Call a live API from Python — including an AI model via Groq — and read its JSON response.
    - Pull data from a public API into a dataframe, with a fallback so your notebook still runs offline.
    - Read a status code and know whether the fix is yours or the server's, and retry with backoff only where it helps.
    - Cache responses to disk, so your work is reproducible, offline-capable and kind to the service.
    - Collect across multiple pages politely, respecting rate limits and terms of service.
    - Handle keys and secrets without leaking them, and apply a minimum-viable privacy check before collecting data about people.

!!! origins "Origins — How data got a common shape"
    APIs feel like plumbing, but their shape was argued into existence. In 2000 Roy Fielding's doctoral dissertation described **REST**: treat every thing as a resource with a URL, and use the web's own verbs to act on it, rather than inventing a new protocol per service. Around the same time Douglas Crockford formalised **JSON**, a notation lifted straight out of JavaScript's object syntax, as a lighter alternative to the XML then dominant. Neither was a committee mandate; both won because they were simpler than the alternative. The consequence is that most of the world's public data now arrives in one predictable shape — which is why the pattern in this chapter transfers to almost any service you'll ever call.

## Reading an API

An API call is just a web request that returns data instead of a page. You send a URL with some parameters; you get back JSON, which maps neatly onto Python dictionaries and then into a dataframe.

```python
import requests
import pandas as pd

url = "https://api.open-meteo.com/v1/forecast"
params = {"latitude": 52.09, "longitude": 5.12,      # Utrecht
          "hourly": "temperature_2m", "forecast_days": 1}

try:
    r = requests.get(url, params=params, timeout=10)   # (1)!
    r.raise_for_status()                               # fail loudly on errors
    payload = r.json()
    hourly = payload["hourly"]
    df = pd.DataFrame({"time": hourly["time"],
                       "temp_c": hourly["temperature_2m"]})
except Exception as e:
    print("Live call failed, using a fallback sample:", e)   # (2)!
    df = pd.DataFrame({"time": ["2026-07-05T00:00", "2026-07-05T01:00"],
                       "temp_c": [17.2, 16.8]})

df.head()
```

1.  `timeout=10` means "give up after 10 seconds": never let a network call hang forever.
2.  Always assume the network can fail. A fallback keeps your notebook runnable offline and during grading, and models the defensive habit real collection needs.

**How it works.** `requests.get` performs the call; `raise_for_status()` turns an HTTP error (a 404, a rate-limit 429) into a Python exception instead of silently returning junk. `r.json()` parses the response into nested dictionaries and lists, and from there it's ordinary pandas: pick the fields you want and build a dataframe. The `try/except` isn't boilerplate; it's the [cleaning mindset](cleaning-messy-data.md) applied at the moment of *intake*: check that what came back is what you expected before you trust it.

Four things make up every call you'll ever write, and it's worth naming them because the documentation of any API will describe them in this order: the **endpoint** (the URL of the resource), the **parameters** (what you're asking for), the **credentials** (a key, if the service requires one), and the **response** (the JSON you get back). Learn to find those four in a set of API docs and you can use a service you've never seen before.

### Reading the status code

`raise_for_status()` turns a failed request into an exception, but *which* failure matters, because the fix differs completely. Every response carries a three-digit status code, and five of them account for nearly everything you'll hit:

| Code | Means | What to do |
|---|---|---|
| `200` | OK | carry on |
| `400` | you sent something malformed | check your parameter names and types against the docs |
| `401` / `403` | not authenticated / not allowed | key missing, wrong, expired, or lacking permission for this endpoint |
| `404` | no such resource | usually a typo in the URL, or an id that doesn't exist |
| `429` | too many requests | you've hit the rate limit — slow down and retry later |
| `5xx` | the server broke | not your fault; wait and retry, and don't hammer it |

```python
r = requests.get(url, params=params, timeout=10)
print(r.status_code, r.reason)     # e.g. 200 OK
if r.status_code == 429:
    print("Rate limited. Retry-After:", r.headers.get("Retry-After"))
```

**How it works.** The distinction that saves you time is **4xx versus 5xx**: a `4xx` means *you* need to change something, a `5xx` means the server does and you should back off. `429` is the one that catches out enthusiastic students, and it's worth noting that many APIs send a `Retry-After` header telling you exactly how long to wait — the polite thing is to read it and obey it.

!!! practice "In Practice — Retry once, politely, then give up"
    Networks fail transiently. A single retry with a pause turns most flaky calls into successful ones, and a bounded loop keeps you from becoming the problem:

    ```python
    import requests, time

    def fetch(url, params=None, attempts=3):
        for i in range(attempts):
            r = requests.get(url, params=params, timeout=10)
            if r.status_code == 200:
                return r.json()
            if r.status_code in (429, 500, 502, 503, 504):   # worth retrying
                wait = int(r.headers.get("Retry-After", 2 ** i))   # (1)!
                print(f"{r.status_code}; waiting {wait}s")
                time.sleep(wait)
                continue
            r.raise_for_status()          # 4xx: retrying won't help, fail loudly
        raise RuntimeError(f"Gave up after {attempts} attempts")
    ```

    1.  `2 ** i` gives 1, 2, 4 seconds — *exponential backoff*, the standard courtesy. Honour the server's own `Retry-After` if it sent one.

    Note what it does *not* retry: a `401` or `404` will never succeed on the second try, so retrying them just wastes everyone's time. Retry what might heal; fail fast on what won't.

!!! practice "In Practice — Fetch once, cache to disk"
    A notebook you re-run twenty times should not call an API twenty times. Save the response the first time and read the file afterwards:

    ```python
    import json, pathlib, requests

    cache = pathlib.Path("data/weather_utrecht.json")
    cache.parent.mkdir(exist_ok=True)

    if cache.exists():
        payload = json.loads(cache.read_text())          # free, instant, offline
    else:
        r = requests.get(url, params=params, timeout=10)
        r.raise_for_status()
        payload = r.json()
        cache.write_text(json.dumps(payload))            # keep it
    ```

    Three things this buys you: your notebook runs offline and during grading, you stop burning rate limit on data you already have, and — most important for provenance — the cached file *is* your snapshot. An API's answer changes over time, so the only way your analysis stays reproducible is if you keep what it said on the day you asked. Commit the cache file alongside your notebook, and note the fetch date.

## Asking a language model, through the same door

An AI model is reached by exactly the mechanics above: an endpoint, a key, a request, a JSON response. Seeing that makes large language models much less mysterious, and it's your first contact with the material of Block C.

```python
import os, requests

API_KEY = os.environ.get("GROQ_API_KEY")     # (1)!
url = "https://api.groq.com/openai/v1/chat/completions"

try:
    r = requests.post(
        url,
        headers={"Authorization": f"Bearer {API_KEY}"},    # (2)!
        json={"model": "llama-3.3-70b-versatile",
              "messages": [{"role": "user",
                            "content": "In one sentence: what is a dataframe?"}]},
        timeout=20,
    )
    r.raise_for_status()
    reply = r.json()["choices"][0]["message"]["content"]    # (3)!
except Exception as e:
    print("Live call failed, using a canned reply:", e)
    reply = "A dataframe is a table of rows and columns you can compute with."

print(reply)
```

1.  Never paste a key into your notebook. Read it from an environment variable (or your platform's secrets store) so it doesn't end up in your git history.
2.  Most authenticated APIs expect the key in a header, not in the URL. Keys in URLs leak into logs and browser history.
3.  The interesting content is buried a few levels down in the JSON. Print `r.json()` once and read the shape before you index into it.

!!! practice "In Practice — Model names go stale faster than anything else in this book"
    `llama-3.3-70b-versatile` is an identifier that will be retired, probably sooner than you expect: providers deprecate models on a timescale of months. If a call returns a `404` or a `400` complaining about the model, that's what happened. Check the provider's current model list rather than debugging your code, and — the transferable habit — **record the exact model name and the date in your provenance note**, because a language model's output is not reproducible across versions. "I asked an AI" is not a method; "I asked `llama-3.3-70b-versatile` on 12 September 2026, with this prompt" is.

**How it works.** It's a `POST` rather than a `GET`, because you're sending a body (your messages) rather than just asking for a resource, and the key travels in a header. Otherwise this is the same four parts as the weather call. Note the last point especially: an API's response *shape* is something you discover by looking, not something you assume — print the raw JSON first, then write the line that extracts what you need.

!!! practice "In Practice — Keys, secrets, and the git history you can't take back"
    An API key is a password. Three rules keep you out of trouble: never type one directly into a notebook cell, never commit one (a key pushed to GitHub is compromised, even if you delete it in the next commit — the history keeps it), and never share a notebook without checking its outputs for a leaked key. Read keys from the environment, and if you ever expose one, revoke and regenerate it rather than hoping nobody looked.

## Collecting more than one page

Most APIs hand back results in **pages** rather than all at once, so collecting a full dataset means politely looping:

```python
# Sketch of paginated collection (pseudo-real):
all_rows = []
for page in range(1, 6):                     # first five pages only
    resp = requests.get(url, params={**params, "page": page}, timeout=10)
    if not resp.ok:
        break
    all_rows.extend(resp.json().get("results", []))
    # time.sleep(1)  # be kind: pause between requests
collected = pd.DataFrame(all_rows)
```

Note the bound on the loop (`range(1, 6)`) and the commented-out pause. Both are courtesies with teeth: an unbounded loop hammering a public endpoint is at best rude and at worst indistinguishable from an attack. **Read the terms** of any service you collect from, respect its stated **rate limits**, and cache what you've already fetched rather than asking twice.

!!! case "Case study — replace the synthetic weather with the real thing"
    In [Chapter 4](combining-datasets.md) the bike-share's `weather` table is hand-made. You can collect it for real instead, and join it in exactly the same way. Pull the daily weather for the trip dates from a public API:

    ```python
    import requests, pandas as pd
    try:
        r = requests.get("https://archive-api.open-meteo.com/v1/archive", timeout=10,
                         params={"latitude": 52.09, "longitude": 5.12,
                                 "start_date": "2026-05-01", "end_date": "2026-05-03",
                                 "daily": "temperature_2m_max,precipitation_sum"})
        r.raise_for_status()
        d = r.json()["daily"]
        weather = pd.DataFrame({"date": pd.to_datetime(d["time"]),
                                "temp_c": d["temperature_2m_max"],
                                "rain_mm": d["precipitation_sum"]})
    except Exception as e:
        print("Live call failed, using fallback:", e)
        weather = pd.DataFrame({"date": pd.to_datetime(["2026-05-01","2026-05-02","2026-05-03"]),
                                "temp_c": [17.5, 14.2, 21.0], "rain_mm": [0.0, 6.4, 0.0]})
    weather
    ```

    The enrichment you'd otherwise fake to learn joins becomes a real, sourced column.

## Handling personal data

The moment your dataset is about *people*, a heavier set of duties applies, and "it was technically public" is not a defence. It helps to distinguish two dangers. **Personally identifiable information (PII)**, such as names, emails, phone numbers, precise location, or national ID numbers, identifies someone directly. **Quasi-identifiers**, such as a postcode, a birth date, or a job title, identify no one alone but pin down an individual *in combination* — the mosaic effect, which you'll meet again when you start joining tables in [Chapter 4](combining-datasets.md). Both need care, and the second is the one people forget.

A practical stance rests on three moves. **Minimise:** collect only the fields your question actually needs, every extra personal column is a liability to store and protect, not an asset. **Respect purpose and consent:** data given for one purpose shouldn't be silently repurposed, and pulling people's public posts into a research dataset is exactly such a repurposing, whatever the terms allow. **Pseudonymise or anonymise:** replace names with meaningless IDs (pseudonymisation), or aggregate and blur until individuals disappear (anonymisation), while remembering that true anonymisation is genuinely hard, because a determined join can re-identify what looked anonymous.

!!! origins "Origins — Why European data law starts from 'collect less'"
    Data-protection law was born of memory. Nazi Germany's censuses were tabulated on Hollerith punched-card machines, and meticulous population registries, the Netherlands' among the most complete, helped identify people for persecution. After the war, that history hardened into a principle: the safest data is the data you never collected. The German state of Hesse passed the first data-protection act in 1970, a lineage running straight to today's **GDPR**, with its insistence on data minimisation, purpose limitation and a right to erasure. When privacy law feels burdensome, it's worth remembering what it is a reaction to.

!!! practice "In Practice — A minimum-viable privacy check"
    Before collecting data about people, answer four questions in your provenance note: (1) Do I actually need each personal field, or am I collecting it because I can? (2) Would the people in this dataset be surprised or troubled to find themselves in it? (3) Can I pseudonymise now, swap names for IDs at the point of collection? (4) Could this dataset, joined to something public, re-identify someone? If any answer worries you, collect less. The most defensible personal data is the personal data you chose not to gather.

!!! further "Going Further — The mechanics you'll actually hit"
    Three things separate a toy collector from a usable one. *Pagination:* loop politely through result pages rather than grabbing the first screen. *Shape validation:* an API response can change or arrive malformed, so check the fields you expect are present before trusting it. *Fallbacks:* assume the network will sometimes fail and degrade gracefully instead of crashing (the `try/except` above). None of this is advanced, but it's the difference between code that works in the demo and code that survives contact with the real web.

An API is the good case: someone built a door and told you how to knock. Keep a record of exactly what you asked for and when, because an API response is a snapshot of a moment that you cannot reproduce later. Next, with a table you sourced and can vouch for, we start asking it questions — beginning with the humblest and most reliable move in all of data work: looking before you conclude.

!!! exercise "Practice on DataCamp"
    The exercises for this chapter live on DataCamp. Short course, and it maps almost one-to-one onto this chapter.

    - [Introduction to APIs in Python](https://app.datacamp.com/learn/courses/introduction-to-apis-in-python) — **Ch. 1 Making API Requests with Python** and **Ch. 2 More API Request Concepts.**
