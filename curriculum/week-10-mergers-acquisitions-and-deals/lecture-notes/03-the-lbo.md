# Lecture 3 — The LBO

> **Duration:** ~2 hours. **Outcome:** You can build a full leveraged buyout — sources & uses, a debt-sized capital structure, a multi-year cash-sweep debt-paydown schedule, and an exit — and compute the sponsor's MOIC and IRR from your own numbers.

CRNM isn't the only party interested in SPSY. A private equity sponsor is evaluating the same company for a **leveraged buyout (LBO)** — buying the whole business with a small amount of the sponsor's own money and a large amount of borrowed debt, running it for several years while debt gets paid down and (hopefully) the business grows, then selling it. This lecture builds that deal from `target_financials` and `lbo_assumptions`, entirely independent of CRNM's strategic math from Lectures 1–2.

## 1. What makes an LBO different from a strategic acquisition

CRNM's deal (Lectures 1–2) is measured by its effect on CRNM's own **EPS** — CRNM already has operations, and the question is whether adding SPSY helps or hurts the combined company's earnings per share. A sponsor has no existing operations to combine into — it is a pool of investor capital looking for a return. Its question is entirely different: **"if I put in $X of my own money and finance the rest with debt, how much cash comes back to me when I sell, and what annualized return does that represent?"** Everything in this lecture serves that one question.

## 2. Sources & uses — the LBO's opening balance sheet

Every LBO starts with a **sources & uses** table: what the deal costs (uses), and where the money to pay for it comes from (sources). They must balance exactly.

**Uses** — what the sponsor has to pay for, from `lbo_assumptions`:

```
Entry EV = entry_multiple_ev_ebitda × target EBITDA
         = 10.0 × 20,000,000
         = $200,000,000

Transaction fees = transaction_fees_pct × Entry EV
                  = 0.02 × 200,000,000
                  = $4,000,000

Total uses = Entry EV + Transaction fees
           = 200,000,000 + 4,000,000
           = $204,000,000
```

Transaction fees (legal, advisory, financing arrangement fees) are real cash the sponsor has to fund on day one — they buy nothing and never generate a return, which matters a great deal in Section 8.

**Sources** — the term loan is sized off a leverage multiple, and the sponsor funds whatever's left:

```
Term loan = term_loan_multiple_ebitda × target EBITDA
          = 5.0 × 20,000,000
          = $100,000,000

Sponsor equity = Total uses − Term loan
               = 204,000,000 − 100,000,000
               = $104,000,000
```

The sponsor is putting in **$104,000,000** of its own capital to control a $200,000,000 business — leverage of 5.0x EBITDA (`100,000,000 / 20,000,000`), meaning debt covers just under half the deal (`100M / 204M ≈ 49%`). Every dollar of that $104,000,000 equity check is what Section 8's return gets measured against.

## 3. Why leverage is the whole point of an LBO

Compare two hypothetical buyers of the identical $200,000,000 business, both of whom see it grow to be worth $255,256,310 in five years (Section 5 derives this number): an **all-equity buyer** who paid $204,000,000 in cash sees their $204M grow to roughly $255M — about a 1.25x return. A **leveraged buyer** who put in only $104,000,000 and financed the rest with debt captures the *entire* increase in enterprise value on a much smaller equity base — because the debt holders' claim doesn't grow with the business, all of the upside accrues to the smaller equity slice. This is **leverage amplifying equity returns**, the same principle Week 5 covered for a corporate capital structure, just deliberately maximized by a financial sponsor instead of managed conservatively by an operating company's CFO. It cuts both ways: if the business shrinks instead of grows, the same leverage amplifies the *loss* just as severely — Challenge 2 has you test exactly that.

## 4. Building the operating projection

The term loan gets paid down out of the business's own free cash flow, so the LBO needs a 5-year operating forecast first, using the drivers in `lbo_assumptions` — the same driver-based approach Week 7 used for CRNM's DCF:

