# Method: how Rolling Loop forecasts are made, logged and scored

*These rules take effect with the first published forecast on Wednesday, November 4, 2026, and will not be changed retroactively. Any future change applies only to forecasts made after the change is announced here, with its date.*

Rolling Loop publishes forecasts before the outcome is known, keeps every one of them, and scores them in public. This page sets out the rules in advance so that nobody, including us, can move the goalposts later.

## 1. What we forecast

- **Opening weekend box office** for every film opening in **2,000 or more theatres in the US and Canada**.
- The number we forecast is the **Friday-to-Sunday domestic gross (US and Canada)**, in US dollars, as the industry reports it. Thursday-night previews count toward Friday.
- Films opening on a holiday Wednesday are still forecast for their Friday-to-Sunday weekend. A five-day figure may be noted but is not scored.

## 2. Which films get a forecast

Every film opening in **2,000 or more theatres in the US and Canada** that weekend gets a forecast, whatever we expect it to do. We do not skip a film we feel unsure about. A film is left out only if an input the model needs is missing: a theatre count from a published source, an English Wikipedia article and its page views, the film's TMDB record, or, for a sequel, the earlier film's opening. We never estimate a missing input. The 2,000-theatre cut is applied to the theatre count we enter when we make the forecast, taken from a published source, with a link to it stored in the forecast. A film counted under 2,000 theatres at that point gets no forecast, and once the lock has passed we do not add one if the count turns out higher. Withdrawals after a forecast is made are covered in section 5.

**Change, November 9, 2026** (applies to films opening from Friday, November 20, 2026; adds forecasts for limited releases; the rules for films in 2,000 or more theatres do not change):

- **Limited releases get a forecast too.** A film whose theatre count is from 600 to 1,999 when we make the forecast gets an opening-weekend forecast of the same Friday-to-Sunday figure. Sections 2 to 7 apply to it in the same way: the count comes from a published source with the link stored in the forecast, and a film is left out only if an input the model needs is missing. A film counted under 600 theatres gets no forecast.
- **A separate model name.** Limited-release forecasts are logged as `ow_lim_v1_ridge_wiki_clean`. It is the same fitted model as `ow_v1_ridge_wiki_clean`, which was already trained on every film that opened in 600 or more theatres. Only the 80% range differs: it is wider, by a factor of 1.106 on the log scale, because the model's misses on limited releases are wider. The comps benchmark is logged beside each forecast, as for wide films. Other films opening the same weekend still count as competition only if they are in 2,000 or more theatres.
- **The backtest.** On the same walk-forward test (2018–19 and 2022–25), 109 films opened in 600 to 1,999 theatres. The median miss was 33.7%, against 40.7% for the comps benchmark. 77% of actuals landed inside the unwidened 80% range. With the range widened by a factor refitted each year from earlier limited releases only, 83% did. The 1.106 factor used for live forecasts was measured on all 109 films. A model trained only on limited releases did worse than the benchmark (42.2% against 40.5% on the 103 films it could forecast) and is not used.
- **Scored separately.** Limited-release forecasts are scored by the rules in sections 6 and 7 and shown in their own scoreboard table. The headline scoreboard covers only films forecast in 2,000 or more theatres, as before.
- **One forecast model per film per forecast window.** A film is forecast as a wide or a limited release by the count we enter at the time. If a later count crosses 2,000 theatres, the existing forecast stands, and we do not add one from the other model in the same forecast window (the lock week, or 4 to 13, 14 to 27, or 28 or more days before the opening Friday).
- **Withdrawals.** After the lock, a limited-release forecast can be withdrawn only if the release moves or the film opens in fewer than 600 theatres, with a published source. A limited-release forecast whose film opens in 2,000 or more theatres is kept and scored in the limited-release table. A wide forecast whose film drops below 2,000 theatres is still withdrawn under section 5; it is not rescored as a limited release.
<!-- source: docs/backtest-limited.md; models/ow_lim_v1_ridge_wiki_clean.json (range_scale_k, fit_date); pipeline/forecast_ow.py (LIMITED_MIN, LIMITED_FROM, universe, log_rows); pipeline/withdraw.py; pipeline/score.py (scoreboard "limited") -->

## 3. When forecasts lock

- Forecasts are published by **12:00 p.m. Eastern on the Wednesday** before opening (Tuesday for Wednesday openers).
- Each forecast carries a **UTC timestamp** and is written to the public forecast log at the same moment.
- No forecast for a film is accepted once its Thursday previews have begun.

We may also log earlier forecasts, 28 or more, 14 to 27, or 4 to 13 days before the opening Friday. They are scored the same way and shown separately; the headline scoreboard uses only the forecast logged three days or fewer before the opening Friday, which is the Wednesday forecast (the Tuesday forecast for a Wednesday opener).

**Change, November 4, 2026** (applies to forecasts made from November 5, 2026; no forecast had been published before it):

