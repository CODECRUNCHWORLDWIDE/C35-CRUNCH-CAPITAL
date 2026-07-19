# Lecture 2 — Debt Instruments and Covenants

> **Duration:** ~2 hours. **Outcome:** You can build a full loan amortization schedule from the annuity formula, in SQL and Python, and write the two covenant tests — leverage and interest coverage — a lender attaches to a loan to protect its money, and compute exactly how much financial cushion a company has under each.

## 1. What "raising debt" actually means

Debt is the mirror image of equity from Lecture 1. Instead of selling a slice of the company forever, you're **borrowing** a fixed amount and promising a fixed schedule of payments back, with **interest** as the price of the loan. Nobody gives up ownership. Nobody gets diluted. But the payments are **contractual, not optional** — miss one, and the lender has real legal remedies a diluted shareholder never gets. That fixed obligation, regardless of how the business actually performs, is the central tradeoff of debt, and it's what the rest of this lecture is built around.

Crunch Machine Co.'s second term sheet: a **$20,000,000, 5-year, fully amortizing term loan at 7.5% annual interest**, from this week's `financing_terms` table:

```sql
SELECT * FROM financing_terms WHERE instrument_type = 'Debt';
```

## 2. The coupon, and what "fully amortizing" means

The **coupon rate** — 7.5% here — is the annual interest rate the lender charges on the *outstanding* balance. "Fully amortizing" means the loan is paid off with **equal payments** over its life, each one a blend of interest and principal, such that the balance hits exactly zero at the final payment. This is the same annuity-payment mechanic from Week 1's amortization-schedule challenge — same formula, now applied to a lender's term sheet with covenants attached:

```
Payment = P × [ r(1 + r)^n ] / [ (1 + r)^n − 1 ]
```

Where `P = 20,000,000` (principal), `r = 0.075` (annual rate — this loan pays **annually**, not monthly, so no rate conversion is needed), and `n = 5` (annual payments over the 5-year term):

```
Payment = 20,000,000 × [0.075 × 1.075^5] / [1.075^5 − 1]
        ≈ 4,943,294.36
```

Every year, Crunch Machine Co. pays the lender **$4,943,294.36**, no more, no less, for five years — split differently between interest and principal each time.

## 3. Building the full schedule

Each year, the split follows the same two-line mechanic from Week 1:

```
interest_this_year  = beginning_balance × 0.075
principal_this_year = payment − interest_this_year
ending_balance       = beginning_balance − principal_this_year
```

Run it for all 5 years:

| Period | Beginning balance | Payment | Interest paid | Principal paid | Ending balance |
|-------:|-------------------:|--------:|---------------:|-----------------:|-----------------:|
| 1 | 20,000,000.00 | 4,943,294.36 | 1,500,000.00 | 3,443,294.36 | 16,556,705.64 |
| 2 | 16,556,705.64 | 4,943,294.36 | 1,241,752.92 | 3,701,541.43 | 12,855,164.21 |
| 3 | 12,855,164.21 | 4,943,294.36 | 964,137.32 | 3,979,157.04 | 8,876,007.17 |
| 4 | 8,876,007.17 | 4,943,294.36 | 665,700.54 | 4,277,593.82 | 4,598,413.35 |
| 5 | 4,598,413.35 | 4,943,294.36 | 344,881.00 | 4,598,413.35 | 0.00 |

Total interest paid across the full 5 years: **$4,716,471.78** — on top of repaying the original $20,000,000 principal, the company pays almost $4.72 million just for the privilege of having the cash five years early. Notice the same front-loading pattern from Week 1: year 1's interest ($1.5M) is more than **4×** year 5's ($344,881), because interest is always charged on whatever balance is still outstanding, and that balance shrinks every year.

## 4. Doing this in SQL and Python

The annuity payment formula translates directly into a query, exactly like Week 1's amortization work:

```sql
-- The level annual payment
SELECT
    20000000 * (0.075 * POWER(1.075, 5)) / (POWER(1.075, 5) - 1) AS annual_payment;
-- ≈ 4943294.36
```

Generating the full year-by-year schedule needs a running balance, which is easiest as an **iterative** calculation (in Python, or a recursive CTE in SQL if your engine supports it) rather than one flat query — each year's interest depends on the *previous* year's ending balance, which is exactly the kind of recurrence relation SQL's `WITH RECURSIVE` exists for, but that plumbing is more naturally expressed as a small Python loop that you then load into a table:

```python
def loan_schedule(principal: float, annual_rate: float, years: int) -> list[dict]:
    """Full amortization schedule for a fixed-rate, fully-amortizing annual-payment loan."""
    payment = principal * (annual_rate * (1 + annual_rate) ** years) / ((1 + annual_rate) ** years - 1)
    balance = principal
    schedule = []
    for period in range(1, years + 1):
        interest = balance * annual_rate
        principal_paid = payment - interest
        balance -= principal_paid
        schedule.append({
            "period": period,
            "payment": round(payment, 2),
            "interest_paid": round(interest, 2),
            "principal_paid": round(principal_paid, 2),
            "ending_balance": round(balance, 2),
        })
    return schedule

rows = loan_schedule(20_000_000, 0.075, 5)
for row in rows:
    print(row)
```

Load the result into SQL exactly as Week 1's challenge did — via `pandas.DataFrame(rows).to_sql("term_loan_schedule", engine, if_exists="replace", index=False)` — and every downstream question ("how much interest in year 3?", "what's still owed after year 2?") becomes a `SELECT`, not a re-run of the Python.

## 5. Seniority — who gets paid first

Not all debt (or equity) sits at the same place in line if the company can't pay everyone. **Seniority** is the ranking of who gets repaid first if the company is liquidated or restructured. From most to least protected:

1. **Senior secured debt** — backed by specific collateral (equipment, real estate, receivables); if the company defaults, this lender can seize and sell that specific collateral before anyone else gets paid. Crunch Machine Co.'s term loan is **secured** — the Midwest distribution hub itself is the collateral.
2. **Senior unsecured debt** — no specific collateral, but still ranks ahead of everything below it in a general claim on the company's assets.
3. **Subordinated (junior) debt** — explicitly ranks behind senior debt; often carries a higher coupon to compensate lenders for that added risk.
4. **Preferred equity** — ranks behind all debt, but ahead of common equity.
5. **Common equity** — last in line. Common shareholders (everyone in this week's `shareholders` table) get paid only after every class of debt and preferred equity above them is made whole — which, in a real liquidation, is frequently nothing.

This ordering is exactly why lenders can accept a lower return than equity investors demand for the same company: secured, senior debt is structurally far safer, and price follows risk. It's also why the term loan's collateral and seniority matter as much as its coupon rate when you're comparing "the cost of debt" to "the cost of equity" — they are not compensating for the same amount of risk.

## 6. Covenants — the conditions attached to the loan

A **covenant** is a contractual condition the borrower must keep meeting for the *life* of the loan, not just at signing. Break one, and the loan can be declared in **technical default** even if every payment has been made on time — which typically lets the lender demand immediate repayment, renegotiate at a higher rate, or seize collateral, depending on what the loan agreement says. Covenants exist because a lender who's owed a fixed amount, no matter how well or badly the business does, wants an early-warning system that fires *before* the company gets close to being unable to pay — not after.

Crunch Machine Co.'s term loan carries two standard **financial covenants**:

### 6.1 Maximum leverage — Total Debt / EBITDA ≤ 3.00×

```
Leverage ratio = Total Debt / EBITDA
```

`EBITDA` (earnings before interest, taxes, depreciation, and amortization) is used here instead of net income because it's a cleaner proxy for the cash a business generates before financing and accounting choices — exactly the cash available to service debt. Post-raise, Crunch Machine Co.'s total debt is its existing $30,000,000 plus the new $20,000,000 term loan:

```sql
SELECT
    (existing_debt + 20000000) AS total_debt_post,
    ebitda,
    ROUND((existing_debt + 20000000) / ebitda, 3) AS leverage_ratio
FROM company_snapshot;
```

`(30,000,000 + 20,000,000) / 22,000,000 = 2.273×` — **within** the 3.00× covenant, with real headroom (EBITDA could fall to about $16.67M before this covenant alone would be breached, holding debt constant).

### 6.2 Minimum interest coverage — EBIT / Interest Expense ≥ 4.00×

```
Interest coverage ratio = EBIT / Total Interest Expense
```

`EBIT` (earnings before interest and taxes — EBITDA minus depreciation and amortization) measures the operating earnings actually available to cover interest payments. This covenant asks: *for every dollar of interest owed this year, how many dollars of operating earnings does the company have to cover it?* A higher number means more cushion.

```sql
SELECT
    (ebitda - depreciation_amort)                                     AS ebit,
    (existing_debt * existing_debt_rate) + 1500000.00                 AS total_interest_year1,
    ROUND((ebitda - depreciation_amort)
        / ((existing_debt * existing_debt_rate) + 1500000.00), 3)     AS interest_coverage
FROM company_snapshot;
```

`EBIT = 22,000,000 − 4,000,000 = 18,000,000`. Year-1 interest is the existing debt's interest (`30,000,000 × 6% = 1,800,000`) plus the new loan's year-1 interest (`$1,500,000`, from the schedule above) `= 3,300,000`. Coverage `= 18,000,000 / 3,300,000 ≈ 5.455×` — comfortably above the 4.00× minimum.

### 6.3 Other common covenant types (not modeled numerically this week, but worth knowing)

- **Negative pledge** — the borrower cannot grant a security interest in its assets to any *other* lender without this lender's consent, protecting the original lender's claim on collateral from being diluted by a later, competing loan.
- **Restricted payments** — caps or bans dividends and share buybacks above a leverage threshold, so the company can't borrow money and then hand cash straight to shareholders instead of using it (or keeping it as a cushion) to service the debt.
- **Affirmative covenants** — ongoing obligations, not limits: deliver audited financials to the lender quarterly, maintain insurance on collateral, keep the business properly licensed.

## 7. Why covenants matter to the *cost* comparison, not just the risk comparison

A loan's quoted 7.5% coupon is not the full price of debt. Covenants constrain what the company can do with its own cash and assets for the life of the loan — no extra secured borrowing without consent, no dividend increases if leverage rises, mandatory financial reporting. None of that shows up in the interest rate, but all of it is a real cost: **lost flexibility**, priced implicitly rather than explicitly. Lecture 3 puts a name to weighing this kind of cost against equity's dilution cost — they are different currencies, and a CFO has to convert both into the same terms to actually compare them.

## 8. Common mistakes

- **Using the wrong balance for a period's interest.** Interest is always computed on the **beginning-of-period** balance (what's still owed *before* this period's payment), not the ending balance or the original principal.
- **Testing a covenant against the wrong debt figure.** Leverage covenants almost always mean **total** debt (existing plus new), not just the new loan being negotiated — a company that forgets its existing $30M when checking the covenant will understate its leverage and get an unpleasant surprise.
- **Confusing EBITDA and EBIT.** They differ by depreciation and amortization — mixing them up in a covenant test changes the answer materially, since D&A is $4,000,000 here, a large enough number to matter.
- **Treating "coupon rate" as "true cost of debt."** The coupon is the interest cost; origination fees, covenant-driven flexibility loss, and (as Week 4 covered) the after-tax adjustment from the interest tax shield all belong in the *true* cost, which Lecture 3 builds explicitly.

## 9. Check yourself

- Write the annuity payment formula from memory, and explain in one sentence why year 1's interest is always the largest of any year in a fully amortizing loan.
- What does "fully amortizing" mean, and how does it differ from a loan that pays interest-only until a single lump-sum payoff at maturity (a "bullet" loan)?
- Why would a *secured* lender accept a lower coupon rate than an *unsecured* lender, for lending to the exact same company?
- In your own words, what is a covenant actually protecting the lender against, and why does it matter that it's tested continuously rather than only checked once at signing?
- Crunch Machine Co.'s leverage ratio post-raise is 2.273×, against a 3.00× covenant. Name one thing that could happen to the company's EBITDA that would push it over that line, holding debt constant.

Lecture 3 puts the equity path from Lecture 1 and the debt path from this lecture side by side, computes the true, all-in cost of each, and works through the control and signaling tradeoffs that decide which one a real CFO picks.

## Further reading

- **Investopedia — "Amortization":** <https://www.investopedia.com/terms/a/amortization.asp>
- **Investopedia — "Debt Covenant":** <https://www.investopedia.com/terms/d/debtcovenant.asp>
- **Investopedia — "Seniority":** <https://www.investopedia.com/terms/s/seniority.asp>
- **Investopedia — "Interest Coverage Ratio":** <https://www.investopedia.com/terms/i/interestcoverageratio.asp>
- **PostgreSQL — Recursive Queries (`WITH RECURSIVE`):** <https://www.postgresql.org/docs/current/queries-with.html>
