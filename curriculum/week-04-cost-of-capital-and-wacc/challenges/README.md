# Week 4 — Challenges

Two open-ended problems, ~1.5 hours each. Unlike the exercises, these don't have one single numeric answer key — the skill is judgment and defensible methodology, not recall. Do them after the three exercises.

1. **[Challenge 1 — Relever beta across structures](challenge-01-relever-beta-across-structures.md)** — you have no traded stock for a target division; unlever four public comparables' betas, average them, and relever at a target capital structure. *(~90 min.)*
2. **[Challenge 2 — WACC sensitivity grid](challenge-02-wacc-sensitivity-grid.md)** — build a two-dimensional grid of WACC across a range of betas and leverage ratios in pandas, and use it to answer "how much does WACC actually move?" questions. *(~90 min.)*

## How these are judged

There's no answer key with a single correct number for every cell. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of method** — did you apply the formulas (Hamada unlevering/relevering, the WACC assembly) correctly, even if your final inputs are debatable?
- **Explicit assumptions** — did you write down every judgment call (which peers to include, which target structure to relever at, which range to grid over)?
- **Interpretation** — do you explain what the numbers *mean*, not just report a table?
- **No spreadsheets** — every table, grid, and calculation lives in pandas/SQL. A pivot table is a `pandas.pivot_table()` or `pd.crosstab()`, never an Excel grid.

Keep your work in `challenge-01.md` / `challenge-01.py` and `challenge-02.md` / `challenge-02.py`, with your code **and** your written reasoning. The reasoning is the point.
