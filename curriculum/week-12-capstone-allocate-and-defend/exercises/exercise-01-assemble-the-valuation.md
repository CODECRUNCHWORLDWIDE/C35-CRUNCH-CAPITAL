# Exercise 1 — Assemble the Target Valuation

**Goal:** Turn `capstone_cash_flows` and `capstone_deal_snapshot` into a reconciled per-share valuation range, and quantify the gap against the market price. This is Lecture 1's method, run by your own hand.

**Estimated time:** 90 minutes.

## Setup

Confirm your data is loaded:

```sql
SELECT COUNT(*) FROM capstone_cash_flows;    -- must print 5
SELECT * FROM capstone_deal_snapshot;         -- must print 1 row
```

Create a folder `exercise-01/` with a `valuation.sql` and a `valuation.py`. You'll do the discounting in **both** — SQL first (to build the muscle), Python second (to cross-check and to set up for Exercise 2, which reuses the Python version).

## Tasks

### Part A — SQL

1. **Present value of the explicit forecast.** Write one query that returns each `fiscal_year`'s `unlevered_fcf` and its present value, discounted at `capstone_deal_snapshot.wacc`. *(Expected: PV values roughly $21.1M, $23.4M, $24.7M, $25.7M, $26.3M for 2026–2030.)*

2. **Sum the explicit-period PV.** *(Expected: ≈ $121.3M.)*

3. **Terminal value, perpetuity growth method.** Compute `TV_2030 = FCF_2030 × (1 + terminal_growth_rate) / (wacc − terminal_growth_rate)`, then discount it back 5 years at `wacc`. *(Expected: TV ≈ $625.2M undiscounted, ≈ $402.7M discounted to today.)*

4. **Terminal value, exit multiple method.** Compute `TV_2030 = EBITDA_2030 × exit_multiple_ev_ebitda`, then discount it back 5 years. *(Expected: TV ≈ $528.1M undiscounted, ≈ $340.1M discounted.)*

5. **Two enterprise values.** Add each discounted terminal value to the explicit-period PV from Task 2. *(Expected: ≈ $524.0M perpetuity, ≈ $461.4M exit multiple.)*

6. **Bridge both to a per-share price.** Subtract `total_debt`, add `cash_and_equivalents`, divide by `shares_outstanding`, for both methods. *(Expected: ≈ $18.71 and ≈ $16.10.)*

### Part B — Comps and blend

7. **Comps-implied per share.** `EV = comps_median_ev_ebitda × ltm_ebitda`, then the same debt/cash/shares bridge. *(Expected: ≈ $12.12.)*

8. **State and justify a weighting.** Pick a weight between the DCF average and the comps answer (Lecture 1 used 60/40 in favor of the DCF; you may use a different split, but you must state it and give one sentence of justification). Compute the blended target price.

9. **Compute the implied upside** vs. `current_share_price`. *(Expected, at 60/40: ≈ 22% upside on a blended target of ≈ $15.29.)*

### Part C — Python cross-check

10. Rebuild Tasks 1–9 in `valuation.py` using `pandas`, reading the same tables via `pandas.read_sql`. Your Python DCF value and SQL DCF value should agree to within rounding.

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg2://localhost/crunch_capital")
cash_flows = pd.read_sql("SELECT * FROM capstone_cash_flows ORDER BY fiscal_year", engine)
deal = pd.read_sql("SELECT * FROM capstone_deal_snapshot", engine).iloc[0]

t = cash_flows["fiscal_year"] - 2025
discount_factors = 1 / (1 + deal["wacc"]) ** t
pv_fcf = (cash_flows["unlevered_fcf"] * discount_factors).sum()
print(f"PV of explicit FCF: ${pv_fcf:,.0f}")
# ... continue with terminal value, both methods, and the bridge
```

## Expected result (spot checks)

| Item | Expected (± a few %) |
|---|---:|
| PV of explicit FCF | ≈ $121.3M |
| DCF EV, perpetuity method | ≈ $524.0M |
| DCF EV, exit multiple method | ≈ $461.4M |
| DCF per share, perpetuity | ≈ $18.71 |
| DCF per share, exit multiple | ≈ $16.10 |
| Comps per share | ≈ $12.12 |
| Blended target (60/40) | ≈ $15.29 |
| Implied upside vs. $12.50 market price | ≈ 22% |

## Done when…

- [ ] `valuation.sql` computes all of Tasks 1–9.
- [ ] `valuation.py` independently reproduces the same DCF enterprise values.
- [ ] You've written one sentence stating and justifying your DCF/comps weighting.
- [ ] Your final blended target price and implied upside match the spot checks within a few percent.

## Stretch

- Build a small sensitivity table: recompute the blended target at WACC ± 0.5pp and terminal growth ± 0.5pp (a 3×3 grid). How much does the target price move for a "small" change in the discount rate versus the growth rate?
- Recompute the comps-implied value using the mean instead of the median multiple, and compare.

## Submission

Commit `exercise-01/` (both files, plus a short `notes.md` with your stated weighting and its justification) to your portfolio under `c35-week-12/exercise-01/`.
