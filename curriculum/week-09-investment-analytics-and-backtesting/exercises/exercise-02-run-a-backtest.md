# Exercise 2 — Run a Backtest with Costs

**Goal:** Turn Exercise 1's top-3 composite signal into equal-weight monthly positions, run it through a from-scratch backtest engine with realistic transaction costs, and produce a daily net-return series and equity curve.

**Estimated time:** 90 minutes.

## Setup

You need Exercise 1's Task 7 output (rebalance_date, ticker, composite_rank) either re-run live or saved to a CSV/table. You also need the full `daily_prices` table pulled into a wide pandas DataFrame (one column per ticker).

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg2://localhost/crunch_capital")
prices_long = pd.read_sql(
    "SELECT ticker, price_date, close_price FROM daily_prices ORDER BY price_date",
    engine, parse_dates=["price_date"]
)
prices_wide = prices_long.pivot(index="price_date", columns="ticker", values="close_price").sort_index()
print(prices_wide.shape)   # expect (504, 7)
```

Create `exercise_02.py`. Build the pieces in this order — each is testable on its own before you chain them.

## Tasks

1. **Convert the signal into equal weights.** Write `size_positions(signal_df)` that takes Exercise 1's top-3 table and returns a DataFrame with columns `rebalance_date, ticker, weight`, where every ticker in a given `rebalance_date`'s top-3 gets `weight = 1/3`. *(If a rebalance date somehow has fewer than 3 names — it won't this week, since top-N always fills exactly 3 — handle it anyway: weight = 1/count.)*

2. **Build the weight matrix.** Write `build_weight_matrix(prices_wide, positions)` that produces a DataFrame shaped like `prices_wide` (same index, same columns) where each cell is the ticker's target weight, **forward-filled from each rebalance date until the next one, taking effect strictly AFTER the rebalance date** — follow Lecture 2 Section 4's `mask` pattern exactly. Print the weight matrix for one month and confirm by eye that exactly 3 columns are non-zero and each non-zero cell reads `0.3333...`.

3. **Compute gross returns.** Write `compute_gross_returns(prices_wide, weight_matrix)` returning a single `pd.Series` of daily portfolio returns, using **yesterday's weights** against **today's price returns** (`weight_matrix.shift(1) * prices_wide.pct_change()`, summed across columns). Explain in a code comment why the `.shift(1)` is there — don't just copy Lecture 2's code without being able to say why in your own words.

4. **Compute turnover and cost.** Write `compute_cost(weight_matrix, cost_bps=10.0)` returning a daily cost `pd.Series`: the sum of absolute weight changes (`weight_matrix.diff().abs().sum(axis=1)`) times `cost_bps / 10000`, non-zero only on days the portfolio actually changed.

5. **Assemble the full engine.** Write `run_backtest(prices_wide, positions, cost_bps=10.0)` that calls steps 1–4 in order and returns a DataFrame with columns `gross_return, cost, net_return, equity_curve` (equity curve starting at 1.0, via `(1 + net_return.fillna(0)).cumprod()`).

6. **Run it and inspect.** Run the full backtest on Exercise 1's signal. Print the first rebalance date's weight change, the total cost paid over the whole backtest (in cumulative return terms), and the final equity curve value.

## Expected result (spot checks)

- The weight matrix, on any date strictly between two rebalance dates, should have **exactly 3 non-zero columns**, each `≈0.3333`, summing to `1.0`.
- `gross_return` on the very first day of the whole series should be `NaN` or `0` (there's no "yesterday" to shift from) — decide how you handle this and note it in a comment.
- Cumulative cost paid, summed over the ~11 rebalances that actually occur in 2025 (since 12-1 momentum only starts producing signals in January 2025), should be a small positive number in the low single-digit percent of total capital — sanity-check this against the arithmetic in Lecture 2 Section 3 (roughly `10bps × 2 × 11 rebalances ≈ 220bps ≈ 2.2%` if turnover is close to 100% each month; your actual number will differ since not every rebalance fully replaces the book).
- `equity_curve.iloc[-1]` should be a number reasonably close to `1.0` (neither wildly above nor catastrophically below) — this is a 7-name, 2-year, cost-aware backtest on synthetic data, not a strategy expected to look spectacular.

## Done when…

- [ ] All 6 functions run in sequence without error and each has been printed/inspected individually, not just chained blindly.
- [ ] You can point to the exact line that prevents look-ahead bias and explain it without looking at Lecture 2.
- [ ] Your cost series is zero on every non-rebalance day and non-zero only when the portfolio's weights actually changed.
- [ ] The final `result` DataFrame (from `run_backtest`) is saved to `backtest_result.csv` for Exercise 3 to consume.

## Stretch

- Re-run with `cost_bps=0` (frictionless) and `cost_bps=25` (a pessimistic, illiquid-stock assumption) and compare the final equity curve value across all three. How much does the strategy's terminal value swing just from the cost assumption alone?
- Add a `risk_free_bps_per_day` parameter that credits idle cash (the fraction of the book not in any of the top-3) with a small daily return, and see how much it matters given this strategy is always ~100% invested (top-3 with equal weight leaves no idle cash this week — note that in a comment, and think about when it *would* matter).

## Submission

Commit `exercise_02.py` and `backtest_result.csv` to your portfolio under `c35-week-09/exercise-02/`.
