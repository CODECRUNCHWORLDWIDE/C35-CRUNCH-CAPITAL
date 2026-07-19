# Week 5 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hand calculation, SQL, Python, and a written explanation. Commit each.

All calculations use `Tc = 0.25`, `rU = 0.10` unless a problem states otherwise, and reference Crunch Machine Co.'s `income_statement` / `balance_sheet` seed from the [README](./README.md).

---

## Problem 1 — Tax shield drills (60 min)

Mix of hand calculation and code — your choice per question, but show at least 4 by hand with the arithmetic visible, and confirm the rest with code. Put everything in `drills.md`.

1. Annual interest tax shield for a company with `interest expense = 2,400,000` and `Tc = 21%` (the US federal statutory rate).
2. Same company, PV of the tax shield assuming permanent debt of `40,000,000` at `Tc = 21%`. *(Note: this doesn't require the interest expense from Q1 at all — why not?)*
3. A company has `D = 15,000,000` of permanent debt and `Tc = 28%` (a higher-tax jurisdiction). PV of its tax shield?
4. Crunch Machine Co.'s FY2022 annual tax shield and perpetual-debt PV (pull `interest_expense` and `total_debt` from the seed tables yourself).
5. Crunch Machine Co.'s FY2023 annual tax shield and perpetual-debt PV.
6. Between FY2022 and FY2023, did the annual tax shield go up or down? Does that match the direction `interest_expense` moved? *(It should — the two are directly proportional.)*
7. A finite 3-year loan of `5,000,000` at `6%`, fully amortizing, level annual payments. Compute the payment (Week 1's annuity formula), then the 3-period amortization schedule.
8. PV of the tax shield over that 3-year schedule, `Tc = 25%`, discounted at `6%`.
9. Compare Q8's answer to the perpetual shortcut `Tc × 5,000,000 = 1,250,000`. By what percentage is the finite-life shield smaller?
10. In one sentence: why does a company that plans to pay off its debt within a few years get a meaningfully smaller tax-shield benefit than the `Tc × D` formula would suggest?

---

## Problem 2 — MM Proposition II, by hand and by code (60 min)

In `mm-prop-ii.md`:

1. A firm has `rU = 12%`, can borrow at `rD = 7%`, no taxes. Compute `rE` at `D/E = 0`, `D/E = 0.5`, `D/E = 1.0`, `D/E = 2.0`. Plot (by hand, a rough sketch is fine, or with `matplotlib`) `rE` against `D/E` — what shape is the line?
2. Same firm, now **with** taxes at `Tc = 25%`. Recompute `rE` at the same four `D/E` levels using the with-taxes formula. Is the line steeper or flatter than Q1's no-tax version? Explain why, referencing the `(1 − Tc)` term.
3. Using Crunch Machine Co.'s actual FY2021 and FY2025 figures (`D`, `E` from the balance sheet; use each year's own **embedded** cost of debt, `interest_expense / total_debt`, as that year's `rD`; `rU = 10%`; `Tc = 25%`), compute `rE` for both years with the with-taxes formula.
4. By how many percentage points did Crunch Machine Co.'s modeled cost of equity rise between 2021 and 2025? What two things changed simultaneously to drive that — name both (one is the `D/E` ratio itself, the other is the embedded `rD`).

---

## Problem 3 — Explain the tradeoff theory to a skeptical board member (45 min)

In `explain-writeup.md`, ≤ 400 words, plain language (imagine a board member who's never seen a derivative):

1. Why doesn't MM's original "capital structure doesn't matter" result hold once you add corporate taxes?
2. Why doesn't the logical conclusion of "debt creates value via the tax shield" mean a company should borrow as much as possible?
3. In one paragraph, using Crunch Machine Co.'s own numbers (no derivatives, no calculus — just the two dollar figures), explain why the company appears to be past the value-maximizing debt level rather than short of it.
4. If the board member asks "so is more debt always bad?" — answer in two sentences, being precise about what the tradeoff model actually implies (an optimum exists; whether a specific company is above, at, or below it is a separate empirical question).

---

## Problem 4 — Query practice across engines (45 min)

The same tasks, run on **both** engines, in `cross-engine.sql`:

1. A query computing `interest_coverage` for every year, on both Postgres and SQLite — confirm identical results.
2. A query using a window function (`SUM(...) OVER (ORDER BY fiscal_year)`) to compute the cumulative interest tax shield across all 5 years. Confirm it runs unchanged on both engines.
3. **Postgres only:** rewrite the DFL computation (`operating_income / pretax_income`) using an explicit `CAST(... AS NUMERIC)` to guarantee decimal (not integer) division, and confirm it matches your original query's output. *(SQLite doesn't need this — note in a comment why integer-division truncation is a Postgres-specific risk here, tied back to Week 1's `5/2` gotcha.)*
4. Note in a comment: which of these three tasks, if any, produced genuinely different results between engines? If none did, say so explicitly — that's a valid and useful finding.

---

## Problem 5 — Extend the model to a real company (60 min)

Pick a real, publicly traded company you can find basic financials for (10-K, annual report, or a finance data site — e.g., a company's EBIT, interest expense, and total debt for its most recent fiscal year).

1. Compute its interest coverage ratio. Which of Lecture 2's coverage bands does it fall into?
2. Assuming a reasonable `Tc` (look up the company's home-country statutory rate, or use its reported effective tax rate if you can find one) and a reasonable `rU` (your own defensible estimate — state your reasoning in 1–2 sentences), compute its unlevered value `VU` and its annual + perpetual-PV tax shield.
3. Using `α = 0.30` (this week's Crunch Machine Co. default — note if you think a different `α` is more appropriate for this specific company's industry, and why), compute the model's value-maximizing debt level `D*` and compare it to the company's actual debt.
4. In 3–4 sentences: does this real company look over-levered, under-levered, or roughly at the model's optimum? Name one industry-specific factor (customer trust, asset resaleability, cyclicality) that might make this company's *true* `α` different from Crunch Machine Co.'s manufacturing-industry assumption.

**Deliver** `real-company-model.md` with your sourced figures (cite where you found them), your calculations, and your written interpretation.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 60 min |
| 2 | 60 min |
| 3 | 45 min |
| 4 | 45 min |
| 5 | 60 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
