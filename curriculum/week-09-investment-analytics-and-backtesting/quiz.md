# Week 9 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 10. A mix of multiple-choice and short "what does this code do / return" questions — the answer key explains the *why*, not just the letter.

---

**Q1.** In this week's framework, what is the precise difference between an investment *thesis* and a *signal*?

- A) They're the same thing, just different words
- B) A thesis is a plain-English belief; a signal is the precise, computable rule the backtest engine actually consumes
- C) A signal only applies to momentum strategies; a thesis applies to all strategies
- D) A thesis is quantitative; a signal is qualitative

---

**Q2.** Classic 12-1 momentum computes the trailing return over approximately 12 months, but deliberately **excludes**:

- A) The oldest month in the window
- B) Weekends and holidays
- C) The most recent month
- D) Any month with a negative return

---

**Q3.** Why does 12-1 momentum exclude that month specifically?

- A) It's a rounding convenience with no real reason
- B) That short-term window is dominated by mean-reversion, a different effect on a different horizon
- C) Data from the most recent month is usually missing
- D) Regulatory rules require it

---

**Q4.** This week's `fundamental_snapshot` table has exactly one `as_of_date` (`2025-12-31`). What is the risk of joining it to `daily_prices` on `ticker` alone, with no date condition?

- A) The join will fail with a SQL error
- B) It silently uses a 2025 fundamentals number for trading decisions in 2024, before that number existed — look-ahead bias
- C) It has no risk; fundamentals don't change often enough to matter
- D) It will return zero rows

---

**Q5.** In the backtest engine's return calculation, `gross_return = (weights.shift(1) * daily_rets).sum(axis=1)`. What does `.shift(1)` specifically prevent?

- A) Negative returns
- B) The portfolio earning today's return using weights that could only be known after today's close
- C) Rounding errors in floating-point math
- D) Duplicate rows in the price table

---

**Q6.** A weight-matrix mask of `(px.index > rdate) & (px.index <= end_date)` means new target weights take effect:

- A) On the rebalance date itself
- B) One day before the rebalance date
- C) Strictly after the rebalance date
- D) Only at month-end, regardless of the rebalance date

---

**Q7.** One-way transaction cost is 6 bps and one-way slippage is 6 bps. A strategy fully turns over its portfolio (sells everything, buys entirely new names) at every one of 12 monthly rebalances in a year. Roughly what fraction of capital does this cost per year?

- A) About 0.12%
- B) About 1.44%
- C) About 2.88%
- D) About 14.4%

---

**Q8.** Which of these is true about slippage versus transaction cost?

- A) They're identical concepts with different names
- B) Transaction cost is a direct fee/spread charge; slippage is the gap between the price a signal was computed on and the price actually achieved at execution
- C) Slippage only applies to short positions
- D) Transaction cost only applies when a strategy loses money

---

**Q9.** Maximum drawdown, computed as `(equity_curve / equity_curve.cummax() - 1).min()`, should always be:

- A) A positive percentage
- B) Exactly zero for a profitable strategy
- C) Less than or equal to zero
- D) Equal to the Sharpe ratio divided by volatility

---

**Q10.** A strategy shows a full-sample (2024–2025) Sharpe ratio of 1.6, but when walk-forward validated, its average out-of-sample Sharpe across rolling windows is 0.1 with a standard deviation larger than the mean. What does this pattern most strongly suggest?

- A) The strategy is even better than the full-sample number implies
- B) The full-sample number was likely inflated by overfitting to that specific sample; the edge doesn't reliably generalize
- C) The walk-forward code has a bug and should be ignored
- D) Nothing — Sharpe ratios are expected to vary this much regardless of overfitting

---

**Q11.** In a proper train/test split, once you've looked at the out-of-sample test result:

- A) You should keep tuning the strategy and re-testing on that same window until it looks good
- B) You may not further tune the strategy and re-test on that same window — doing so turns the test set into a disguised training set
- C) The test result should be discarded entirely
- D) You must immediately publish the result regardless of quality

---

**Q12.** Survivorship bias, in this week's context, specifically refers to:

