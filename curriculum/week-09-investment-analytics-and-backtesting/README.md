# Week 9 — Investment analytics & backtesting

> **Goal:** by Sunday you can take an investment thesis stated in plain English — "cheap, improving-momentum industrial names outperform" — encode it as a precise rules-based signal, run it through a backtest engine you built yourself with realistic transaction costs and monthly rebalancing, compute its Sharpe ratio and maximum drawdown, and prove (or disprove) that it survives out-of-sample instead of just curve-fitting the past.

Welcome back to **C35 · Crunch Capital**. Week 7 valued one company by discounting its cash flows and comparing it to peers. Week 8 took a *basket* of names and found the mean-variance-optimal way to hold them, assuming you already knew their expected returns and covariances. This week asks a different, more honest question: **how would you have actually decided what to buy and sell, week after week, using only information you had at the time — and would it have made money after costs?** That is what "backtesting" means, and it is simultaneously the most useful and the most abused tool in investing. Anyone can find a rule that would have worked on the last two years of data; the entire skill of this week is building a backtest rigorous enough to tell the difference between a real edge and an accident of the sample you happened to test on.

This week's universe is the same seven-name **industrial machinery** sector you built comps for in Week 7 — `CRNM` (Crunch Machine Co.) and its six peers `TFI`, `MMS`, `AFC`, `VDT`, `FPG`, `IMW`. You'll generate two years of realistic daily prices for all seven with a seeded, reproducible Python script (never hand-typed, never a spreadsheet), load them into `crunch_capital`, and then do four things in order: turn a thesis into a signal, turn a signal into positions and a return series, turn a return series into honest performance numbers, and — the part most self-taught backtesters skip — stress the whole thing for the biases that make backtests lie.

## Learning objectives

By the end of this week, you will be able to:

- **Translate an investment thesis into a rules-based signal** — encode momentum ("recent relative strength predicts near-term returns"), value ("cheap relative to fundamentals outperforms"), and mean-reversion ("sharp short-term moves partly revert") as precise, computable SQL/Python rules, not vibes.
- **Build a backtest engine from a price database** — turn a signal into a dated position table, apply a rebalancing schedule, and compute a daily portfolio return series from first principles (no backtesting library doing the work for you).
- **Model realistic frictions** — transaction costs (commissions/fees in basis points) and slippage (the gap between the price you signaled on and the price you actually traded at), and show, with numbers, how much of a "great" gross strategy costs disappear.
- **Compute the standard performance metrics** — total and annualized return (CAGR), annualized volatility, the Sharpe ratio, maximum drawdown, and hit rate (win rate) — and know exactly what each one is, and is not, telling you.
- **Detect and prevent look-ahead bias** — the single most common reason a backtest looks great and a live strategy doesn't, including the specific, realistic case of joining price data to a fundamentals snapshot without respecting *when* that snapshot became known.
- **Validate out-of-sample** — split history into a training window (where you're allowed to tune) and a held-out test window (where you're not), and walk that split forward through time to see whether a strategy's edge survives contact with data it never saw.

## Prerequisites

