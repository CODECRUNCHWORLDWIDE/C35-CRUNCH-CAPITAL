# Challenge 2 — Stress Diversification in a Crash

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **The finding matters more than the exact numbers.**

## The scenario

Every calculation this week — the covariance matrix, the frontier, the GMV and tangency portfolios — was built from five years of *relatively calm* historical data. There's a well-documented and uncomfortable fact about real markets that this kind of calm-period optimization quietly ignores: **correlations are not stable. In a genuine crisis, assets that looked weakly or negatively correlated in normal times tend to move together far more than their historical correlation would predict** — the "correlations go to 1 in a crash" phenomenon that caught a lot of supposedly well-diversified portfolios off guard in 2008 and again in March 2020. Volatilities spike too, not just correlations.

Your job: take the portfolios you already built this week, subject them to a stressed covariance matrix that reflects a crash scenario, and find out how much of their calm-market diversification benefit actually survives.

## Your task

Work in `challenge-02.py`, with your written interpretation in `challenge-02.md`.

### Part A — Build a stressed covariance matrix

1. Start from your calm-market volatilities (Exercise 1) and correlation matrix (Exercise 1). Build a **crisis volatility vector** by scaling each asset's volatility up — riskier, more equity-like assets should spike more than defensive ones. A reasonable starting assumption (state your own if you prefer): equities and REITs roughly **1.5–1.6×** their calm volatility, high-yield bonds roughly **1.5×**, core bonds roughly **1.3×** (bonds are not immune to a crisis, especially one driven by rate shocks or a liquidity crunch), gold roughly **1.1–1.2×** (a traditional, if imperfect, safe haven).

2. Build a **crisis correlation matrix** by blending each calm off-diagonal correlation toward a high "crisis" target (try `0.85`) — but *not uniformly*: blend gold's correlations with everything else much less aggressively than the other pairs (say, 20% of the way to the crisis target, versus 70% for every other pair) to reflect gold's reputation as a partial, imperfect diversifier even under stress. State your blend weights explicitly.

3. Assemble the crisis covariance matrix: `Sigma_crisis = outer(vol_crisis, vol_crisis) * corr_crisis` (with the diagonal of `corr_crisis` reset to exactly 1.0). Print it.

### Part B — Re-evaluate your existing portfolios under stress

**Do not re-optimize.** The whole point is to ask: what happens to the portfolios you already built for calm markets, if a crisis hits *before you get a chance to rebalance*?

4. For each of the GMV portfolio, the tangency portfolio, and equal-weight (all from Exercise 3 — same weights, unchanged), compute portfolio volatility under **both** the calm covariance matrix and your new crisis covariance matrix. Report both, and the ratio (crisis vol ÷ calm vol) for each portfolio.

5. For comparison, compute the same calm-vs-crisis vol ratio for a **single-asset** position (100% in `CRNQ` alone, no diversification at all). Compare this single-asset ratio to your three portfolios' ratios from Task 4.

6. **The core question**: which portfolio's volatility *increases by the largest proportional multiple* when the crisis hits — and is it the portfolio you'd have guessed? Connect your answer explicitly to *why* that portfolio was low-risk in calm markets in the first place (hint: look back at which assets and correlations it was leaning on most heavily).

### Part C — Quantify the diversification benefit, calm vs. crisis

7. For each portfolio, compute the **"diversification benefit"** two ways: (a) calm markets — the difference between the naive weighted-average of the individual assets' calm volatilities and the portfolio's actual calm volatility; (b) crisis — the same comparison, but using crisis volatilities and the crisis portfolio volatility. Report both, in percentage-point terms, for all three portfolios.

8. Convert Task 7's numbers into a **percentage of risk removed** (diversification benefit ÷ naive weighted-average volatility) for both regimes. Does the *percentage* of risk that diversification removes go up, down, or stay about the same, moving from calm markets to your crisis scenario? State the number for at least the GMV portfolio.

## Constraints

- Every volatility and correlation number must come from your actual computed calm-market Exercise 1 outputs — don't invent the calm-market baseline, only the crisis assumptions.
- State every crisis assumption explicitly (vol multipliers, correlation blend weights) in `challenge-02.md` — there's no single "correct" crisis scenario, but a defensible one needs its assumptions on the table.
- All matrix construction and every computation happens in numpy/pandas.

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Crisis assumptions | Arbitrary, unexplained multipliers | Stated, defensible multipliers reflecting real asset-class behavior (bonds/gold spike less than equities; correlations converge but gold lags) |
| Mechanics | Portfolios re-optimized under stress (misses the point) | Same calm-market weights held fixed, only the risk *measurement* changes |
| The core finding (Task 6) | Just reports three ratios with no comparison | Explicitly identifies which portfolio suffers the largest proportional vol increase, and explains *why*, tied back to that portfolio's specific reliance on low/negative correlations |
| Part C interpretation | Numbers reported without a stated conclusion | A clear answer to "does diversification's protective *percentage* hold up in a crash," with the actual number |

## Why this challenge exists

Mean-variance optimization, exactly as taught this week, is silent about *regime*. It hands you one "optimal" answer built from one covariance matrix, estimated from one historical window, and says nothing about whether that covariance matrix will still describe the world when it matters most. The uncomfortable, well-documented pattern in real markets is that diversification benefits are often smallest exactly when investors need them most — during systemic crises, when nearly everything falls together. A portfolio manager who only ever looks at a calm-period optimization and never asks "what does this look like in a crash" is missing exactly the risk this challenge makes you quantify.

## Submission

Commit `challenge-02.md` and `challenge-02.py` to your portfolio under `c35-week-08/challenge-02/`.
