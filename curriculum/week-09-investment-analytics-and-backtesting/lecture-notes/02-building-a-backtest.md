# Lecture 2 — Building a Backtest

> **Duration:** ~2 hours. **Outcome:** You can turn a signal table into a dated position schedule, apply a monthly rebalance, model transaction costs and slippage, and compute a strategy's daily return series and cumulative equity curve — in Python, pulling prices from SQL, with no backtesting library doing the work for you.

A signal tells you what you'd *like* to hold. A backtest tells you what actually would have happened if you'd held it — including the frictions that separate a spreadsheet fantasy from a tradable strategy. This lecture builds that machine from four parts, in order: **rebalancing** (when do positions change), **position sizing** (how much of each name), **costs** (what trading actually costs you), and **the return engine** (turning daily prices and dated positions into a single portfolio return series).

## 1. Why you rebalance instead of trading continuously

A signal like 12-1 momentum changes value every single trading day — yesterday's winner might rank slightly differently today. If you traded every day to keep the portfolio *exactly* matched to the day's ranking, you'd generate enormous transaction costs relative to any expected edge (this is called "churning," and it's the fastest way to turn a profitable strategy into an unprofitable one). Real systematic strategies **rebalance on a fixed schedule** — monthly is standard for a momentum/value strategy with a multi-month horizon — and hold the resulting portfolio unchanged between rebalance dates, letting daily price moves do what they will.

```python
import pandas as pd

def rebalance_dates(price_dates: pd.DatetimeIndex) -> pd.DatetimeIndex:
    """Last trading day of each calendar month, drawn from actual price dates
    (never assume the 30th/31st exists as a trading day)."""
    df = pd.DataFrame(index=price_dates)
    df["year_month"] = df.index.to_period("M")
    last_day_per_month = df.groupby("year_month").apply(lambda g: g.index.max())
    return pd.DatetimeIndex(last_day_per_month.values)
```

Given our two years of business-day prices, this returns 24 dates — the last trading day of every month from January 2024 through December 2025. On each of those 24 dates, and *only* those dates, the strategy is allowed to change what it holds.

## 2. Position sizing: equal weight, top-N

With a top-N signal (Lecture 1), the simplest defensible sizing rule is **equal weight**: if 3 names are long, each gets 1/3 of the portfolio's capital; if the signal only surfaces 2 names on a given rebalance (say a threshold rule left the book smaller), each gets 1/2, and the rest sits in cash earning nothing (a simplification — a real fund would earn the risk-free rate on idle cash, which you can add as a stretch).

```python
def size_positions(longs: list[str]) -> dict[str, float]:
    """Equal-weight the names on the long list; empty book = 100% cash."""
    if not longs:
        return {}
    weight = 1.0 / len(longs)
    return {ticker: weight for ticker in longs}
```

