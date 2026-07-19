# Week 7 — Exercises

Three guided exercises, ~60–90 min each. Together they build the full pipeline this week's mini-project depends on: a cash-flow forecast (Exercise 1), a terminal value with a built-in cross-check (Exercise 2), and a comps analysis (Exercise 3). Do them in order — Exercise 2 discounts the exact forecast Exercise 1 builds, and the mini-project assembles all three.

1. **[Exercise 1 — Project free cash flows](exercise-01-project-free-cash-flows.md)** — derive revenue, EBITDA, and unlevered FCF from `revenue_forecast`'s drivers, in SQL and Python.
2. **[Exercise 2 — Terminal value methods](exercise-02-terminal-value-methods.md)** — compute terminal value by perpetuity growth and by exit multiple, discount both, and cross-check each method's implied parameter against the other's.
3. **[Exercise 3 — Build a comp set](exercise-03-build-a-comp-set.md)** — compute EV and multiples from `trading_comps`, summarize them defensibly, and apply the result to Crunch Machine Co.

## Before you start

- You've read all three lectures.
- You ran this week's seed from the [README](../README.md) and all four sanity-check counts (`1`, `5`, `6`, `5`) came back correct.
- You have a Postgres or SQLite shell open, and a Python environment with `pandas`, `numpy`, and `sqlalchemy` installed and activated.

## Suggested workflow

- Work Exercise 1 fully before starting Exercise 2 — Exercise 2's discounting depends on Exercise 1's forecast being correct. If your Exercise 1 numbers don't match this week's Lecture 1 table, fix them before moving on; an error there propagates into everything after it.
- For every calculation, write the formula or query before running it, then check your output against the "Expected results" section — a valuation model that "looks about right" but doesn't match a known-correct check is not a trustworthy model.
- Save your work: `solutions.sql` for SQL tasks, `solutions.py` for Python tasks, one section per exercise under a numbered comment. You'll reuse and extend these files in the mini-project.

## Note on precision

Financial models compound small rounding differences fast — five years of forecasting, then discounting, then applying a multiple, can turn a one-cent rounding choice into a few-hundred-dollar difference in a large enterprise value. The "Expected results" in each exercise give you a value with enough tolerance (usually ±0.5%) to accommodate reasonable rounding-order differences — if you're within that band, you're right. If you're off by more than that, you have a real bug, not a rounding artifact.
