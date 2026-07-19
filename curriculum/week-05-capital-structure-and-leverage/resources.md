# Week 5 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

Nothing new this week beyond what Weeks 1–4 already set up. Confirm you have:

- **PostgreSQL 16+** or **SQLite 3.35+** — either, with `crunch_capital` loaded (see the [week README](./README.md)).
- **Python 3.10+** with `pandas`, `numpy`, and **`matplotlib`** — the only new package this week, needed for the distress-cost curve chart:

```bash
pip install matplotlib
```

## Required reading (this week's core)

- **Investopedia — "Modigliani-Miller Theorem":** <https://www.investopedia.com/terms/m/modigliani-miller.asp>
  *Why: the clearest short, free walkthrough of Proposition I and II, with and without taxes — read after Lecture 1 to reinforce, not replace it.*
- **Investopedia — "Trade-Off Theory":** <https://www.investopedia.com/terms/t/tradeofftheory.asp>
  *Why: a second, independent explanation of the tax-shield-versus-distress-cost framework from Lecture 2.*
- **Investopedia — "Financial Distress":** <https://www.investopedia.com/terms/f/financial_distress.asp>
  *Why: real-world texture on what distress looks like operationally, beyond the stylized model this week builds.*
- **Investopedia — "Degree of Financial Leverage (DFL)":** <https://www.investopedia.com/terms/d/degreeoffinancialleverage.asp>
  *Why: a second worked example of the DFL formula from Lecture 3, with different numbers than Crunch Machine Co.'s.*
- **Investopedia — "Interest Coverage Ratio":** <https://www.investopedia.com/terms/i/interestcoverageratio.asp>
  *Why: the exact ratio Lecture 2 uses as this week's leading distress indicator — includes industry-typical benchmark ranges.*

## Reference (keep in tabs)

- **PostgreSQL — Window Functions:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: the `SUM(...) OVER (ORDER BY ...)` pattern used for the cumulative tax-shield query in Exercise 1 and the homework.*
- **matplotlib — Pyplot tutorial:** <https://matplotlib.org/stable/tutorials/pyplot.html>
  *Why: everything needed for the value-vs-leverage chart in Exercise 3, Challenge 1, and the mini-project — line plots, labeled points, legends.*
- **pandas — `idxmax` / `idxmin`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.idxmax.html>
  *Why: the exact method used to find the grid-search optimum in Exercise 3 and Challenge 1.*

## Primary sources (the actual papers, for the curious)

- **Modigliani & Miller (1958) — "The Cost of Capital, Corporation Finance and the Theory of Investment":** <https://www.jstor.org/stable/1809766>
  *Why: the original Proposition I/II paper. Dense, but this is where the whole week's foundation comes from.*
- **Modigliani & Miller (1963) — "Corporate Income Taxes and the Cost of Capital: A Correction":** <https://www.jstor.org/stable/1809167>
  *Why: the follow-up paper that adds the `Tc × D` tax-shield result — the correction to their own 1958 no-tax model.*
- **Myers (1984) — "The Capital Structure Puzzle":** <https://www.jstor.org/stable/2327916>
  *Why: a foundational critique of the tradeoff theory's limits and an introduction to the competing "pecking order" theory — previewed for Week 6.*

## Deeper background (optional this week)

- **Damodaran (NYU Stern) — "Capital Structure: The Choices and the Trade Off":** <https://pages.stern.nyu.edu/~adamodar/pdfiles/ovhds/capstr.pdf>
  *Why: one of the most respected free, public treatments of exactly this week's material, with real-company data and a more rigorous treatment of estimating distress-cost fractions than this week's stylized `α` assumption.*
- **NYU Stern — Damodaran's data page (industry-average debt ratios):** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/data.html>
  *Why: real, industry-by-industry capital structure benchmarks — useful for Homework Problem 5's real-company extension, to sanity-check whether your chosen company's leverage looks typical for its sector.*

## Glossary

| Term | Definition |
|------|------------|
| **Capital structure** | The mix of debt and equity a company uses to finance its assets. |
| **MM Proposition I** | Without taxes: `VL = VU`, capital structure doesn't affect firm value. With taxes: `VL = VU + Tc·D`. |
| **MM Proposition II** | The formula for how the cost of equity `rE` rises with leverage, with or without a tax adjustment. |
| **Interest tax shield** | The reduction in taxes a company enjoys because interest expense is tax-deductible. |
| **Financial distress** | The state of struggling to meet debt obligations — not necessarily bankruptcy itself, but the costly zone that often precedes or accompanies it. |
| **Direct distress costs** | Out-of-pocket bankruptcy/restructuring costs — legal, accounting, court fees. |
| **Indirect distress costs** | Harder-to-see costs: lost customers, lost suppliers, talent flight, underinvestment, fire-sale asset losses. |
| **Tradeoff theory** | The theory that optimal capital structure balances the tax shield's benefit against expected distress costs, producing an interior value-maximizing debt level. |
| **Interest coverage ratio** | `EBIT / interest expense` — how many times over a company's operating earnings could pay its interest bill. |
| **Degree of Financial Leverage (DFL)** | `EBIT / pretax income` — the multiplier by which a percentage change in EBIT translates into a percentage change in net income. |
| **Covenant** | A condition in a loan agreement (e.g., a minimum interest coverage ratio) that, if breached, can trigger default even if payments are current. |

---

*Broken link? Open an issue or PR.*
