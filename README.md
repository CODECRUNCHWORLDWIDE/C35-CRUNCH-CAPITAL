# C35 · Crunch Capital

> A free, open-source 12-week course on corporate finance, investing, and strategic decision-making powered by technology — capital budgeting, valuation, portfolios, M&A, and cap tables, analyzed with SQL, Python, and AI.

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![PostgreSQL · Python](https://img.shields.io/badge/stack-PostgreSQL_·_Python-2463EB.svg)](#stack)
[![Built in the open](https://img.shields.io/badge/built-in%20the%20open-2463EB.svg)](https://github.com/CODECRUNCHWORLDWIDE)

C35 is a Tier-2 applied course: it takes you from "what is a cash flow?" to running a full deal — a DCF valuation, a comparables model, a portfolio backtest, and a cap table — all quantified in **SQL and Python**, never a spreadsheet. It pairs naturally with [C33 Crunch SQL](../C33-CRUNCH-SQL/) (the query foundation it assumes), [C27 Crunch Data](../C27-CRUNCH-DATA/), and [C5 Crunch AI & Data Science](../C5-CRUNCH-AI-DATA-SCIENCE/). This course makes you the person who can allocate capital like an operator and defend the number.

---

## Pathway summary

- **Full-time:** 12 weeks · ~28 hrs/week · ~336 hours
- **Working-professional pace:** 6 months · ~14 hrs/week
- **Evening pace:** 12 months · ~7 hrs/week

See [`SYLLABUS.md`](SYLLABUS.md).

---

## What you will be able to do at the end of 12 weeks

- **Read the money:** parse the three financial statements, tie them together, and compute the ratios that actually predict trouble — all from a query, not a spreadsheet.
- **Value a project:** run NPV, IRR, MIRR, and payback on real cash flows, and know exactly when IRR lies to you.
- **Price capital:** build a cost of equity (CAPM), a cost of debt, and a full WACC, and understand what moves each one.
- **Choose a capital structure:** model leverage, the tax shield, financial distress, and find the debt level that maximizes value.
- **Raise money:** compare equity vs. debt financing, model dilution, coupons, covenants, and the true cost of each round.
- **Value a company:** build a DCF from the ground up and a trading-comparables model, and triangulate a defensible range.
- **Build a portfolio:** apply mean-variance optimization, the efficient frontier, Sharpe ratios, and honest asset allocation.
- **Backtest a strategy:** turn an investment thesis into a rules-based backtest with realistic costs — and measure whether it survives.
- **Analyze a deal:** model an M&A transaction — accretion/dilution, synergies, and the financing mix.
- **Model a startup:** build a cap table through priced rounds, SAFEs, option pools, and an exit waterfall.
- **Govern the risk:** stress-test, size positions, and reason about governance and the incentives behind the numbers — the capstone.

---

## Curriculum (12 weeks)

| Week | Topic | You leave able to… |
|------|-------|--------------------|
| 1 | [Finance foundations & data tooling](curriculum/week-01-finance-foundations-and-tooling/) | Model time value of money in SQL + Python, not a spreadsheet. |
| 2 | [Financial statements & ratio analysis](curriculum/week-02-financial-statements-and-ratios/) | Tie the three statements together and compute health ratios. |
| 3 | [Capital budgeting — NPV & IRR](curriculum/week-03-capital-budgeting-npv-irr/) | Rank projects with NPV, IRR, MIRR, and payback. |
| 4 | [Cost of capital & WACC](curriculum/week-04-cost-of-capital-and-wacc/) | Build cost of equity, cost of debt, and a full WACC. |
| 5 | [Capital structure & leverage](curriculum/week-05-capital-structure-and-leverage/) | Model the tax shield, distress, and an optimal debt level. |
| 6 | [Equity & debt financing](curriculum/week-06-equity-and-debt-financing/) | Compare raising equity vs. debt and model dilution + coupons. |
| 7 | [Valuation & comparables](curriculum/week-07-valuation-and-comparables/) | Build a DCF and a trading-comps model to a value range. |
| 8 | [Portfolio theory & allocation](curriculum/week-08-portfolio-theory-and-allocation/) | Optimize a mean-variance portfolio on the efficient frontier. |
| 9 | [Investment analytics & backtesting](curriculum/week-09-investment-analytics-and-backtesting/) | Backtest a rules-based strategy with realistic costs. |
| 10 | [M&A & deal analysis](curriculum/week-10-mergers-acquisitions-and-deals/) | Model accretion/dilution, synergies, and a financing mix. |
| 11 | [Startup finance & cap tables](curriculum/week-11-startup-finance-and-cap-tables/) | Build a cap table through rounds and an exit waterfall. |
| 12 | [Capstone — allocate & defend](curriculum/week-12-capstone-allocate-and-defend/) | Run a full deal end-to-end and defend the number. |

---

## How to navigate a week

Every week folder holds the same structure:

- **`README.md`** — the week overview + how the pieces fit + the week's goal.
- **`lecture-notes/`** — 3 lectures (~2 hrs each), the conceptual core.
- **`exercises/`** — 3 short, guided reps against a real dataset.
- **`challenges/`** — 2 open-ended problems with no single right answer.
- **`mini-project/`** — one build that ties the week together.
- **`homework.md`**, **`quiz.md`**, **`resources.md`** — practice, self-check, and further reading.

---

## Stack

PostgreSQL (16+) as the primary data store and Python (pandas, NumPy) for modeling and analytics, with SQLite for zero-setup practice. **We never use Excel or any spreadsheet as a database or model store** — every cash flow, price series, cap table, and deal model lives in SQL and Python so it is queryable, versionable, and reproducible. (Spreadsheets are taught only in [C41 Crunch Excel](../C41-CRUNCH-EXCEL/).) You'll also use an AI assistant for reasoning checks and a seed dataset of statements, prices, and deals shipped with the course. Everything is free and runs on macOS, Linux, and Windows.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · [Browse all courses](https://codecrunchglobal.vercel.app/courses)*
