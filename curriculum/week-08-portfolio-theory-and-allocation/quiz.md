# Week 8 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 9. A mix of multiple-choice and short numeric/conceptual questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** A portfolio's variance, $\sigma_p^2 = w^T \Sigma w$, is:

- A) Always equal to the weighted average of the individual assets' variances
- B) A quadratic form that includes every pairwise covariance term, weighted by the product of the two assets' weights
- C) Only defined when all correlations are positive
- D) Independent of the covariance matrix

---

**Q2.** Two assets have correlation exactly +1. Combining them in any proportion:

- A) Eliminates all portfolio risk
- B) Produces exactly the weighted average of their individual volatilities — no diversification benefit
- C) Is mathematically undefined
- D) Always reduces risk below either asset's own volatility

---

**Q3.** Going from monthly to annual, covariance scales by:

- A) `√12`
- B) `12`
- C) `12²`
- D) It doesn't scale — covariance is already annual

---

**Q4.** In this week's seed data, `CRNG` (gold) has a correlation with the other five funds that is:

- A) Strongly negative with everything
- B) Close to zero with most of them — a genuine diversifier
- C) Above +0.9 with `CRNQ`
- D) Undefined, because gold has no meaningful variance

---

**Q5.** The global-minimum-variance (GMV) portfolio's weight formula, $w_{\text{GMV}} = \Sigma^{-1}\mathbf{1} / (\mathbf{1}^T \Sigma^{-1}\mathbf{1})$:

- A) Requires knowing each asset's expected return
- B) Depends only on the covariance matrix, not on expected returns at all
- C) Only works if all weights are constrained to be non-negative
- D) Is the same portfolio as the tangency portfolio, always

---

**Q6.** The "two-fund separation theorem" says:

- A) You should never hold more than two assets
- B) Every portfolio on the efficient frontier is a linear combination of any two distinct frontier portfolios
- C) The frontier only exists for exactly two assets
- D) Bonds and stocks must always be held in equal amounts

---

**Q7.** In this week's seed data, the unconstrained GMV portfolio puts a large **negative** weight on `CRNH`. This means:

- A) The formula has an error
- B) The optimizer wants to short `CRNH`, using the proceeds to increase other holdings, because of how `CRNH` interacts with the other assets' risk and correlations
- C) `CRNH`'s expected return must be negative
- D) `CRNH` cannot be included in any portfolio

---

**Q8.** The Sharpe ratio of a portfolio is defined as:

- A) $\mu_p / \sigma_p$
- B) $\sigma_p / \mu_p$
- C) $(\mu_p - r_f) / \sigma_p$
- D) $\mu_p - r_f$

---

**Q9.** Once a risk-free asset is available, the efficient frontier becomes:

- A) Identical to the risky-only frontier — nothing changes
- B) A straight line from $(0, r_f)$ tangent to the risky-asset frontier at the tangency portfolio
- C) A vertical line at the tangency portfolio's volatility
- D) Undefined

---

**Q10.** The slope of the capital market line equals:

- A) The risk-free rate
- B) The GMV portfolio's volatility
- C) The tangency portfolio's Sharpe ratio
- D) 1.0, always

---

**Q11.** An asset's beta, $\beta_i = \text{Cov}(r_i, r_m) / \text{Var}(r_m)$, close to zero means:

- A) The asset has zero volatility
- B) The asset's returns show little linear relationship with the market's returns
- C) CAPM cannot be applied to that asset
- D) The asset is risk-free

---

**Q12.** CAPM says the market compensates investors for bearing:

- A) Total risk (systematic + idiosyncratic)
- B) Only idiosyncratic risk
- C) Only systematic (market/beta) risk — idiosyncratic risk is assumed diversifiable and therefore not priced
- D) No risk at all; CAPM prices only liquidity

---

**Q13.** In Challenge 2's crisis stress test, correlations spiking toward a common high value in a crash tends to:

