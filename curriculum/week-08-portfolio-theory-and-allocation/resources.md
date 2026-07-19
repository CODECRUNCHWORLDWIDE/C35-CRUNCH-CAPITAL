# Week 8 — Resources

Curated, free, official-first. You do not need to read everything here — this is a reference to come back to, not a reading assignment.

## Install / environment (reused from Week 1)

If you haven't already set up your workspace, do this first — everything else in this week assumes it's running.

- **PostgreSQL 16+** — [postgresql.org/download](https://www.postgresql.org/download/). macOS: [Postgres.app](https://postgresapp.com/) is the fastest path. Linux: `sudo apt install postgresql` (Debian/Ubuntu) or `sudo dnf install postgresql-server` (Fedora/RHEL). Windows: the EDB installer.
- **SQLite 3.35+** (zero-setup fallback) — ships with macOS and most Linux distributions. Verify with `sqlite3 --version`.
- **Python 3.10+ with pandas, numpy, and scipy** —
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate     # Windows: .venv\Scripts\activate
  pip install pandas numpy scipy sqlalchemy psycopg[binary]
  ```
  `scipy` is new this week — you'll need `scipy.optimize.minimize` for Challenge 1's constrained optimization. It wasn't required in earlier weeks but is a standard, free, well-maintained scientific-Python package.
- **matplotlib** (optional but recommended for Exercise 2's frontier plot) — `pip install matplotlib`.

## Foundational papers (the actual sources this week's theory comes from)

- **Markowitz, H. (1952). "Portfolio Selection."** *Journal of Finance*, 7(1), 77–91. The original mean-variance optimization paper — short, readable, and the direct ancestor of everything in Lectures 1 and 2. <https://www.jstor.org/stable/2975974>
- **Sharpe, W. F. (1964). "Capital Asset Prices: A Theory of Market Equilibrium under Conditions of Risk."** *Journal of Finance*, 19(3), 425–442. The paper that introduced CAPM. <https://www.jstor.org/stable/2977928>
- **Sharpe, W. F. (1966/1994). "The Sharpe Ratio."** The original and later-refined definitions of the ratio you used throughout Lectures 2–3: <https://web.stanford.edu/~wfsharpe/art/sr/sr.htm>
- **Fama, E. F., & French, K. R. (1993). "Common Risk Factors in the Returns on Stocks and Bonds."** *Journal of Financial Economics*, 33(1), 3–56. The paper that launched multi-factor investing, referenced in Lecture 3's factor-tilt discussion. <https://doi.org/10.1016/0304-405X(93)90023-5>

## Portfolio theory, taught deeper

- **CFA Institute — Portfolio Risk and Return, Part I & II (Level I curriculum overview):** <https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/portfolio-risk-and-return-part-i> — the industry-standard treatment of exactly this week's material, at the next level of formality.
- **Investopedia — Modern Portfolio Theory (MPT):** <https://www.investopedia.com/terms/m/modernportfoliotheory.asp>
- **Investopedia — Efficient Frontier:** <https://www.investopedia.com/terms/e/efficientfrontier.asp>
- **Investopedia — Capital Market Line:** <https://www.investopedia.com/terms/c/cml.asp>
- **Aswath Damodaran (NYU Stern) — free corporate finance and investments teaching materials:** <https://pages.stern.nyu.edu/~adamodar/> — his investment-management course notes cover portfolio theory in more depth than this week, useful if you want to go further now.

## Real market data (for the mini-project)

- **Stooq** — <https://stooq.com/> — free historical daily/monthly CSV downloads with no signup, e.g. `https://stooq.com/q/d/l/?s=<TICKER>.us&i=m` for monthly US-listed data.
- **FRED — `DGS10` (10-Year Treasury Constant Maturity Rate)** — <https://fred.stlouisfed.org/series/DGS10> — free, no-signup source for a risk-free rate proxy, same as Week 4.
- **Yahoo Finance historical data** — manual CSV export from a browser, a reasonable fallback if Stooq lacks a ticker.

## Python/numerical references used this week

- **numpy — Linear algebra (`linalg.inv`, matrix/vector products):** <https://numpy.org/doc/stable/reference/routines.linalg.html>
- **numpy — `polyfit`:** <https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html>
- **pandas — `DataFrame.cov()` / `.corr()` / `.rolling()`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cov.html>
- **scipy — `optimize.minimize` (used in Challenge 1 for constrained optimization):** <https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.minimize.html>
- **matplotlib — `pyplot` basics (for Exercise 2's frontier plot):** <https://matplotlib.org/stable/tutorials/pyplot.html>

## Where this week leads

Week 9 of this course turns "what's the best static allocation" (this week) into "would a *rule* for changing that allocation over time actually have worked, net of trading costs" — rules-based backtesting. Keep your `asset_prices` table and your covariance-matrix / optimizer code; you'll extend both directly.
