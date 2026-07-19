# Exercise 1 — Build an accretion/dilution model

**Time:** ~90 minutes. **Builds on:** Lecture 1 (deal structure and price) and Lecture 2, Sections 1–5 (accretion/dilution, no synergies).

## Goal

Build CRNM's full pro forma combined income statement — no synergies yet — and prove whether the base-case deal (55% cash / 45% stock, from `deal_terms`) is accretive or dilutive to CRNM's standalone EPS.

## Setup

You need `acquirer_financials`, `target_financials`, and `deal_terms` from this week's [README](../README.md) seed. Confirm all three sanity checks pass before starting.

## Part A — Standalone EPS (SQL)

Write a query that computes CRNM's own standalone EPS from `acquirer_financials` alone — EBIT, interest, EBT, tax, net income, and EPS, exactly as Lecture 2, Section 2 walks through by hand.

```sql
-- Your query here. Compute:
--   ebit, interest_expense, ebt, tax, net_income, standalone_eps
-- from acquirer_financials.
```

**Expected result:** standalone EPS = **$0.85** (net income $20,400,000 over 24,000,000 shares).

## Part B — Deal mechanics (Python)

In Python, compute the pieces Lecture 1 built:

1. The control premium.
2. The equity purchase price and implied enterprise value.
3. The cash consideration, stock consideration, new shares issued, and pro forma share count.

```python
import pandas as pd

# load acquirer_financials, target_financials, deal_terms into DataFrames

# your code here — compute:
#   control_premium, equity_purchase_price, implied_ev,
#   cash_consideration, stock_consideration,
#   new_shares_issued, pro_forma_shares
```

**Expected results:** control premium 30.0%; equity purchase price $195,000,000; implied EV $207,000,000; cash consideration $107,250,000; stock consideration $87,750,000; new shares issued 4,387,500; pro forma shares 28,387,500.

## Part C — Pro forma combined income statement (SQL or Python, your choice)

Build the full pro forma statement from Lecture 2, Section 3, **without synergies**:

1. Combined EBIT = CRNM's EBIT + SPSY's EBIT.
2. Combined interest = CRNM's existing interest + SPSY's existing interest + new interest on the acquisition debt (`cash_consideration × new_acquisition_debt_rate`).
3. Combined EBT, tax, and net income.
4. Pro forma EPS = combined net income ÷ pro forma shares.

**Expected result:** pro forma EPS ≈ **$0.9113**.

## Part D — The verdict

Compute the accretion/dilution percentage:

```
accretion_pct = (pro_forma_eps - standalone_eps) / standalone_eps
```

**Expected result:** ≈ **+7.2%** — the base-case deal is accretive, even with zero synergies.

Write one paragraph (in a comment or a short markdown note) explaining, in your own words, *why* — reference the P/E gap between CRNM (~23.5x) and the implied deal multiple for SPSY (~17.6x) from Lecture 2, Section 5.

## Verify your work

| Metric | Expected value |
|---|---:|
| Standalone EPS | $0.85 |
| Equity purchase price | $195,000,000 |
| Implied EV | $207,000,000 |
| New shares issued | 4,387,500 |
| Pro forma shares | 28,387,500 |
| Pro forma EPS (no synergies) | ≈ $0.9113 |
| Accretion/(dilution) | ≈ +7.2% |

If your accretion percentage is negative, double-check you added — not subtracted — SPSY's EBIT into the combined figure, and that you're dividing by the *pro forma* share count, not CRNM's original 24,000,000.
