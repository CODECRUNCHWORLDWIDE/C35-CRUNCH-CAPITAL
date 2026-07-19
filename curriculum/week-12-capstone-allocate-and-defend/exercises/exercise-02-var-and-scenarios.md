# Exercise 2 — Compute VaR and Scenarios

**Goal:** Quantify how much the CRNM position could lose (VaR, two ways), how much the whole portfolio could lose with it added, and what the valuation looks like under each of the six named scenarios, probability-weighted.

**Estimated time:** 90 minutes.

## Setup

```sql
SELECT COUNT(*) FROM capstone_price_history WHERE ticker = 'CRNM';   -- must print 504
SELECT COUNT(*) FROM capstone_portfolio_context;                       -- must print 7
SELECT ROUND(SUM(probability), 4) FROM capstone_scenarios;              -- must print 1.0000
```

Create `exercise-02/` with `var.sql`, `var.py`, and `scenarios.py`. Assume a **$10,000,000** book and an **8% CRNM weight** (so an $800,000 position) throughout.

## Tasks

### Part A — Parametric VaR

1. Using `capstone_portfolio_context.annual_volatility` for CRNM (0.28), compute daily volatility (`annual_vol / √252`).
2. Compute 1-day 95% and 99% parametric VaR in **dollars**, for the $800,000 position, using `z = 1.645` (95%) and `z = 2.326` (99%). *(Expected: ≈ $23,200 and ≈ $32,800.)*
3. Convert to a **10-day** VaR by scaling with `√10` (the standard square-root-of-time rule). State the assumption this rule relies on (returns are i.i.d. — no autocorrelation) in one sentence.

### Part B — Historical VaR

4. In `var.sql`, compute CRNM's daily returns from `capstone_price_history` using `LAG` (see Lecture 2, Section 3) and pull the 5th and 1st percentile with `PERCENTILE_CONT`.
5. Convert both percentiles to a **dollar** 1-day VaR for the $800,000 position (flip the sign — the percentile is a negative return).
6. Compare to your parametric answer from Task 2. Are they close (within ~25%)? If not, investigate before moving on — a large gap on this week's simulated (log-normal) data usually means a bug, not a genuine finding.

### Part C — Portfolio VaR

7. Compute the weighted-average correlation between the six core funds and CRNM, using each core fund's `target_weight` (renormalized to sum to 1 across the six core funds only) and its `corr_with_crnm`. *(Expected: ρ ≈ 0.27.)*
8. Using the core book's aggregate volatility (given: 8.5%) and CRNM's volatility (28%) at their target weights (92% / 8%), compute portfolio volatility with the two-asset formula from Lecture 2, Section 4. *(Expected: ≈ 8.70%.)*
9. Compute the **whole $10M portfolio's** 1-day 95% and 99% VaR using `port_vol` from Task 8. Compare this dollar figure to the VaR you'd have computed if you (incorrectly) ignored correlation entirely and just summed CRNM's standalone VaR onto the core book's own VaR. How much does ignoring correlation overstate the risk?

### Part D — Scenario valuation

10. In `scenarios.py`, for **each** of the six rows in `capstone_scenarios`, rebuild the five-year FCF stream: apply `revenue_growth_shock` to each year's growth rate and `ebitda_margin_shock` to each year's margin (both are additive shocks to the Week-7-style base-case drivers — you do not have the raw driver table this week, so approximate FY2026 base growth as 8% and base margin as 21%, holding the shock constant across all 5 years for simplicity — state this simplification in your notes).
11. Re-discount each scenario's stream at `wacc + wacc_shock`, using `exit_multiple_ev_ebitda + exit_multiple_shock` for the terminal value, and bridge to a per-share price.
12. Compute the **probability-weighted expected per-share value**: `Σ (probability_i × per_share_i)`.
13. Compare the probability-weighted expected value to the single-point $15.29 blended target from Exercise 1. Which is more conservative, and by how much?

## Expected result (spot checks)

| Item | Expected (± tolerance) |
|---|---:|
| 1-day 95% parametric VaR ($800k position) | ≈ $23,200 |
| 1-day 99% parametric VaR | ≈ $32,800 |
| 10-day 95% parametric VaR | ≈ $73,300 |
| Historical VaR (95%, 1-day) | within ~25% of parametric |
| Core-to-CRNM weighted correlation | ≈ 0.27 |
| Portfolio volatility with CRNM added | ≈ 8.70% (vs. 8.50% core-only) |
| Probability-weighted expected per-share value | below the $15.29 point estimate |

## Done when…

- [ ] `var.sql` and `var.py` produce parametric and historical VaR that agree within ~25%, and you've investigated any bigger gap.
- [ ] `scenarios.py` produces six per-share values and one probability-weighted expected value.
- [ ] You can state, in one sentence, why the probability-weighted expected value is more conservative than the single-point blended target.
- [ ] You've stated your simplifying assumption from Task 10 explicitly.

## Stretch

- Recompute portfolio VaR at the 99% level and 10-day horizon, and report all four combinations (95%/99% × 1-day/10-day) in one small table.
- Instead of holding the shock constant across all 5 years (Task 10), let the shock apply only to FY2026–FY2027 and taper to zero by FY2029 (a "shock fades" assumption). How much does that change the probability-weighted value?

## Submission

Commit `exercise-02/` to your portfolio under `c35-week-12/exercise-02/`.
