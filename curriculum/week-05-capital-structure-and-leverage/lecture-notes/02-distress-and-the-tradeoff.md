# Lecture 2 — Distress and the Tradeoff

> **Duration:** ~2 hours. **Outcome:** You can define direct and indirect costs of financial distress, read a company's interest coverage trend as a distress signal, build a stylized tradeoff model combining the tax shield against expected distress costs, and solve that model for a value-maximizing debt level.

Lecture 1 ended on a cliffhanger: the interest tax shield grows steadily as a company borrows more, but for Crunch Machine Co. it was a rounding error next to the value the company lost as its operating income collapsed. If `Tc × D` were the whole story, every firm would lever up until debt was nearly 100% of the balance sheet. Real companies don't do that. This lecture supplies the missing force that stops them.

## 1. What "financial distress" actually means

**Financial distress** is the state a company enters when it struggles to meet its debt obligations — not necessarily bankruptcy itself, but the zone of difficulty that often precedes it: missed or renegotiated payments, breached loan covenants, credit-rating downgrades, suppliers demanding cash on delivery instead of extending terms, key employees updating their résumés.

Distress is costly, and the costs split into two categories.

### Direct costs

The out-of-pocket costs of an actual bankruptcy filing or restructuring: legal fees, accounting fees, court costs, investment-banker fees for a restructuring advisory. Empirically, these tend to run **3%–5% of firm value** for a typical bankruptcy — real money, but usually not the dominant piece of the story.

### Indirect costs

The much larger, harder-to-see costs that show up *before* any formal filing, often even if the company never actually goes bankrupt:

