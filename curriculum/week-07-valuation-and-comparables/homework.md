# Week 7 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hand calculation, SQL, Python, and a written explanation. Commit each.

All queries and calculations run against this week's seed tables (`valuation_assumptions`, `revenue_forecast`, `trading_comps`, `precedent_transactions`) from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Reproduce the base case (45 min)

Before changing anything, prove you can exactly reproduce this week's headline numbers from the raw seed data.

1. Load all four tables and confirm the sanity-check counts (`1`, `5`, `6`, `5`).
2. Reproduce Lecture 1's full five-year forecast table (revenue through unlevered FCF) in either SQL or Python.
3. Reproduce both DCF enterprise values: perpetuity growth (≈$523,893,110) and exit multiple (≈$461,366,567).
4. Reproduce the trading-comps summary statistics: mean EV/EBITDA ≈8.63×, median ≈8.50×.

**Deliver** `reproduction.md` with all four reproduced numbers and, for any that don't match within 0.5%, a note on what you found and fixed.

---

## Problem 2 — Twenty valuation drills (100 min)

Mix of hand calculation, SQL, and Python — your choice per question, but do at least 6 by hand with the arithmetic shown, and confirm the rest with code. Put everything in `drills.md` with the question number, method used, and answer.

1. By hand: FY2028 revenue, compounding FY2027's computed revenue ($242,676,000) forward by FY2028's 6% growth rate.
2. FY2028 EBITDA, using FY2028's 22.5% margin.
3. FY2028's full unlevered FCF calculation, all the way through (D&A, EBIT, tax, NOPAT, capex, ΔNWC).
4. The discount factor at WACC=9.2% for `t=3`.
5. The present value of FY2028's unlevered FCF, using your Task 4 discount factor.
6. Recompute the `t=3` discount factor at a hypothetical WACC of 10.0% instead of 9.2%. How much smaller is it?
7. Recompute the perpetuity-growth terminal value using `g=2.0%` instead of 2.5% (WACC unchanged at 9.2%, FY2030 UFCF unchanged).
8. Discount your Task 7 terminal value back to today (`t=5`).
9. The resulting full enterprise value (explicit-period PV unchanged at ≈$121,271,769, plus your Task 8 terminal value).
10. Compare your Task 9 answer to the base case's $523,893,110 — how much smaller, in dollars and as a percentage?
11. By hand: Meridian Motion Systems' (`MMS`) market capitalization, from its share price and shares outstanding in `trading_comps`.
12. `MMS`'s enterprise value (add debt, subtract cash).
13. `MMS`'s EV/Revenue and EV/EBITDA multiples.
14. Bantam Tool & Die's EV/Revenue and EV/EBITDA multiples, from `precedent_transactions`.
15. Which single trading comp has the **lowest** implied leverage (`total_debt / enterprise_value`)? *(SQL.)*
16. Crunch Machine Co.'s net debt, from `valuation_assumptions`.
17. The base-case perpetuity-growth equity value and per-share price, using your Task 16 net debt.
18. The trading-comps-low equity value and per-share price (implied EV ≈$362,000,000 from Lecture 2/Exercise 3).
19. The control premium: the percentage gap between the precedent-transactions median EV/EBITDA (10.97×) and the trading-comps median EV/EBITDA (8.50×).
20. In one sentence: based on Challenge 1's sensitivity grid, which single lever — WACC or terminal growth rate — moved enterprise value more over the ranges tested, and does that match how the Gordon growth formula behaves as WACC and `g` get closer together?

---

## Problem 3 — Explain a valuation range to a non-finance colleague (45 min)

In `explain-writeup.md`, write no more than 450 words, in plain language (imagine a colleague from product or engineering who's never built a valuation model):

1. In one paragraph, explain what a DCF is actually doing — without using the word "discount" more than twice, and without writing a single formula.
2. In one paragraph, explain why a trading-comps analysis can produce a *different* answer than a DCF for the same company on the same day, without either one being "wrong."
3. If your colleague asked "so what's the company actually worth — just give me one number," explain in 2–3 sentences why you'd push back and give them a range with a base case instead, tying back to Lecture 3, Section 4.

---

## Problem 4 — Query practice across engines (45 min)

The same tasks, run on **both** engines, in `cross-engine.sql`:

1. The full `trading_comps` EV/multiple query from Lecture 2, Section 3, run on both Postgres and SQLite — confirm identical results.
2. A query computing `median_ev_ebitda` using `PERCENTILE_CONT` on Postgres. Note in a comment what you had to do differently (or in Python) to get the same number on SQLite.
3. **Postgres only:** rewrite the recursive revenue-compounding CTE from Lecture 1, Section 4 using `POWER()` and a plain (non-recursive) join against a generated series of exponents instead — i.e., `revenue_t = ltm_revenue × PRODUCT(1 + growth_rate)` computed a different way. *(Hint: `EXP(SUM(LN(1 + growth_rate)) OVER (...))` is one non-recursive path to a running product.)* Confirm it reproduces the same five revenue figures as the recursive version.
4. Note in a comment: does SQLite support `WITH RECURSIVE`? Since what version?

---

## Problem 5 — Value a company you actually understand (75 min)

Pick a real company (public, so you can find real numbers) that you understand reasonably well — something you've worked at, use as a customer, or follow closely.

1. Find its most recent trailing-twelve-month revenue and EBITDA (or a close proxy) from a source like its investor-relations site or a free data source (see [`resources.md`](./resources.md)).
2. Build a 5-year revenue-forecast driver table for it, in the same shape as this week's `revenue_forecast` — your own defensible guesses at growth, margin, capex, and NWC assumptions, with a sentence justifying each one.
3. Run your Exercise 1/2 forecast-and-discount logic against it (reuse your code — a discount rate of your own reasonable estimate is fine; you don't have a real WACC for it unless you want to redo Week 4's work).
4. Find 3–4 genuinely comparable public companies and compute their EV/EBITDA multiples from real, current data.
5. In 4–5 sentences, compare your DCF's implied value to what the comps suggest, and say whether you think the real company is currently over- or under-valued relative to your own analysis — and how confident you actually are in that conclusion, given how many assumptions went into it.

**Deliver** `real-company-valuation.py` (or `.sql`+`.py`) and `real-company-writeup.md`.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 100 min |
| 3 | 45 min |
| 4 | 45 min |
| 5 | 75 min |
| **Total** | **~5.2 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
