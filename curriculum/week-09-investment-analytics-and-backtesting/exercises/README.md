# Week 9 — Exercises

Three guided exercises, ~1–1.5 hours each, that build on each other in order: encode a signal, run it through a backtest with costs, then score the result. **Type every query and every line of Python yourself** — don't copy-paste. By the end of Exercise 3 you'll have a working, if simple, backtest engine you built with your own hands.

1. **[Exercise 1 — Encode a signal](exercise-01-encode-a-signal.md)** — build a composite momentum + value signal table in SQL against `daily_prices` and `fundamental_snapshot`.
2. **[Exercise 2 — Run a backtest](exercise-02-run-a-backtest.md)** — turn Exercise 1's signal into monthly-rebalanced positions and a cost-adjusted daily return series in Python.
3. **[Exercise 3 — Performance metrics](exercise-03-performance-metrics.md)** — compute CAGR, annualized volatility, Sharpe ratio, maximum drawdown, and hit rate from Exercise 2's return series.

## Before you start

- You've completed all three lectures.
- You ran the setup in the [week README](../README.md): `SELECT COUNT(*) FROM daily_prices;` returns **3528**, and `SELECT COUNT(*) FROM fundamental_snapshot;` returns **7**.
- You have a Python environment with `pandas`, `numpy`, and either `sqlalchemy`+`psycopg2` (Postgres) or `sqlite3` (standard library, SQLite).
- You have a database shell open too — `psql crunch_capital` or `sqlite3 crunch_capital.db` — for writing and checking the SQL parts before you wire them into Python.

## Suggested workflow

- Write and test each SQL query directly in your database shell first — confirm row counts and spot-check a few tickers by eye — *then* wrap it in Python (`pd.read_sql`).
- Keep every exercise's code in its own file (`exercise_01.py`/`.sql`, `exercise_02.py`, `exercise_03.py`) — Exercise 2 imports the signal logic from Exercise 1, and Exercise 3 imports the backtest engine from Exercise 2.
- Print intermediate results as you go (a signal table, a positions table, a returns table) and eyeball them for sanity before trusting a final number. A backtest with a silent bug rarely announces itself — it just quietly reports a number that's wrong.
- If a number surprises you (a Sharpe ratio that looks too good, a drawdown of exactly 0%), stop and figure out why before moving on — Lecture 3 exists because that instinct is the single most valuable habit in this entire discipline.

## Note on reproducibility

Because `daily_prices` was generated with a fixed random seed (`35090`), every student's numbers should match exactly, to the penny, given identical code. If your Sharpe ratio or drawdown doesn't match a classmate's, the bug is in your code, not in randomness — a useful property for debugging.
