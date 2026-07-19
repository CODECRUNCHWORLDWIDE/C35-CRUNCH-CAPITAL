# Lecture 1 — MM and the Tax Shield

> **Duration:** ~2 hours. **Outcome:** You can state Modigliani-Miller's Proposition I and II with and without taxes, explain exactly which assumption taxes relax, and compute the interest tax shield — both its annual value and its present value — for any company, in SQL and Python.

## 1. The question capital structure asks

A company's **capital structure** is just the mix of debt and equity it uses to fund its assets — how much of the balance sheet's right-hand side is `total_liabilities` versus `total_equity`. Every dollar of assets has to be financed somehow, and every company chooses some mix of borrowed money and owners' money to do it.

Here's the puzzle. Debt is almost always *cheaper* than equity — lenders take less risk than shareholders (they get paid first, and a fixed amount), so they demand a lower return. Debt also comes with a tax break: interest is deductible, dividends aren't. So why doesn't every company borrow as much as it possibly can?

Two economists, Franco Modigliani and Merton Miller, answered a narrower version of this question first, in 1958 — and their answer is the foundation everything else this week builds on.

## 2. MM Proposition I — without taxes

**Claim:** in a world with no taxes, no bankruptcy costs, no transaction costs, and no information asymmetry (a "perfect capital market"), **the value of a firm does not depend on how it is financed.**

$$V_L = V_U$$

`V_L` is the value of the levered firm (some debt, some equity); `V_U` is the value of the same firm's assets if financed entirely with equity. MM's proof is an arbitrage argument: if `V_L` were somehow *higher* than `V_U`, an investor could borrow on their own account, buy the unlevered firm's shares, replicate the exact cash flows of the levered firm's equity, and pocket the difference risk-free. That opportunity can't survive in an efficient market — so the two values must converge.

The intuition without the arbitrage machinery: **slicing a pizza into more pieces doesn't make the pizza bigger.** Debt and equity are just two different claims on the *same* underlying pile of operating cash flows. Repackaging those claims — more debt, less equity, or vice versa — doesn't change the size of the pile.

This is a genuinely surprising result, and it's easy to under-appreciate how radical it was in 1958: it says capital structure, the entire subject you're about to spend a week on, *doesn't matter*, full stop, as long as markets are perfect. Everything from here forward is about what happens once you relax that "perfect markets" assumption — and this week focuses on the two that matter most: **taxes** (this lecture) and **distress costs** (Lecture 2).

## 3. MM Proposition II — without taxes

Proposition I says total firm value doesn't move. Proposition II says something about the *pieces* — specifically, what happens to the cost of equity as leverage rises.

$$r_E = r_U + \frac{D}{E}(r_U - r_D)$$

Where `r_E` is the required return on levered equity, `r_U` is the required return on the firm's unlevered assets, `r_D` is the cost of debt, and `D/E` is the debt-to-equity ratio. Read it as: **equity holders demand a higher return as leverage rises, and the size of the increase is exactly proportional to the debt-to-equity ratio, scaled by the spread between the asset return and the debt return.**

Why does this have to be true, given Proposition I? Because debt is safer than the underlying assets (`r_D < r_U`), adding debt makes the *remaining* equity claim riskier — equity now absorbs 100% of the asset risk on a smaller sliver of the capital structure, like concentrating the same total risk onto fewer shares. The cost of equity rises exactly enough to offset the cheaper debt, so the blended cost of capital (WACC) **stays flat** as leverage changes. That flat-WACC result is the mechanical restatement of Proposition I: if the average cost of capital never moves, and it discounts the same operating cash flows, firm value can't move either.

**Worked example.** A firm has `r_U = 10%` and can borrow at `r_D = 6%`. At `D/E = 0.5`:

```
r_E = 0.10 + 0.5 × (0.10 − 0.06) = 0.10 + 0.02 = 0.12  →  12%
```

At `D/E = 2.0` (much more levered):

```
r_E = 0.10 + 2.0 × (0.10 − 0.06) = 0.10 + 0.08 = 0.18  →  18%
```

