# Challenge 2 — WACC Sensitivity Grid

**Time:** ~90 minutes. **Difficulty:** Medium. **Deliverable is a grid, not a single number.**

## The scenario

Every WACC you've computed this week is a **point estimate** built on assumptions an analyst chose — a particular beta, a particular tax rate, a particular capital structure. A CFO reviewing your WACC of 9.16% is going to ask, reasonably, "how much does this actually move if I'm wrong about beta? What if we take on more debt?" A single number can't answer that. A **sensitivity grid** — WACC recomputed across a whole range of two inputs at once — can.

Your job: build that grid for Crunch Machine Co., entirely in pandas (never a spreadsheet), and use it to answer three concrete questions a CFO would actually ask.

## Your task

Work in `challenge-02.py`, with your written interpretation in `challenge-02.md`.

### Part A — Build the grid

1. Pull Crunch Machine Co.'s fixed inputs from `capital_structure` ($R_f$, $ERP$, tax rate) and its after-tax cost of debt from your Exercise 3 pipeline (or recompute it — your choice, note which).

2. Define two ranges to vary:
   - **Beta:** 0.90 to 1.70, in steps of 0.10 (9 values).
   - **Debt-to-total-capital ($D/V$):** 0% to 45%, in steps of 5% (10 values).

3. For every combination of beta and $D/V$ (81 combinations — a Cartesian product), compute:
   - Cost of equity via CAPM (using that row's beta).
   - $E/V = 1 - D/V$.
   - WACC via the standard formula, using the *fixed* after-tax cost of debt from Task 1 (hold cost of debt constant across the grid — you're isolating the effect of beta and leverage specifically, not remixing every input at once).

   Build this as a `pandas.DataFrame` with one row per combination — don't write 81 separate calculations by hand. (Hint: `itertools.product`, or build two Series and use a full outer merge / cross join.)

4. **Pivot it into a readable grid.** Use `pandas.pivot_table` (or `.pivot()`) to reshape your long DataFrame into a wide grid: rows = $D/V$, columns = beta, values = WACC. Print it with values formatted as percentages.

### Part B — Answer three CFO questions from the grid

5. **"If our beta estimate is wrong by ±0.2, how much does WACC move, at our current ~21% leverage?"** Look up (or interpolate near) the row closest to $D/V = 20\%$, and compare the WACC at beta = 1.04 (≈1.24 − 0.2) versus beta = 1.44 (≈1.24 + 0.2). Report the swing in basis points.

6. **"If we doubled our leverage to ~40% D/V, holding beta at our regression estimate, how much would WACC change — and is that direction what you'd expect?"** Compare WACC at $D/V ≈ 20\%$ vs. $D/V ≈ 40\%$, both at beta ≈ 1.24 (note: this comparison, as specified, deliberately holds beta *fixed* even though you know from Lecture 2 that more leverage would actually *raise* beta in reality — flag that simplification explicitly in your write-up).

7. **"Which input — a beta swing of ±0.2, or a leverage swing of ±10 points of D/V — moves WACC more, in absolute terms, over this grid?"** Use your grid to answer with a specific comparison, not a guess.

### Part C — A more honest version (stretch difficulty, but do at least attempt it)

8. Question 6 asked you to hold beta fixed while leverage changes, which you correctly flagged as unrealistic. Now do it properly: for each $D/V$ value in your grid, **relever** a starting *unlevered* beta (back out $\beta_U$ from your regression beta of 1.2429 at Crunch Machine Co.'s *current* $D/E \approx 0.2623$ first, using Hamada) at the *implied* $D/E$ for that row's $D/V$, and use *that* relevered beta — not a fixed one — to compute cost of equity and WACC for that row. Rebuild the pivoted grid with this corrected logic. Compare the corrected WACC at $D/V = 40\%$ to your (incorrect) fixed-beta answer from Task 6. Which one is higher, and does that match the intuition that more debt should raise both the cost of equity *and*, likely, the cost of debt (which this exercise still holds fixed — say so)?

## Constraints

- The entire grid — construction, pivoting, and every lookup — happens in pandas. No spreadsheet, no manually typed-out table.
- State every simplification explicitly (Task 6 asks you to; do the same for anything else you hold fixed that realistically wouldn't be).
- Round only for *display*. Keep full precision through every calculation.

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|---------------|
| Grid construction | Loops with hardcoded values, or a handful of manual calculations | A proper Cartesian product / cross join producing all 81 rows programmatically |
| Readability | Long-format table only | Pivoted, formatted-as-percentage grid that a CFO could actually read |
| Part B answers | Numbers pulled from the grid with no comparison stated | Explicit basis-point comparisons answering each question directly |
| Honesty about simplifications | Doesn't mention that Task 6 unrealistically holds beta fixed | Flags it explicitly, and Part C fixes it properly |
| Interpretation | Just the numbers | A short, clear written answer to "which matters more, beta or leverage?" |

## Submission

Commit `challenge-02.md` and `challenge-02.py` to your portfolio under `c35-week-04/challenge-02/`.