```
Revenue_t = Revenue_(t-1) × (1 + revenue_growth_rate)      -- 5% flat
EBITDA_t  = Revenue_t × (EBITDA margin, held flat at 20%)
D&A_t     = Revenue_t × da_pct_revenue                       -- 4%
EBIT_t    = EBITDA_t − D&A_t
Capex_t   = Revenue_t × capex_pct_revenue                    -- 3%
ΔNWC_t    = (Revenue_t − Revenue_(t-1)) × nwc_pct_incremental_revenue  -- 10%
```

Starting from SPSY's own FY2025 revenue ($100,000,000):

| Year | Revenue | EBITDA | D&A | EBIT | Capex | ΔNWC |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 105,000,000 | 21,000,000 | 4,200,000 | 16,800,000 | 3,150,000 | 500,000 |
| 2 | 110,250,000 | 22,050,000 | 4,410,000 | 17,640,000 | 3,307,500 | 525,000 |
| 3 | 115,762,500 | 23,152,500 | 4,630,500 | 18,522,000 | 3,472,875 | 551,250 |
| 4 | 121,550,625 | 24,310,125 | 4,862,025 | 19,448,100 | 3,646,519 | 578,813 |
| 5 | 127,628,156 | 25,525,631 | 5,105,126 | 20,420,505 | 3,828,845 | 607,753 |

## 5. The cash sweep — the mechanic that makes an LBO an LBO

Each year, the business generates free cash flow, and `lbo_assumptions` specifies a **100% cash sweep**: every dollar of free cash flow is applied straight to paying down the term loan (no dividends, no bolt-on acquisitions — the simplest, most conservative case). Because interest is calculated on the *beginning* balance each year, and that balance falls every year the sweep runs, this genuinely has to be computed one year at a time — it's a **path-dependent** calculation, not a single formula.

```
Interest_t     = Beginning debt_t × term_loan_rate                    -- 8.0%
EBT_t          = EBIT_t − Interest_t
Tax_t          = EBT_t × tax_rate                                     -- 25%
Net income_t   = EBT_t − Tax_t
Free cash flow_t = Net income_t + D&A_t − Capex_t − ΔNWC_t
Debt paydown_t = Free cash flow_t × cash_sweep_pct                    -- 100%
Ending debt_t  = Beginning debt_t − Debt paydown_t
```

Walking Year 1 by hand:

```
Interest_1  = 100,000,000 × 0.08 = 8,000,000
EBT_1       = 16,800,000 − 8,000,000 = 8,800,000
Tax_1       = 8,800,000 × 0.25 = 2,200,000
NI_1        = 6,600,000
FCF_1       = 6,600,000 + 4,200,000 − 3,150,000 − 500,000 = 7,150,000
Paydown_1   = 7,150,000
Ending debt_1 = 100,000,000 − 7,150,000 = 92,850,000
```

Carrying the same mechanic forward through Year 5:

| Year | Beg. debt | Interest | EBT | Tax | Net income | FCF | Paydown | End. debt |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 1 | 100,000,000 | 8,000,000 | 8,800,000 | 2,200,000 | 6,600,000 | 7,150,000 | 7,150,000 | 92,850,000 |
| 2 | 92,850,000 | 7,428,000 | 10,212,000 | 2,553,000 | 7,659,000 | 8,236,500 | 8,236,500 | 84,613,500 |
| 3 | 84,613,500 | 6,769,080 | 11,752,920 | 2,938,230 | 8,814,690 | 9,421,065 | 9,421,065 | 75,192,435 |
| 4 | 75,192,435 | 6,015,395 | 13,432,705 | 3,358,176 | 10,074,529 | 10,711,222 | 10,711,222 | 64,481,213 |
| 5 | 64,481,213 | 5,158,497 | 15,262,008 | 3,815,502 | 11,446,506 | 12,115,034 | 12,115,034 | 52,366,179 |

Notice the debt falls from **$100,000,000 to $52,366,179** — nearly cut in half — over five years, entirely funded by the business's own operations, with no additional capital from the sponsor. Each year's paydown accelerates slightly, because a shrinking debt balance means shrinking interest, which leaves more cash free for the *next* year's paydown — a self-reinforcing effect that is the entire financial engine of a leveraged buyout.