- **Week 7 of this course** (valuation & comparables) — this week reuses the same seven-company industrial-machinery peer set and the EV/EBITDA multiples you computed there as the raw material for the value factor. If those names and numbers aren't familiar, skim Week 7's README before starting.
- **Week 8 of this course** (portfolio theory & allocation) — you should already be comfortable computing a return series' mean, standard deviation, and Sharpe ratio in Python; this week extends that into a full time series built from daily prices rather than a single assumed number.
- Working SQL (`SELECT`, `JOIN`, `GROUP BY`, window functions like `LAG`/`LEAD` and `AVG(...) OVER (...)`) — from [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) or equivalent.
- Python 3.10+ with `pandas` and `numpy`. No new libraries required — everything this week, including the backtest engine itself, is code you write, not a black-box package. (`resources.md` lists optional backtesting frameworks worth knowing about *after* you've built one by hand.)
- **No spreadsheets.** Same [data tooling rule](../../SYLLABUS.md#data-tooling-rule) as every week: two years of daily prices for seven tickers is exactly the kind of dataset that's miserable and error-prone in a spreadsheet and trivial in a database. Every price, every signal, every position, and every return this week lives in SQL and Python.

## Set up this week's data (do this first)

You need the same PostgreSQL 16+ (or SQLite 3.35+) database from prior weeks, plus Python with `pandas`, `numpy`, and either `psycopg2`/`sqlalchemy` (Postgres) or the standard-library `sqlite3` module.

**Step 1 — create the two new tables.**

```bash
psql crunch_capital              # PostgreSQL
# or
sqlite3 crunch_capital.db        # SQLite fallback
```

```sql
-- 1. The seven-name universe this week trades. Same companies as Week 7's
--    trading comps, so a value factor built from those multiples is real,
--    not invented for this week.
CREATE TABLE securities (
    ticker      TEXT PRIMARY KEY,
    company     TEXT    NOT NULL,
    sector      TEXT    NOT NULL
);

INSERT INTO securities VALUES
('CRNM', 'Crunch Machine Co.',          'Industrial Machinery'),
('TFI',  'TitanForge Industrial',       'Industrial Machinery'),
('MMS',  'Meridian Motion Systems',     'Industrial Machinery'),
('AFC',  'Anchor Fabrication Corp',     'Industrial Machinery'),
('VDT',  'Vantage Drive Technologies',  'Industrial Machinery'),
('FPG',  'Falcon Precision Group',      'Industrial Machinery'),
('IMW',  'Ironclad Machine Works',      'Industrial Machinery');

-- 2. Daily prices — this is what you generate with the script below.
CREATE TABLE daily_prices (
    ticker       TEXT    NOT NULL REFERENCES securities(ticker),
    price_date   DATE    NOT NULL,
    close_price  NUMERIC NOT NULL,
    volume       BIGINT  NOT NULL,
    PRIMARY KEY (ticker, price_date)
);

-- 3. A single point-in-time fundamentals snapshot — EV/EBITDA multiples
--    AS OF 2025-12-31, carried over from Week 7's trading_comps. On purpose
--    there is only one snapshot date. That's not laziness: it's the setup
--    for Lecture 3 and Challenge 2, where you'll discover what happens when
--    a backtest silently uses a fundamentals number *before* it existed.
CREATE TABLE fundamental_snapshot (
    ticker       TEXT    NOT NULL REFERENCES securities(ticker),
    as_of_date   DATE    NOT NULL,
    ev_ebitda    NUMERIC NOT NULL,
    PRIMARY KEY (ticker, as_of_date)
);

INSERT INTO fundamental_snapshot VALUES
('CRNM', '2025-12-31', 16.40),
('TFI',  '2025-12-31', 23.58),
('MMS',  '2025-12-31', 17.69),
('AFC',  '2025-12-31', 15.19),
('VDT',  '2025-12-31', 32.44),
('FPG',  '2025-12-31', 19.94),
('IMW',  '2025-12-31', 10.58);
```

**Step 2 — generate two years of daily prices.** This is a reproducible geometric Brownian motion simulation, seeded so everyone in the course gets identical numbers. Save this as `generate_prices.py`:

```python
import numpy as np
import pandas as pd

# One seed for the whole course = every student gets the same 3,528 rows.
rng = np.random.default_rng(35090)

# ticker: (start_price, annual_drift, annual_vol)
# VDT is the expensive, high-vol "story stock" prone to sharp reversals.
# IMW is statistically cheap and cyclical — high vol, low drift.
# TFI has the strongest recent momentum. AFC and FPG are steady compounders.
PARAMS = {
    "CRNM": (46.00, 0.08, 0.24),
    "TFI":  (38.00, 0.14, 0.30),
    "MMS":  (27.00, 0.04, 0.22),
    "AFC":  (31.00, 0.05, 0.18),
    "VDT":  (52.00, 0.10, 0.38),
    "FPG":  (41.00, 0.09, 0.20),
    "IMW":  (19.00, 0.03, 0.33),
}

dates = pd.bdate_range("2024-01-02", "2025-12-31")   # business days only
n = len(dates)

rows = []
for ticker, (start, mu, sigma) in PARAMS.items():
    daily_mu = mu / 252
    daily_sigma = sigma / np.sqrt(252)
    z = rng.standard_normal(n)
    log_returns = daily_mu - 0.5 * daily_sigma**2 + daily_sigma * z
    price_path = start * np.exp(np.cumsum(log_returns))
    volume = rng.lognormal(mean=13.0, sigma=0.35, size=n).astype(int)  # ~450k shares/day median
    for d, p, v in zip(dates, price_path, volume):
        rows.append((ticker, d.date().isoformat(), round(float(p), 2), int(v)))

prices = pd.DataFrame(rows, columns=["ticker", "price_date", "close_price", "volume"])
prices.to_csv("daily_prices_seed.csv", index=False)
print(f"generated {len(prices)} rows for {len(PARAMS)} tickers over {n} business days")
```

**Step 3 — load the CSV into the database.**

```python
# PostgreSQL (sqlalchemy + psycopg2)
from sqlalchemy import create_engine
import pandas as pd

prices = pd.read_csv("daily_prices_seed.csv", parse_dates=["price_date"])
engine = create_engine("postgresql+psycopg2://localhost/crunch_capital")
prices.to_sql("daily_prices", engine, if_exists="append", index=False, method="multi", chunksize=1000)
```

```python
# SQLite fallback
import sqlite3
import pandas as pd

prices = pd.read_csv("daily_prices_seed.csv")
conn = sqlite3.connect("crunch_capital.db")
prices.to_sql("daily_prices", conn, if_exists="append", index=False)
conn.close()
```

**Sanity check** — this should print `3528` (7 tickers × 504 business days):

```sql
SELECT COUNT(*) FROM daily_prices;
```

And this should print 7 rows, each with a plausible price in the $15–$140 range after two years of drift:

```sql
SELECT ticker, price_date, close_price
FROM daily_prices
WHERE price_date = (SELECT MAX(price_date) FROM daily_prices)
ORDER BY ticker;
```

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace).

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Data setup; thesis → signal | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Momentum, value, mean-reversion signals | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Wednesday | Building the backtest engine | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Costs, slippage, performance metrics | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Overfitting, look-ahead bias, walk-forward | 2h | 0h | 1h | 0.5h | 1h | 1.5h | 6h |
| Saturday | Mini-project (full strategy backtest) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-from-thesis-to-signal.md](./lecture-notes/01-from-thesis-to-signal.md) | Encoding momentum, value, and mean-reversion theses as precise SQL/Python rules | 2h |
| 2 | [lecture-notes/02-building-a-backtest.md](./lecture-notes/02-building-a-backtest.md) | Positions, rebalancing, transaction costs, slippage, and a return-series engine | 2h |
| 3 | [lecture-notes/03-overfitting-and-honesty.md](./lecture-notes/03-overfitting-and-honesty.md) | Look-ahead bias, overfitting, and out-of-sample / walk-forward testing | 2h |
| 4 | [exercises/exercise-01-encode-a-signal.md](./exercises/exercise-01-encode-a-signal.md) | Build a momentum + value signal table in SQL | 1h |
| 5 | [exercises/exercise-02-run-a-backtest.md](./exercises/exercise-02-run-a-backtest.md) | Turn the signal into positions and a cost-adjusted return series | 1.5h |
| 6 | [exercises/exercise-03-performance-metrics.md](./exercises/exercise-03-performance-metrics.md) | Compute CAGR, vol, Sharpe, drawdown, and hit rate | 1.5h |
| 7 | [challenges/challenge-01-walk-forward-validation.md](./challenges/challenge-01-walk-forward-validation.md) | Walk-forward validate the strategy across rolling windows | 1.5h |
| 8 | [challenges/challenge-02-expose-a-lookahead-bug.md](./challenges/challenge-02-expose-a-lookahead-bug.md) | Find and fix a real look-ahead bug in a given backtest script | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Backtest a rules-based strategy with realistic costs, out-of-sample | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, papers on backtest overfitting, and further reading | — |

## By the end of this week you can…

- Take a sentence like "buy what's gone up, avoid what's expensive" and turn it into an unambiguous, backtestable rule.
- Build a backtest engine from a price table with your own hands — no framework doing the thinking for you.
- Put a real number on transaction costs and explain how much of a strategy's edge they eat.
- Compute and correctly interpret Sharpe ratio, max drawdown, and hit rate — and explain what each one hides.
- Spot a look-ahead bug on sight, and design a test that would have caught it before it cost real money.
- Tell the difference between a strategy that has an edge and one that got lucky on the sample you tested.

## Up next

[Week 10 — M&A & deal analysis](../week-10-mergers-acquisitions-and-deals/) — from picking which stocks to own to buying the whole company: modeling a merger, accretion/dilution, and the financing mix.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
