# Week 8 — Challenges

Two open-ended problems, ~90 minutes each. Unlike the exercises, these don't have one single numeric answer key — the skill is judgment and defensible methodology, not recall. Do them after the three exercises.

1. **[Challenge 1 — Constrained allocation](challenge-01-constrained-allocation.md)** — the unconstrained optimizer wants to short two of your six funds; re-solve under no-short and position-cap constraints and compare the results. *(~90 min.)*
2. **[Challenge 2 — Diversification stress test](challenge-02-diversification-stress-test.md)** — correlations that looked friendly in calm markets tend to spike toward 1 in a crash; rebuild your "optimal" portfolio's risk profile under a stressed correlation matrix and see how much of the diversification benefit survives. *(~90 min.)*

## How these are judged

There's no answer key with a single correct number for every cell. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of method** — did you apply the formulas (constrained optimization, portfolio variance under a stressed covariance matrix) correctly, even if your final inputs are debatable?
- **Explicit assumptions** — did you write down every judgment call (which constraints to impose, how much to stress the correlations)?
- **Interpretation** — do you explain what the numbers *mean* for an investor, not just report a table?
- **No spreadsheets** — every table, grid, and calculation lives in pandas/numpy. A constrained optimization is a `scipy.optimize.minimize()` call or a documented grid search, never an Excel Solver add-in.

Keep your work in `challenge-01.md` / `challenge-01.py` and `challenge-02.md` / `challenge-02.py`, with your code **and** your written reasoning. The reasoning is the point.
