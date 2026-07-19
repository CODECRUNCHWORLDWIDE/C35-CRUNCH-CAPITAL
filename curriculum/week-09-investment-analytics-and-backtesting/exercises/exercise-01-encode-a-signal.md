# Exercise 1 — Encode a Rules-Based Signal

**Goal:** Build a composite momentum + value signal table entirely in SQL — the exact table shape Lecture 1 described — and produce a top-3 long list for every monthly rebalance date across 2024–2025.

**Estimated time:** 60 minutes.

## Setup

Confirm your data is loaded:

```sql
SELECT COUNT(*) FROM daily_prices;          -- must print 3528
SELECT COUNT(DISTINCT ticker) FROM daily_prices;  -- must print 7
SELECT COUNT(*) FROM fundamental_snapshot;  -- must print 7
```

Create a file `exercise_01.sql` and put each task's query under a `-- Task N` comment. You'll also write a short `exercise_01.py` at the end to pull the result into pandas.

## Tasks

1. **Trading-day index.** Write a query that numbers every `(ticker, price_date)` row with `ROW_NUMBER() OVER (PARTITION BY ticker ORDER BY price_date)`. Call the output column `day_num`. *(This is the building block every later task self-joins against.)*

2. **12-1 momentum.** Using the lecture's self-join pattern (`day_num - 252` and `day_num - 21`), compute `momentum_12_1` for every ticker on every date that has at least 252 prior trading days of history. *(Expected: the earliest `signal_date` in your result should be roughly 253 trading days after 2024-01-02 — around January 2025.)*

3. **5-day reversal (for context, not this exercise's final signal).** Compute `return_5d` for every ticker/date with at least 5 prior trading days. You won't use this in the composite score below, but you'll need it working for Exercise 2's stretch goal and Lecture 1's Section 4 makes clear why it lives on a different horizon than momentum.

4. **Value ranks.** From `fundamental_snapshot` (the single `2025-12-31` row per ticker), rank all 7 tickers by `ev_ebitda` ascending (`RANK() OVER (ORDER BY ev_ebitda ASC)`), aliased `value_rank`. *(Expected: `IMW` = rank 1, `VDT` = rank 7.)*

5. **Momentum ranks, per rebalance date.** For each of the monthly rebalance dates (the last trading day of every month from January 2025 through December 2025 — the only months with enough history for 12-1 momentum), rank the 7 tickers by `momentum_12_1` descending, aliased `mom_rank`. *(Hint: get the list of month-end dates with `MAX(price_date)` grouped by `EXTRACT(YEAR FROM price_date), EXTRACT(MONTH FROM price_date)` in Postgres, or `strftime('%Y-%m', price_date)` in SQLite.)*

6. **Composite score.** Join Task 5's momentum ranks to Task 4's value ranks on `ticker`, and compute `composite_rank = (mom_rank + val_rank) / 2.0` for every ticker on every rebalance date. *(Expected: 7 tickers × ~12 monthly rebalance dates in 2025 = roughly 84 rows.)*

7. **Top-3 long list.** From Task 6, for each rebalance date, select the 3 tickers with the lowest `composite_rank` (best combined momentum + value), and output `rebalance_date, ticker, composite_rank`, ordered by date then rank. This is your signal's final output — the table Exercise 2 will consume.

## Wrap it in Python

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg2://localhost/crunch_capital")

# paste Task 7's final query here as a triple-quoted string
signal_sql = """
-- your Task 7 query
"""

signal = pd.read_sql(signal_sql, engine, parse_dates=["rebalance_date"])
print(signal.head(15))
print(f"\nTotal (rebalance_date, ticker) rows: {len(signal)}")
print(f"Unique rebalance dates: {signal['rebalance_date'].nunique()}")
```

## Expected result (spot checks)

- Task 4 → `IMW` ranked 1 (cheapest), `VDT` ranked 7 (richest).
- Task 6 → roughly 84 rows (7 tickers × ~12 months).
- Task 7 → roughly 36 rows (3 tickers × ~12 months), and a given ticker should NOT appear in every single month's top 3 — the composite score should visibly rotate membership over the year as momentum shifts.

## Done when…

- [ ] `exercise_01.sql` has all 7 tasks, each under a `-- Task N` comment, and each runs without error.
- [ ] Task 7's output, loaded into pandas, shows `rebalance_date`, `ticker`, `composite_rank` for roughly 36 rows.
- [ ] You can explain, in one sentence, why Task 6 averages **ranks** rather than raw `momentum_12_1` and `ev_ebitda` values directly.
- [ ] You've eyeballed the top-3 list for at least 3 different months and confirmed it changes over time (if the exact same 3 tickers appear every month, something's likely wrong upstream — check Task 5's ranking direction).

## Stretch

- Add a fourth factor: 5-day reversal, inverted (rank ascending, since you want the biggest *losers*), and blend all three ranks into the composite instead of two. Does the top-3 list change much?
- Instead of top-3, implement a threshold rule (`composite_rank <= 2.5`) and note how the resulting portfolio size varies month to month.

## Submission

Commit `exercise_01.sql` and `exercise_01.py` to your portfolio under `c35-week-09/exercise-01/`.
