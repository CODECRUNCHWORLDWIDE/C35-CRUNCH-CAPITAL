# Week 4 — Resources

Curated, free, and official wherever possible. Everything this week runs on the PostgreSQL/SQLite + Python stack from Week 1 — no new tools to install this week, only new *data sources* to know about (the mini-project pulls from a few of these).

## Core references, by lecture

### Lecture 1 — Cost of equity, CAPM

- **Federal Reserve Economic Data (FRED) — 10-Year Treasury Constant Maturity Rate (`DGS10`):** <https://fred.stlouisfed.org/series/DGS10> — the standard free source for a real-world risk-free rate; free CSV download, updated daily, no signup.
- **Aswath Damodaran (NYU Stern) — Equity Risk Premiums, data and methodology:** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/ctryprem.html> — the closest thing the industry has to a standard, freely published, regularly updated ERP reference, both historical and implied.
- **Investopedia — Capital Asset Pricing Model (CAPM):** <https://www.investopedia.com/terms/c/capm.asp>
- **CFA Institute — Cost of Capital (refresher reading, freely accessible):** <https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/cost-capital>

### Lecture 2 — Estimating beta from data

- **pandas — `DataFrame.pct_change`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pct_change.html>
- **pandas — `DataFrame.pivot`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pivot.html>
- **NumPy — `polyfit`:** <https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html>
- **statsmodels — Ordinary Least Squares (OLS), for the full regression report (optional, `pip install statsmodels`):** <https://www.statsmodels.org/stable/regression.html>
- **Aswath Damodaran — "Estimating Risk Parameters and Betas" (levered/unlevered beta, the pure-play/comparables method, in full depth):** <https://pages.stern.nyu.edu/~adamodar/pdfiles/papers/beta.pdf>

### Lecture 3 — Cost of debt and WACC

- **Investopedia — Yield to Maturity (YTM):** <https://www.investopedia.com/terms/y/yieldtomaturity.asp>
- **Investopedia — Weighted Average Cost of Capital (WACC):** <https://www.investopedia.com/terms/w/wacc.asp>
- **Aswath Damodaran — "Estimating the Cost of Capital" (full derivation, market-value weighting explained in depth):** <https://pages.stern.nyu.edu/~adamodar/pdfiles/valn2ed/ch4.pdf>
- **Federal Reserve — Monetary policy overview**, useful background for reasoning about what moves $R_f$: <https://www.federalreserve.gov/monetarypolicy.htm>

## Free data sources for real market data (used in the mini-project)

- **Stooq — free historical price CSV downloads, no signup:** <https://stooq.com/> — e.g. `https://stooq.com/q/d/l/?s=<TICKER>.us&i=m` for monthly US-listed price history. The easiest scriptable source for a real company's and a real index's/ETF's price series.
- **SEC EDGAR full-text search — free access to every US public company's filings:** <https://www.sec.gov/edgar/search/> — use this to find a company's most recent 10-K/10-Q for shares outstanding and its debt footnote.
- **FRED — thousands of free, downloadable economic and rate series beyond just `DGS10`:** <https://fred.stlouisfed.org/>
- **Yahoo Finance — historical data export (manual, browser-based) as a fallback if Stooq doesn't have your ticker:** <https://finance.yahoo.com/> — search a ticker, open "Historical Data," set the date range, and use "Download."

## Tools you already have (no new installs this week)

- **PostgreSQL 16+** or **SQLite 3.35+** — from Week 1.
- **Python 3.10+** with `pandas`, `numpy`, `sqlalchemy`, and a Postgres driver (`psycopg[binary]`) — from Week 1.
- **Optional, for Exercise 2's stretch goal:** `statsmodels`, for a full regression report with standard errors and confidence intervals:
  ```bash
  pip install statsmodels
  ```

## Further, optional reading

- **John Lintner (1965) and William Sharpe (1964) — the original CAPM papers.** Not required, but if you want to see where the formula actually came from, Sharpe's *"Capital Asset Prices: A Theory of Market Equilibrium under Conditions of Risk"* (Journal of Finance) is the canonical citation.
- **Eugene Fama and Kenneth French — "The Cross-Section of Expected Stock Returns" (1992)**, the paper that first argued single-factor CAPM beta alone doesn't fully explain realized returns, and proposed adding size and value factors. Useful context for why CAPM, despite being the industry-standard *starting point*, is not universally accepted as the *final word* on required returns. A concise summary: <https://www.investopedia.com/terms/f/famaandfrenchthreefactormodel.asp>
- **Robert Hamada (1972) — "The Effect of the Firm's Capital Structure on the Systematic Risk of Common Stocks"**, the paper the levered/unlevered beta formula in Lecture 2 is named for.
