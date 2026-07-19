# Exercise 3 — Compute Performance Metrics

**Goal:** Take Exercise 2's daily net-return series and compute the five numbers every performance report needs: CAGR, annualized volatility, Sharpe ratio, maximum drawdown, and hit rate — each written as its own small, testable function.

**Estimated time:** 60 minutes.

## Setup

```python
import pandas as pd
import numpy as np

result = pd.read_csv("backtest_result.csv", index_col=0, parse_dates=True)
net_returns = result["net_return"].dropna()
equity_curve = result["equity_curve"].dropna()
print(net_returns.describe())
```

Create `exercise_03.py`. Write each function below, print its output for the sanity checks, then assemble a one-call summary at the end.

## Tasks

1. **Total return and CAGR.** Total return is simply `equity_curve.iloc[-1] - 1`. CAGR (compound annual growth rate) annualizes it over the actual number of years the backtest ran:

   ```python
   def cagr(equity_curve: pd.Series, periods_per_year: int = 252) -> float:
       n_periods = len(equity_curve)
       years = n_periods / periods_per_year
       total_return = equity_curve.iloc[-1] / equity_curve.iloc[0]
       return total_return ** (1 / years) - 1
   ```

   Run it. *(Expected: a small number, likely single-digit percent either sign — this is a short, cost-aware backtest on 7 synthetic names, not a headline hedge-fund result.)*

2. **Annualized volatility.** The standard deviation of daily returns, scaled to a yearly figure by the square root of trading days per year:

   ```python
   def annualized_vol(daily_returns: pd.Series, periods_per_year: int = 252) -> float:
       return daily_returns.std() * np.sqrt(periods_per_year)
   ```

   *(Expected: given our universe's 18–38% annual vol per name and 3-name diversification, somewhere in the 15–30% range.)*

3. **Sharpe ratio.** Excess return per unit of risk, using a risk-free rate you must state explicitly (don't hardcode `0` silently — as of this course's writing, a reasonable short-term risk-free assumption is in the 3–5% range; pick one and name it in a comment):

   ```python
   def sharpe_ratio(daily_returns: pd.Series, risk_free_annual: float = 0.04, periods_per_year: int = 252) -> float:
       excess_daily = daily_returns - (risk_free_annual / periods_per_year)
       return (excess_daily.mean() / excess_daily.std()) * np.sqrt(periods_per_year)
   ```

   Compute it with your stated `risk_free_annual`, then recompute with `risk_free_annual=0.0` and note how much the number moves. *(A Sharpe ratio above 1.0 for a real, cost-adjusted, multi-year equity strategy is genuinely good — be suspicious of anything above 2.0 on a backtest this small; it's far more likely to be a sign of overfitting or a bug than a sign of skill. Revisit Lecture 3 if yours comes out that high.)*

4. **Maximum drawdown.** The largest peak-to-trough decline in the equity curve — the single number that best captures "how bad did it get, at the worst possible moment to be watching":

   ```python
   def max_drawdown(equity_curve: pd.Series) -> float:
       running_max = equity_curve.cummax()
       drawdown = equity_curve / running_max - 1
       return drawdown.min()   # most negative value = the worst drawdown, expressed as a negative fraction
   ```

   Also find and print the **date** of the trough (`drawdown.idxmin()`) and the date of the peak that preceded it (`equity_curve[:drawdown.idxmin()].idxmax()`) — a drawdown number without its dates tells you far less than the number plus context.

5. **Hit rate.** The fraction of rebalance-to-rebalance periods (not individual days — individual daily hit rate is a much noisier, less useful number for a monthly-rebalanced strategy) where the strategy's net return was positive:

   ```python
   def hit_rate(daily_returns: pd.Series, rebalance_dates: pd.DatetimeIndex) -> float:
       period_returns = []
       for start, end in zip(rebalance_dates[:-1], rebalance_dates[1:]):
           period = daily_returns[(daily_returns.index > start) & (daily_returns.index <= end)]
           period_returns.append((1 + period).prod() - 1)
       period_returns = pd.Series(period_returns)
       return (period_returns > 0).mean()
   ```

   *(Expected: some fraction between 0 and 1 — with roughly 11 monthly periods this year, the number will jump around a lot per period; say so in your write-up rather than over-interpreting a ratio built on 11 data points.)*

6. **Assemble a one-call summary.** Write `summarize_performance(result, rebalance_dates, risk_free_annual=0.04)` that calls all five functions and returns a single dict/Series with all five metrics, printed to two decimal places (percentages) or two decimal places (Sharpe, unitless).

## Expected result (spot checks)

- `cagr` and `annualized_vol` should both be finite, non-`NaN`, non-`inf` numbers — if you get `inf` or `NaN`, check for a `0` in `equity_curve.iloc[0]` or an all-zero returns window.
- `max_drawdown` must be **≤ 0** (it's a decline, by definition) — a positive number means you inverted a sign somewhere.
- `hit_rate` must be between `0.0` and `1.0` inclusive.
- Print all five metrics together and read them as a *paragraph*, not five isolated numbers: "The strategy returned X% CAGR with Y% volatility (Sharpe Z), the worst drawdown was W% around [date], and it was profitable in V% of monthly rebalance periods." If you can't turn your five numbers into that sentence, you don't understand them yet — go back and check each one individually.

## Done when…

- [ ] All five metric functions run independently and pass their sign/range sanity checks above.
- [ ] `summarize_performance` produces one clean dict/Series you can print or hand to a report.
- [ ] You've written the one-paragraph plain-English summary described above, using your actual computed numbers.
- [ ] You've stated your `risk_free_annual` assumption explicitly in a comment, and shown how the Sharpe ratio changes if you'd used `0.0` instead.

## Stretch

- Compute a **rolling 3-month Sharpe ratio** (recompute Sharpe on each trailing ~63-trading-day window) and plot or print it over time. Is the strategy's risk-adjusted performance stable, or does it swing wildly between good and bad stretches? What does that tell you about how much you should trust the single full-period Sharpe ratio?
- Compute the same five metrics for a naive **equal-weight-all-7-tickers, buy-and-hold** benchmark (no signal, no rebalancing after day 1) and compare. Did the signal actually add value over just owning the whole universe?

## Submission

Commit `exercise_03.py` to your portfolio under `c35-week-09/exercise-03/`, including the printed one-paragraph summary.
