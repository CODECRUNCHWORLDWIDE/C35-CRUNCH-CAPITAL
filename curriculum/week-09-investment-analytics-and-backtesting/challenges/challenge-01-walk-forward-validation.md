# Challenge 1 — Walk-Forward Validate a Strategy

**Time:** ~90 minutes. **Difficulty:** Hard. **No single right answer.**

## The scenario

Your Exercise 2/3 backtest reported one Sharpe ratio and one max drawdown, computed over the entire 2024–2025 window at once. Lecture 3 warned you that a single full-sample number tells you almost nothing about whether a strategy has a real, repeatable edge or just got lucky on the two years you happened to test. Your job now is to actually build the walk-forward validation Lecture 3 described, run it on the momentum + value composite strategy from Exercise 1, and write an honest verdict on whether it holds up.

## Your task

1. **Build the rolling windows.** Implement `walk_forward_windows()` from Lecture 3 Section 4 (or your own version) with a **6-month train / 3-month test** roll across the full `daily_prices` date range. Print every `(train_start, train_end, test_start, test_end)` tuple you get — you should end up with roughly 4–6 windows, depending on exactly where you start the roll and how you handle the fact that 12-1 momentum itself needs ~12 months of history before it can produce a signal at all.

2. **For each window, "train."** In this challenge, "training" means picking the strategy's parameters using only data through `train_end` — for a first pass, keep it simple: use the exact composite momentum+value, top-3, equal-weight, 10bps-cost strategy from Exercises 1–2 unchanged (the parameters aren't the thing under test yet; the *out-of-sample stability* of the strategy's basic edge is). As a stretch, choose top-N per window based on which of top-2/top-3/top-4 had the better Sharpe ratio in that window's training period, then test the *chosen* N out-of-sample — this is closer to how real walk-forward parameter selection works, and sets up a much more interesting (and more honest) result.

3. **For each window, test.** Run the (possibly per-window-chosen) strategy on the `test_start`–`test_end` slice only, using prices and signals computed **only** from information available by each date in that slice (no peeking forward — this is exactly the discipline from Lecture 3). Record each window's net Sharpe ratio, CAGR, and max drawdown.

4. **Aggregate and report.** Build a small table: one row per window, columns `train_start, train_end, test_start, test_end, test_sharpe, test_cagr, test_max_drawdown`. Compute the **mean** and **standard deviation** of `test_sharpe` across all windows.

5. **Write the verdict.** In `challenge-01.md`, answer explicitly:
   - Is the out-of-sample Sharpe ratio positive, on average, across windows? How does the *average* out-of-sample Sharpe compare to the single full-sample Sharpe you computed in Exercise 3?
   - How much does the Sharpe ratio vary window to window (the standard deviation you computed)? A strategy with a real, stable edge should show *some* consistency; wild swings between strongly positive and strongly negative windows are a warning sign, not proof of nothing (with only 4–6 windows here, don't over-claim either way — say so).
   - If you did the stretch (per-window parameter selection), did the "best in training" top-N choice actually transfer to being the best choice out-of-sample? If it didn't reliably transfer, what does that tell you about how much you should trust in-sample parameter tuning generally?
   - Given everything above, would you trade this exact strategy with real capital? Why or why not — and what, specifically, would you want to see before you would?

## Constraints

- Every window's "test" period must use only data available within that period — verify explicitly that your momentum/value signal computation for a test window's rebalance dates doesn't reach forward into a later window's data. Write one sentence confirming how you checked this.
- Report **all** windows' results, not just the favorable ones. Cherry-picking which windows to show is exactly the sin this challenge exists to prevent.
- If a window has too little history to compute a valid 12-1 momentum signal (early in 2024), skip it and say so explicitly rather than silently producing a wrong number.

## Hints

<details>
<summary>On why 4–6 windows feels uncomfortably small</summary>

It is small — that's honest, not a flaw in your implementation. Two years of data, a 12-month momentum lookback that eats the first year, and a 6-month train / 3-month test roll simply doesn't leave room for many independent windows. A real institutional walk-forward study runs over 10–20+ years specifically to get enough independent test windows to draw a real conclusion. Say this limitation out loud in your write-up — recognizing the limits of your sample size *is* the skill this challenge is testing, not a confession of failure.

</details>

<details>
<summary>On what "the strategy failed walk-forward" would actually look like</summary>

Not necessarily a negative average Sharpe — it could also look like a full-sample Sharpe of, say, 1.4 with an average out-of-sample Sharpe across windows of 0.2 and a standard deviation larger than the mean. That pattern (big in-sample number, weak and noisy out-of-sample number) is the single most common signature of overfitting described in Lecture 3, and it's worth specifically checking for even if neither number is negative.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Window construction | Windows overlap incorrectly or leak future data | Windows are cleanly separated and verified not to peek forward |
| Completeness | Reports only favorable windows | Reports every window, favorable or not |
| Statistical humility | Treats a 5-window average as definitive proof | States the sample size explicitly and calibrates confidence accordingly |
| Comparison | Never compares to the full-sample Exercise 3 number | Explicitly contrasts full-sample vs. walk-forward Sharpe and explains the gap |
| Verdict | No clear recommendation | A specific, defensible "would/wouldn't trade this, because…" |

## Submission

Commit `challenge-01.md` (with your windows table and written verdict) and your code to your portfolio under `c35-week-09/challenge-01/`.