- A) A stock's price surviving a market crash
- B) A backtest universe selected with hindsight, silently excluding companies that failed or were delisted along the way
- C) A signal that survives multiple parameter changes
- D) A rebalance date that survives a holiday closure

---

**Q13.** When combining a momentum score and an EV/EBITDA value score into one composite signal, Lecture 1 recommends averaging:

- A) The raw momentum and EV/EBITDA numbers directly
- B) Each factor's **rank** within the universe, not the raw values
- C) Only the momentum score, ignoring value entirely
- D) The z-score of trading volume instead

---

**Q14.** Why does averaging raw `momentum_12_1` (e.g., `0.18`) directly with raw `ev_ebitda` (e.g., `18.0`) produce a meaningless composite score?

- A) It doesn't — this is a valid approach
- B) The two numbers live on completely different, incomparable scales, so one factor would dominate the average by pure coincidence of units, not economic importance
- C) SQL cannot add a percentage to a multiple
- D) EV/EBITDA is always negative

---

**Q15.** A backtest reports a stellar net Sharpe ratio of 3.2 on a simple, single-factor, long-only equity strategy tested over two years on seven stocks. What is the most appropriate reaction?

- A) Trade it immediately with maximum capital
- B) Treat it as excellent evidence the factor works and publish it
- C) Be skeptical — a Sharpe that high on that small a sample is far more likely to reflect a bug or overfitting than genuine, durable skill; re-check for look-ahead bias and out-of-sample degradation before trusting it
- D) Sharpe ratios above 3 are common and unremarkable for equity strategies

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — a thesis is the plain-English belief; a signal is what you get after forcing it to answer exactly which securities, on exactly which date, using exactly what data known at that moment. The backtest engine only ever sees the signal.
2. **C** — the most recent month is excluded.
3. **B** — that short window is where mean-reversion, a different and roughly opposite-horizon effect, tends to dominate; mixing it into the momentum window muddies both signals.
4. **B** — joining on `ticker` alone lets a 2024 rebalance decision "see" a fundamentals number computed at the end of 2025 — classic look-ahead bias, and exactly the bug Challenge 2 is built around.
5. **B** — you can only trade tomorrow on information known through today's close; `.shift(1)` ensures the portfolio earns each day's return using the weights decided as of the *prior* day, not weights that would require future knowledge.
6. **C** — strictly after the rebalance date. The rebalance date itself is "decision day"; you didn't yet hold the new weights while that day's return happened.
7. **C** — about 2.88%. One-way cost is 12 bps (6+6); a full turnover both sells and buys, so each rebalance costs `12 bps × 2 = 24 bps`; over 12 rebalances, `24 bps × 12 = 288 bps ≈ 2.88%`.
8. **B** — transaction cost is a direct fee/spread; slippage is the gap between the price the signal was computed on and the price actually achieved, driven by time lag and market impact.
9. **C** — max drawdown is a decline from a prior peak, so it's always ≤ 0 (0 only if the equity curve never fell below any prior high).
10. **B** — a large gap between a strong in-sample number and a weak, noisy out-of-sample number is the textbook signature of overfitting described in Lecture 3, not evidence of a stronger true edge.
11. **B** — further tuning against an already-viewed test window turns it into a second training set, defeating the purpose of holding it out at all.
12. **B** — selecting today's universe with hindsight silently drops companies that went bankrupt or were delisted, making historical strategies (especially momentum) look better than they actually performed.
13. **B** — average each factor's rank within the universe, not the raw values.
14. **B** — the two numbers are on incompatible scales (a fractional return vs. a multiple in the teens/twenties), so a direct average would be dominated by whichever number happens to be numerically larger, not by which factor is economically more important.
15. **C** — extreme Sharpe ratios on small samples and simple strategies should raise suspicion of a bug (most often look-ahead bias) or overfitting before they're treated as a genuine, tradable edge — precisely Lecture 3's central lesson.

</details>

**Scoring:** 13+ → start Week 10. 10–12 → re-read the lecture sections behind your misses, especially Lecture 3. <10 → re-read all three lectures from the top; this week's discipline compounds directly into every remaining backtest you'll ever build.
