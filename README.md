# Rolling Loop forecasts

The public, timestamped log of every Rolling Loop forecast and its score.

- `forecasts.csv`: one row per forecast (opening weekends, streaming dates, awards), with its
  p10/p50/p90 range or probability, the model version and the time it was made.
- `scores.csv`: one row per scored forecast, with the actual result, its source and the error.
- `METHOD.md`: the published forecasting and scoring rules.

Rows are append-only: no row is ever edited or deleted. A correction is a new row whose note says
`supersedes <id>`. All timestamps are UTC.

This repository is updated automatically from the Rolling Loop source repo, and every update is
checked to only add rows. Its git history is the audit trail: each commit shows exactly when a
forecast became public.

Method: https://rollingloop.ca/method.html

This product uses the TMDB API but is not endorsed or certified by TMDB.
