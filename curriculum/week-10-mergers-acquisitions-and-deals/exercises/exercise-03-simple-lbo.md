# Exercise 3 — Build a simple LBO

**Time:** ~90 minutes. **Builds on:** Lecture 3 (the LBO) in full.

## Goal

Build the sponsor's 5-year LBO of SPSY from `target_financials` and `lbo_assumptions`: sources & uses, a year-by-year cash-sweep debt schedule, an exit, and the resulting MOIC and IRR.

## Part A — Sources & uses (SQL or Python)

Compute:

1. Entry EV (`entry_multiple_ev_ebitda × target EBITDA`).
2. Transaction fees (`transaction_fees_pct × Entry EV`).
3. Total uses (Entry EV + fees).
4. Term loan (`term_loan_multiple_ebitda × target EBITDA`).
5. Sponsor equity (Total uses − Term loan).

**Expected results:** Entry EV $200,000,000; fees $4,000,000; total uses $204,000,000; term loan $100,000,000; sponsor equity $104,000,000.

## Part B — The 5-year operating projection

Build the revenue/EBITDA/D&A/EBIT/capex/ΔNWC table from Lecture 3, Section 4, using `lbo_assumptions`' drivers (5% revenue growth, 20% EBITDA margin held flat, 4% D&A, 3% capex, 10% of incremental revenue for ΔNWC).

**Expected result:** Year 5 revenue ≈ $127,628,156; Year 5 EBITDA ≈ $25,525,631.

## Part C — The cash-sweep debt schedule

This is the heart of the exercise. Write a **Python loop** (not a single vectorized formula — the calculation is sequential, exactly as Lecture 3, Section 9 explains) that, for each of the 5 years:

1. Computes interest on the *beginning* debt balance.
2. Computes EBT, tax, and net income.
3. Computes free cash flow (`net income + D&A − capex − ΔNWC`).
4. Applies the cash sweep percentage to get that year's debt paydown.
5. Computes the ending debt balance, which becomes next year's beginning balance.

```python
# starter shape — fill in the body
debt = term_loan
revenue = target_revenue
rows = []
for year in range(1, hold_period_years + 1):
    # ... your code here ...
    rows.append(dict(year=year, beg_debt=..., interest=..., net_income=..., fcf=..., end_debt=...))
    debt = ...  # roll forward
```

**Expected results (rounded to the nearest dollar):**

| Year | Beg. debt | Interest | Net income | FCF | End. debt |
|---:|---:|---:|---:|---:|---:|
| 1 | 100,000,000 | 8,000,000 | 6,600,000 | 7,150,000 | 92,850,000 |
| 2 | 92,850,000 | 7,428,000 | 7,659,000 | 8,236,500 | 84,613,500 |
| 3 | 84,613,500 | 6,769,080 | 8,814,690 | 9,421,065 | 75,192,435 |
| 4 | 75,192,435 | 6,015,395 | 10,074,529 | 10,711,222 | 64,481,213 |
| 5 | 64,481,213 | 5,158,497 | 11,446,506 | 12,115,034 | 52,366,179 |

If your Year 1 numbers match but later years drift, you likely computed each year's interest off the *original* $100,000,000 balance instead of rolling the debt balance forward — go back to Lecture 3, Section 9 and check your loop updates `debt` at the end of each iteration.

## Part D — Exit, MOIC, and IRR

Using the exit multiple from `lbo_assumptions` (10.0x, same as entry) applied to **Year 5's EBITDA**, and Year 5's **ending debt**:

```
Exit EV            = exit_multiple × Year 5 EBITDA
Exit equity value  = Exit EV − Year 5 ending debt
MOIC               = Exit equity value / Sponsor equity (Part A)
IRR                = MOIC^(1 / hold_years) − 1
```

**Expected results:** Exit EV ≈ $255,256,310; exit equity value ≈ $202,890,131; MOIC ≈ **1.95x**; IRR ≈ **14.3%**.

## Verify your work

| Metric | Expected value |
|---|---:|
| Sponsor equity (entry) | $104,000,000 |
| Term loan (entry) | $100,000,000 |
| Ending debt (Year 5) | ≈ $52,366,179 |
| Exit equity value | ≈ $202,890,131 |
| MOIC | ≈ 1.95x |
| IRR | ≈ 14.3% |

## Stretch (optional, feeds Challenge 2)

Re-run Part C and D with `cash_sweep_pct` set to 0.50 instead of 1.00 — a sponsor that lets management keep half the free cash flow for growth capex instead of sweeping 100% to debt. How much lower does the ending debt balance come out, and what happens to MOIC? (Don't overthink the mechanism yet — Challenge 2 has you formally decompose it.)
