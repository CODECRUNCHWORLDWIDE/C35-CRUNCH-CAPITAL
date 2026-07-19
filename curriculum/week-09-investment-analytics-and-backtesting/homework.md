# Week 9 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on SQL/Python, a written explanation, and a small research task. Commit each.

All work runs against the `crunch_capital` database and the `daily_prices`/`fundamental_snapshot` tables from the [week README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Twenty signal warm-ups (75 min)

Write and run each as SQL (or a short pandas snippet where noted). Put them in `warmups.sql` / `warmups.py` with a numbered comment and the actual result beneath each.

1. Which ticker had the single highest closing price on the last trading day of the dataset?
2. Which ticker had the single lowest closing price on the first trading day of the dataset?
3. Compute each ticker's simple return from the first trading day to the last (no rolling windows — just `last_price / first_price - 1`).
4. Rank all seven tickers by that full-period return, highest first.
5. Compute each ticker's 21-trading-day trailing volatility (standard deviation of daily returns over the most recent 21 days) as of the last date in the dataset.
6. Which ticker had the highest 21-day trailing volatility as of that date? Does that match your intuition from the `PARAMS` table in the README's generator script?
7. Compute the correlation matrix of daily returns across all seven tickers (`prices_wide.pct_change().corr()` in pandas). Which pair is most correlated? Least?
8. Using `fundamental_snapshot`, compute the average EV/EBITDA across the universe.
9. How many tickers trade above that average (i.e., are "expensive" relative to the group)?
10. Compute 12-1 momentum (Lecture 1) for every ticker as of the single most recent date with enough history. Which ticker currently ranks best?
11. Compute 5-day reversal for every ticker as of the most recent date. Which ticker just had the sharpest 5-day drop?
12. Is the ticker with the best momentum (Q10) the same as the ticker with the cheapest value rank (Task 4 of Exercise 1)? If not, which factor would you trust more for a 3-month holding period, and why?
13. Build the composite rank (Exercise 1 Task 6) for the most recent rebalance date and list the top-3.
14. How many distinct months have at least 252 trading days of prior history available (i.e., how many months can 12-1 momentum actually be computed for)?
15. What is the minimum, maximum, and average daily trading volume across all seven tickers over the full dataset?
16. Using `daily_prices`, find the single worst one-day return (largest daily decline) for any ticker on any date, and which ticker/date it was.
17. Using the weight-matrix logic from Lecture 2, what does the *turnover* (sum of absolute weight changes) look like on a rebalance date where the top-3 list is completely unchanged from the prior month? What should it be, numerically, and why?
18. If one-way transaction cost is 8 bps and a $50,000 portfolio fully turns over (sells everything, buys entirely new names) at one rebalance, what is the total dollar cost of that single rebalance (both legs)?
19. Annualize a daily return standard deviation of `0.014` (1.4%) to a yearly volatility figure. Show the formula and the number.
20. If a strategy's annualized volatility is 22% and its Sharpe ratio is 0.8 with a 4% risk-free rate, what is its CAGR? (Rearrange the Sharpe formula and solve.)

---

## Problem 2 — Explain the cost drag (45 min)

In `cost-writeup.md`, answer in prose (no more than 400 words total):

1. Run your Exercise 2 backtest at `cost_bps = 0`, `cost_bps = 10`, and `cost_bps = 30`. Report the final equity curve value at each.
2. In your own words, explain *why* the gap between the `0` and `30` bps results is larger than a naive "30 bps costs 0.3% per trade" intuition would suggest, given monthly rebalancing over two years.
3. Describe one realistic change to the strategy (fewer rebalances, larger position sizes, a longer holding period) that would reduce cost drag, and estimate — roughly, with arithmetic, not just a guess — how much it would help.
4. If you were advising a friend who wanted to run a version of this strategy with real money in a small brokerage account, what cost assumption would you tell them to use, and why might it be higher than this week's 10 bps default?

---

## Problem 3 — Momentum vs. value vs. reversal, head to head (60 min)

Build three separate single-factor strategies — pure momentum top-3, pure value top-3, pure 5-day-reversal top-3 — each with identical rebalancing (monthly), sizing (equal weight), and cost assumptions (10 bps) as the exercises. Run all three through your Exercise 2 backtest engine.

Deliver `factor-comparison.md` with:

- A table: strategy, CAGR, annualized vol, Sharpe, max drawdown, hit rate — one row per factor.
- One paragraph: which single factor performed best on this dataset, and — critically — do you trust that result as evidence the factor is genuinely better, or is it more likely an artifact of the specific two-year synthetic sample and small universe? Justify your answer using what you learned in Lecture 3.

---

## Problem 4 — A walk-forward sanity check, small scale (45 min)

Using just **one** train/test split (not the full walk-forward machinery from Challenge 1) — train on 2024 data, test on 2025 data — compute the momentum + value composite strategy's Sharpe ratio separately for each half.

Deliver `train-test.md` with the two Sharpe ratios side by side and 150–250 words on what the gap (or lack of one) tells you, referencing Lecture 3's discussion of what a large in-sample-vs-out-of-sample gap usually signals.

---

## Problem 5 — Read and summarize a real backtesting failure (60 min)

Find one real, publicly documented example of a systematic trading strategy or backtested model that performed well in testing and then failed or underperformed once traded live (a well-known quant fund's public post-mortem, a documented "quant quake" event, or a reputable financial-press article analyzing why a strategy stopped working — not a random blog post with no sourcing).

Deliver `real-world-failure.md`, 250–350 words:

1. What was the strategy or fund, in one or two sentences?
2. What went wrong, in the reporting's own account?
3. Which specific concept from Lecture 3 (look-ahead bias, overfitting, survivorship bias, regime change, or a friction the backtest underweighted) best explains the failure, based on what you read? Quote at most one short phrase (under 15 words) from your source, with attribution.
4. What would you have wanted to see in that strategy's original backtest report — using this week's "reading a backtest skeptically" checklist — that might have raised a flag earlier?

---

## Submission

Commit all five problems' files to your portfolio under `c35-week-09/homework/`.
