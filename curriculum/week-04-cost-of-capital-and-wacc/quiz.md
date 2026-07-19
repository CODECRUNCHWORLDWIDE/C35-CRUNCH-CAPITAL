# Week 4 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 5. A mix of multiple-choice and short "compute this" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In the CAPM formula $R_e = R_f + \beta(R_m - R_f)$, the term $(R_m - R_f)$ is called the:

- A) Risk-free rate
- B) Equity risk premium
- C) Cost of debt
- D) Beta adjustment

---

**Q2.** Why is the 10-year US Treasury yield the conventional choice for $R_f$?

- A) It's the highest-yielding government bond available.
- B) It's long enough to match typical corporate investment horizons and safe enough that default risk is negligible.
- C) It's set directly by the Federal Reserve and never changes.
- D) It's required by GAAP accounting rules.

---

**Q3.** A stock has $\beta = 0.60$. What does that say about how it has historically moved relative to the market?

- A) It moves in the opposite direction from the market.
- B) It's unrelated to the market entirely.
- C) It moves in the same direction as the market, but more mildly.
- D) It moves in the same direction as the market, but more aggressively.

---

**Q4.** Compute the cost of equity: $R_f = 4.0\%$, $\beta = 1.50$, $ERP = 5.0\%$.

- A) 9.0%
- B) 10.5%
- C) 11.5%
- D) 13.5%

---

**Q5.** In Lecture 2's regression, why must you compute *returns* from the price series before regressing, rather than regressing the raw prices directly?

- A) SQL cannot store price data as a floating-point number.
- B) Prices trend upward over time for reasons unrelated to risk, so a price-on-price regression wouldn't isolate the risk relationship CAPM cares about.
- C) `numpy.polyfit` only accepts values between 0 and 1.
- D) Returns are always normally distributed and prices never are.

---

**Q6.** Crunch Machine Co.'s regression beta (≈1.24) came out lower than its vendor-supplied beta (1.30). What is the correct conclusion?

- A) The regression must contain a bug, since it disagrees with the vendor.
- B) Both are estimates built on different windows/methods; disagreement is expected and neither is automatically "the truth."
- C) The vendor's number should always be discarded in favor of a self-computed one.
- D) Betas below 1.30 are impossible for industrial companies.

---

**Q7.** In this week's regression, R² came out to about 0.58. What does that mean?

- A) 58% of the market's variance is explained by the stock.
- B) The regression has a 58% chance of being wrong.
- C) About 58% of the stock's monthly return variance is explained by market-wide moves; the rest is idiosyncratic.
- D) Beta is equal to 0.58.

---

**Q8.** Write the Hamada unlevering formula from memory (in words or symbols) and explain why more debt raises a company's levered beta even if its underlying business is unchanged.

*(Short answer — no options. Check your answer against the key.)*

---

**Q9.** A peer has $\beta_L = 1.20$, $D/E = 0.50$, $t = 0.25$. Its unlevered beta is closest to:

- A) 0.60
- B) 0.87
- C) 1.20
- D) 1.50

---

**Q10.** A bond trades **below** par (price < 100). Relative to its stated coupon rate, its yield to maturity is:

- A) Always lower than the coupon
- B) Always higher than the coupon
- C) Always exactly equal to the coupon
- D) Unrelated to the coupon

---

**Q11.** Why does WACC use the **after-tax** cost of debt, rather than the raw pre-tax yield?

- A) Because bond yields are always quoted after tax already.
- B) Because interest payments are tax-deductible, so the government effectively subsidizes part of the interest cost — a benefit equity dividends don't receive.
- C) Because after-tax numbers are always smaller and easier to work with.
- D) Because the IRS requires WACC to be computed on an after-tax basis.

---

**Q12.** WACC uses **market-value** weights for $E$ and $D$, not book-value weights, primarily because:

- A) Market values are always smaller than book values, which simplifies the math.
- B) Book value reflects historical accounting figures that can be very different from what the capital is actually worth today.
- C) Regulators require market-value weighting by law.
- D) Book values are not available for public companies.

---

**Q13.** Compute WACC: $E/V = 75\%$, $R_e = 11\%$, $D/V = 25\%$, $R_d = 6\%$ (pre-tax), $t = 20\%$.

- A) 8.25%
- B) 9.45%
- C) 10.05%
- D) 11.00%

---

**Q14.** According to Lecture 3's summary table, which of the following is a WACC input that a company's management **substantially controls**, as opposed to one set almost entirely by the macro environment?

- A) The risk-free rate
- B) The equity risk premium
- C) How much debt the company carries (and, through it, its beta and its $D/V$ weight)
- D) Broad market interest-rate policy

