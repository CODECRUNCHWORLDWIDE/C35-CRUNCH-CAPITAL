# Week 12 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before calling C35 complete. A mix of multiple-choice and short "what does this compute?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In `capstone_deal_snapshot`, the gap between the perpetuity-growth DCF value (≈$524.0M EV) and the exit-multiple DCF value (≈$461.4M EV) should be interpreted as:

- A) A calculation error — the two methods should always agree exactly
- B) Information — it shows the perpetuity method's terminal growth assumption is, at this WACC, more generous than what the market currently pays via the exit multiple
- C) Proof that the exit-multiple method is always more conservative
- D) Irrelevant, since only the higher number matters

---

**Q2.** Why is the terminal value typically the largest single component of a multi-year DCF's enterprise value?

- A) It's a modeling convention with no economic basis
- B) The explicit forecast period is deliberately kept short; the terminal value stands in for "everything after that," which is most of a going concern's life
- C) Terminal value is always computed without discounting
- D) It isn't — the explicit period is always larger

---

**Q3.** A 1-day 95% VaR of $23,200 means:

- A) You will never lose more than $23,200 in a single day
- B) On a typical day, there's a 95% chance your loss is smaller than $23,200 — the other 5% of days can be worse, sometimes much worse
- C) You are guaranteed to lose exactly $23,200 on 5% of days
- D) The position's maximum possible value is $23,200

---

**Q4.** Which statement correctly distinguishes parametric VaR from historical VaR?

- A) They always produce identical answers on any dataset
- B) Parametric VaR assumes a normal distribution and uses a volatility-based formula; historical VaR uses the empirical percentile of actual past returns with no distributional assumption
- C) Historical VaR can only be computed in SQL; parametric VaR can only be computed in Python
- D) Parametric VaR requires more historical data than historical VaR

---

**Q5.** On this week's simulated (geometric Brownian motion) price data, historical VaR and parametric VaR come out close to each other. What should you conclude about real market data?

- A) The same closeness will always hold, because VaR methods are interchangeable
- B) Real market returns typically have fatter tails than a normal distribution, so historical VaR at extreme percentiles is often noticeably worse than parametric VaR predicts
- C) Real market data always makes historical and parametric VaR diverge in the opposite direction
- D) VaR is meaningless on real data

---

**Q6.** Adding an 8%-weighted position in a 28%-volatility asset to a portfolio raised the *portfolio's* volatility from 8.50% to only 8.70%. The single biggest reason this increase was so small is:

- A) The position size was too small to matter at all
- B) The new position's correlation to the existing book (≈0.27) is moderate, not extreme, so much of its standalone risk diversifies away at the portfolio level
- C) Volatility calculations always understate risk
- D) The core book had zero volatility to begin with

---

**Q7.** What is the structural difference between "scenario analysis" and "stress testing," as this week's lectures used the terms?

- A) They are the same technique with different names
- B) Scenario analysis uses named, probability-weighted futures; stress testing combines extreme components deliberately, without assigning a probability, to test survivability
- C) Stress testing is always more optimistic than scenario analysis
- D) Scenario analysis requires Python; stress testing requires SQL

---

**Q8.** `capstone_scenarios` probabilities sum to 1.00 across six named scenarios. Why is a "probability-weighted expected per-share value" (Σ probability × per_share) typically more conservative than the single-point blended DCF/comps target?

- A) It never is — they're always equal
- B) It explicitly incorporates the downside scenarios (Recession, Margin Compression, etc.) that a single base-case point estimate ignores
- C) Probability-weighting always maximizes the value, not averages it
- D) The scenarios table doesn't affect valuation at all

---

**Q9.** In the DCF/comps weighting exercise, why is 60/40 (favoring the DCF) described as a *judgment call* rather than a formula-derived answer?

- A) It isn't a judgment call — 60/40 is the industry-mandated standard
- B) The weighting reflects an analyst's view on how much to trust company-specific driver assumptions (DCF) versus market-implied peer pricing (comps), and reasonable analysts can disagree
- C) The weighting is randomly generated
- D) Comps should always receive 100% weight

---

**Q10.** In Lecture 3's incentive-quantification example, a revenue-only bonus plan paid out in full under the "Margin Compression" scenario despite margin falling 3 percentage points. What does this demonstrate?

- A) Revenue-only bonus plans are illegal
- B) A metric that ignores profitability can reward a year that actually destroyed value, because it can't distinguish "grew profitably" from "grew unprofitably"
- C) Margin never matters for bonus design
- D) The scenario itself was computed incorrectly

---

**Q11.** According to Lecture 3, what makes a claim in a risk memo count as "evidence" rather than just an assertion?

- A) It's stated confidently and in a table
- B) It's attached to a specific, re-runnable query or script that produces the same number when someone else runs it
- C) It's signed off by a manager
- D) It appears in the Recommendation section