- **Lost customers.** Buyers of durable goods, warranties, or long-term service contracts avoid a supplier who might not exist in two years. A car buyer thinks twice about a manufacturer rumored to be in trouble.
- **Lost suppliers and tighter terms.** Vendors demand cash upfront instead of net-30 terms, straining the very cash position that's already weak.
- **Talent flight.** Skilled employees — the ones with the most outside options — leave first, taking institutional knowledge with them.
- **Underinvestment.** Distressed firms cut R&D, maintenance capex, and promising-but-risky projects, even ones with positive NPV, because equity holders (who bear the marginal risk in this cash-strapped range) and creditors (who'd rather see any spare cash used to repay them) don't agree on how to spend scarce cash. This specific breakdown has a name — the **underinvestment problem**, one of several "agency costs of debt" that compound distress costs beyond the direct legal bill.
- **Fire-sale asset losses.** A company forced to sell assets quickly, under pressure, to raise cash typically gets far less than the asset's true value in an orderly sale.

Empirical estimates of total distress costs (direct + indirect combined) vary widely by industry — capital-light service or brand-dependent businesses (where customer trust *is* the asset) tend to see costs on the higher end, sometimes estimated at **15%–30%+ of firm value**; asset-heavy manufacturers with resaleable equipment tend to see somewhat lower indirect costs, because their physical assets retain value even in a fire sale. This week uses **30% of unlevered firm value** as Crunch Machine Co.'s stylized total distress-cost estimate — high enough to reflect that a mid-size industrial manufacturer's value depends partly on ongoing customer relationships and skilled staff, not just resaleable machines.

## 2. Reading distress risk before it happens: interest coverage

You can't observe "probability of distress" directly, but you can read a strong leading indicator: **interest coverage**, `EBIT / interest expense` — how many times over a company's operating earnings could pay its interest bill.

```sql
SELECT fiscal_year,
       ROUND(operating_income / interest_expense, 2) AS interest_coverage
FROM income_statement
ORDER BY fiscal_year;
```

| Year | EBIT | Interest | Coverage |
|-----:|-----:|---------:|---------:|
| 2021 | 7,140 | 620 | 11.52x |
| 2022 | 7,200 | 690 | 10.43x |
| 2023 | 6,045 | 810 | 7.46x |
| 2024 | 4,140 | 1,050 | 3.94x |
| 2025 | 2,225 | 1,380 | **1.61x** |

There's no single universal cutoff, but a common rule of thumb bands coverage roughly like this: **above 8x** is comfortable investment-grade territory; **4x–8x** is moderate, still healthy; **2x–4x** is elevated risk, the range where lenders start tightening covenants and rating agencies start downgrading; **below 2x** is high risk of covenant breach or default — a single bad quarter can push pretax income to zero or negative. Crunch Machine Co. crossed from "comfortable" to "high risk" in just five years, without a single year of coverage *increasing*. That trajectory — not any one year's number — is the real signal.

## 3. Modeling the probability of distress

To build a model you can actually solve, you need probability of distress as a function of leverage — not a real, empirically estimated function (that requires a dataset of thousands of companies and their eventual outcomes, a whole research literature on its own), but a **stylized** one that captures the right shape: distress probability starts near zero at low leverage, and rises **convexly** — slowly at first, then faster — as debt grows relative to the firm's underlying value.

This week's model expresses leverage as `D / V_U` (debt relative to unlevered firm value) and uses a simple quadratic:

$$p(D) = \left(\frac{D}{V_U}\right)^2$$

At `D = 0`, `p = 0` — no debt, no distress risk from leverage. As `D` approaches `V_U`, `p` approaches 1 — debt equal to the entire unlevered value of the firm is, unsurprisingly, treated as near-certain distress in this stylized model. The **squaring** is what creates the convex shape: doubling leverage from a low base barely moves `p`, but doubling it from a high base moves `p` a lot. That convexity is the mechanism that eventually makes each *additional* dollar of debt more dangerous than the last one — exactly the shape real distress risk has, even though this exact functional form is a teaching simplification, not an estimated empirical model.

## 4. The tradeoff model

Combine everything into one value equation — the **tradeoff theory of capital structure**:

$$V_L(D) = V_U + \underbrace{T_c \times D}_{\text{tax shield}} - \underbrace{p(D) \times \alpha \times V_U}_{\text{expected distress cost}}$$

Where `α` is the fraction of unlevered value destroyed *if* distress fully occurs (this week: `α = 0.30` for Crunch Machine Co.), and `p(D) × α × V_U` is the **expected** cost — probability times magnitude, the same logic as any expected-value calculation. Substituting the quadratic probability function:

$$V_L(D) = V_U + T_c D - \frac{\alpha}{V_U} D^2$$

This is a downward-opening parabola in `D` (the `D²` term has a negative coefficient) — which means it has a single interior maximum. Debt creates value at the margin while the tax shield's marginal benefit exceeds the marginal expected distress cost; past that point, more debt actively **destroys** value, because the convex distress-cost term is growing faster than the linear tax-shield term.

## 5. Solving for the value-maximizing debt level

Take the derivative of `V_L(D)` with respect to `D` and set it to zero — the standard first-order condition for a maximum:

$$\frac{dV_L}{dD} = T_c - \frac{2\alpha}{V_U} D = 0 \quad\Longrightarrow\quad D^* = \frac{T_c \times V_U}{2\alpha}$$

**Worked example — Crunch Machine Co., FY2025.** Using `EBIT = 2,225`, `Tc = 0.25`, `rU = 0.10`, `α = 0.30`:

```
VU = 2,225 × 0.75 / 0.10 = 16,687.5

D* = (0.25 × 16,687.5) / (2 × 0.30)
   = 4,171.875 / 0.60
   = 6,953.125
```

The model says the value-maximizing debt level for Crunch Machine Co., given FY2025's operating performance, is roughly **$6.95 million**. Crunch Machine Co.'s **actual** FY2025 total debt is **$19.7 million** — short-term debt (5,200) plus long-term debt (14,500).

```
19,700 / 6,953.125 ≈ 2.83
```

**Crunch Machine Co. is carrying roughly 2.8 times the debt this model says maximizes its value.** That's not a subtle overshoot — it's a company that levered up while its operating income was collapsing, compounding two problems into one. This is the number the mini-project asks you to reproduce, verify, and chart across a full grid of `D`, not just at the single optimum point.

## 6. Seeing the parabola before you solve it

Calculus gives you the *exact* peak in one step, but it's worth evaluating `V_L(D)` at a few round numbers first, so the shape isn't just an abstract "downward-opening parabola" — it's a curve you can watch rise, peak, and fall using the same formula from Section 4, `V_L(D) = VU + Tc·D − (α/VU)·D²`, with Crunch Machine Co.'s FY2025 numbers:

| `D` | Tax shield (`Tc·D`) | Expected distress cost | `V_L(D)` |
|----:|---------------------:|------------------------:|----------:|
| 0 | 0.0 | 0.0 | 16,687.5 |
| 2,500 | 625.0 | 112.4 | 17,200.1 |
| 5,000 | 1,250.0 | 449.4 | 17,488.1 |
| 7,000 | 1,750.0 | 880.2 | **17,556.6** |
| 10,000 | 2,500.0 | 1,797.8 | 17,389.7 |
| 15,000 | 3,750.0 | 4,044.9 | 16,392.6 |
| 19,700 (actual) | 4,925.0 | 6,976.9 | 14,635.6 |
| 25,000 | 6,250.0 | 11,236.0 | 11,701.5 |

Read down the `V_L(D)` column: it climbs from `16,687.5` at zero debt, peaks right around `D = 7,000` (matching Section 5's calculus answer of `D* ≈ 6,953`), and then falls — past `10,000` it's already declining, and by the time you reach Crunch Machine Co.'s actual `19,700`, firm value has fallen more than **$2.9 million** below the peak, and more than **$2.05 million** below what the company would be worth with *zero* debt at all. That last comparison is worth sitting with: this stylized model says Crunch Machine Co.'s current debt load is so far past the optimum that the company would be worth *more* unlevered entirely than it is at its actual, heavily-levered capital structure — even though the tax shield by itself is unambiguously a good thing at any debt level below the peak.

## 7. Why this matters more than the sign of `dV_L/dD` alone

It's tempting to treat `D*` as a precise engineering answer. It isn't — `α`, `rU`, and the quadratic shape of `p(D)` are all stylized assumptions, not measured constants. What the model *does* deliver reliably is the **qualitative** result that matters for decision-making: value is **not** monotonically increasing in debt once distress costs are in the picture, there **is** an interior optimum, and a company whose interest coverage is collapsing while its debt keeps climbing is moving in the wrong direction on this curve, regardless of the exact number you pick for `α`. Sensitivity to those assumptions — how much `D*` moves if `α` is 20% instead of 30% — is exactly what Challenge 1 asks you to test.

Look at the `D*` formula itself for a shortcut to that sensitivity intuition: `D* = Tc·VU / (2α)`. `D*` moves in **direct proportion** to `Tc` and `VU` (double either one, and `D*` doubles), and in **inverse proportion** to `α` (double `α`, and `D*` is cut in half). So a company in a low-tax jurisdiction, or one whose industry has unusually severe indirect distress costs (a brand-dependent consumer business, say, versus an asset-heavy manufacturer), should rationally carry *less* debt than this model's Crunch Machine Co. defaults — not because the formula changes, but because the inputs a real analyst would plug into it differ by company and industry.

## 8. Real markets confirm the shape, if not the exact number

Credit-rating agencies (S&P, Moody's, Fitch) effectively price this same tradeoff every time they assign a bond rating — coverage ratios map, roughly, onto rating bands, and rating bands map onto the interest rate a company pays on new debt:

| Interest coverage (rough) | Typical rating band | What it signals |
|---------------------------|----------------------|-------------------|
| Above ~12x | AAA / Aaa | Minimal credit risk |
| ~6x–12x | A to AA | Strong, low risk |
| ~3x–6x | BBB (investment grade) | Adequate, watch trends |
| ~1.5x–3x | BB to B ("junk"/high yield) | Speculative, meaningfully higher borrowing cost |
| Below ~1.5x | CCC and below | Substantial risk of default |

Crunch Machine Co.'s FY2025 coverage of 1.61x sits right at the edge of speculative-grade territory — exactly where a real lender would demand a materially higher interest rate (consistent with the rising embedded `rD` you'd compute from `interest_expense / total_debt` across the five years) to compensate for the distress risk this lecture has been modeling abstractly. The market doesn't need to know the exact `α` this course assumes — it prices the risk directly into the interest rate a company like this actually has to pay, which is its own confirmation that the tradeoff theory's basic shape (more leverage eventually costs more, not less) shows up outside the classroom too.

One caveat worth flagging honestly: the tradeoff theory is not the *only* accepted explanation for how real companies choose capital structure. An influential competing view, the **pecking order theory** (Myers, 1984), argues companies don't target a specific optimal `D*` at all — instead they simply prefer internal cash first, then debt, then equity as a last resort, because of information gaps between managers and outside investors. Real financing behavior often looks more consistent with pecking order than with a precise tradeoff optimum. This course uses the tradeoff model because it's solvable and teaches the tax-shield/distress-cost tension clearly — Week 6 picks the pecking order thread back up when comparing debt and equity financing directly.

## 9. Check yourself

- Name two direct costs and three indirect costs of financial distress. Which category tends to be larger empirically?
- What does an interest coverage ratio of 1.61x tell you that a 3.94x ratio doesn't?
- Why is `p(D) = (D/VU)²` convex, and why does that convexity matter for the shape of `V_L(D)`?
- Write the tradeoff-theory value equation from memory. Which term is linear in `D`, and which is quadratic?
- Derive `D*` from `V_L(D)` by taking the derivative and setting it to zero. What are the two things `D*` is directly proportional to, and the one thing it's inversely proportional to?
- For Crunch Machine Co., is the company under-levered, near-optimal, or over-levered relative to the model's `D*`? By roughly what multiple?
- Using `D* = Tc·VU / (2α)`, if `α` doubled (distress got twice as costly, industry-wide), what would happen to `D*`? What if `Tc` doubled instead?
- In one sentence, how does the pecking order theory's explanation of financing choices differ from the tradeoff theory's?

## Further reading

- **Investopedia — "Trade-Off Theory":** <https://www.investopedia.com/terms/t/tradeofftheory.asp>
- **Investopedia — "Financial Distress":** <https://www.investopedia.com/terms/f/financial_distress.asp>
- **Investopedia — "Interest Coverage Ratio":** <https://www.investopedia.com/terms/i/interestcoverageratio.asp>
- **Investopedia — "Credit Rating":** <https://www.investopedia.com/terms/c/creditrating.asp>
- **Damodaran (NYU Stern) — "The Trade Off on Debt: Costs and Benefits":** <https://pages.stern.nyu.edu/~adamodar/pdfiles/ovhds/capstr.pdf>
- **Myers (1984) — "The Capital Structure Puzzle"** (context for why the tradeoff theory alone doesn't fully explain observed financing behavior — the "pecking order" alternative, previewed for Week 6): <https://www.jstor.org/stable/2327916>
