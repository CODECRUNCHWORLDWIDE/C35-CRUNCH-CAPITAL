# Week 3 — Quiz

Fourteen questions. Lectures closed. Aim for 12/14 before starting Week 4. A mix of multiple-choice and short "what does this return?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** What is the correct decision rule for a single, independent project using NPV?

- A) Accept if NPV is the highest of all projects under consideration.
- B) Accept if NPV > 0; reject if NPV < 0.
- C) Accept if NPV > IRR.
- D) Accept if NPV equals the initial investment.

---

**Q2.** Why is a project's period-0 cash flow never discounted?

- A) Period-0 cash flows are always positive.
- B) `(1 + r)^0 = 1`, so dividing by it changes nothing — it's already expressed in today's dollars.
- C) Discounting only applies to revenue, not investment.
- D) It's a modeling convention with no mathematical basis.

---

**Q3.** This week's `cash_flows` table gives six different hurdle rates to six different projects. What is the primary justification for that?

- A) Larger projects always get lower hurdle rates.
- B) Hurdle rates should vary with project risk — riskier projects require a higher required return.
- C) It's arbitrary; any single company-wide rate would work equally well.
- D) Hurdle rates are set by whichever department sponsors the project.

---

**Q4.** IRR is formally defined as:

- A) The average annual cash flow divided by the initial investment.
- B) The discount rate that makes a project's NPV exactly zero.
- C) The hurdle rate the company has chosen for the project.
- D) The rate at which the project's raw cash-flow total exceeds its cost.

---

**Q5.** Two mutually exclusive projects: Small has a 55% IRR and costs $8,000. Large has a 22% IRR and costs $600,000. At the company's 9% hurdle rate, which project is virtually certain to have the higher NPV, and why?

- A) Small, because 55% is a much higher percentage return.
- B) Large, because IRR alone can't account for scale, and Large's return is applied to vastly more capital.
- C) They must be equal, since both clear the hurdle rate.
- D) Neither — you cannot compare projects of different sizes at all.

---

**Q6.** What is a "crossover rate" in the context of two mutually exclusive projects with different cash-flow timing?

- A) The rate at which both projects' IRRs become equal.
- B) The discount rate at which the two projects' NPVs are equal; the ranking by NPV flips on either side of it.
- C) The company's cost of debt.
- D) The rate at which a project's payback period equals its discounted payback period.

---

**Q7.** A cash-flow stream is: −100, +230, −132 (three periods). How many sign changes does this stream have, and what does that imply about its IRR?

- A) One sign change; exactly one IRR is guaranteed.
- B) Two sign changes; the stream may have more than one IRR (and in fact does: 10% and 20%).
- C) Zero sign changes; there is no IRR.
- D) Three sign changes; the IRR is undefined by definition.

---

**Q8.** What does MIRR fix that plain IRR does not?

- A) MIRR removes the need to specify a hurdle rate at all.
- B) MIRR lets you specify a realistic reinvestment rate for interim cash flows instead of implicitly assuming reinvestment at the IRR itself, and as a result cannot have multiple roots.
- C) MIRR is only used for projects with negative NPV.
- D) MIRR converts NPV into a percentage with no other change.

---

**Q9.** For this week's New CNC Machine project (NPV +$71,712.84 at a 7% hurdle rate, IRR 20.17%), MIRR (computed at 7%/7%) came out to 14.42% — noticeably *below* IRR. What does that gap tell you?

- A) The project actually has negative NPV.
- B) Standard IRR overstated the project's realistic annualized return by assuming interim cash flows get reinvested at 20.17%, an unrealistically high rate; MIRR corrects that to the achievable 7%.
- C) MIRR is simply a rounding error and should be ignored.
- D) The company should use a 14.42% hurdle rate instead of 7%.

---

**Q10.** What does the Profitability Index (PI) measure that NPV alone does not?

- A) The percentage annual return of the project.
- B) Value created per dollar of capital invested — an efficiency measure, not an absolute-dollar measure.
- C) The number of years until the investment is recouped.
- D) The project's risk tier.

---

**Q11.** Under a fixed, limited capital budget, with multiple independent positive-NPV projects competing for the same dollars, which ranking approach tends to maximize total value created?

