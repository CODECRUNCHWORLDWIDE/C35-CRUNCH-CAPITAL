# Lecture 3 — Leverage and Returns

> **Duration:** ~2 hours. **Outcome:** You can compute the Degree of Financial Leverage (DFL) for any company, explain why leverage amplifies both the upside and the downside of equity returns by the same multiple, and build a scenario table showing exactly how much bigger a swing in net income and ROE is than the swing in EBIT that caused it.

Lectures 1 and 2 were about firm **value** — what the whole pie is worth under different debt levels. This lecture is about something a shareholder feels more directly: what leverage does to the **volatility of their own return**, independent of whether the tradeoff model says the debt level is value-maximizing or not. Two firms can have identical assets and identical EBIT, but the levered one's equity holders will experience a rollercoaster the unlevered one's equity holders never feel — and you're about to learn to put a precise multiple on how much bigger that rollercoaster is.

## 1. Fixed cost, variable earnings — the mechanical source of amplification

Interest expense is (mostly) a **fixed cost** — a company that borrowed a fixed principal at a fixed rate owes the same interest payment whether EBIT is high or low that year (variable-rate debt complicates this at the margin, but the core mechanic holds: the payment doesn't scale down automatically with a bad year the way, say, a percentage-of-revenue royalty would). Net income is what's left over *after* that fixed obligation is paid — so net income has to absorb 100% of any change in EBIT, on top of already being a smaller base number than EBIT.

**Simple illustration.** A firm with `EBIT = 1,000` and `interest = 400` has `pretax income = 600`. If EBIT falls 10% to `900`, pretax income falls to `500` — a swing of `100` on a base of `600`, or **16.7%**, not 10%. The interest payment absorbed none of the shock; net income absorbed all of it, on a smaller starting base. That gap between the percentage change in EBIT and the (larger) percentage change in what's left for equity is exactly what leverage amplification measures.

## 2. Degree of Financial Leverage (DFL)

$$DFL = \frac{\% \Delta \text{ net income (or EPS)}}{\% \Delta \text{ EBIT}} = \frac{EBIT}{EBIT - \text{interest}} = \frac{EBIT}{\text{pretax income}}$$

Read `DFL = 2.0` as: **a 1% change in EBIT produces roughly a 2% change in net income (and EPS, if share count is fixed).** DFL is always `≥ 1` for a firm with any positive interest expense, and it climbs toward infinity as `EBIT` approaches `interest` (pretax income approaches zero) — the closer a firm sits to its breakeven coverage point, the more violently a small EBIT wobble gets amplified into net income.

**Worked example, continued from Section 1.** `EBIT = 1,000`, `interest = 400`, `pretax = 600`:

```
DFL = 1,000 / 600 = 1.67
```

A 10% EBIT decline produces roughly a `10% × 1.67 = 16.7%` net income decline — matching the exact swing computed by hand above. That's not a coincidence; the formula is derived precisely so the ratio holds for small percentage changes around the current EBIT level.

## 3. Where the formula comes from

It's worth deriving `DFL = EBIT / pretax income` once, so it doesn't feel like a memorized rule. Net income is `NI = (EBIT − Interest) × (1 − Tc)` — interest is fixed, so any change in EBIT flows straight through to a change in `(EBIT − Interest)`, and the `(1 − Tc)` tax factor multiplies both the before-change and after-change value identically. Write `%ΔNI` as `ΔNI / NI`:

```
%ΔNI = [ΔEBIT × (1 − Tc)] / [(EBIT − Interest) × (1 − Tc)] = ΔEBIT / (EBIT − Interest)
```

The `(1 − Tc)` term cancels top and bottom — **DFL doesn't depend on the tax rate at all**, only on the ratio of EBIT to pretax income. Now divide by `%ΔEBIT = ΔEBIT / EBIT`:

```
DFL = %ΔNI / %ΔEBIT = [ΔEBIT / (EBIT − Interest)] / [ΔEBIT / EBIT] = EBIT / (EBIT − Interest)
```

The `ΔEBIT` terms cancel too — which is *why* DFL is a single fixed number at a given EBIT level, valid for a change of any size in either direction, not just the specific 10% used in Section 1's illustration. (If you're computing DFL for **EPS** rather than total net income, and the company has preferred stock outstanding, subtract preferred dividends from the numerator before dividing — preferred dividends are, like interest, a fixed claim paid before what's left over for common shareholders. This week's Crunch Machine Co. has no preferred stock, so plain net income and EPS amplify identically.)