Triple the leverage, and the cost of equity jumps 6 percentage points. Shareholders are pricing in exactly the extra risk concentration leverage creates — no more, no less. Under Proposition I, none of this changes what the *firm* is worth; it only reshuffles who bears the risk and at what price.

## 4. Adding taxes — the assumption that breaks the tie

Real tax codes let corporations deduct interest expense before computing taxable income, but they do **not** let corporations deduct dividends. That single asymmetry is enough to break Proposition I's "capital structure doesn't matter" result, because now the two financing choices don't split the *same* pie — debt financing lets the company hand a smaller slice to the tax authority first.

**MM Proposition I, with taxes:**

$$V_L = V_U + T_c \times D$$

Where `T_c` is the marginal corporate tax rate and `D` is the (permanent) face value of debt. The term `T_c × D` is the **present value of the interest tax shield** — the extra value created purely because interest payments are tax-deductible. This is the headline result of MM's 1963 follow-up paper, and it says the opposite of the 1958 result: with taxes, more debt *does* increase firm value, dollar for dollar with `T_c × D`, holding everything else fixed.

**MM Proposition II, with taxes** (the cost-of-equity version adjusts too):

$$r_E = r_U + \frac{D}{E}(1 - T_c)(r_U - r_D)$$

The `(1 − T_c)` term shrinks the leverage penalty on equity's required return — part of the extra risk shareholders would otherwise demand compensation for is offset by the fact that debt is subsidized by the tax deduction.

## 5. Why the shield is worth `Tc × D`

Walk through the mechanics once, because the formula should feel inevitable, not memorized.

Every period, a levered firm pays `r_D × D` in interest. Because that interest is tax-deductible, the firm's taxable income falls by `r_D × D`, so its tax bill falls by `T_c × r_D × D`. That reduction in taxes — money the firm *would have* paid the government but now keeps — is the **annual interest tax shield**:

```
Annual tax shield = Tc × interest expense = Tc × rD × D
```

If the firm maintains that same level of debt `D` forever (the "permanent debt" assumption MM originally used), that shield is a **perpetuity**. What's the right rate to discount a perpetuity of tax shields? MM's original argument: since the tax shield's size is driven by the debt payment itself, it's roughly as risky as the debt, so discount it at `r_D`:

```
PV(tax shield) = (Tc × rD × D) / rD = Tc × D
```

The `r_D` cancels cleanly out of the formula — that's *why* the tax shield's present value doesn't depend on the interest rate at all, only on the tax rate and the debt level. (Some models discount the shield at `r_U` instead, arguing it's as risky as the firm's *assets*, not its debt — that produces a different, smaller formula. Different textbooks pick different conventions; this course uses the `Tc × D` perpetuity form, the one MM themselves proposed and still the most commonly taught.)

## 6. Finite-life debt — when the perpetuity shortcut doesn't apply

`Tc × D` assumes the debt is rolled over forever. Real debt has a maturity — a 5-year term loan, a 10-year bond. If debt won't be permanent, don't use the perpetuity shortcut; instead, discount each period's actual shield back individually:

$$PV(\text{tax shield}) = \sum_{t=1}^{n} \frac{T_c \times \text{interest}_t}{(1 + r_D)^t}$$