## 6. Exit — realizing the return

At the end of Year 5, the sponsor sells the business. `lbo_assumptions` specifies an **exit multiple equal to the entry multiple** (10.0x EBITDA) — the conservative base case, assuming no change in how the market values this kind of business over the hold period:

```
Exit EV = exit_multiple_ev_ebitda × Year 5 EBITDA
        = 10.0 × 25,525,631
        = $255,256,310

Exit equity value = Exit EV − Ending debt (Year 5)
                   = 255,256,310 − 52,366,179
                   = $202,890,131
```

That **$202,890,131** is the check the sponsor receives at exit, in exchange for the $104,000,000 equity check it wrote on day one.

## 7. MOIC and IRR — the two numbers every sponsor reports

**MOIC** (multiple on invested capital) — the simplest possible return metric, ignoring time entirely:

```
MOIC = Exit equity value / Entry equity check
     = 202,890,131 / 104,000,000
     ≈ 1.95×
```

**IRR** (internal rate of return) — the annualized return that accounts for the 5-year holding period, computed from a single lump sum in and a single lump sum out:

```
IRR = (Exit equity value / Entry equity check)^(1/hold_years) − 1
    = (1.9509)^(1/5) − 1
    ≈ 14.3%
```

A ~1.95x MOIC and ~14.3% IRR over 5 years, with **zero multiple expansion assumed**, is a solidly attractive (if unspectacular) private equity result — real sponsors typically target IRRs in the high teens to low 20s, which means this deal would need either faster growth, more leverage, some multiple expansion at exit, or some combination of the three to clear a typical fund's return hurdle. Challenge 2 decomposes exactly where this 1.95x came from and what levers would move it.

## 8. Computing this with a recursive CTE in SQL

The cash sweep is inherently sequential — exactly the kind of problem Week 7's revenue-compounding `WITH RECURSIVE` was built for:

```sql
WITH RECURSIVE lbo AS (
    -- Year 1 anchor
    SELECT
        1                                                       AS yr,
        t.revenue * (1 + l.revenue_growth_rate)                 AS revenue,
        t.revenue                                                AS prior_revenue,
        l.term_loan_multiple_ebitda * t.ebitda                  AS beg_debt,
        l.da_pct_revenue, l.capex_pct_revenue,
        l.nwc_pct_incremental_revenue, l.term_loan_rate,
        l.tax_rate, l.cash_sweep_pct,
        (t.ebitda / t.revenue)                                  AS ebitda_margin
    FROM target_financials t
    JOIN lbo_assumptions l ON l.target = t.company

    UNION ALL

    SELECT
        yr + 1,
        revenue * (1 + 0.05)                                     AS revenue,
        revenue                                                  AS prior_revenue,
        beg_debt - (
            -- prior year's ending debt = beg_debt - prior year's paydown
            ((revenue * ebitda_margin - revenue * da_pct_revenue)
             - beg_debt * term_loan_rate) * (1 - tax_rate)
            + revenue * da_pct_revenue
            - revenue * capex_pct_revenue
            - (revenue - prior_revenue) * nwc_pct_incremental_revenue
        ) * cash_sweep_pct                                       AS beg_debt,
        da_pct_revenue, capex_pct_revenue, nwc_pct_incremental_revenue,
        term_loan_rate, tax_rate, cash_sweep_pct, ebitda_margin
    FROM lbo
    WHERE yr < 5
)
SELECT yr, ROUND(revenue, 0) AS revenue, ROUND(beg_debt, 0) AS beg_debt
FROM lbo ORDER BY yr;
```

In practice, most analysts find a **plain iterative loop clearer** than folding the whole debt-paydown formula into one recursive expression — Section 9 shows why, and Exercise 3 has you build both so you can compare.

## 9. Computing this in Python — the clearer approach for a path-dependent schedule