## 4. DFL is symmetric — it cuts both ways

The same multiple that turns a bad year worse turns a good year *better*. If that same firm's EBIT instead **rises** 10% to `1,100`, pretax income rises to `700` — a swing of `100` on a base of `600`, again **16.7%**, matching `DFL = 1.67` exactly. This symmetry is the entire reason leverage is attractive when times are good and dangerous when times are bad: it doesn't choose a direction, it just multiplies whatever direction EBIT happens to move.

This is why "leverage" is the right word — like a mechanical lever, it doesn't create force, it **redirects and amplifies** whatever force (EBIT growth or EBIT decline) is already being applied.

## 5. Crunch Machine Co. — DFL rising as coverage falls

```sql
SELECT fiscal_year,
       operating_income                                  AS ebit,
       pretax_income,
       ROUND(operating_income / pretax_income, 2)          AS dfl
FROM income_statement
ORDER BY fiscal_year;
```

| Year | EBIT | Pretax income | DFL |
|-----:|-----:|---------------:|----:|
| 2021 | 7,140 | 6,520 | 1.10 |
| 2022 | 7,200 | 6,510 | 1.11 |
| 2023 | 6,045 | 5,235 | 1.15 |
| 2024 | 4,140 | 3,090 | 1.34 |
| 2025 | 2,225 | 845 | **2.63** |

In 2021, Crunch Machine Co.'s leverage barely amplified anything — DFL of 1.10 means a 10% EBIT move produced roughly an 11% net-income move, hardly different from an all-equity firm. By 2025, that same 10% EBIT move produces roughly a **26.3%** net-income move. The company's earnings became far more volatile purely as a mechanical consequence of the rising interest burden relative to a shrinking EBIT base — this is the same trajectory Lecture 2 read through interest coverage, seen from the return-volatility side instead of the distress-probability side. They're two views of one underlying fact: fixed interest claims are eating a bigger and bigger share of a smaller and smaller operating profit.

## 6. Return on equity: unlevered vs. levered, same EBIT swing

DFL tells you the *ratio* of the swings. It's often more intuitive to see the actual **ROE** numbers side by side for a levered firm versus a hypothetical unlevered version of the same assets, under a good year and a bad year.

Take a simplified example firm with `total assets = 20,000`, split two ways: **Unlevered** (`equity = 20,000`, `debt = 0`) versus **Levered** (`debt = 10,000` at `rD = 7%`, `equity = 10,000`). Base-case `EBIT = 2,500`, `Tc = 25%`. Compute net income and ROE for a **bad year** (`EBIT` down 20% to `2,000`) and a **good year** (`EBIT` up 20% to `3,000`):

```python
import pandas as pd

def net_income(ebit: float, interest: float, tax_rate: float) -> float:
    pretax = ebit - interest
    return pretax * (1 - tax_rate)

scenarios = {"Bad year (EBIT -20%)": 2000, "Base case": 2500, "Good year (EBIT +20%)": 3000}
rows = []
for label, ebit in scenarios.items():
    ni_unlevered = net_income(ebit, interest=0,   tax_rate=0.25)
    ni_levered   = net_income(ebit, interest=700, tax_rate=0.25)   # 7% x 10,000
    rows.append({
        "scenario": label,
        "roe_unlevered": round(ni_unlevered / 20000, 4),   # equity = 20,000
        "roe_levered":   round(ni_levered   / 10000, 4),   # equity = 10,000
    })

print(pd.DataFrame(rows))
```

```
               scenario  roe_unlevered  roe_levered
0  Bad year (EBIT -20%)         0.0750       0.0975
1             Base case         0.0938       0.1350
2 Good year (EBIT +20%)         0.1125       0.1725
```