---

**Q12.** The governance checklist item "does the audit/oversight cadence match the pace of the business" is important because:

- A) More meetings always prevent fraud
- B) A control that reviews activity only once a year can leave a material problem undetected for most of that year — cadence, not just existence, of a control matters
- C) It has no effect on outcomes
- D) Audit committees should never meet more than once a year

---

**Q13.** Why does this week's capstone deliberately give you `capstone_cash_flows` as an already-derived 5-year FCF stream, rather than raw growth/margin drivers to build from scratch (as Week 7 did)?

- A) Because the capstone doesn't use cash flows at all
- B) Because this week's skill is assembly and defense of an end-to-end case built from pieces you already know how to derive, not re-deriving the drivers from scratch
- C) Because raw drivers can't be discounted
- D) Because Week 7 never covered driver-based forecasting

---

**Q14.** In the "who benefits if you're wrong" governance question from Lecture 3, the question is aimed primarily at:

- A) The target company's competitors
- B) The analyst's own incentives — whether their compensation or performance evaluation is pointed toward getting the analysis right, or toward talking themselves into a position that looks good short-term
- C) The IC's catering budget
- D) Nothing — it's a rhetorical question with no real answer expected

---

**Q15.** A red team recomputes the blended valuation at DCF/comps weightings of 50/50, 40/60, and 0/100, and finds the implied upside falls to exactly zero at a weighting of roughly 15/85. What is the correct interpretation?

- A) The thesis is definitely wrong
- B) You'd need to discount the DCF's credibility down to only about 15% weight (i.e., trust the market-implied comps view almost entirely) before the thesis stops showing any upside at all — a useful, quantified statement of how much conviction the thesis actually requires
- C) The comps value must be recalculated
- D) 15/85 is meaningless without more context

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — a gap between independent valuation methods is information about what's driving the disagreement (here, the perpetuity method's growth assumption vs. the market-implied exit multiple), not an error to be papered over.
2. **B** — the explicit forecast is deliberately short (5 years here); the terminal value represents the remaining, much longer life of the business, so it's structurally the largest piece.
3. **B** — VaR describes a typical bad day at a stated confidence level. It explicitly does not bound the worst possible outcome; the tail beyond the confidence level is, by construction, excluded.
4. **B** — parametric VaR is a formula built on an assumed (normal) distribution and a volatility input; historical VaR reads the empirical percentile directly off actual past returns, with no distributional assumption.
5. **B** — this week's price data was generated by geometric Brownian motion, which is log-normal by construction, so the two VaR methods agree closely. Real markets have fatter tails, so this closeness should not be expected to hold on live data.
6. **B** — correlation, not position size alone, determines how much of a new position's risk shows up at the portfolio level. A moderate correlation (≈0.27) means much of CRNM's standalone volatility diversifies away.
7. **B** — scenario analysis assigns probabilities to named futures and computes an expected value; stress testing deliberately combines extreme, individually-improbable components with no probability attached, to test whether the position survives a plausible worst case.
8. **B** — probability-weighting explicitly folds in the downside scenarios (Recession, Margin Compression, Stagflation, Credit Crunch) that a single base-case point estimate does not represent at all.
9. **B** — the weighting encodes how much you trust company-specific DCF assumptions versus market-implied peer pricing; there's no formula that derives "correct" trust, so it must be stated and defended, not asserted as objective.
10. **B** — a metric that only measures top-line growth cannot distinguish healthy growth from growth achieved at the cost of profitability, so it can reward a genuinely bad year if that year still cleared the growth bar.
11. **B** — Lecture 3's reproducibility standard: a claim becomes evidence when it's attached to a specific, re-runnable artifact that produces the same number for anyone else who runs it.
12. **B** — the risk isn't the existence of a control, it's whether that control's frequency is fast enough to catch a problem before it compounds; an annual-only review can leave months of undetected drift.
13. **B** — this week's point is assembling and defending a case from pieces you already know how to build (Week 7 taught driver-based forecasting); re-deriving drivers from scratch would just repeat Week 7, not test the capstone's actual skill.
14. **B** — the question probes whether the analyst's own incentives point toward accuracy or toward a self-serving narrative, which is a governance risk applied to the analyst, not just the target company.
15. **B** — a break-even weighting is a quantified statement of how fragile the thesis is to your own DCF-vs-market trust assumption; it doesn't declare the thesis wrong, it tells you exactly how much conviction the DCF needs to retain for the thesis to still hold.

</details>

**Scoring:** 13+ → you've completed C35 Crunch Capital. 10–12 → re-read the lecture section behind each miss before calling the course done. <10 → re-read all three lectures from the top; this week deliberately compounds everything before it, so gaps here often trace back to an earlier week's concept worth revisiting.
