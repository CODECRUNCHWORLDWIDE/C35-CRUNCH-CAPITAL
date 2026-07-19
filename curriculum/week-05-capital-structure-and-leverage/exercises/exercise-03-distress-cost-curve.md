# Exercise 3 — The Distress Cost Curve

**Goal:** Turn the tradeoff-theory formula from Lecture 2 into a table and a chart — firm value across a full grid of debt levels — and see the interior maximum with your own eyes instead of just deriving it with calculus.

**Estimated time:** 90 minutes.

## Setup

Have Lecture 2, Sections 3–5 open beside you. You'll need `VU`, `Tc`, `α`, and the tradeoff formula. Create `distress_curve.py`. This week's parameters, fixed for this exercise: `VU = 16,687.5` (Crunch Machine Co., FY2025), `Tc = 0.25`, `α = 0.30`.

## Part A — The model functions

1. **Task 1.** Write three small functions, each taking `D` (and any other needed parameters) and returning a single number:

```python
def prob_distress(D: float, VU: float) -> float:
    """p(D) = (D / VU)^2"""
    ...

def expected_distress_cost(D: float, VU: float, alpha: float) -> float:
    """p(D) * alpha * VU"""
    ...

def levered_value(D: float, VU: float, tax_rate: float, alpha: float) -> float:
    """VU + tax_rate * D - expected_distress_cost(D, VU, alpha)"""
    ...
```

2. **Task 2.** Sanity-check `levered_value(0, VU, Tc, alpha)` — it should return exactly `VU`, since zero debt means zero tax shield and zero distress cost. Confirm this with an `assert`.

## Part B — Build the grid

3. **Task 3.** Build a `pandas` DataFrame with one row for every `D` from `0` to `25,000` in steps of `1,000` (26 rows), with columns `D`, `prob_distress`, `expected_distress_cost`, `tax_shield` (`= Tc × D`), and `levered_value`.

4. **Task 4.** Find the row with the **maximum** `levered_value` in your grid (`df.loc[df["levered_value"].idxmax()]`). What `D` does it land on? How close is that to the exact calculus answer, `D* = Tc × VU / (2α)`? *(It won't match exactly — your grid steps by 1,000, and the true optimum is unlikely to land precisely on a multiple of 1,000. That gap is the whole reason Challenge 1 asks you to also solve this with calculus, not just grid search.)*

## Part C — Chart it

5. **Task 5.** Using `matplotlib`, plot `levered_value` (y-axis) against `D` (x-axis) as a line chart. Add a vertical line (or marked point) at your Task 4 grid-maximum `D`, and a second vertical line at Crunch Machine Co.'s **actual** FY2025 debt, `D = 19,700`. Label both. Save the figure as `distress_curve.png`.

6. **Task 6.** Looking at your chart: is `D = 19,700` to the left or right of the peak? Is the curve still rising there, flat, or falling? What does the chart's shape at that point tell you about whether Crunch Machine Co. would gain or lose value from paying down debt, according to this model?

## Expected results (spot checks)

- Task 2 → `levered_value(0, ...) == 16,687.5` exactly.
- Task 3, `D = 5,000` row → `tax_shield = 1,250.0`, `expected_distress_cost ≈ 449.4`, `levered_value ≈ 17,488.1`.
- Task 3, `D = 20,000` row → `levered_value ≈ 14,496.5` — already noticeably below the peak.
- Task 4 → the grid maximum should land at `D = 7,000`, with `levered_value ≈ 17,551.3` — close to (but not exactly at) the true calculus optimum of `D* ≈ 6,953.1`.
- Task 6 → at `D = 19,700`, the curve should be well past the peak and **clearly falling** — the model says Crunch Machine Co. would gain value by reducing debt, not adding more.

## Done when…

- [ ] All three Task 1 functions are written and pass the Task 2 assertion.
- [ ] Your grid (Task 3) has exactly 26 rows, `D` from 0 to 25,000.
- [ ] `distress_curve.png` exists, shows a clear inverted-U shape, and both vertical markers (grid optimum, actual debt) are visible and labeled.
- [ ] You can answer Task 6 out loud in one sentence, pointing at the shape of your own chart.

## Stretch

- Add a Task 7: re-run the whole exercise with `α = 0.15` (a much less severe distress-cost assumption) instead of `0.30`. How much does the grid-optimum `D` shift? Plot both curves (`α = 0.30` and `α = 0.15`) on the same axes with a legend. This previews Challenge 1's sensitivity question.

## Submission

Commit `distress_curve.py` and `distress_curve.png` to your portfolio under `c35-week-05/exercise-03/`.