In SQL, if you have a loan amortization schedule (built exactly like Week 1's Challenge 1) with one row per period and an `interest_paid` column, the finite-life shield is one query:

```sql
SELECT SUM(0.25 * interest_paid / POWER(1 + 0.08, period)) AS pv_tax_shield
FROM warehouse_loan_schedule;
```

In Python, the same sum over a `pandas` DataFrame:

```python
import pandas as pd

def pv_finite_tax_shield(schedule: pd.DataFrame, tax_rate: float, discount_rate: float) -> float:
    """PV of the interest tax shield over a finite amortization schedule."""
    shields = tax_rate * schedule["interest_paid"]
    discount_factors = 1 / (1 + discount_rate) ** schedule["period"]
    return float((shields * discount_factors).sum())
```

The finite-life sum will always be **smaller** than `Tc × D`, because a loan that eventually gets paid off stops generating a tax shield — `Tc × D` implicitly assumes the shield never ends.

## 7. Crunch Machine Co. — pricing the shield, 2021 vs. 2025

Pull the numbers straight from the seed tables:

```sql
SELECT i.fiscal_year,
       i.operating_income                          AS ebit,
       i.interest_expense,
       b.short_term_debt + b.long_term_debt          AS total_debt,
       ROUND(0.25 * i.interest_expense, 1)            AS annual_tax_shield,
       ROUND(0.25 * (b.short_term_debt + b.long_term_debt), 1) AS pv_tax_shield_perpetual
FROM income_statement i
JOIN balance_sheet b ON b.fiscal_year = i.fiscal_year
WHERE i.fiscal_year IN (2021, 2025)
ORDER BY i.fiscal_year;
```

| Year | EBIT | Interest | Total debt | Annual shield (`Tc × interest`) | PV of shield (`Tc × D`, perpetual) |
|-----:|-----:|---------:|-----------:|---------------------------------:|-------------------------------------:|
| 2021 | 7,140 | 620 | 11,300 | 155.0 | 2,825.0 |
| 2025 | 2,225 | 1,380 | 19,700 | 345.0 | 4,925.0 |

Crunch Machine Co.'s effective tax rate has held almost exactly at **25%** all five years (`tax_expense / pretax_income` — verify it yourself), which is why `Tc = 0.25` is used everywhere this week. As the company borrowed more, both the annual shield and its present value roughly doubled — debt looks, on this slice of the picture alone, like a clean win.

Now compute what happened to the **unlevered** value over the same period, using `r_U = 10%` (this week's stated assumption for Crunch Machine Co.):

```
VU = EBIT × (1 − Tc) / rU

VU(2021) = 7,140 × 0.75 / 0.10 = 53,550.0
VU(2025) = 2,225 × 0.75 / 0.10 = 16,687.5
```

`V_U` fell by **36,862.5** over four years — the tax shield gained about **2,100** over the same stretch. **The tax shield is real, but for a company whose operating income is collapsing, it is a rounding error next to the damage done by the operating decline itself.** Keep that comparison in mind heading into Lecture 2: the tax shield alone never explains why a company would choose to be *cautious* about debt. Something has to sit on the other side of the ledger and grow faster than the shield as leverage rises — that something is the expected cost of financial distress.

## 8. Check yourself

- In one sentence, what assumption does MM Proposition I require that real markets don't actually have?
- Why does `r_E` rise with leverage under Proposition II, even though total firm value doesn't move?
- Write the formula for the annual interest tax shield. What two things multiply together to produce it?
- Why does the discount rate `r_D` cancel out of the perpetual tax shield formula, leaving just `Tc × D`?
- When should you use the finite-life sum instead of the `Tc × D` shortcut?
- For Crunch Machine Co., which grew more from 2021 to 2025: the interest tax shield, or the loss in unlevered value? What does that comparison suggest about how much weight the tax shield alone should carry in a real financing decision?

## Further reading

- **Modigliani & Miller (1958) — "The Cost of Capital, Corporation Finance and the Theory of Investment"** (the original Proposition I/II paper — summary and context): <https://www.jstor.org/stable/1809766>
- **Modigliani & Miller (1963) — "Corporate Income Taxes and the Cost of Capital: A Correction"** (the paper that adds `Tc × D`): <https://www.jstor.org/stable/1809167>
- **Investopedia — "Modigliani-Miller Theorem":** <https://www.investopedia.com/terms/m/modigliani-miller.asp>
- **Investopedia — "Tax Shield":** <https://www.investopedia.com/terms/t/taxshield.asp>
- **Damodaran (NYU Stern) — "Capital Structure: The Choices and the Trade Off":** <https://pages.stern.nyu.edu/~adamodar/pdfiles/ovhds/capstr.pdf>
