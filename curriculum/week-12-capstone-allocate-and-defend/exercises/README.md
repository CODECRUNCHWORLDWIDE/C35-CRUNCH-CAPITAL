# Week 12 — Exercises

Three guided exercises, ~1.5 hours each, that build in sequence toward the mini-project. Each one produces an artifact you'll reuse directly on Saturday — treat these as the actual first draft of your capstone deliverable, not throwaway practice.

1. **[Exercise 1 — Assemble the target valuation](exercise-01-assemble-the-valuation.md)** — discount `capstone_cash_flows`, bridge to a per-share range, and compare it to the market price.
2. **[Exercise 2 — Compute VaR and scenarios](exercise-02-var-and-scenarios.md)** — historical and parametric VaR, plus a probability-weighted scenario valuation.
3. **[Exercise 3 — Draft the risk memo](exercise-03-draft-the-risk-memo.md)** — assemble Exercises 1 and 2, plus a governance read, into one reproducible memo.

## Before you start

- You've completed all three lectures.
- You ran the full setup from the [week README](../README.md) and every sanity check there passes.
- You have a working PostgreSQL or SQLite connection **and** a Python environment with `pandas`/`numpy`.

## Suggested workflow

- Work each exercise in order — Exercise 3 explicitly assembles the outputs of Exercises 1 and 2.
- Keep every number's SQL or Python source next to it. You are practicing the reproducibility standard from Lecture 3 starting now, not just at the mini-project.
- Save your work as you go: `exercise-01/`, `exercise-02/`, `exercise-03/` subfolders in your portfolio, each with the queries/scripts and a short `notes.md` of your results.

## Note on tolerance

This week's expected results are given with a small tolerance (typically ±2–3%) to account for rounding at different points in a multi-step calculation. If your answer is close but not exact, trace which intermediate step diverged before assuming your logic is wrong — a rounding difference two steps back compounds by the end of a five-year discounted cash flow.
