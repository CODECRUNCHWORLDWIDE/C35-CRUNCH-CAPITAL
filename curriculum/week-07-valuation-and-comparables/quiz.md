# Week 7 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 8. A mix of multiple-choice, short calculation, and "what does this mean" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** What does "unlevered" mean in "unlevered free cash flow," and why does a standard DCF use it instead of cash flow *after* interest expense?

- A) It means the company has no debt at all.
- B) It means the cash flow is measured before any payments to or from lenders or shareholders, so it matches the capital-structure-neutral basis of WACC — mixing a levered cash flow with WACC double-counts the cost of debt.
- C) It means the cash flow ignores taxes entirely.
- D) It's just another name for EBITDA.

---

**Q2.** In this week's forecast, `nwc_pct_incremental_revenue` (15%) is multiplied by which quantity to get the change in net working capital for a given year?

- A) That year's total revenue.
- B) That year's total EBITDA.
- C) The dollar *increase* in revenue over the prior year.
- D) The prior year's unlevered free cash flow.

---

**Q3.** Roughly what share of this week's base-case DCF enterprise value (perpetuity-growth method) came from the terminal value alone, rather than the five explicit forecast years?

- A) About 10%
- B) About 35%
- C) About 55%
- D) About 77%

---

**Q4.** Why is that large a share of total value coming from the terminal value considered normal, not a red flag?

- A) It isn't normal — it means the model has a bug.
- B) A 5-year explicit forecast is a finite window against what is, by construction, an infinite remaining life for the business — the terminal value is standing in for everything beyond year 5.
- C) Terminal values are always fabricated to make the DCF match a target price.
- D) Explicit-period cash flows are never discounted, so they're always small.

---

**Q5.** In the perpetuity-growth terminal value formula `TV = FCF_(n+1) / (WACC − g)`, what happens to `TV` as the terminal growth rate `g` gets closer to `WACC`?

- A) `TV` approaches zero.
- B) `TV` is unaffected by how close `g` gets to `WACC`.
- C) `TV` grows without bound — the denominator shrinks toward zero, which is why terminal growth rates are conventionally kept well below WACC.
- D) `TV` becomes negative automatically whenever `g > 0`.

---

**Q6.** This week's exit-multiple method (8.0× EV/EBITDA) produces a lower enterprise value than the perpetuity-growth method (2.5% terminal growth). When you solve each method for the *other* method's implied parameter, what do you find?

- A) The 8.0× multiple implies a growth rate around 1.36% — lower than the 2.5% explicitly assumed elsewhere — confirming the exit-multiple method is the more conservative one.
- B) Both methods imply exactly the same growth rate, so the EV difference must be a calculation error.
- C) The implied parameters can't be computed from a terminal value; they require refitting the entire five-year forecast.
- D) The 8.0× multiple implies a growth rate of 8.0% — the multiple and the growth rate are numerically the same thing.

---

**Q7.** Why do valuation analysts typically use **EV/EBITDA** rather than **P/E** when comparing companies with meaningfully different amounts of debt?

- A) P/E is illegal to use in professional valuation work.
- B) EBITDA is always a larger number than net income, so the ratio looks better.
- C) P/E is distorted by financing choices (interest expense flows through to earnings), while EV/EBITDA is capital-structure-neutral on both sides of the ratio — EV already reflects total capital, and EBITDA sits above interest expense.
- D) There is no real difference; the two ratios always produce the same ranking of companies.

---

**Q8.** On a small comp set (six companies, in this week's `trading_comps`), why is the **median** multiple generally preferred over the **mean** as the primary summary statistic?

- A) The median is always mathematically larger than the mean.
- B) A single outlier (like `VDT`'s 10.60× EV/EBITDA, well above the next-highest comp) pulls a six-number average further than it pulls the median, which only depends on the two middle-ranked values.
- C) SQL cannot compute a mean on more than five rows.
- D) The mean and median are interchangeable; the choice is arbitrary.

---

**Q9.** This week's precedent-transactions median EV/EBITDA (≈10.97×) is meaningfully higher than the trading-comps median EV/EBITDA (≈8.50×). What is the name for the effect responsible for most of that gap, and what causes it?

- A) It's a data error — the two medians should be identical.
- B) The "control premium" — an acquirer buying 100% of a company (and the ability to change its strategy and extract synergies) typically pays more per dollar of EBITDA than a public-market investor buying a minority stake.
- C) Precedent transactions always use a different accounting standard than trading comps.
- D) It's caused entirely by inflation between the transaction dates and today.

---

**Q10.** What does the EV-to-equity bridge's **net debt** term equal, in the simplest case (no preferred stock or minority interest)?

- A) Total debt alone.
- B) Total debt minus cash and equivalents.
- C) Cash and equivalents minus total debt.
- D) Enterprise value minus market capitalization.

