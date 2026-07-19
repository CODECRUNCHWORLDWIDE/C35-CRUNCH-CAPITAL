# Week 5 — Exercises

Three guided exercises, ~60–90 min each. **Type every query and every function yourself** — don't copy-paste. Reading a formula and being able to reproduce it under a slightly different scenario are different skills, and this week's quiz and mini-project test the second one.

1. **[Exercise 1 — Value the tax shield](exercise-01-value-the-tax-shield.md)** — compute the annual and present-value interest tax shield for Crunch Machine Co., all five years, in SQL and Python.
2. **[Exercise 2 — Leverage return amplification](exercise-02-leverage-return-amplification.md)** — build a full DFL and ROE-amplification scenario table in pandas.
3. **[Exercise 3 — Distress cost curve](exercise-03-distress-cost-curve.md)** — compute and plot firm value against a grid of debt levels under the tradeoff model.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and `SELECT COUNT(*) FROM income_statement;` / `SELECT COUNT(*) FROM balance_sheet;` both return **5**.
- You have a shell open: `psql crunch_capital` (Postgres) or `sqlite3 crunch_capital.db` (SQLite), and a Python environment with `pandas`, `numpy`, and `matplotlib` installed.

## Suggested workflow

- Open the exercise file beside your terminal.
- For each task, **write the formula out on paper first** — `Tc × D`, `DFL`, `V_L(D)` — before translating it into SQL or Python. If you can't write the formula from memory, go back to the lecture before continuing.
- Check every numeric answer against the "Expected" note. If a number doesn't match, don't move on — find the bug first; carrying a wrong number into the next exercise (or the mini-project) compounds the confusion.
- Save your work per exercise as instructed. You'll commit all three to your portfolio.

## Note on this week's numbers

Every exercise uses `Tc = 0.25` (Crunch Machine Co.'s effective tax rate, confirmed in Lecture 1), `rU = 0.10` (this week's stated unlevered cost of capital), and, where a distress-cost parameter is needed, `α = 0.30` (Lecture 2's stylized total distress-cost fraction) — unless an exercise explicitly tells you to vary one of them. Keeping these fixed across exercises means your Exercise 1 tax-shield numbers, Exercise 3's distress-curve numbers, and the mini-project should all agree with each other where they overlap — a good internal consistency check.
