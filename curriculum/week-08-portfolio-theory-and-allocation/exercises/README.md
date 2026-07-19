# Week 8 — Exercises

Three guided exercises, ~1.5 hours each. **Type every query and every line of Python yourself** — don't copy-paste out of the lectures. Reproducing a covariance matrix and a closed-form optimization from a live table, on your own, is what makes the linear algebra stick.

1. **[Exercise 1 — Covariance from returns](exercise-01-covariance-from-returns.md)** — pull `asset_prices` into pandas, compute monthly returns, and build the full 6×6 annualized covariance and correlation matrices.
2. **[Exercise 2 — Trace the frontier](exercise-02-trace-the-frontier.md)** — solve the mean-variance optimization in closed form across a grid of target returns and trace the efficient frontier.
3. **[Exercise 3 — Max-Sharpe portfolio](exercise-03-max-sharpe-portfolio.md)** — solve for the global-minimum-variance and maximum-Sharpe (tangency) portfolios and compare them to equal-weight.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) — `SELECT COUNT(*) FROM asset_prices;` returns **427**, `SELECT COUNT(*) FROM capital_market_assumptions;` returns **1**.
- Your Python environment (from Week 1) has `pandas`, `numpy`, and `sqlalchemy` installed and can connect to `crunch_capital`.
- You have a shell open: `psql crunch_capital` (PostgreSQL) or `sqlite3 crunch_capital.db` (SQLite).

## Suggested workflow

- Open the exercise file beside your terminal/editor.
- For each task, **write the formula down first** (in matrix notation), then the code, then run it and check against the "Expected" note.
- If a number surprises you — a negative weight, a Sharpe ratio that seems too high or too low — stop and figure out *why* before moving on. That confusion is the lesson, especially in a week this matrix-heavy.
- Save your work: Python in `solutions.py` (or a notebook), one section per exercise. You'll commit it.

## Note on precision

Weights and betas are unforgiving of rounding, exactly like WACC was in Week 4. Keep at least 4 decimal places on weights (`0.2347`, not `0.23`) and betas until your final answer, and only round for display. A weight rounded too early, multiplied through a $6\times6$ matrix inversion, can visibly shift your reported portfolio volatility.

## A note on `numpy.linalg.inv`

Every closed-form solution this week inverts the covariance matrix, `Sigma_inv = np.linalg.inv(Sigma)`. If your matrix is built correctly (six real assets, a genuine sample covariance matrix from 60 months of returns), this will always succeed — a real sample covariance matrix built from more time periods than assets is essentially guaranteed to be invertible. If you ever get a `LinAlgError: Singular matrix`, the almost-certain cause is a bug in how you built `Sigma` (e.g., accidentally including a duplicate column, or a column of all-identical values) — debug the matrix itself before assuming numpy is at fault.
