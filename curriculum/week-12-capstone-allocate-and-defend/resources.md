# Week 12 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference for the specific question you're chasing during the mini-project.

## Install / confirm first

- **PostgreSQL 16+ and SQLite 3.35+** — the same `crunch_capital` database from every prior week. If either is missing, see [Week 1's resources](../week-01-finance-foundations-and-tooling/resources.md).
- **Python 3.10+ with `pandas`, `numpy`, and `sqlalchemy`/`psycopg2`** — no new libraries this week; everything is built on what you've already installed.
- Optional: **`matplotlib`**, if you want to actually plot the sensitivity grids from Challenge 1 instead of tabulating them — not required, but a picture makes a football-field-style valuation range easier to defend in a live presentation.

## Required reading (this week's core)

- **NYU Stern — Aswath Damodaran, "Valuation" course materials (free, the standard public reference for DCF/comps triangulation):** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/valuation.html>
  *Why: the clearest free treatment anywhere of exactly this week's Lecture 1 problem — reconciling a DCF against a market-implied multiple.*
- **RiskMetrics Technical Document (J.P. Morgan, the founding parametric-VaR paper):** <https://www.msci.com/documents/10199/5915b101-4206-4ba0-aee2-3449d5c7e95a>
  *Why: the original, still-standard reference for the parametric VaR formula you used in Lecture 2.*
- **Basel Committee on Banking Supervision — "Messages from the academic literature on risk measurement for the trading book":** <https://www.bis.org/publ/bcbs_wp19.htm>
  *Why: a regulator-level, honest treatment of VaR's known limitations — required reading precisely because it argues with the method you just learned.*
- **OECD — "G20/OECD Principles of Corporate Governance":** <https://www.oecd.org/corporate/principles-corporate-governance/>
  *Why: the internationally-referenced governance framework behind Lecture 3's checklist.*

## Reference (keep in tabs)

- **PostgreSQL — Ordered-set aggregates (`PERCENTILE_CONT`, `PERCENTILE_DISC`):** <https://www.postgresql.org/docs/current/functions-aggregate.html#FUNCTIONS-ORDERED-SET-TABLE>
  *Why: the exact function you used for historical VaR's empirical percentiles.*
- **PostgreSQL — Window functions (`LAG`, for daily-return calculations):** <https://www.postgresql.org/docs/current/tutorial-window.html>
- **SEC — EDGAR full-text search:** <https://www.sec.gov/cgi-bin/browse-edgar>
  *Why: where Homework Problem 4 sends you to read a real proxy statement or 8-K.*
- **NumPy — `numpy.random.Generator` (the `default_rng` API used in this week's price-generation script):** <https://numpy.org/doc/stable/reference/random/generator.html>
- **pandas — `DataFrame.pct_change` and `.quantile`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pct_change.html>

## On the specific techniques this week

- **Investopedia — "Value at Risk (VaR)":** <https://www.investopedia.com/terms/v/var.asp>
  *Why: a fast, free refresher if the Lecture 2 mechanics didn't fully stick — good second explanation, different voice.*
- **CFA Institute — "Scenario Analysis, Stress Testing" (refresher-reading style overview):** <https://www.cfainstitute.org/en/membership/professional-development/refresher-readings>
  *Why: the professional-exam-level framing of exactly the scenario-vs-stress distinction from Lecture 2, Section 7.*
- **Cadbury Report (1992), "Financial Aspects of Corporate Governance"** — the founding modern governance-code document, still short and readable:
  <https://ecgi.global/code/cadbury-report-1992>
  *Why: understand where the modern board-independence and audit-cadence norms in Lecture 3's checklist actually came from.*

## Deeper background (optional this week)

- **Markowitz, "Portfolio Selection" (1952)** — the original mean-variance paper behind the portfolio-VaR math you used this week (and behind all of Week 8): <https://www.jstor.org/stable/2975974> (freely readable via most university libraries; also widely mirrored)
  *Why: the two-asset variance formula in Lecture 2, Section 4 is a direct, simplified application of this paper's core result.*
- **Kahneman, Lovallo, & Sibony, "Before You Make That Big Decision" (HBR, 2011)** — on pre-mortem analysis, the technique behind Challenge 1: <https://hbr.org/2011/06/before-you-make-that-big-decision>
  *Why: the original popularization of the "imagine it already failed, work backward" technique this week's red-team challenge is built on.*

## Glossary

| Term | Definition |
|------|------------|
| **Enterprise value (EV)** | The value of a business's operations, independent of how it's financed — before subtracting debt or adding cash. |
| **Terminal value (TV)** | The value assigned to all cash flows beyond the explicit forecast period, computed by perpetuity growth or an exit multiple. |
| **EV-to-equity bridge** | `Equity value = EV − total debt + cash`; the mechanics of getting from a whole-business value to a per-share price. |
| **Value at Risk (VaR)** | The loss not expected to be exceeded, at a stated confidence level, over a stated horizon — a statement about a typical bad outcome, not the worst possible one. |
| **Parametric VaR** | VaR computed from a formula assuming a return distribution (usually normal) and a volatility input. |
| **Historical VaR** | VaR read directly off the empirical percentile of actual past returns — no distributional assumption. |
| **Scenario analysis** | Computing an outcome (here, a valuation) under each of several named, probability-weighted futures. |
| **Stress test** | A deliberately extreme, often-combined shock with no probability attached, used to test survivability rather than likelihood. |
| **Fat tails** | The tendency of real return distributions to have more extreme outcomes than a normal distribution predicts. |
| **Governance risk** | The risk that the people controlling a company's cash flows are not incentivized, or not adequately overseen, to deliver those cash flows to shareholders as modeled. |
| **Reproducibility standard** | The practice of attaching a specific, re-runnable query or script to every quantitative claim in an analysis or memo. |
| **Red team / pre-mortem** | Deliberately attacking your own analysis's weakest assumptions, quantified, before someone else does. |

---

*Broken link? Open an issue or PR.*