Equal weight is deliberately unsophisticated — it ignores volatility differences (VDT is nearly twice as volatile as AFC in our universe, so an equal-weight book is implicitly *overweight* VDT's risk contribution). Volatility-scaled or risk-parity weighting is a real refinement, and Week 8's mean-variance machinery is exactly the tool for it — but layering that on top of a signal you haven't yet proven has real, cost-adjusted, out-of-sample edge is solving the wrong problem first. Get the simple version honestly right before making it fancier.

## 3. Transaction costs and slippage — the two frictions that kill strategies

**Transaction cost** is the direct cost of trading: commissions, exchange fees, and the bid-ask spread you cross every time you buy or sell. We model it simply, as a fixed number of **basis points (bps)** of the traded notional — 1 bp = 0.01%. A retail-realistic assumption for a liquid mid-cap stock is **5–10 bps per trade** (each side — buying costs bps, selling costs bps again).

**Slippage** is subtler and often larger: the price you actually get filled at differs from the price your signal was computed on, because time passes between "decide" and "execute," and because your own order moves the market if it's large relative to the stock's volume. We'll model it the same way — as additional bps on top of transaction cost — but it's worth understanding *why* it's a separate concept: transaction cost is a fee you're charged; slippage is a price you pay because markets aren't frictionless and instantaneous, even with zero fees.

```python
TRANSACTION_COST_BPS = 5     # commissions + spread, one-way
SLIPPAGE_BPS = 5              # execution slippage, one-way
TOTAL_COST_BPS = TRANSACTION_COST_BPS + SLIPPAGE_BPS   # 10 bps one-way

def trade_cost(notional_traded: float, cost_bps: float = TOTAL_COST_BPS) -> float:
    """Dollar cost of one trade (buy OR sell) of the given notional value."""
    return notional_traded * (cost_bps / 10_000)
```

At 10 bps one-way, fully turning over a $100,000 portfolio (sell everything, buy something new) costs `$100,000 × 0.0010 × 2 = $200` — trivial-looking per trade, but it compounds: a strategy that fully turns over monthly pays roughly `10 bps × 2 × 12 = 240 bps`, or 2.4%, of its capital to trading *every year*, before it's made a single dollar of alpha. **A strategy with a 3% annual gross edge and 2.4% in annual costs has a 0.6% net edge** — and that's before slippage gets worse in a real fill (our flat-bps model is optimistic; real slippage grows with order size and shrinks with liquidity). This is the single most common reason a backtest that "worked" loses money live: costs were left out, or modeled too generously.

## 4. The return engine: from positions to a portfolio return series

This is the core of the backtest. The algorithm, stated precisely:

1. On each rebalance date, compute the new target positions (weights) from the signal.
2. Compare the new weights to yesterday's weights; the difference (in absolute value) is how much notional gets traded, which determines cost.
3. Between rebalance dates, hold weights fixed and let each name's daily return flow through — the portfolio return on any day is the weighted sum of the day's held names' returns.
4. Subtract the day's trading cost (if any) from the day's return.
5. Compound daily returns into a cumulative equity curve.

```python
import pandas as pd
import numpy as np

def run_backtest(
    prices: pd.DataFrame,        # columns: ticker, price_date, close_price (long format)
    positions: pd.DataFrame,     # columns: rebalance_date, ticker, weight
    cost_bps: float = 10.0,
) -> pd.DataFrame:
    """Returns a DataFrame indexed by date with columns:
    gross_return, cost, net_return, equity_curve (starts at 1.0)."""

    px = prices.pivot(index="price_date", columns="ticker", values="close_price").sort_index()
    daily_rets = px.pct_change()

    rebalance_dates = sorted(positions["rebalance_date"].unique())
    weights = pd.DataFrame(0.0, index=px.index, columns=px.columns)

    # Forward-fill target weights: each rebalance date's weights persist
    # until the NEXT rebalance date, and no earlier.
    for i, rdate in enumerate(rebalance_dates):
        w_today = positions[positions["rebalance_date"] == rdate].set_index("ticker")["weight"]
        end_date = rebalance_dates[i + 1] if i + 1 < len(rebalance_dates) else px.index[-1]
        mask = (px.index > rdate) & (px.index <= end_date)   # STRICTLY after the rebalance date
        for ticker, w in w_today.items():
            weights.loc[mask, ticker] = w

    gross_return = (weights.shift(1) * daily_rets).sum(axis=1)   # yesterday's weights earn today's return

    # Cost: charged on rebalance dates only, proportional to turnover
    # (sum of absolute weight changes) — one round-trip cost per unit turned over.
    turnover = weights.diff().abs().sum(axis=1)
    cost = turnover * (cost_bps / 10_000)

    net_return = gross_return - cost
    equity_curve = (1 + net_return.fillna(0)).cumprod()

    return pd.DataFrame({
        "gross_return": gross_return,
        "cost": cost,
        "net_return": net_return,
        "equity_curve": equity_curve,
    })
```

Two lines deserve a second look, because they're where beginners' backtests go quietly wrong:

**`weights.shift(1) * daily_rets`** — the portfolio earns *today's* return using *yesterday's* weights, not today's. This is not a stylistic choice; it's the difference between a legitimate backtest and a look-ahead-biased one. You cannot know today's closing price (and therefore today's target weight, if the signal depends on it) until the market closes today — so you can only trade *tomorrow* based on it. If you multiplied today's weight by today's return, you'd be implicitly assuming you traded at this morning's open using information only available at tonight's close. Lecture 3 dissects this exact bug in much more depth.