---

**Q11.** Crunch Machine Co.'s DCF (perpetuity growth) produces a per-share value of about $18.70, while the top of the trading-comps range is about $15.53. According to this week's reconciliation lecture, what is the *specific, quantified* way to investigate that gap rather than just noting "they're different models"?

- A) There is no way to investigate it further — one model must simply be wrong.
- B) Convert the trading-comps multiple into an *implied growth rate* using the same Gordon-growth relationship the DCF uses, and compare it directly to the DCF's own explicit terminal-growth assumption.
- C) Always trust the DCF over comps, since it uses more decimal places.
- D) Average the two numbers and stop, without further analysis.

---

**Q12.** According to this week's triangulation process (Lecture 3, Section 3), what is the strongest kind of evidence that a valuation range is landing in the right place?

- A) Every method producing the exact same number.
- B) The highest single method's number, taken alone.
- C) Independently derived methods' ranges (built from different evidence — a company's own forecast versus market pricing) overlapping with each other.
- D) Whichever method was computed first.

---

**Q13.** A colleague asks you for "just one number" for what a company is worth. According to this week's lecture, what's the strongest reason to push back and offer a range with a stated base case instead?

- A) A single number is illegal to report in a valuation memo.
- B) A range takes less time to compute than a point estimate.
- C) A single point number implies a level of precision the underlying analysis — built on forecast assumptions, comp selection, and a chosen discount rate — usually can't actually support.
- D) Ranges are always required by regulation, regardless of context.

---

**Q14.** Name one situation where a DCF is the more trustworthy method, and one situation where trading/transaction comps are the more trustworthy method (per Lecture 2, Section 7).

- A) DCF is always more trustworthy; comps should never be used.
- B) DCF is stronger when no good comp set exists or the whole sector is temporarily mispriced; comps are stronger when a genuinely similar peer set exists and you distrust your own multi-year forecasting ability.
- C) Comps are always more trustworthy; DCF should never be used.
- D) The two methods are only ever used for entirely different industries and never compared to each other.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — unlevered means "before financing effects" on both the cash-flow and discount-rate side; mixing a levered cash flow with WACC double-counts the cost of debt.
2. **C** — the driver is applied to the *change* in revenue over the prior year, not to total revenue — this is the single most common bug in building this week's forecast.
3. **D** — roughly 77% of the perpetuity-growth EV (≈$402.6M of ≈$523.9M) came from the discounted terminal value alone.
4. **B** — a finite explicit forecast window is being compared against an effectively infinite remaining business life; the terminal value is doing the work of representing everything past year 5.
5. **C** — as `g` approaches `WACC`, the denominator `(WACC − g)` shrinks toward zero and `TV` grows without bound, which is exactly why terminal growth rates are kept conservative relative to WACC.
6. **A** — solving the Gordon-growth relationship backward from an 8.0× exit multiple gives an implied growth rate of about 1.36%, noticeably below the 2.5% assumed directly elsewhere — confirming the exit-multiple method is the more conservative terminal-value choice here.
7. **C** — EV/EBITDA is capital-structure-neutral on both sides of the ratio (EV already reflects total capital; EBITDA sits above interest expense), while P/E is distorted by how much debt a company carries.
8. **B** — a median depends only on the middle-ranked value(s) and is far less sensitive to one extreme outlier than a six-number mean is.
9. **B** — the control premium: paying for the ability to control and change a business's strategy (and capture synergies) commands a higher price per dollar of EBITDA than a passive minority stake.
10. **B** — net debt = total debt minus cash and equivalents, in the simplest EV-to-equity bridge with no preferred stock or minority interest.
11. **B** — converting a trading multiple into its implied growth rate (via the same TV formula the DCF itself uses) gives a specific, comparable number instead of a vague "different models give different answers."
12. **C** — overlap between ranges built from genuinely independent evidence (a company's own forecast vs. the market's current pricing of peers) is stronger confirmation than agreement between two variants of the same method.
13. **C** — the underlying analysis (forecast assumptions, comp selection, a chosen discount rate) has real uncertainty baked in; a single point number misrepresents how precise the answer actually is.
14. **B** — DCF is stronger with no good comp set or a temporarily mispriced sector; comps are stronger when a genuinely similar peer set exists and multi-year forecasting is unreliable — neither method is universally superior.

</details>

**Scoring:** 12+ → start Week 8. 9–11 → re-read the lecture sections behind your misses (especially Lecture 1, Sections 6–7 if you missed Q3–Q6, or Lecture 3 if you missed Q11–Q13). <9 → re-read all three lectures from the top; this week's mechanics compound directly into Week 10's M&A modeling and Week 12's capstone.
