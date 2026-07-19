# Exercise 3 — Solve the Max-Sharpe Portfolio

**Goal:** Solve for the global-minimum-variance and maximum-Sharpe (tangency) portfolios in closed form, compare both to equal-weight, and confirm the tangency portfolio really does sit where the capital market line touches the frontier — without looking at the lecture code while you write it.

**Estimated time:** 1.5 hours.

## Setup

Reuse your six-fund `mu`, `Sigma`, `Sigma_inv`, and `ones` from Exercises 1–2. Pull the risk-free rate from the database rather than hardcoding it:

```sql
SELECT risk_free_rate, equity_risk_premium FROM capital_market_assumptions WHERE assumption_set = 'base_case';
```

```python
cma = pd.read_sql("SELECT * FROM capital_market_assumptions WHERE assumption_set = 'base_case'", engine)
rf = cma["risk_free_rate"].iloc[0]
```

Create `solutions.py`. Put each task under a comment with the task number.

## Tasks

1. **Global minimum-variance portfolio.** Compute `w_gmv = Sigma_inv @ ones / (ones @ Sigma_inv @ ones)`. Print each fund's weight, plus the portfolio's expected return and volatility. Confirm the weights sum to 1.

2. **Maximum-Sharpe (tangency) portfolio.** Compute `w_tan` using the excess-return formula from Lecture 2, Section 5. Print each fund's weight, expected return, volatility, and Sharpe ratio. Confirm the weights sum to 1.

3. **Equal-weight benchmark.** Compute the expected return, volatility, and Sharpe ratio of the naive equal-weight portfolio (`1/6` in each fund). Print all three, side by side with the GMV and tangency portfolios' numbers, in one small comparison table.

4. **Which portfolio for which investor?** In 3–5 sentences, explain which of the three portfolios (GMV, tangency, equal-weight) you'd recommend to (a) a retiree who cannot tolerate large drawdowns, and (b) a 25-year-old investor with a 40-year horizon who wants to maximize long-run growth and can tolerate short-term losses. Justify each answer using the actual numbers you just computed, not generic advice.

5. **Confirm the tangency point sits ON the frontier.** Using your `frontier_portfolio()` function from Exercise 2, call it at `target_return = tan_return` (the tangency portfolio's own expected return from Task 2). Compare the *weights* it returns to your Task 2 tangency weights directly. They should match closely — explain in one sentence why they must (both formulas are solving the same underlying constrained optimization; the tangency portfolio is simply the one point on the frontier where a specific extra condition — maximum Sharpe ratio — also happens to hold).

6. **Sharpe-ratio sensitivity to the risk-free rate.** Recompute the tangency portfolio's weights and Sharpe ratio for two other risk-free rate assumptions: `rf = 0.01` and `rf = 0.05`. Print all three tangency solutions (rf=1%, 3%, 5%) side by side. Does the *tangency portfolio's own composition* stay roughly stable across these three risk-free rates, or does it move a surprising amount? Pay particular attention to what happens at `rf = 0.05` — compare it to the GMV portfolio's own expected return from Task 1, and explain in your own words why the tangency formula becomes numerically unstable as `rf` approaches that number from below.

## Expected results (spot checks)

- Task 1 (GMV): expected return ≈ **5.55%**, volatility ≈ **3.98%**. `CRNB` should carry the largest weight (roughly 94%); `CRNH` should be meaningfully negative (roughly −27%).
- Task 2 (tangency, at rf=3%): expected return ≈ **8.04%**, volatility ≈ **5.59%**, Sharpe ≈ **0.90**.
- Task 3 (equal-weight): expected return ≈ **9.84%**, volatility ≈ **8.58%**, Sharpe ≈ **0.80** — lower Sharpe than the tangency portfolio despite a higher raw return.
- Task 5: your two sets of tangency weights should match to at least 3 decimal places.
- Task 6: at `rf = 1%` the tangency portfolio stays broadly similar in shape to the `rf = 3%` solution (still bond-heavy, still long in every fund but `CRNI`). At `rf = 5%` — close to the GMV portfolio's own ≈5.55% expected return — the solution becomes extreme (a large short in `CRNB`, a large long in `CRNH`) and much less trustworthy as a real-world allocation.

## Done when…

- [ ] `solutions.py` runs top to bottom with no errors, against a freshly seeded `crunch_capital` database.
- [ ] Your risk-free rate is pulled from `capital_market_assumptions`, not hardcoded, in Tasks 1–5.
- [ ] Task 4's answer references specific numbers (Sharpe ratios, volatilities) from your own output, not just abstract reasoning.
- [ ] Task 6 states a clear, specific finding (not just "the numbers changed a bit").

## Stretch

- The tangency portfolio at `rf = 3%` had a small negative weight in `CRNI` (roughly −0.6%). Is that economically meaningful, or is it small enough to be noise from estimation error in `mu` and `Sigma`? Zero out that one weight, renormalize the rest to sum to 1, and check how much the resulting Sharpe ratio changes. What does that tell you about how sensitive the "optimal" solution is to very small allocations?
- Build a `wacc_results`-style table (recall Week 4) called `portfolio_results` (`portfolio_name`, `crnq_weight`, `crni_weight`, ..., `expected_return`, `volatility`, `sharpe_ratio`, `computed_date`) in your database and insert your GMV, tangency, and equal-weight results into it — so future weeks (or a future you) can query past portfolio solutions instead of recomputing them from scratch.

## Submission

Commit `solutions.py` to your portfolio under `c35-week-08/exercise-03/`.
