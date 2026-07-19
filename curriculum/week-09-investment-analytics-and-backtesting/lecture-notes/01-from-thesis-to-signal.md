# Lecture 1 — From Thesis to Signal

> **Duration:** ~2 hours. **Outcome:** You can take an investment thesis stated in plain English and turn it into a precise, computable **signal** — a table of `(ticker, date, score, position)` that a backtest engine can consume without any further human judgment. You'll build momentum, value, and mean-reversion signals against the seven-ticker `daily_prices` table.

Every systematic investment strategy starts life as a sentence a human believes: *"stocks that have gone up tend to keep going up for a while,"* or *"cheap companies eventually get repriced upward,"* or *"a stock that just fell 8% in a day for no fundamental reason tends to bounce back a little."* That sentence is a **thesis**. A thesis is not a strategy. A strategy is what you get after you force the thesis to answer three unforgiving questions: *exactly which securities, on exactly which date, using exactly what data known at that moment, do you buy or sell?* The object that answers those three questions is a **signal**, and building one honestly is this lecture's entire job.

## 1. What a signal actually is

A signal is a table, not a feeling. For every ticker, on every date you might trade, it says one of three things: **long** (own it), **short** (bet against it — we won't short this week, see the note below), or **flat** (don't hold it). Concretely, by the end of this lecture you will have built something that looks like:

| ticker | signal_date | momentum_score | value_score | position |
|--------|------------|----------------:|------------:|----------|
| TFI | 2025-06-30 | 0.184 | -1.02 | long |
| IMW | 2025-06-30 | -0.041 | 1.35 | long |
| VDT | 2025-06-30 | 0.062 | -1.61 | flat |
| ... | ... | ... | ... | ... |

That table — and only that table — is what Lecture 2's backtest engine is allowed to see. It does not know *why* TFI is long. It does not know your thesis. It just executes. This separation is deliberate and important: it forces every piece of judgment into the signal-construction step, where you can inspect, debate, and audit it, instead of letting judgment leak invisibly into the backtest itself.

**A note on shorting.** A real long/short strategy holds long positions in names it expects to outperform and short positions in names it expects to underperform, which lets it profit even in a falling market. This week's backtest is **long-only with a cash residual** — we hold the strategy's chosen names and leave the rest of the portfolio in cash — because it's simpler to reason about and matches how most individual investors and long-only funds actually operate. Every technique here (ranking, scoring, position sizing) extends directly to long/short; we call it out explicitly rather than pretending the simplification doesn't exist.

## 2. Momentum — "recent relative strength persists"

The momentum thesis, in its classic academic form (Jegadeesh & Titman, 1993): stocks that outperformed over the past 3–12 months tend to keep outperforming over the next 1–3 months. The standard implementation is **12-1 momentum** — the trailing 12-month return, *excluding the most recent month*. The most recent month is excluded on purpose: very short-term returns are dominated by mean-reversion (Section 4), and mixing the two signals together muddies both.

In SQL, against `daily_prices`, this means: for each ticker, find the close price roughly 252 trading days ago and the close price roughly 21 trading days ago, and compute the return between them.

```sql
WITH ranked AS (
    SELECT
        ticker,
        price_date,
        close_price,
        ROW_NUMBER() OVER (PARTITION BY ticker ORDER BY price_date) AS day_num
    FROM daily_prices
),
lagged AS (
    SELECT
        r.ticker,
        r.price_date          AS signal_date,
        r.close_price          AS price_t,
        lag12.close_price      AS price_t_minus_252,   -- ~12 months back
        lag1.close_price       AS price_t_minus_21      -- ~1 month back
    FROM ranked r
    JOIN ranked lag12
      ON lag12.ticker = r.ticker AND lag12.day_num = r.day_num - 252
    JOIN ranked lag1
      ON lag1.ticker = r.ticker AND lag1.day_num = r.day_num - 21
)
SELECT
    ticker,
    signal_date,
    ROUND( (price_t_minus_21 / price_t_minus_252 - 1)::numeric, 4) AS momentum_12_1
FROM lagged
ORDER BY signal_date, ticker;
```

