# Week 7 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

Nothing new this week beyond prior weeks' stack:

- **PostgreSQL 16+** (or **SQLite 3.35+** fallback) — same `crunch_capital` database from prior weeks.
- **Python 3.10+** with `pandas`, `numpy`, `sqlalchemy` — already installed from Week 1.
- **`matplotlib`** (optional, for Challenge 1's football-field chart): `pip install matplotlib`. A formatted-text table is an acceptable substitute if you'd rather not install it.

## Required reading (this week's core)

- **Damodaran (NYU Stern) — "DCF Valuation: The Steps":** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/valuation/dcf.htm>
  *Why: the single best free, public walkthrough of exactly this week's DCF mechanics, from one of the most cited voices in valuation.*
- **Investopedia — "Discounted Cash Flow (DCF)":** <https://www.investopedia.com/terms/d/dcf.asp>
  *Why: a second, shorter explanation of the same core idea — read after Lecture 1 to reinforce, not replace it.*
- **Investopedia — "Enterprise Value (EV)":** <https://www.investopedia.com/terms/e/enterprisevalue.asp>
  *Why: the exact EV formula Lecture 2 builds every multiple on top of.*
- **Investopedia — "Comparable Company Analysis":** <https://www.investopedia.com/terms/c/comparable-company-analysis.asp>
  *Why: the standard walkthrough of trading-comps mechanics, matching Lecture 2's approach.*

## Reference (keep in tabs)

- **Investopedia — "Precedent Transaction Analysis":** <https://www.investopedia.com/terms/p/precedenttransactionanalysis.asp>
  *Why: the standard explanation of deal multiples and the control premium, backing Lecture 2, Section 5.*
- **Damodaran (NYU Stern) — "The Value of Control":** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/valcontrol.htm>
  *Why: a deeper, still-free treatment of exactly what a control premium is compensating for.*
- **Investopedia — "Football Field Chart":** <https://www.investopedia.com/terms/f/football-field-chart.asp>
  *Why: the standard visual format Challenge 1 and the mini-project build.*
- **PostgreSQL — `PERCENTILE_CONT` and ordered-set aggregates:** <https://www.postgresql.org/docs/current/functions-aggregate.html#FUNCTIONS-ORDEREDSET-TABLE>
  *Why: the exact function signature for computing a median in SQL, used throughout Lecture 2 and Exercise 3.*
- **PostgreSQL — Recursive Queries (`WITH RECURSIVE`):** <https://www.postgresql.org/docs/current/queries-with.html#QUERIES-WITH-RECURSIVE>
- **SQLite — Common Table Expressions:** <https://www.sqlite.org/lang_with.html>
  *Why: both needed for Lecture 1's revenue-compounding query; note SQLite's recursive CTE syntax and support level.*
- **pandas — `.median()`, `.quantile()`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.median.html>
  *Why: SQLite has no built-in median — this is your fallback for Exercise 3.*
- **matplotlib — `barh` (horizontal bar charts):** <https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.barh.html>
  *Why: the exact function Challenge 1's football field is built with.*

## Free data, if you do Homework Problem 5 (a real company)

- **SEC EDGAR — full-text search and filings:** <https://www.sec.gov/edgar/search/>
  *Why: the authoritative, free, public source for any US public company's actual income statement, balance sheet, and cash-flow statement.*
- **stockanalysis.com** — free financial summaries, no signup for basic data: <https://stockanalysis.com/>
  *Why: a fast way to pull a real company's trailing revenue, EBITDA, share count, and debt without wading through a full 10-K.*
- **Yahoo Finance** — free quotes, historical prices, and basic financials: <https://finance.yahoo.com/>
  *Why: a quick source for a real company's current share price and shares outstanding, needed for Homework Problem 5.*

## Deeper background (optional this week)

- **Damodaran (NYU Stern) — full valuation dataset downloads (industry multiples, growth rates, betas):** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/data.html>
  *Why: real, free, regularly-updated industry-level multiples — a genuine reference point if you want to sanity-check whether this week's fictional comp set's multiples are realistic for the actual industrial-equipment sector.*
- **Damodaran (NYU Stern) — "Valuation: Art, Science, or Craft?" (free PDF):** <https://pages.stern.nyu.edu/~adamodar/pdfiles/papers/artofvaluation.pdf>
  *Why: the best short defense of why triangulating multiple methods (this week's whole point) beats trusting any single model.*
- **McKinsey — "Valuation: Measuring and Managing the Value of Companies" (publisher page):** <https://www.mckinsey.com/capabilities/strategy-and-corporate-finance/our-insights/valuation-measuring-and-managing-the-value-of-companies>
  *Why: the standard practitioner reference text in the field — not free, but the publisher page previews the framework this week's lectures are built in the tradition of.*

## Glossary

| Term | Definition |
|------|------------|
| **Unlevered free cash flow (UFCF)** | Cash generated by the business before any payments to or from debt or equity holders. |
| **Enterprise value (EV)** | The value of a whole business's operations, capital-structure-neutral: market cap + total debt − cash. |
| **Terminal value (TV)** | The value of all cash flows beyond the explicit forecast period, computed as a single lump sum. |
| **Perpetuity growth (Gordon growth) method** | Terminal value computed by assuming cash flow grows at a constant rate forever: `TV = FCF_(n+1) / (WACC − g)`. |
| **Exit multiple method** | Terminal value computed by applying a market-based multiple to the terminal year's EBITDA (or another metric). |
| **EV/Revenue, EV/EBITDA** | Enterprise-value multiples used to compare companies regardless of capital structure. |
| **Trading comps** | A valuation method using public peers' current market pricing and multiples. |
| **Precedent transactions** | A valuation method using multiples paid in past, similar M&A deals. |
| **Control premium** | The extra amount an acquirer typically pays, per dollar of EBITDA, to own and control 100% of a business versus a passive minority stake. |
| **EV-to-equity bridge** | The set of adjustments (subtract net debt and any senior claims, add back non-operating cash) converting enterprise value to common equity value. |
| **Net debt** | Total debt minus cash and equivalents. |
| **Football field (chart)** | A horizontal-bar chart showing multiple valuation methods' low–high ranges on one shared axis. |
| **Triangulation** | Reconciling multiple independent valuation methods into one defensible range or point estimate. |

---

*Broken link? Open an issue or PR.*
