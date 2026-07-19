# Week 1 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hand calculation, SQL, Python, and a written explanation. Commit each.

All queries and calculations run against the `cash_flows` seed from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Set up both engines (45 min)

Get **both** PostgreSQL and SQLite working, plus your Python environment, so you can feel the difference firsthand and always have a fallback.

1. Install PostgreSQL 16+ and SQLite 3.35+ (see [`resources.md`](./resources.md)).
2. Create `crunch_capital` in Postgres and `crunch_capital.db` in SQLite, and load the seed into **both**.
3. Run `SELECT COUNT(*) FROM cash_flows;` in each — both must return `38`.
4. Confirm Python can connect to both (`SELECT version();` via Postgres, `SELECT sqlite_version();` via SQLite, each pulled through `pandas.read_sql`).

**Deliver** `setup.md` with: the version of each engine and Python/pandas, and one sentence on a connection-string or syntax difference you noticed between the two.

---

## Problem 2 — Twenty time-value drills (90 min)

Mix of hand calculation, SQL, and Python — your choice per question, but do at least 5 by hand first with the arithmetic shown, and confirm the rest with code. Put everything in `drills.md` with the question number, method used, and answer.

1. FV of $2,000 at 5% for 3 years.
2. FV of $2,000 at 5% for 3 years, compounded **monthly** instead of annually — how much bigger is it, and why?
3. PV of $25,000 due in 6 years at a 10% discount rate.
4. PV of $25,000 due in 6 years — recompute at a 20% discount rate. How much value did doubling the rate destroy?
5. At what rate (try values until you're within $50) does $8,000 grow to $12,000 in exactly 4 years?
6. How many years (to the nearest whole year) does it take $10,000 to double at a 9% annual rate? Compare your answer to the Rule of 72's estimate (72 ÷ 9 ≈ 8 years).
7. The New CNC Machine's period-1 cash flow ($55,000), discounted at its 7% hurdle rate.
8. The same $55,000, discounted at the European Market Entry project's 18% hurdle rate instead — how much smaller is the present value, purely from the rate change?
9. The Solar Rooftop Retrofit's total raw (undiscounted) cash inflow, summed across all periods.
10. The same project's NPV, properly discounted at its 6% hurdle rate — how much did discounting "cost" the raw total?
11. Effective annual rate of a 10% nominal rate compounded quarterly.
12. Effective annual rate of that same 10% nominal rate compounded monthly — is it higher or lower than the quarterly version, and does that match your intuition about compounding frequency?
13. PV of the AI Quality-Control R&D project's single largest cash flow, at its own hurdle rate.
14. Which single seed project has the highest raw (undiscounted) total inflow? *(SQL.)*
15. Which single seed project has the highest NPV once properly discounted? *(SQL + the discount formula, or your Exercise 3 functions.)* Is it the same project as #14?
16. Sum of every capex outflow across all 6 projects — the company's total period-0 cash commitment if it funded everything at once.
17. FV, at 8%, of that total period-0 commitment (#16), 5 years forward — what it would be worth if simply invested instead of spent on these projects.
18. PV of a hypothetical $200,000 cash flow arriving in year 10, at a 12% discount rate.
19. Same $200,000 in year 10 — how much does the present value change if the timing slips to year 12 instead, same 12% rate?
20. In one sentence: which single lever — rate or time — had a bigger effect on present value across today's drills, and does that match Lecture 2's discount-factor table?

---

## Problem 3 — Explain discounting to a non-finance colleague (45 min)

In `explain-writeup.md`, write no more than 400 words, in plain language (imagine a colleague from product or engineering who's never seen a discount formula):

1. Why is $60,000 four years from now worth less than $60,000 today, even if you're 100% certain you'll get it? (This is the pure time-value effect, separate from risk — be precise about the distinction.)
2. Using one seed project as your example, explain what its `hurdle_rate` represents and why a riskier project gets a higher one.
3. Explain, without using the word "exponent," why doubling the number of years doesn't simply double the discounting effect.
4. If your colleague asked "so is a spreadsheet or your SQL/pandas setup better for this?" — answer in two sentences, tying back to Lecture 3.

---

## Problem 4 — Query practice across engines (45 min)

The same tasks, run on **both** engines, in `cross-engine.sql`:

1. Every row where `flow_type = 'Opex'`, on both Postgres and SQLite — confirm identical results.
2. A query using `POWER()` to compute the discount factor for the row with `cash_flow_id = 14` — run it on both engines and confirm the same number.
3. **Postgres only:** the same discount-factor query, rewritten using the `^` exponent operator instead of `POWER()`. Confirm the result matches.
4. Note in a comment: does SQLite support the `^` operator the way Postgres does? *(Check the docs if unsure — don't guess.)*

---

## Problem 5 — Extend the engine with a real-world project (60 min)

Pick a real capital decision you actually understand — a personal one is fine (buying vs. leasing a car, a home renovation, paying down debt early vs. investing) or a business one you've read about.

1. Model it as a cash-flow stream: a period-0 outflow, and at least 4 future periods of inflow or savings. Make a defensible guess at a discount rate and state why you chose it (what's the risk/opportunity-cost story?).
2. Write it as `INSERT` statements into a **new** table `my_project_cash_flows` (same shape as `cash_flows`, or a trimmed version — your choice, documented).
3. Run your Exercise 3 (or mini-project) `pv_of_stream` function against it and report the NPV.
4. In 3–4 sentences, say whether the NPV supports doing this project, and name one real-world uncertainty your simple model ignores (e.g., the discount rate might be wrong, a cash flow might not materialize on schedule).

**Deliver** `my-project.sql` and `my-project-writeup.md`.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 90 min |
| 3 | 45 min |
| 4 | 45 min |
| 5 | 60 min |
| **Total** | **~4.75 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