```python
import pandas as pd

t = pd.read_sql("SELECT * FROM target_financials WHERE company = 'Solstice Precision Systems, Inc.';", engine).iloc[0]
l = pd.read_sql("SELECT * FROM lbo_assumptions WHERE target = 'Solstice Precision Systems, Inc.';", engine).iloc[0]

entry_ev = l.entry_multiple_ev_ebitda * t.ebitda
fees = l.transaction_fees_pct * entry_ev
total_uses = entry_ev + fees
term_loan = l.term_loan_multiple_ebitda * t.ebitda
sponsor_equity = total_uses - term_loan

rows = []
revenue = t.revenue
debt = term_loan
for year in range(1, l.hold_period_years + 1):
    prior_revenue = revenue
    revenue = revenue * (1 + l.revenue_growth_rate)
    ebitda = revenue * (t.ebitda / t.revenue)     # margin held flat at SPSY's current 20%
    da = revenue * l.da_pct_revenue
    ebit = ebitda - da
    interest = debt * l.term_loan_rate
    ebt = ebit - interest
    tax = ebt * l.tax_rate
    ni = ebt - tax
    capex = revenue * l.capex_pct_revenue
    delta_nwc = (revenue - prior_revenue) * l.nwc_pct_incremental_revenue
    fcf = ni + da - capex - delta_nwc
    paydown = fcf * l.cash_sweep_pct
    end_debt = debt - paydown
    rows.append(dict(year=year, revenue=revenue, ebitda=ebitda, interest=interest,
                      net_income=ni, fcf=fcf, beg_debt=debt, end_debt=end_debt))
    debt = end_debt

schedule = pd.DataFrame(rows)
print(schedule.round(0).to_string(index=False))

exit_ebitda = schedule.iloc[-1].ebitda
exit_ev = l.exit_multiple_ev_ebitda * exit_ebitda
exit_equity = exit_ev - schedule.iloc[-1].end_debt
moic = exit_equity / sponsor_equity
irr = (moic) ** (1 / l.hold_period_years) - 1

print(f"\nSponsor equity in: ${sponsor_equity:,.0f}")
print(f"Exit equity value: ${exit_equity:,.0f}")
print(f"MOIC: {moic:.2f}x   IRR: {irr:.1%}")
```

This reproduces Section 5's table and Section 7's MOIC/IRR exactly. Keep `schedule` — Exercise 3 and Challenge 2 both build directly on top of it.

## 10. Common mistakes

- **Computing interest on the ending debt balance instead of the beginning balance.** Interest accrues on the debt outstanding *during* the year, which is the balance at the start of the year — using the (lower) ending balance understates interest expense and overstates the paydown, compounding the error every subsequent year.
- **Forgetting transaction fees in the sources & uses table.** They're real cash the sponsor spends and never gets back directly — omitting them overstates the sponsor's true return.
- **Applying the exit multiple to entry-year EBITDA instead of the final year's EBITDA.** The business has grown over the hold — using stale EBITDA understates the exit value substantially.
- **Treating MOIC and IRR as interchangeable.** A 1.95x MOIC is a great outcome over 3 years and a mediocre one over 10 — MOIC ignores time entirely, which is exactly why sponsors always report both.

## 11. Check yourself

- Why must "sources" and "uses" always be equal, and what are the two components of "uses" in this lecture's LBO?
- Explain, without the formula in front of you, why interest expense has to be computed on the *beginning* debt balance in a cash-sweep schedule.
- If the exit multiple had been 9.0x instead of 10.0x (multiple compression), would you expect MOIC to rise or fall, and roughly why?
- What two numbers get compared to compute MOIC, and what does IRR add that MOIC alone doesn't tell you?

This closes out the week's three lecture arcs. The mini-project asks you to run CRNM's strategic deal and the sponsor's LBO of the same target side by side, and recommend which buyer SPSY's own board should prefer.

## Further reading

- **Investopedia — "Leveraged Buyout (LBO)":** <https://www.investopedia.com/terms/l/leveragedbuyout.asp>
- **Investopedia — "Cash Sweep":** <https://www.investopedia.com/terms/c/cash-sweep.asp>
- **Investopedia — "Internal Rate of Return (IRR)":** <https://www.investopedia.com/terms/i/irr.asp>
- **PostgreSQL — Recursive Queries (`WITH RECURSIVE`):** <https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-RECURSIVE>