- A) Increase every portfolio's volatility by the same proportional amount, regardless of how diversified it was in calm markets
- B) Hurt diversified portfolios *proportionally more* than a concentrated single-asset position, because their calm-market low risk relied heavily on correlations that are precisely what breaks down in a crisis
- C) Have no effect on portfolios that were already well-diversified
- D) Only affect the expected return, not the volatility, of a portfolio

---

**Q14.** A "factor tilt," as described in Lecture 3, is best described as:

- A) An output the mean-variance optimizer produces automatically
- B) A deliberate, judgment-based deviation from the Sharpe-optimal portfolio, based on a view the historical optimization can't see
- C) Illegal under most investment mandates
- D) Only applicable to individual stocks, never to funds

---

**Q15.** You add a `w_i ≥ 0` (no-short) constraint to the mean-variance optimization. What happens to the closed-form solution from Lecture 2?

- A) Nothing changes — the closed-form formulas already respect the constraint
- B) The closed-form formulas no longer apply directly; you need a numerical solver like `scipy.optimize.minimize` with inequality constraints
- C) The problem becomes unsolvable
- D) The GMV and tangency portfolios become identical

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — portfolio variance is the full quadratic form $\sum_i \sum_j w_i w_j \text{Cov}(r_i, r_j)$, not a simple weighted average of individual variances.
2. **B** — at correlation +1, the cross term in the two-asset variance formula collapses the result to the straight-line weighted average of the two volatilities; there's no risk reduction from combining them.
3. **B** — covariance (like variance) scales linearly with time, `× 12` monthly-to-annual. Only volatility (a square root of variance) uses the `√12` scaling.
4. **B** — in the seed data, `CRNG` correlates close to zero with the other five funds (roughly 0.0 to 0.2), which is exactly why it earns a meaningful allocation in the tangency portfolio despite a modest expected return.
5. **B** — the GMV formula uses only $\Sigma^{-1}$ and a vector of ones; expected returns ($\mu$) never enter the calculation. GMV cares only about risk, not return.
6. **B** — two-fund separation: given any two distinct portfolios that sit on the efficient frontier, every other frontier portfolio is some linear combination of those two.
7. **B** — a negative weight in an unconstrained mean-variance solution means the optimizer wants to short that asset; it's a mathematically valid output of the unconstrained problem, not an error, though it may not be practically investable (Challenge 1).
8. **C** — Sharpe ratio is excess return over risk-free rate, divided by volatility: $(\mu_p - r_f)/\sigma_p$.
9. **B** — with a risk-free asset available, the efficient set becomes the capital market line: a straight line from $(0, r_f)$ tangent to the risky-only frontier at the tangency portfolio.
10. **C** — the CML's slope is $(\mu_{\text{tan}} - r_f)/\sigma_{\text{tan}}$, which is exactly the tangency portfolio's own Sharpe ratio.
11. **B** — a beta near zero means the asset's historical returns show little linear co-movement with the market benchmark; it says nothing about the asset's own total volatility, which can still be substantial (idiosyncratic risk).
12. **C** — CAPM's central claim: only systematic (beta) risk is priced, because idiosyncratic risk can in principle be diversified away for free, so the market doesn't compensate anyone for bearing it.
13. **B** — this is Challenge 2's core finding: portfolios whose calm-market low risk depended heavily on low/negative correlations (like the GMV portfolio, bond-heavy) tend to see a *larger* proportional volatility increase in a correlation-spike crisis than a single concentrated position does, because the very thing that made them low-risk is what breaks down.
14. **B** — a factor tilt is a deliberate choice to move away from the mathematically Sharpe-optimal portfolio, based on a view (about beta pricing, about a specific risk to avoid or embrace) that a backward-looking optimization can't incorporate on its own.
15. **B** — the closed-form GMV/tangency/frontier formulas solve the *unconstrained* (or equality-constrained-only) problem. Adding inequality constraints like `w_i ≥ 0` requires a numerical solver such as `scipy.optimize.minimize` with `SLSQP` or a similar method.

</details>

**Scoring:** 12+ → start Week 9. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; this week's matrix machinery compounds directly into Week 9's backtesting.