---

**Q15.** A company recomputes its WACC and finds it rose by 40 basis points from last year. Its own leverage, debt structure, and business mix are all unchanged. What's the most likely explanation?

- A) A calculation error, since WACC cannot change without a company-specific decision.
- B) A macro shift — e.g., the risk-free rate rose, or the market's equity risk premium widened — since the company-controlled inputs didn't move.
- C) The company's tax rate must have been eliminated.
- D) WACC only changes once every five years by convention.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — $(R_m - R_f)$ is the equity risk premium: how much extra return investors demand for holding risky stocks over the risk-free asset.
2. **B** — long enough to roughly match multi-year corporate investment horizons, liquid, and safe enough that default risk is negligible. It isn't the highest-yielding bond (A), the Fed doesn't set it directly (C — it's market-determined, though influenced by Fed policy), and it has nothing to do with GAAP (D).
3. **C** — a beta between 0 and 1 means the stock moves with the market but more mildly (dampened), not oppositely (A), unrelated (B), or amplified (D, which would need $\beta > 1$).
4. **C** — $4.0\% + 1.50 \times 5.0\% = 4.0\% + 7.5\% = 11.5\%$.
5. **B** — prices trend up over time from growth, inflation, and compounding, none of which is what CAPM's risk relationship is about; returns strip that trend out and isolate period-to-period co-movement.
6. **B** — the two betas are estimates from different data/methods; disagreement is normal and expected, not a sign either is wrong. Neither should be blindly trusted or blindly discarded (A, C are both overconfident conclusions); nothing about 1.24 vs 1.30 says anything about what's "impossible" (D).
7. **C** — R² is the fraction of the *stock's* return variance explained by the market regressor; the unexplained remainder (42% here) is idiosyncratic, company-specific variance. It says nothing about the market's own variance (A) or about statistical "chance of being wrong" (B) — that's not what R² measures. It's also not beta itself (D).
8. **Sample answer:** $\beta_U = \dfrac{\beta_L}{1+(1-t)(D/E)}$. More debt raises leverage even with an unchanged business because debt holders are paid first and equity absorbs all remaining variability on a *smaller* base of capital — the same operating cash-flow swings translate into *larger* percentage swings for equity as more of the capital structure is fixed-obligation debt instead of residual-claim equity.
9. **B** — $\beta_U = \dfrac{1.20}{1+(1-0.25)(0.50)} = \dfrac{1.20}{1+0.375} = \dfrac{1.20}{1.375} \approx 0.87$.
10. **B** — a bond priced below par means the market is demanding more return than the stated coupon offers, so YTM is above the coupon; the reverse (A) holds for bonds priced above par, and equality (C) only holds exactly at par.
11. **B** — interest is tax-deductible, reducing the company's tax bill — the "interest tax shield" — which is a real, government-subsidized reduction in the effective cost of debt that dividends (paid after tax) don't get. (A), (C), and (D) are all false statements about how yields or tax law work.
12. **B** — book value reflects historical accounting entries (cost basis, retained earnings) that can diverge sharply from what the market is actually willing to pay for the capital today; WACC should reflect current, not historical, capital costs. (A) is not reliably true, and (C)/(D) are fabricated claims.
13. **B** — after-tax cost of debt first: $R_d(1-t) = 6\%\times(1-0.20)=4.8\%$. Then weight both pieces: $WACC = (0.75\times11\%) + (0.25\times4.8\%) = 8.25\% + 1.20\% = 9.45\%$. The most common error here is weighting the *pre-tax* 6% instead of the after-tax 4.8% — that mistake alone lands on option A (8.25% + 1.50% miscomputed) or elsewhere; always apply $(1-t)$ before weighting, not after.
14. **C** — leverage is the lever management most directly controls, and it moves both beta (via financial risk) and the $D/V$ weight simultaneously; the risk-free rate (A), equity risk premium (B), and broad rate policy (D) are all macro-determined, outside any single company's control.
15. **B** — with every company-specific input unchanged, a WACC move has to come from the macro inputs ($R_f$ or $ERP$) that no single company controls; (A), (C), and (D) are all incorrect claims about how or when WACC can change.

</details>

**Scoring:** 12+ → start Week 5. 9–11 → re-read the lecture sections behind your misses, especially anywhere you missed a sign or a $(1-t)$ adjustment. <9 → re-read all three lectures from the top; every later week's discount rate depends on getting this one right.

**A note on Q13:** the key above shows the correct worked arithmetic step by step precisely because this is the single most common place students drop a step (forgetting the tax adjustment, or weighting the wrong side) — if you got it wrong, trace through the two weighted terms individually before moving on.
