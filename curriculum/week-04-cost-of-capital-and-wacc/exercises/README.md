# Week 4 — Exercises

Three guided exercises, ~1–1.5 hours each. **Type every query and every line of Python yourself** — don't copy-paste out of the lectures. Reading a formula and reproducing it from a live table are different skills, and only the second one sticks.

1. **[Exercise 1 — CAPM cost of equity](exercise-01-capm-cost-of-equity.md)** — compute cost of equity by hand, in SQL, and in Python for several beta/rate scenarios.
2. **[Exercise 2 — Regress a beta](exercise-02-regress-a-beta.md)** — pull `stock_prices` into pandas, compute monthly returns, and regress CRNM against MKT yourself.
3. **[Exercise 3 — Assemble a WACC](exercise-03-assemble-a-wacc.md)** — price Crunch Machine Co.'s debt from `debt_tranches` and assemble its full WACC end to end.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) — `SELECT COUNT(*) FROM stock_prices;` returns **122**, `SELECT COUNT(*) FROM debt_tranches;` returns **3**, `SELECT COUNT(*) FROM capital_structure;` returns **1**.
- Your Python environment (from Week 1) has `pandas`, `numpy`, and `sqlalchemy` installed and can connect to `crunch_capital`.
- You have a shell open: `psql crunch_capital` (PostgreSQL) or `sqlite3 crunch_capital.db` (SQLite).

## Suggested workflow

- Open the exercise file beside your terminal/editor.
- For each task, **write the formula down first**, then the code, then run it and check against the "Expected" note.
- If a number surprises you, stop and figure out *why* before moving on — that confusion is the lesson.
- Save your work: SQL in a `solutions.sql`, Python in a `solutions.py` (or a notebook), one section per exercise. You'll commit both.

## Note on precision

Money and rates are unforgiving of rounding. Keep at least 4 decimal places on rates (`0.1046`, not `0.10`) until your final answer, and only round for display. A rate rounded too early can shift a multi-hundred-million-dollar WACC calculation by real dollars once you scale it up.
