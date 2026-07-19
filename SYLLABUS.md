# C35 · Crunch Capital — Syllabus

**Format:** 12 weeks · self-paced · open-source (GPL-3.0). Each week = 3 lectures, 3 exercises, 2 challenges, a mini-project, homework, and a quiz.

**Prerequisites:** Comfort with basic algebra and running commands in a terminal. Working SQL (`SELECT`, joins, `GROUP BY`) and beginner Python are assumed — take [C33 Crunch SQL](../C33-CRUNCH-SQL/) first if you need them. No prior finance or accounting knowledge required — this course goes from zero to expert.

**Data tooling rule:** all financial data — statements, cash flows, price series, cap tables, deal models — is stored and modeled in **SQL (PostgreSQL 16, SQLite fallback) and Python (pandas)**, never in Excel or a spreadsheet. Where a finance workflow traditionally lives in a spreadsheet, we rebuild it in SQL + Python so it is queryable, reproducible, and version-controlled.

**Assessment (honor-based):** tick each lesson complete in the reader; finish the weekly mini-project and quiz. A certificate is issued on 100% completion to One-Stop members.

| Week | Theme | Mini-project |
|------|-------|--------------|
| 1 | Finance foundations + time value of money in SQL/Python | Build a discounting engine over a cash-flow table |
| 2 | Financial statements + ratio analysis + statement linkage | Query a ratio dashboard across 3 linked statements |
| 3 | Capital budgeting — NPV, IRR, MIRR, payback | Rank a project slate by NPV and defend the cutoff |
| 4 | Cost of capital — CAPM, cost of debt, WACC | Compute a company's WACC from market data |
| 5 | Capital structure — leverage, tax shield, distress | Find the value-maximizing debt level under a model |
| 6 | Equity vs. debt financing — dilution, coupons, covenants | Model a financing round two ways and compare cost |
| 7 | Valuation — DCF + trading comparables | Value a company by DCF and comps to a range |
| 8 | Portfolio theory — mean-variance, frontier, Sharpe | Optimize an allocation on the efficient frontier |
| 9 | Investment analytics + rules-based backtesting | Backtest a strategy with costs and report the metrics |
| 10 | M&A + deal analysis — accretion/dilution, synergies | Model an acquisition and its financing mix |
| 11 | Startup finance + cap tables — rounds, SAFEs, waterfall | Build a cap table through 3 rounds to an exit |
| 12 | Capstone — full deal, risk, and governance | Run an end-to-end deal and defend the number |

**Outcome:** value a company, build and backtest a portfolio, model a deal and a cap table, and defend every number — all quantified in SQL and Python, like an operator allocating capital.