- The lock moves to **12:00 p.m. Eastern on the day before the film opens**: Thursday for a Friday opener, Wednesday for a Thursday opener, and Tuesday for a Wednesday opener. Each film's forecast is published by its own lock, and the forecast and withdrawal rules above that refer to the lock use this time.
- The opening day is the film's US release date on TMDB. Where TMDB's date is wrong, we use the date from a published source and store a link to it in the forecast. A date set this way moves only the lock, never a model input.
- The model's inputs do not move. Page views are still counted up to the Tuesday before the opening Friday (the Monday when TMDB lists the film as opening on the Wednesday), as in the data the model was trained on, and no other input depends on the lock.
- The headline scoreboard still uses only the forecast logged three days or fewer before the opening Friday, which is now the forecast made by the film's lock.
- The market comparison in section 7 uses the latest complete market snapshot taken before the day of this lock.

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
- Before a film's lock, a forecast can be withdrawn for any reason. The withdrawal and its reason stay in the public log. After the lock, a forecast can only be withdrawn if the release moves or the film opens in fewer than 2,000 theatres, with a published source. Once any forecast for a film's weekend has been scored, none of that film's forecasts can be withdrawn, for any reason; a scored miss stays on the scoreboard.

## 6. What we score against

- The **studio-reported weekend actual published on Monday**, not Sunday's estimate.
- Each actual is logged with its source and the time we recorded it.
- If a studio later revises the Monday figure, we keep the Monday figure for scoring.

## 7. How we score

For each forecast:

- **P50 error**: how far the central estimate was from the actual, as a percentage of the actual.
- **Range hit**: whether the actual landed inside the P10 to P90 range.
- **Brier score** (yes/no forecasts only): (probability − outcome)², where outcome is 1 if it happened and 0 if not. Lower is better; always guessing 50% scores 0.25.

Across the season, the scoreboard shows:

- **Median P50 error** for all scored forecasts.
- **Range hit rate**, which should be close to 80%. Much higher means our ranges are too wide; much lower means we are overconfident.
- **Comparisons**: where we have matched a prediction market (Kalshi or Polymarket) to the opening, the This Weekend page shows the market's implied median beside our forecast, from the latest complete market snapshot taken before the lock day. When a newer model version goes live, the older version keeps running alongside it so the two can be compared on the same films.

**Every forecast is scored, including the misses.** We do not remove or hide forecasts that went badly.

## 8. Data sources

- Film details and release dates: TMDB. *This product uses the TMDB API but is not endorsed or certified by TMDB.*
- Attention data: Wikipedia page views from the Wikimedia API.
- Market prices: public data from Kalshi and Polymarket.
- Weekend actuals: studio-reported figures, as published.
- Training data for the opening-weekend model: past opening grosses and theatre counts from English Wikipedia: film articles and its lists of weekly number-one films. Where Wikipedia gives no theatre count, or a figure that published reports contradict, we entered it by hand from a published news report and kept the link to that report. Films that opened in a few theatres before going wide, and releases whose studio did not report grosses, are left out.

We do not use scraped data from sites that do not permit it.

**Change, November 5, 2026** (adds a backtest result; no rule changes): We tested the opening-weekend model walk-forward: each month of 2018–19 and 2022–25 was forecast by a model trained only on films that opened before that month (2020–21 are not in the data). On the 442 films that opened in 2,000 or more theatres, the median miss was 31.6%, against 41.7% for the comps benchmark, and 80% of actuals landed inside the 80% range.
<!-- source: docs/horizon-backtest.md, results table row 1 (lock week) -->

**Change, November 9, 2026** (applies to forecasts made from November 10, 2026; no rule changes): We corrected the opening-weekend model's training data and refit the model. A hand check of flagged films against published reports changed 17 opening grosses and 3 theatre counts and added 43 films that opened in 600 to 1,999 theatres, so the training file went from 706 films to 749. On the same walk-forward test, the median miss on the 442 films that opened in 2,000 or more theatres went from 31.6% to 31.2%, against 41.3% for the comps benchmark (41.7% before), and 82% of actuals landed inside the 80% range (80% before). The model keeps its name, `ow_v1_ridge_wiki_clean`. Forecasts logged before November 10, 2026 came from the earlier fit and are not changed. The corrections and the link to each source are listed in `research/task113b/overrides.csv`.
<!-- source: docs/backtest-v1-refit.md, "Before and after" table; models/ow_v1_ridge_wiki_clean.json fit_date -->

**Change, November 9, 2026** (applies to weekend actuals logged from November 9, 2026; no change to how forecasts are made): A weekend actual is entered by hand from a page that publishes the studio-reported figure, and the link to that page is stored with the actual. That page can be a trade report or a box-office site's film page, such as The Numbers or Box Office Mojo. We never collect actuals or other data from these sites by automated means.
<!-- source: forecasts/inputs/README.md (Monday actuals); pipeline/score.py (actual_source) -->

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
