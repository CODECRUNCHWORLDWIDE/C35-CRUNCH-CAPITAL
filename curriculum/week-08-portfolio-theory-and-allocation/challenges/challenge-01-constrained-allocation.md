# Challenge 1 — Optimize Under Constraints

**Time:** ~90 minutes. **Difficulty:** Medium. **Deliverable is a comparison, not just a single answer.**

## The scenario

Exercise 3's unconstrained tangency portfolio put a small short position in `CRNI` (roughly −0.6%) and, in the GMV portfolio, a much larger short in `CRNH` (roughly −27%). In practice, most funds and most individual investors simply **cannot** take those short positions — either their mandate forbids shorting entirely, or the cost/complexity of actually borrowing shares to short a bond or gold fund makes it impractical, or a compliance policy caps how much can go into any single position regardless of direction. Crunch Capital Advisors' investment committee has told you: **no short positions, and no single fund above 30% of the portfolio.** Your job is to find the best allocation the committee will actually let you run, and to show them exactly what that constraint costs.

## Your task

Work in `challenge-01.py`, with your written interpretation in `challenge-01.md`.

### Part A — Re-solve under constraints

You cannot use the closed-form formulas from Lecture 2 once you add inequality constraints (`w_i ≥ 0`, `w_i ≤ 0.30`) — those formulas only solve the *unconstrained* (or equality-constrained-only) problem. Use `scipy.optimize.minimize` with the `SLSQP` method instead:

1. **Constrained maximum-Sharpe portfolio.** Minimize the negative Sharpe ratio, subject to weights summing to 1, every weight between 0 and 0.30. Use `scipy.optimize.minimize(fun, x0, method="SLSQP", bounds=bounds, constraints=constraints)` — `x0` can be equal weights as a starting guess. Print the resulting weights, expected return, volatility, and Sharpe ratio.

2. **Constrained global-minimum-variance portfolio.** Same constraints, but minimize portfolio variance directly instead of negative Sharpe. Print the resulting weights, expected return, and volatility.

3. **Sanity-check your optimizer.** Re-run Task 1 with the position cap *removed* (bounds `(0, 1)` instead of `(0, 0.30)`, keeping the no-short constraint) and confirm your constrained result is *close* to Exercise 3's unconstrained tangency portfolio wherever the unconstrained solution didn't actually violate the no-short constraint in the first place (i.e., every weight that was already non-negative in the unconstrained solution should barely move). This confirms your `scipy` setup is solving the right problem before you trust the fully constrained answer.

### Part B — Quantify what the constraints cost

4. **The efficiency cost of "no shorting."** Compare the no-short-only tangency Sharpe ratio (Task 1 with the 30% cap *removed*) to the fully unconstrained tangency Sharpe ratio from Exercise 3. Report the difference in Sharpe ratio, and separately, the difference in expected return at matched risk (you'll need to either accept the small volatility difference directly, or use your Exercise 2 frontier function to find the unconstrained portfolio at the *same* volatility as your constrained solution, for an apples-to-apples return comparison).

5. **The additional cost of the 30% position cap.** Compare the fully constrained (no-short + 30% cap) tangency Sharpe ratio to the no-short-only Sharpe ratio from Task 4. How much *additional* Sharpe ratio do you give up specifically because of the position cap, on top of what you already gave up from removing shorting?

6. **Write the memo.** In `challenge-01.md`, write a half-page memo to the investment committee: state the final recommended (fully constrained) weights, the resulting Sharpe ratio, and — in plain language a non-quant committee member could follow — how much risk-adjusted performance the two constraints together are costing versus the theoretical unconstrained optimum. Be honest about whether you think the constraints are worth it.

## Constraints

- The optimization itself must use `scipy.optimize.minimize` (or an equivalent solver) — no manual grid search standing in for a real constrained optimizer, and no spreadsheet Solver.
- Every reported number must come from actually running your code, not estimated by eye from the unconstrained solution.
- State your optimizer's starting guess (`x0`) and confirm the result doesn't change meaningfully if you start from a different `x0` (try starting from the unconstrained tangency weights, clipped to the constraint bounds, as a second starting point) — constrained nonlinear optimizers can occasionally converge to different local solutions depending on where they start, and part of doing this rigorously is checking that didn't happen here.

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Optimizer setup | Bounds or constraints misspecified, or manual weight-guessing instead of `scipy.optimize` | Correct `SLSQP` setup with proper bounds and equality constraint, verified against a sanity check |
| Robustness check | No check that the optimizer found a genuine optimum | Multiple starting points tried, confirmed to converge to the same answer |
| Quantifying the cost | Just reports the constrained portfolio with no comparison | Explicit Sharpe-ratio and return deltas isolating the cost of shorting-restriction vs. the position cap separately |
| Communication | Numbers dumped with no narrative | A clear memo a non-quant reader could act on, with an honest recommendation |

## Submission

Commit `challenge-01.md` and `challenge-01.py` to your portfolio under `c35-week-08/challenge-01/`.
