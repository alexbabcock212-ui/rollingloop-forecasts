# Method: how Rolling Loop forecasts are made, logged and scored

*These rules take effect with the first published forecast on Wednesday, November 4, 2026, and will not be changed retroactively. Any future change applies only to forecasts made after the change is announced here, with its date.*

Rolling Loop publishes forecasts before the outcome is known, keeps every one of them, and scores them in public. This page sets out the rules in advance so that nobody, including us, can move the goalposts later.

## 1. What we forecast

- **Opening weekend box office** for every film opening in **2,000 or more theatres in the US and Canada**.
- The number we forecast is the **Friday-to-Sunday domestic gross (US and Canada)**, in US dollars, as the industry reports it. Thursday-night previews count toward Friday.
- Films opening on a holiday Wednesday are still forecast for their Friday-to-Sunday weekend. A five-day figure may be noted but is not scored.

## 2. Which films get a forecast

Every film opening in **2,000 or more theatres in the US and Canada** that weekend gets a forecast, whatever we expect it to do. We do not skip a film we feel unsure about. A film is left out only if an input the model needs is missing: a theatre count from a published source, an English Wikipedia article and its page views, the film's TMDB record, or, for a sequel, the earlier film's opening. We never estimate a missing input. Withdrawals after a forecast is made are covered in section 5.

## 3. When forecasts lock

- Forecasts are published by **12:00 p.m. Eastern on the Wednesday** before opening (Tuesday for Wednesday openers).
- Each forecast carries a **UTC timestamp** and is written to the public forecast log at the same moment.
- No forecast for a film is accepted once its Thursday previews have begun.

We may also log earlier forecasts, 28, 14 and 7 days before opening. They are scored the same way and shown separately; the headline scoreboard uses only the Wednesday forecast.

## 4. What a forecast contains

- **P50**: our central estimate. The actual should land above it half the time and below it half the time.
- **P10 to P90**: an 80% range. If we are well calibrated, about 8 in 10 actuals land inside it.
- **Model version**: which model produced the number, so changes to the model can be tracked.

A forecast is a probability statement, not a promise. A single miss outside the range is expected about 2 times in 10. A pattern of misses is what matters, and the scoreboard shows the pattern.

For yes/no questions, such as whether a film will be nominated for an award, a forecast is a single probability instead of a range.

## 5. The forecast log is append-only

- The log lives in a public GitHub repository. Its full history, including every timestamp, is visible to anyone.
- Rows are **never edited or deleted**.
- A correction is a **new row** marked `supersedes <id>`. The original stays visible, and **only the original forecast is scored**.
- If a film's release moves or it drops below 2,000 theatres after we forecast it, we add a row marking the forecast **withdrawn**, with the reason. Withdrawn forecasts stay in the log but are not scored.
- Before a film's lock, a forecast can be withdrawn for any reason. The withdrawal and its reason stay in the public log. After the lock, a forecast can only be withdrawn if the release moves or the film opens in fewer than 2,000 theatres, with a published source.

## 6. What we score against

- The **studio-reported weekend actual published on Monday**, not Sunday's estimate.
- Each actual is logged with its source and the time we recorded it.
- If a studio later revises the Monday figure, we keep the Monday figure for scoring and note the revision.

## 7. How we score

For each forecast:

- **P50 error**: how far the central estimate was from the actual, as a percentage of the actual.
- **Range hit**: whether the actual landed inside the P10 to P90 range.
- **Brier score** (yes/no forecasts only): (probability − outcome)², where outcome is 1 if it happened and 0 if not. Lower is better; always guessing 50% scores 0.25.

Across the season, the scoreboard shows:

- **Median P50 error** for all scored forecasts.
- **Range hit rate**, which should be close to 80%. Much higher means our ranges are too wide; much lower means we are overconfident.
- **Comparisons**: where a prediction market (Kalshi or Polymarket) lists the same opening, the This Weekend page shows the market's implied median beside our forecast, from the latest complete market snapshot taken before the lock day. When a newer model version goes live, the older version keeps running alongside it so the two can be compared on the same films.

**Every forecast is scored, including the misses.** We do not remove or hide forecasts that went badly.

## 8. Data sources

- Film details and release dates: TMDB. *This product uses the TMDB API but is not endorsed or certified by TMDB.*
- Attention data: Wikipedia page views from the Wikimedia API.
- Market prices: public data from Kalshi and Polymarket.
- Weekend actuals: studio-reported figures, as published.
- Training data for the opening-weekend model: past opening grosses and theatre counts from English Wikipedia: film articles and its lists of weekly number-one films. Where Wikipedia gives no theatre count, or a figure that published reports contradict, we entered it by hand from a published news report and kept the link to that report. Films that opened in a few theatres before going wide, and releases whose studio did not report grosses, are left out.

We do not use scraped data from sites that do not permit it.

## 9. Conflicts of interest

- **We do not trade** on any prediction market for a film or award we forecast.
- Rolling Loop is an independent project. It is not affiliated with any studio, distributor, exhibitor or prediction market, and receives no payment from them for forecasts.
- Nothing on this site is financial or investment advice.

## 10. File formats

The public log uses two CSV files.

The public, append-only copy of `forecasts.csv` and `scores.csv` is at https://github.com/alexbabcock212-ui/rollingloop-forecasts, and its commit history shows when each row became public.

**`forecasts.csv`** (one row per forecast; append-only)

| column | meaning |
|---|---|
| `forecast_id` | unique id, e.g. `opening_weekend-2026-11-06-<tmdb_id>-h2-ow_v0_comps_clean` |
| `created_utc` | when the forecast was logged (ISO 8601, UTC) |
| `model_version` | e.g. `ow_v0_comps_clean` |
| `target` | `opening_weekend`, or a yes/no target such as `oscar_bp_nom` |
| `tmdb_id` | TMDB film id |
| `title` | film title |
| `event_date` | the date the outcome happens (Friday of the weekend, or the announcement date) |
| `horizon_days` | days between `created_utc` and `event_date` |
| `p10`, `p50`, `p90` | US dollars, for dollar targets only; empty otherwise |
| `prob` | probability from 0 to 1, for yes/no targets only; empty otherwise |
| `status` | `active`, or `withdrawn` |
| `supersedes` | id of the row this corrects, or empty |
| `inputs` | short JSON of the model's main inputs |
| `note` | short reason for a correction or withdrawal, or empty |

**`scores.csv`** (one row per scored forecast)

| column | meaning |
|---|---|
| `forecast_id` | matches `forecasts.csv` |
| `actual_usd` | Monday studio-reported weekend gross |
| `outcome` | 1 or 0 for yes/no targets; empty otherwise |
| `actual_source` | where the actual came from |
| `actual_logged_utc` | when we recorded the actual |
| `p50_error_pct` | (P50 − actual) ÷ actual × 100 |
| `in_range` | `true` if P10 ≤ actual ≤ P90 |
| `brier` | (prob − outcome)², for yes/no targets; empty otherwise |
| `scored_utc` | when the score was written |