Read that query the way you'd read English: for every trading day that has at least 252 prior trading days of history, compute the return from 252 days ago to 21 days ago, and call that number `momentum_12_1`. A ticker with `momentum_12_1 = 0.18` returned 18% over that 11-month window and *ranks high* on the momentum factor; a ticker with `-0.05` ranks low.

**Why exclude the most recent month specifically, not zero or two?** This is an empirical convention from the original research, not a law of nature — but it's worth internalizing *why* a convention like this exists at all: it was chosen because it maximized the strategy's performance in-sample on decades of US equity data. That should make you a little uneasy, and it should — we come back to exactly this uneasiness in Lecture 3.

## 3. Value — "cheap relative to fundamentals outperforms"

The value thesis: a company priced cheaply relative to what it actually earns (or generates in cash) tends to outperform a company priced richly relative to the same yardstick, because the market eventually re-rates it. You built the yardstick already, in Week 7: **EV/EBITDA**. A low multiple means the market is paying less for each dollar of the company's operating earnings — all else equal, cheap. This week's `fundamental_snapshot` table carries exactly those Week 7 multiples forward, dated `2025-12-31`.

```sql
SELECT
    ticker,
    ev_ebitda,
    -- rank 1 = cheapest (lowest multiple) = strongest value signal
    RANK() OVER (ORDER BY ev_ebitda ASC) AS value_rank
FROM fundamental_snapshot
WHERE as_of_date = '2025-12-31';
```

```
 ticker | ev_ebitda | value_rank
--------+-----------+-----------
 IMW    |     10.58 |         1
 AFC    |     15.19 |         2
 CRNM   |     16.40 |         3
 MMS    |     17.69 |         4
 FPG    |     19.94 |         5
 TFI    |     23.58 |         6
 VDT    |     32.44 |         7
```

`IMW` is the cheapest name in the universe; `VDT` is the richest. Notice something important and slightly uncomfortable: **this table has exactly one date.** In the real world you'd have a fundamentals snapshot updating every quarter (as new earnings come out), and a value signal built correctly would join each trading day to the *most recent snapshot available on or before that day* — never a snapshot from the future. We deliberately gave you only the `2025-12-31` snapshot so that building the value signal "naively" (joining on ticker alone, ignoring dates) makes an error that's *invisible until you go looking for it*. Hold that thought — it's the entire subject of Lecture 3 and Challenge 2.

## 4. Mean-reversion — "sharp short-term moves partly revert"

The mean-reversion thesis operates on the opposite time horizon from momentum: over days, not months, a stock that just moved sharply tends to give some of that move back, because the move often overshot the news that caused it (or was pure noise/liquidity pressure with no news at all). A simple, honest implementation is **5-day reversal**: rank names by their trailing 5-trading-day return, and go long the *biggest losers* (expecting a bounce) rather than the biggest winners.

```sql
WITH ranked AS (
    SELECT
        ticker, price_date, close_price,
        ROW_NUMBER() OVER (PARTITION BY ticker ORDER BY price_date) AS day_num
    FROM daily_prices
),
five_day AS (
    SELECT
        r.ticker,
        r.price_date AS signal_date,
        r.close_price / lag5.close_price - 1 AS return_5d
    FROM ranked r
    JOIN ranked lag5
      ON lag5.ticker = r.ticker AND lag5.day_num = r.day_num - 5
)
SELECT ticker, signal_date, ROUND(return_5d::numeric, 4) AS return_5d
FROM five_day
WHERE signal_date = '2025-06-30'
ORDER BY return_5d ASC;   -- most negative (biggest recent loser) first
```

Momentum and mean-reversion are not contradictory theses — they operate over different, roughly non-overlapping horizons (months vs. days), and both have decades of published evidence behind them at their *own* horizon. The mistake beginners make is applying a reversal signal over a 6-month window (where momentum dominates instead) or a momentum signal over a 5-day window (where reversal dominates instead). **Horizon is not a detail — it's the core of the thesis.**

