# Week 1 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 2. A mix of multiple-choice, short calculation, and "what does this return?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** According to Lecture 1, corporate finance is best understood as optimizing which three things?

- A) Revenue, profit, and market share
- B) Cash, time, and risk
- C) Assets, liabilities, and equity
- D) Growth, margin, and headcount

---

**Q2.** Why does corporate finance care about **cash flow** rather than accounting **profit** as its primary unit of analysis?

- A) Profit is illegal to report publicly.
- B) A company can be profitable on paper and still be unable to pay its bills if the cash isn't there when needed.
- C) Cash flow is always a bigger number than profit.
- D) They are always identical, so it doesn't matter which one you use.

---

**Q3.** A dollar today is worth more than a dollar in a year for two independent reasons. Name both.

- A) Inflation and taxes
- B) Opportunity cost and time preference
- C) Interest rates and exchange rates
- D) Risk and inflation only

---

**Q4.** In the seed `cash_flows` table, the **European Market Entry** project has a hurdle rate of 18% while the **Solar Rooftop Retrofit** has a hurdle rate of 6%. What best explains that 12-point gap, for the same company?

- A) It's arbitrary — hurdle rates are set randomly.
- B) The Europe project's cash flows are far less certain (new market, new regulation), so a higher required return compensates for that added risk.
- C) The solar project is smaller in dollar terms, so it automatically gets a lower rate.
- D) Hurdle rates only depend on how many years the project runs.

---

**Q5.** What is the future value of $2,000 invested today at 5% annual interest, after 2 years (compounded annually)?

- A) $2,100.00
- B) $2,200.00
- C) $2,205.00
- D) $2,210.00

---

**Q6.** What is the present value of $50,000 due in 5 years, discounted at 8% annually? *(Round to the nearest whole dollar.)*

- A) $50,000
- B) $40,000
- C) $34,029
- D) $29,600

---

**Q7.** Why is a cash flow occurring at **period 0** never discounted?

- A) Period-0 cash flows are always negative, and negative numbers can't be discounted.
- B) `(1 + r)^0 = 1`, so dividing by it leaves the amount unchanged — it's already expressed in today's dollars.
- C) SQL and Python can't compute `POWER(x, 0)`.
- D) Period 0 is a rounding convention with no real meaning.

---

**Q8.** Holding the future cash flow and the number of periods constant, what happens to present value as the discount rate **increases**?

- A) Present value increases.
- B) Present value decreases.
- C) Present value is unaffected by the discount rate.
- D) Present value becomes negative.

---

**Q9.** A loan is quoted at a 12% **nominal** annual rate, compounded monthly. What is true about its **effective** annual rate (EAR)?

- A) EAR is exactly 12%, since nominal and effective rates are always the same.
- B) EAR is lower than 12%, because monthly compounding reduces the total interest.
- C) EAR is higher than 12%, because interest compounds more frequently than once a year.
- D) EAR cannot be computed without knowing the loan's principal.

---

**Q10.** What does the **net present value (NPV)** of a cash-flow stream represent, mechanically?

- A) The sum of every cash flow's absolute value, ignoring sign.
- B) The single largest cash flow in the stream.
- C) The sum of every cash flow's present value (each individually discounted to today), including the signed period-0 investment.
- D) The average cash flow across all periods.

---

**Q11.** In the seed data, why might a project with the **largest raw, undiscounted total cash flow** (e.g., European Market Entry) NOT necessarily have the **highest NPV**?

- A) NPV calculations ignore large numbers.
- B) Bigger projects are always penalized by the formula regardless of timing or rate.
- C) A higher hurdle rate and cash flows arriving later both shrink present value — a big but risky, slow, or heavily front-loaded-cost project can be outranked by a smaller but faster-paying, lower-risk one.
- D) This can never happen; the biggest raw total is always the highest NPV.

---

**Q12.** In this week's `cash_flows` table, what does a **negative** `amount` represent?

