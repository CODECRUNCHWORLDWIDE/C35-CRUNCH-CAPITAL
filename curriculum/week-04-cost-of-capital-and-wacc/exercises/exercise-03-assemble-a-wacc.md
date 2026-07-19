# Exercise 3 — Assemble a WACC

**Goal:** Build the full WACC pipeline yourself, end to end — bond YTMs, market-value weights, and the final blended rate — reproducing Lecture 3 without copying its code.

**Estimated time:** 1.5 hours.

## Setup

Confirm all three tables are seeded and readable:

```sql
SELECT COUNT(*) FROM stock_prices;      -- 122
SELECT COUNT(*) FROM debt_tranches;     -- 3
SELECT COUNT(*) FROM capital_structure; -- 1
```

Create `solutions.py`. You may reuse your regression beta from Exercise 2 (recompute it at the top of this file, or hardcode the number you already verified — your choice, but say which you did).

## Tasks

### Part A — Cost of debt

1. **Pull `debt_tranches`** into a DataFrame. Add a column `annual_coupon` = `coupon_rate × 100` (coupon per $100 face).

2. **Approximate YTM.** Add a column `ytm_approx` using the approximation formula:
   $$YTM \approx \frac{C + (F-P)/n}{(F+P)/2}$$
   Print the table with `description`, `coupon_rate`, `price_pct_par`, `ytm_approx`. *(Expected YTMs: Senior Notes ≈ 5.79%, Term Loan B ≈ 6.33%, Green Bond ≈ 4.10%.)*

3. **Market value per tranche.** Add a column `market_value` = `face_value × (price_pct_par / 100)`. Print total debt market value. *(Expected: $322,687,500.)*

4. **Weighted pre-tax cost of debt.** Compute the market-value-weighted average of `ytm_approx` across all three tranches. *(Expected: ≈ 5.56%.)*

5. **After-tax cost of debt.** Pull `tax_rate` from `capital_structure` and apply the tax shield. *(Expected: ≈ 4.17%.)*

### Part B — Capital weights

6. **Market value of equity.** Pull `shares_outstanding` from `capital_structure` and the most recent `adj_close` for `CRNM` from `stock_prices` (write a query that gets the latest date automatically — don't hardcode the date). Compute equity market value. *(Expected: $1,230,300,000.)*

7. **Weights.** Compute `E/V` and `D/V` using the equity market value from Task 6 and the debt market value from Task 3. Confirm they sum to 1.0 (print the sum as a check). *(Expected: E/V ≈ 79.22%, D/V ≈ 20.78%.)*

### Part C — Assemble

8. **Cost of equity.** Using your beta from Exercise 2 (regression-derived, ≈1.2429) and `capital_structure`'s $R_f$/$ERP$, compute cost of equity.

9. **WACC.** Combine Tasks 5, 7, and 8 into the full WACC formula. Print every intermediate number (cost of equity, after-tax cost of debt, E/V, D/V) *and* the final WACC, clearly labeled. *(Expected: WACC ≈ 9.16%.)*

10. **Sensitivity check.** Recompute WACC using the **vendor beta** (1.30) instead of your regression beta, keeping everything else the same. Print both WACCs side by side and the difference between them in basis points (1 bp = 0.01%). *(Expected: vendor-based WACC ≈ 9.38%, a difference of about 23 bps.)*

## Expected results (spot checks)

| Quantity | Expected |
|---|---:|
| Weighted pre-tax cost of debt | ≈ 5.56% |
| After-tax cost of debt | ≈ 4.17% |
| Equity market value | $1,230,300,000 |
| Debt market value | $322,687,500 |
| E/V | ≈ 79.22% |
| D/V | ≈ 20.78% |
| WACC (regression beta) | ≈ 9.16% |
| WACC (vendor beta) | ≈ 9.38% |

## Done when…

- [ ] `solutions.py` runs top to bottom against a freshly seeded database with no hardcoded dates or magic numbers pulled from the lecture (every number is either queried or computed).
- [ ] Task 9's WACC matches the table above to within 0.02 percentage points.
- [ ] You can explain, out loud, why the WACC formula weights $R_e$ by $E/V$ and $R_d(1-t)$ by $D/V$ — and not the other way around.
- [ ] Your Task 10 write-up states, in one sentence, which WACC (vendor or regression beta) you'd actually recommend using and why.

## Stretch

- Write your whole pipeline as a single function `compute_wacc(engine, beta)` that takes a SQLAlchemy engine and a beta value, queries everything it needs, and returns the WACC as a float. Call it twice — once with the vendor beta, once with the regression beta — and confirm you get the same two numbers as Task 10.
- Persist your result: write a one-row DataFrame with columns `company`, `beta_source`, `cost_of_equity`, `after_tax_cost_of_debt`, `wacc` back into a new table `wacc_results` in `crunch_capital` using `.to_sql()`, with `if_exists="append"`. Run your Task 9 and Task 10 calculations again and confirm both rows land in the table.

## Submission

Commit `solutions.py` to your portfolio under `c35-week-04/exercise-03/`.
