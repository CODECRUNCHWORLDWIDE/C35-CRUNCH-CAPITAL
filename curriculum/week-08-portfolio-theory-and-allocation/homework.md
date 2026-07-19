# Week 8 — Homework

Extra practice, spread across the week (~5 hours total). Four parts: conceptual short answers, extended pandas/numpy work against the Crunch Capital Advisors universe, a second, smaller universe for contrast, and a short writing task. Do these after the exercises — they lean on the covariance matrix and optimizer machinery you've already built.

---

## Part A — Conceptual short answers (~1 hour)

Answer each in 2–4 sentences. No code required.

1. A colleague says: "Fund X had 20% volatility last year, and Fund Y had 20% volatility too, so a 50/50 mix of them must also have 20% volatility." Explain precisely why that reasoning is wrong, and under what specific condition (name it) it would actually be correct.
2. Two assets have a correlation of exactly −1 (perfectly negatively correlated). Explain, using the portfolio variance formula from Lecture 1, why it's possible in principle to construct a *zero-volatility* combination of the two, and what fraction of each you'd need (in terms of their individual volatilities, not a specific number).
3. Why does the global-minimum-variance portfolio not depend on the assets' expected returns at all — only on the covariance matrix? Point to the specific place in the GMV formula where expected return simply doesn't appear.
4. Explain, in plain language, why the tangency (max-Sharpe) portfolio's *composition* can become unstable or extreme as the assumed risk-free rate gets close to the GMV portfolio's own expected return (you saw this directly in Exercise 3, Task 6).
5. A friend says "diversification is a free lunch — you get lower risk for the same return, no cost at all." Using what you learned in Challenge 2, name one honest caveat to that claim.

## Part B — Extended pandas/numpy work against Crunch Capital Advisors (~2 hours)

Work against this week's seeded six-fund universe. Write each as Python in `homework.py`.

6. **Rolling volatility.** Compute `CRNQ`'s **rolling 12-month annualized volatility** (`.rolling(12).std() * sqrt(12)`) across the full return series. Print the resulting series (it will have `NaN` for the first 11 months). In which 12-month window was `CRNQ`'s rolling volatility highest? Lowest?
7. **Rolling correlation.** Compute the **rolling 12-month correlation** between `CRNQ` and `CRNB` (`.rolling(12).corr()`). Does it stay reasonably stable across the five years, or does it swing meaningfully? Report the minimum and maximum values you find, with their approximate dates.
8. **A three-asset sub-universe.** Restrict to just `CRNQ`, `CRNB`, and `CRNG` (drop the other three funds). Recompute the GMV and tangency portfolios for this smaller universe only. How do the resulting weights and Sharpe ratio compare to the full six-fund solutions from Exercise 3? Is the three-asset tangency Sharpe ratio higher or lower than the six-asset one — and does that match your intuition about what happens to the achievable frontier when you *remove* assets from the optimization?
9. **Marginal contribution to risk.** For the six-fund tangency portfolio (from Exercise 3), compute each asset's **marginal contribution to portfolio risk**: `MCR_i = (Σw)_i / σ_p` (the i-th entry of `Sigma @ w`, divided by portfolio volatility), then each asset's **percentage contribution to risk**: `w_i × MCR_i / σ_p`. Print both for all six funds, and confirm the percentage contributions sum to 100%. Which single fund contributes the *most* to the tangency portfolio's total risk, and is that the fund with the largest *weight*, or a different one? (These two don't have to be the same fund — that's the point of this exercise.)
10. **Two Sharpe ratios, one comparison.** Compute the Sharpe ratio of the tangency portfolio using the **realized 5-year sample mean** for `mu` (what you've done all week) versus using the **CAPM-implied expected returns** from Lecture 3 (Section 3) in place of the sample means, keeping the same `Sigma`. Report both Sharpe ratios. Which is higher, and does that make sense given what Lecture 3 found about realized returns sitting above CAPM-required returns across the board this period?

## Part C — A smaller universe for contrast (~1.5 hours)

Here is monthly price data for a two-asset universe over the same window — a fictional pure large-cap tech ETF (**CRTQ**) and a fictional short-term Treasury fund (**CRST**), sampled at year-ends only (dollars per share):

| Date | CRTQ | CRST |
|---|---:|---:|
| 2020-12-31 | 100.00 | 50.00 |
| 2021-12-31 | 128.40 | 50.85 |
| 2022-12-31 | 96.30 | 51.53 |
| 2023-12-31 | 141.75 | 53.35 |
| 2024-12-31 | 179.42 | 54.98 |
| 2025-12-31 | 165.87 | 56.85 |

11. Load these two tickers into your own small table (or reuse `real_asset_prices` if you already made it in the mini-project — use a fresh, clearly named table if not), compute **annual** returns (five yearly returns from six year-end prices — annual data, not monthly, so no `× 12` annualization needed on the mean, and no `× √12` on the volatility; the numbers you compute *are* already annual), the covariance, and the correlation between `CRTQ` and `CRST`.
12. Using that covariance, compute the GMV weights for this two-asset universe by hand (you can use the same closed-form formula, just with `n=2`) and report the resulting portfolio's expected return and volatility. Given only five annual observations, how much would you trust this GMV estimate compared to the six-fund, 60-month estimate from the rest of this week? Explain your answer in terms of sample size.

## Part D — Short writing (~30 minutes)

13. In 150–250 words, explain to someone with no finance background what "you can't diversify away all your risk" means, and why. Use the systematic-vs-idiosyncratic decomposition from Lecture 3 as your concrete example — walk through what happens to a portfolio's risk as you add more and more assets, and explain what risk remains no matter how many you add.

---

## Submission

Commit `homework.py` (Parts B and C's code) and `writeup.md` (Parts A, C.12, and D's written answers) to your portfolio under `c35-week-08/homework/`.