Two things to notice. First, the **levered ROE is higher in every scenario** here — that's not a leverage law, it's specific to this example because `rU` (the assets' return) exceeds `rD` (7%), so debt is "working for" equity holders even before considering amplification (this is the same MM Proposition II mechanic from Lecture 1, viewed through realized returns rather than required returns). Second, and more importantly: the **spread** between the bad-year and good-year ROE is far wider for the levered firm (`0.0975` to `0.1725`, a range of `0.075`) than for the unlevered firm (`0.0750` to `0.1125`, a range of `0.0375`) — **exactly double**, matching a DFL you can verify directly: `DFL = 2500/(2500-700) = 2500/1800 = 1.39`, and `1.39` applied to the unlevered ROE range doesn't quite double it on its own — the extra amplification here also comes from equity being a smaller base (`10,000` vs `20,000`) doing double duty, on top of the DFL effect. Disentangling exactly how much of the total gap is "DFL on income" versus "smaller equity base" is precisely what Exercise 2 has you build out in full.

## 7. Operating leverage and combined leverage — the other lever

DFL isn't the only amplifier in a company's income statement. **Degree of Operating Leverage (DOL)** measures the same kind of amplification, but from **fixed operating costs** (rent, salaried staff, depreciation on owned equipment) rather than fixed financing costs:

$$DOL = \frac{\% \Delta \text{ EBIT}}{\% \Delta \text{ revenue}}$$

A company with a lot of fixed operating cost relative to variable cost (a factory with expensive machinery and a small variable-labor component, say) has high DOL — a small revenue swing produces a large EBIT swing, *before* interest even enters the picture. Multiply the two amplifiers together and you get the **Degree of Combined Leverage (DCL)**:

$$DCL = DOL \times DFL = \frac{\% \Delta \text{ net income}}{\% \Delta \text{ revenue}}$$

This week's course stays focused on DFL — the financing-side lever, which is what a capital-structure decision directly controls — but DCL is worth keeping in the back of your mind: a company with **both** high operating leverage and high financial leverage is the most fragile kind of business, because a modest revenue miss gets amplified twice before it reaches net income. A capital-intensive manufacturer like Crunch Machine Co. — heavy fixed costs from `ppe_net` and depreciation, on top of the rising interest burden this lecture has focused on — plausibly carries meaningful DOL too, which would make its true combined fragility worse than the DFL-only view in Section 5 shows on its own. Quantifying DOL properly requires splitting costs into fixed and variable components, which needs cost data this week's `income_statement` table doesn't break out — a natural next step if you ever model this company's operations, not something this week's seed data supports directly.

## 8. The other side of amplification: leverage doesn't add value, it redistributes risk

Tie this back to Lecture 1's Proposition II. Under perfect markets (no taxes), the *extra* expected return equity holders get from leverage is exactly offset, on a risk-adjusted basis, by the extra volatility they're taking on — there's no free lunch, just a repackaging of the same underlying asset risk onto a smaller equity base. What Lectures 1 and 2 add on top of that baseline picture is real: the tax shield genuinely creates value (Lecture 1), and expected distress costs genuinely destroy it (Lecture 2) — but the *pure leverage-amplification* effect on returns, by itself, is risk redistribution, not value creation. A CFO deciding how much debt to carry has to weigh all three effects together — which is exactly the setup for this week's mini-project.

## 9. Check yourself

- Write the DFL formula two ways — in terms of `EBIT` and `interest`, and in terms of `EBIT` and `pretax income`. Confirm they're the same thing.
- Why is DFL always `≥ 1` for a firm with positive interest expense?
- If DFL = 3.0 and EBIT falls 5%, roughly what happens to net income?
- Explain, in one sentence, why DFL amplification is symmetric — it doesn't distinguish between good news and bad news.
- For Crunch Machine Co., what happened to DFL between 2021 and 2025, and what single balance-sheet and income-statement trend (from Lecture 2) is driving that change?
- In the unlevered-vs-levered ROE example, why is the levered firm's ROE range wider than its DFL alone would suggest? Name the second factor at work.
- Write the DCL formula in terms of DOL and DFL. Why would a company with high values of *both* be more fragile than one with a high value of only one?

## Further reading

- **Investopedia — "Degree of Financial Leverage (DFL)":** <https://www.investopedia.com/terms/d/degreeoffinancialleverage.asp>
- **Investopedia — "Financial Leverage":** <https://www.investopedia.com/terms/l/leverage.asp>
- **Investopedia — "Return on Equity (ROE)":** <https://www.investopedia.com/terms/r/returnonequity.asp>
- **Investopedia — "Degree of Operating Leverage (DOL)":** <https://www.investopedia.com/terms/d/degreeofoperatingleverage.asp>
- **Investopedia — "Degree of Combined Leverage":** <https://www.investopedia.com/terms/c/combinedleverage.asp>
- **Damodaran (NYU Stern) — "Financial Leverage: Boon or Bane?":** <https://pages.stern.nyu.edu/~adamodar/pdfiles/ovhds/capstr.pdf>
