# Mini-Project — Optimize a Multi-Asset Allocation on the Efficient Frontier

> Build your own investable universe of real, publicly traded assets (funds, ETFs, or individual securities across at least three genuinely different asset classes), pull real historical price data, and run this week's full pipeline end to end: covariance matrix, efficient frontier, global-minimum-variance portfolio, and maximum-Sharpe portfolio. Report the max-Sharpe weights along with their full risk profile, and defend the allocation the way an advisor would to a client.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

Every calculation this week ran against Crunch Capital Advisors' fictional-but-realistic six-fund universe. This project asks you to do the identical analysis on a universe of assets that actually exist, using free, real data — the same jump from "seeded example" to "real analyst work" that Week 4's mini-project made for WACC.

---

## Choosing your universe

Pick **at least 4, at most 8** real, investable assets (ETFs are the easiest path — they trade daily, have long price histories, and represent whole asset classes in one ticker) spanning **at least 3 distinct asset classes**. A good starting shape, adaptable to your own interests:

- A broad US equity ETF (e.g., a total-market or S&P 500 fund).
- A broad international or emerging-market equity ETF.
- A core/aggregate bond ETF.
- One "diversifier" — gold, real estate (REIT), commodities, or a genuinely different asset class from the first three.
- Optionally, 1–4 more assets of your choosing (a sector fund, a specific stock you're curious about, a second bond fund at a different maturity/credit-quality point, etc.).

Write down your chosen universe and **why you picked it** (2–3 sentences — what gap in the "boring but effective" diversification story are you trying to fill?) in `report.md` before pulling data.

## Gathering the data (all free, no API key required)

You need at least **3 years of monthly price history** (5 years is better, and is what this week's lectures used) for every asset in your universe, all over the *identical* date range so your return series line up.

- **Stooq** (<https://stooq.com/>) — free historical CSV downloads with no signup, e.g. `https://stooq.com/q/d/l/?s=<TICKER>.us&i=m` for monthly US-listed data. This is the easiest scriptable source, and the same one Week 4's mini-project pointed you to.
- Yahoo Finance's historical data page (manual CSV export) works as a fallback if Stooq doesn't have a ticker.
- **Record your source and pull date** for every ticker in `report.md` — an allocation built on data nobody can trace back to a source isn't usable in real work, exactly as Week 4 insisted for WACC inputs.

For the risk-free rate, reuse **FRED's `DGS10`** series (10-Year Treasury) as in Week 4: <https://fred.stlouisfed.org/series/DGS10>.

## Deliverable

A directory in your portfolio `c35-week-08/mini-project/` containing:

1. **`load_data.py`** — loads your gathered price data into a new table in your `crunch_capital` database (e.g., `real_asset_prices`, with the same `(price_id, ticker, price_date, adj_close)` shape as `asset_prices` — name it clearly distinct from the seed data so you don't overwrite it).
2. **`optimize_allocation.py`** — the full pipeline: pulls your loaded prices back out with pandas, computes returns and the covariance matrix, traces the efficient frontier, and solves for the GMV and maximum-Sharpe portfolios. Should run start to finish and print every intermediate number (mean returns, volatilities, correlation matrix, frontier points, final weights).
3. **`report.md`** — see the required sections below.

## Required sections in `report.md`

1. **Your universe and why you picked it** (2–3 sentences, per above).
2. **Data sources** — one line per ticker: where you got the price history (URL), the date range, and your pull date. Plus the risk-free rate source, value, and date.
3. **Your covariance and correlation matrix** — printed in full. Identify the single highest and single lowest pairwise correlation in your universe, by name.
4. **Your efficient frontier** — at least 10 traced points (target return → volatility), and a note on where the GMV portfolio sits on it.
5. **Your global-minimum-variance portfolio** — weights, expected return, volatility.
6. **Your maximum-Sharpe portfolio** — weights, expected return, volatility, Sharpe ratio.
7. **A comparison to equal-weight** — expected return, volatility, and Sharpe ratio of a naive 1/n allocation across your same universe, set directly against your max-Sharpe portfolio's numbers.
8. **A gut check and a recommendation**: does the max-Sharpe allocation feel reasonable given what you know about these assets? Are any weights surprisingly large, small, negative, or absent? If your optimizer wants a short position, say which asset and by how much, and whether you'd actually recommend a client take it or you'd instead recommend the no-short-constrained version from Challenge 1's method. End with one paragraph: if a real client handed you $100,000 today, what would you actually tell them to buy, and why — referencing your own numbers, not generic advice.

## Rules

- **Real data only.** Every number in `report.md` must trace to a real, cited source — no invented figures.
- **Same pipeline shape as this week.** SQL for storage, pandas/numpy for the modeling — no spreadsheets, per the course's data tooling rule, even though you're pulling from external CSV sources.
- **Disclose every simplification.** If you used a shorter price history than 5 years, used monthly rather than weekly/daily data, or treated an ETF's price history as fully representative despite a fund that recently changed strategy or benchmark — say so plainly.
- **Sanity-check your covariance matrix.** If two assets you expect to be highly correlated (say, two different US large-cap equity ETFs) come back with a correlation below 0.5, or two assets from wildly different asset classes come back above 0.9, something is likely wrong with your date alignment (mismatched trading calendars/holidays between exchanges is the most common cause) — debug before moving on.

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Data sourcing | 15% | Every price series and the risk-free rate trace to a specific, real, cited source with a pull date |
| Pipeline correctness | 30% | Returns, covariance matrix, frontier, GMV, and max-Sharpe solutions are all mechanically correct |
| SQL + pandas/numpy discipline | 15% | Data genuinely lives in a SQL table and is queried, not read from a CSV straight into a script and never touching the database |
| Disclosed assumptions | 15% | Every simplification (shorter window, monthly vs. daily, etc.) is explicitly stated |
| Interpretation and recommendation | 25% | The gut-check and closing recommendation show real judgment — specific, numbers-referenced, and actionable — not just a table left to speak for itself |

## Stretch goals

- Add a **factor tilt** (per Lecture 3) to your max-Sharpe allocation — pick a defensible reason (e.g., you want lower market sensitivity ahead of an expected downturn, or you're deliberately overweighting an inflation hedge) — and report the tilted portfolio's expected return, volatility, and beta to a market benchmark, alongside the untilted max-Sharpe numbers for comparison.
- Re-run Challenge 1's constrained (no-short, position-capped) optimization on your *own* universe and report both the constrained and unconstrained max-Sharpe portfolios side by side.
- Re-run Challenge 2's crisis stress test on your own universe's covariance matrix and report how much your max-Sharpe portfolio's volatility would increase in a correlation-spike scenario.
- Pull in a real market benchmark (e.g., a total-market index ETF) and compute each of your universe's assets' CAPM betas against it, exactly as Lecture 3 did for the seed data.

## Why this matters

Every other calculation this week ran against Crunch Capital Advisors' clean, fictional, fully-seeded universe. Real portfolio construction starts with a much messier first step — deciding *which* assets even belong in the universe you're optimizing over, then actually finding and cleaning their price histories, before a single covariance matrix gets built. An analyst who can only run mean-variance optimization when someone hands them a tidy six-column price table isn't yet useful in a real seat; one who can build that table themselves, from real tickers, and still produce a defensible allocation at the end, is exactly the skill this project is built to force.

When done: push, then take the [quiz](../quiz.md).