- A) An error in the data.
- B) A cash outflow — money the company pays out (capex or opex).
- C) A cash inflow that was recorded incorrectly.
- D) A project that has been cancelled.

---

**Q13.** According to Lecture 3, what is the core structural problem with a formula copied down many spreadsheet cells, compared to a single SQL query or Python function?

- A) Spreadsheets can't do arithmetic correctly.
- B) There is no way to guarantee every copy of the formula stays identical — one cell can silently diverge with no visible warning, while a query or function applies the same logic everywhere, every time, by construction.
- C) Spreadsheets are always slower to compute than a database.
- D) Excel doesn't support the `POWER` function.

---

**Q14.** According to Lecture 3, which of the following is named as a **legitimate, honest** use case for a spreadsheet, even in this course's SQL/Python-first world?

- A) Storing a company's entire multi-year cap table.
- B) A true one-off, throwaway back-of-envelope estimate that nobody will need to reproduce or audit.
- C) Running a production loan amortization schedule that Finance queries every quarter.
- D) Modeling a portfolio of thousands of positions.

---

**Q15.** In pandas, what is the advantage of computing a `discount_factor` column across an entire DataFrame at once (`1 / (1 + df["hurdle_rate"]) ** df["period"]`) instead of looping row by row in Python?

- A) It produces a different, more accurate answer.
- B) It's vectorized — the same operation applies to every row at once, which is both faster and closer to how you'd express the same idea in SQL.
- C) Looping row by row is required in pandas; vectorized operations don't exist.
- D) There is no difference; both approaches are always identical in speed and code length.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — cash, time, and risk. Every technique in this course is one of these three questions asked about a bigger or more complicated thing.
2. **B** — a company can report a profit and still fail if it lacks the cash to pay its bills when they're due. This is why cash flow, not accounting profit, is the unit of analysis.
3. **B** — opportunity cost (money today can start earning a return immediately) and time preference (a weaker, behavioral preference for sooner over later).
4. **B** — the gap reflects risk, not size or duration alone. A newer, less certain market commands a higher required return to compensate for the added uncertainty of actually collecting those cash flows.
5. **C** — `2000 × 1.05^2 = 2000 × 1.1025 = 2205.00`.
6. **C** — `50000 / 1.08^5 = 50000 / 1.469328 ≈ 34,029`.
7. **B** — `(1+r)^0 = 1` for any `r`, so dividing by 1 leaves the value unchanged; the cash flow is already in today's dollars.
8. **B** — present value and discount rate move inversely: a higher rate shrinks the present value of the same future cash flow (see Lecture 2's discount-factor table).
9. **C** — more frequent compounding always produces an effective rate at or above the nominal rate; monthly compounding of a 12% nominal rate gives an EAR of about 12.68%.
10. **C** — NPV is the sum of each individually-discounted cash flow's present value, signs included — not an average, not an absolute-value sum, not just the largest flow.
11. **C** — discounting and timing can reorder a ranking: higher hurdle rates and later-arriving cash flows both shrink present value more than a naive "add up the raw numbers" approach would suggest.
12. **B** — negative `amount` values represent cash outflows (capex, opex); positive values represent inflows (revenue, salvage).
13. **B** — the structural risk is that nothing enforces every copy of a formula staying identical; a single query or function, applied once, cannot silently diverge the way scattered spreadsheet cells can.
14. **B** — a genuine one-off, throwaway estimate nobody will reproduce or audit is the honest, legitimate spreadsheet use case named in Lecture 3; the other three options are exactly the reused/scaled/audited scenarios the lecture argues belong in SQL + Python instead.
15. **B** — vectorized pandas operations apply across the whole column/DataFrame at once without an explicit Python loop, which is faster and mirrors how the same logic reads as a single SQL expression.

</details>

**Scoring:** 13+ → start Week 2. 10–12 → re-read the lecture sections behind your misses (especially Lecture 2's discount-factor table if you missed Q5–Q9). <10 → re-read all three lectures from the top; the discounting mechanics compound directly into Week 3's capital budgeting.