**`mask = (px.index > rdate) & (px.index <= end_date)`** — weights take effect strictly *after* the rebalance date, not on it. The rebalance date itself is "decision day" — you compute the new weights using data through that day's close, but you can't have already earned that day's return with the new weights, because you didn't own them yet when the day started.

## 5. Running it end to end

```python
from sqlalchemy import create_engine
import pandas as pd

engine = create_engine("postgresql+psycopg2://localhost/crunch_capital")
prices = pd.read_sql("SELECT ticker, price_date, close_price FROM daily_prices ORDER BY price_date", engine)
prices["price_date"] = pd.to_datetime(prices["price_date"])

# positions built in Exercise 1/2 — one row per (rebalance_date, ticker, weight)
positions = build_momentum_positions(prices, top_n=3)   # from Lecture 1's signal logic

result = run_backtest(prices, positions, cost_bps=10.0)
print(result.tail())
print(f"\nFinal equity: {result['equity_curve'].iloc[-1]:.4f}")
print(f"Total cost paid (cumulative, in return terms): {result['cost'].sum():.4f}")
```

A well-formed backtest **always** reports the gross return series *and* the cost series *and* the net return series, side by side — never just the final net number. Showing only "the strategy returned 34% over two years" without showing gross-vs-net is how strategies get sold that would have lost money the moment someone tried to trade them for real.

## 6. What this engine deliberately does NOT model (and why that's honest to say)

- **Market impact beyond flat slippage.** A real large order moves the price against you *while you're filling it*; our flat-bps model doesn't scale with order size or the stock's own volume. Fine for a small backtest like this week's; a real fund models this explicitly.
- **Partial fills and liquidity constraints.** We assume every trade fills completely, instantly, at the modeled price. Real markets sometimes don't have enough volume to fill a large order at any single price.
- **Short-sale mechanics** (borrow cost, availability, margin calls) — irrelevant this week since we're long-only, but essential the moment a strategy shorts.
- **Taxes.** A very real cost for a taxable account, ignored here as it depends entirely on account type and jurisdiction.

None of these omissions are secret — a professional backtest report states its simplifying assumptions explicitly, in writing, right next to the results. That habit is worth more than any individual modeling refinement.

## 7. Check yourself

- Why rebalance monthly instead of trading every day the signal changes?
- Walk through, in words, what `weights.shift(1) * daily_rets` prevents.
- If a strategy's gross annual return is 9% and its one-way trading cost is 10 bps with monthly full turnover, roughly what is its net return? Show the arithmetic.
- What's the difference between transaction cost and slippage, conceptually?
- Name two frictions this week's engine does not model, and one situation where each would matter a lot.

Lecture 3 takes this exact engine and shows you the ways it can lie to you even when every line of code above is correct — because the sin isn't always in the engine, it's often in the signal or the sample you tested it on.

## Further reading

- **pandas — `pct_change`, `pivot`, `shift` (the three functions this engine leans on):** <https://pandas.pydata.org/docs/reference/frame.html>
- **pandas — Time series / date functionality:** <https://pandas.pydata.org/docs/user_guide/timeseries.html>
- **Investopedia — "Slippage":** <https://www.investopedia.com/terms/s/slippage.asp>
- **CFA Institute — "Transaction Cost Analysis" (overview, free):** <https://www.cfainstitute.org/en/research/foundation/2018/transaction-cost-analysis>