- A) Rank by raw NPV, fund top-down until the budget runs out.
- B) Rank by IRR, fund top-down until the budget runs out.
- C) Rank by Profitability Index (PI), fund top-down until the budget runs out (the Lorie-Savage rule) — with a brute-force check when the project count is small and the budget is "lumpy."
- D) Fund the project with the shortest payback period first, regardless of NPV.

---

**Q12.** In this week's `cash_flows` table, only two of six projects have positive NPV at their own hurdle rate. What is the correct response to a project with negative NPV, regardless of how large the available capital budget is?

- A) Fund it anyway if the budget allows, since a larger budget makes even weak projects worth doing.
- B) Reject it — no amount of available capital makes a value-destroying project (NPV < 0) a good use of that capital.
- C) Fund it only if its IRR is positive, even if less than the hurdle rate.
- D) Fund it if its raw, undiscounted cash-flow total is positive.

---

**Q13.** What does simple (undiscounted) payback period ignore that discounted payback period corrects for?

- A) The sign of each cash flow.
- B) The time value of money — simple payback treats a dollar recovered in year 1 identically to a dollar recovered in year 4.
- C) The size of the initial investment.
- D) Whether the project is mutually exclusive or independent.

---

**Q14.** A project's discounted payback period never resolves within its stated lifespan (it returns `inf` under the standard payback-period algorithm). What does that tell you about the project's NPV?

- A) Nothing — payback and NPV are unrelated.
- B) The project's NPV is negative — if the discounted cumulative cash flow never turns positive within the project's life, the full discounted sum (NPV) can't be positive either.
- C) The project's NPV must be exactly zero.
- D) The project's IRR must be higher than its hurdle rate.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — accept if NPV > 0, reject if NPV < 0. For mutually exclusive projects the rule extends to "pick the higher NPV among those that clear zero"; for independent projects, accept every one that clears zero.
2. **B** — `(1+r)^0 = 1` by definition, so a period-0 cash flow is already expressed in today's dollars; discounting it further would be a bug.
3. **B** — risk. A dollar of uncertain future cash flow is worth less than a dollar of certain future cash flow, so riskier projects need a higher hurdle rate to be worth doing. This week's table ranges from 6% (Low risk) to 18% (High risk).
4. **B** — IRR is the discount rate that zeroes out NPV. There's usually no closed-form solution; you find it numerically.
5. **B** — this is the scale trap. IRR is a percentage and can't see that Large's lower percentage return is being earned on roughly 75x the capital; at almost any realistic hurdle rate, Large's absolute-dollar NPV dwarfs Small's.
6. **B** — the crossover rate is where the two NPV profiles intersect; the higher-IRR project is not automatically the higher-NPV project on both sides of that rate.
7. **B** — two sign changes (− to +, then + to −) means Descartes' rule of signs allows up to two real roots, and this exact stream has both: r = 10% and r = 20% each zero its NPV.
8. **B** — MIRR lets you set an explicit, realistic reinvestment rate (instead of implicitly assuming reinvestment at the IRR itself) and moves all cash flows to just two points in time, which also eliminates the multiple-root problem by construction.
9. **B** — the large IRR-minus-MIRR gap for a strongly positive-NPV project happens because standard IRR assumes interim cash gets reinvested at the (high) IRR itself; MIRR corrects that to a defensible rate (here, the 7% hurdle rate), which is why MIRR comes in lower.
10. **B** — PI expresses value created per dollar invested (`1 + NPV / |investment|`), which is exactly the information you need when capital, not project count, is the binding constraint.
11. **C** — ranking by PI and funding top-down (the Lorie-Savage rule) tends to maximize total value under a fixed budget, because it explicitly optimizes value per unit of the scarce resource. It's a heuristic, not a guarantee, when the budget doesn't divide evenly among the top projects — hence the brute-force check for small slates.
12. **B** — a negative-NPV project destroys value regardless of budget size; screening on NPV happens before, and independently of, any budget-ranking step.
13. **B** — simple payback ignores the time value of money entirely; discounted payback applies the same walk to the discounted cash flows, which is why it's always longer than or equal to simple payback for a normal project.
14. **B** — if the cumulative *discounted* cash flow never turns non-negative within the project's own life, the full sum of all discounted cash flows (which is exactly what NPV is) cannot be positive either — a discounted payback of `inf` is a direct signal of negative NPV.

</details>

**Scoring:** 12+ → start Week 4. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; Week 4 (cost of capital) assumes you can compute and interpret every metric in this quiz on demand.
