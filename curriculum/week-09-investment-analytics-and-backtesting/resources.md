# Week 9 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install / confirm first

- **PostgreSQL 16+** or **SQLite 3.35+** — same engines as every prior week: <https://www.postgresql.org/download/> · <https://www.sqlite.org/download.html>
- **Python 3.10+ with `pandas` and `numpy`** — if not already installed: `pip install pandas numpy sqlalchemy psycopg2-binary`
- No new packages required beyond that. This week's backtest engine is code you write, not a library you import — see the "optional frameworks" section below for what exists once you've built one by hand and want to compare.

## Required reading (this week's core)

- **Jegadeesh & Titman (1993), "Returns to Buying Winners and Selling Losers: Implications for Stock Market Efficiency"** — the original, foundational momentum paper:
  <https://www.jstor.org/stable/2328882>
  *Why: everything Lecture 1 says about 12-1 momentum traces back to this paper's specific methodology choices.*
- **Bailey, Borwein, López de Prado & Zhu (2014), "Pseudo-Mathematics and Financial Charlatanism: The Effects of Backtest Overfitting on Out-of-Sample Performance"**:
  <https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2308659>
  *Why: the single best explanation, with math, of exactly why Lecture 3 exists. Read at least the abstract and introduction even if the full derivation is heavy.*
- **pandas — Time series / date functionality:**
  <https://pandas.pydata.org/docs/user_guide/timeseries.html>
  *Why: `pct_change`, `shift`, `resample`, `.diff()` are the four pandas operations this week's entire engine is built from — know them cold.*
- **PostgreSQL — Window Functions Tutorial:**
  <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `RANK()`, `LAG`/`LEAD`, and self-joins on a row-number index are how every signal in this week's SQL is built.*

## Reference (keep in tabs)

- **PostgreSQL — Window Function reference:** <https://www.postgresql.org/docs/current/functions-window.html>
- **numpy — Random Generator (`default_rng`) reference:** <https://numpy.org/doc/stable/reference/random/generator.html>
  *Why: understand exactly what `rng.standard_normal()` and the seeded generator in the README's price simulator are doing.*
- **pandas — `pivot`, `melt`, reshaping guide:** <https://pandas.pydata.org/docs/user_guide/reshaping.html>
  *Why: the long-to-wide price-table conversion (`prices_long.pivot(...)`) trips people up the first time.*
- **Investopedia — "Sharpe Ratio":** <https://www.investopedia.com/terms/s/sharperatio.asp>
- **Investopedia — "Maximum Drawdown (MDD)":** <https://www.investopedia.com/terms/m/maximum-drawdown-mdd.asp>
- **Investopedia — "Slippage":** <https://www.investopedia.com/terms/s/slippage.asp>

## Deeper background on honesty in backtesting

- **López de Prado, "Advances in Financial Machine Learning" (2018), Ch. 11 — "The Dangers of Backtesting"** — publisher page (not free, but the single deepest treatment available, and worth owning if you continue in quant work): <https://www.wiley.com/en-us/Advances+in+Financial+Machine+Learning-p-9781119482086>
- **Harvey & Liu (2015), "Backtesting"** (Journal of Portfolio Management, free SSRN preprint) — on multiple-testing corrections when many strategies are tried against the same data: <https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2345489>
- **scikit-learn — "Cross-validation: evaluating estimator performance"** — the general statistical-learning framing of train/test and walk-forward splits, applicable well beyond finance: <https://scikit-learn.org/stable/modules/cross_validation.html>
- **AQR Capital Management, "A Century of Evidence on Trend-Following Investing"** (free, publicly hosted white paper) — a real, long-sample, out-of-sample-honest momentum study worth reading after this week's small-sample exercise, to see what a rigorous version looks like at scale: search "AQR Century of Evidence Trend-Following" or check AQR's public insights library at <https://www.aqr.com/Insights>

## Optional backtesting frameworks (look at these *after* you've built one by hand)

These are legitimate, widely used tools — but this course deliberately has you build the engine yourself first, because using a framework before you understand what it's doing under the hood is exactly how look-ahead bugs slip through unnoticed (a library will happily run a leaky signal without complaint).

- **Backtrader** (Python, open-source): <https://www.backtrader.com/>
- **Zipline / zipline-reloaded** (Python, open-source, originally built by Quantopian): <https://github.com/stefan-jansen/zipline-reloaded>
- **vectorbt** (Python, open-source, vectorized/fast): <https://vectorbt.dev/>

## Glossary

| Term | Definition |
|------|------------|
| **Signal** | A precise, computable rule that assigns a position (long/flat/short) to each security on each date, with no human judgment left at trade time. |
| **Rebalancing** | Updating a portfolio's holdings on a fixed schedule (e.g., monthly), rather than continuously. |
| **Transaction cost** | The direct cost of trading — commissions, fees, and the bid-ask spread crossed on every trade. |
| **Slippage** | The gap between the price a decision was based on and the price actually achieved at execution. |
| **Basis point (bp/bps)** | 1/100th of one percent (0.01%); the standard unit for quoting small costs and yields in finance. |
| **CAGR** | Compound Annual Growth Rate — the constant annual growth rate that would produce the same total return over the period actually observed. |
| **Sharpe ratio** | Annualized excess return (over the risk-free rate) divided by annualized volatility — return per unit of risk taken. |
| **Maximum drawdown** | The largest peak-to-trough percentage decline in an equity curve over the backtest period. |
| **Hit rate** | The fraction of trading periods with a positive return. |
| **Look-ahead bias** | A backtest error where a strategy uses information that would not actually have been available on the date it's used. |
| **Survivorship bias** | A backtest error where the tested universe is selected with hindsight, excluding entities that failed or disappeared along the way. |
| **Overfitting (backtest overfitting)** | Tuning a strategy's rules against the same data used to evaluate it, until it fits that sample's noise rather than a real, repeatable pattern. |
| **Out-of-sample / walk-forward validation** | Testing a strategy on data it was not tuned against, either as a single held-out window or as a series of rolling train/test windows moving forward through time. |