## 5. Turning a score into a position: ranking and thresholds

A raw score (`momentum_12_1 = 0.184`) is not yet a position. You need a rule that converts scores across the whole universe, on a given date, into a small number of discrete decisions. The two standard approaches:

**Top-N selection.** Rank all seven names by the score and go long the top `N` (say, top 3). Simple, always holds exactly `N` names, easy to reason about.

```sql
WITH scored AS (
    SELECT ticker, momentum_12_1,
           RANK() OVER (ORDER BY momentum_12_1 DESC) AS mom_rank
    FROM momentum_signal
    WHERE signal_date = '2025-06-30'
)
SELECT ticker, momentum_12_1,
       CASE WHEN mom_rank <= 3 THEN 'long' ELSE 'flat' END AS position
FROM scored
ORDER BY mom_rank;
```

**Threshold selection.** Go long anything above a fixed cutoff (say, `momentum_12_1 > 0.05`), which can hold anywhere from zero to all seven names depending on the market. More faithful to "only trade when the signal is strong," but the portfolio's size becomes unpredictable — a real operational problem when you need to size positions.

This week's exercises use **top-N** because it keeps the backtest engine's position sizing simple (equal-weight across whatever's currently long) and makes it easy to compare strategies apples-to-apples on turnover and cost.

## 6. Combining signals: a composite score

A single-factor strategy (momentum only, or value only) is rarely what serious investors run — they blend factors, because momentum and value tend to fail at *different times* (value did well in 2000–2002, when momentum crashed; momentum did well through much of the 2010s bull market, when value struggled). A simple, transparent way to combine two ranked signals is to average their **ranks** (not their raw scores, which live on different, incomparable scales — a momentum score of `0.18` and an EV/EBITDA of `18` are not the same kind of number):

```sql
WITH mom AS (
    SELECT ticker, RANK() OVER (ORDER BY momentum_12_1 DESC) AS mom_rank
    FROM momentum_signal WHERE signal_date = '2025-06-30'
),
val AS (
    SELECT ticker, RANK() OVER (ORDER BY ev_ebitda ASC) AS val_rank
    FROM fundamental_snapshot WHERE as_of_date = '2025-12-31'
)
SELECT
    mom.ticker,
    mom.mom_rank,
    val.val_rank,
    (mom.mom_rank + val.val_rank) / 2.0 AS composite_rank
FROM mom
JOIN val ON val.ticker = mom.ticker
ORDER BY composite_rank ASC;
```

A ticker that's *both* cheap and has strong momentum gets a low (good) composite rank and rises to the top of the buy list; a ticker that's expensive with weak momentum sinks to the bottom. This is exactly the mechanic behind real multi-factor equity strategies, just with two factors instead of five or six.

## 7. Check yourself

- What is the precise difference between an investment *thesis* and a *signal*? Why does the backtest engine only get to see the signal?
- Why does 12-1 momentum exclude the most recent month, and what thesis does that excluded month belong to instead?
- What's wrong with joining `daily_prices` to `fundamental_snapshot` on `ticker` alone, with no date condition? What would that let a backtest running in January 2024 "know" that it shouldn't?
- What's the operational trade-off between top-N selection and threshold selection?
- Why average **ranks** rather than raw scores when combining momentum and value into a composite?

If those are automatic, Lecture 2 turns a signal table like the ones above into an actual position schedule, a rebalancing routine, and a return series — the backtest engine itself.

## Further reading

- **Jegadeesh & Titman (1993), "Returns to Buying Winners and Selling Losers"** — the original momentum paper: <https://www.jstor.org/stable/2328882> (abstract free; full text often available via university/library access)
- **Fama & French (1998), "Value versus Growth: The International Evidence"** — cross-country evidence for the value premium: <https://doi.org/10.1111/0022-1082.00080>
- **PostgreSQL — Window Functions Tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
- **PostgreSQL — Window Function reference (`RANK`, `LAG`, `ROW_NUMBER`):** <https://www.postgresql.org/docs/current/functions-window.html>
