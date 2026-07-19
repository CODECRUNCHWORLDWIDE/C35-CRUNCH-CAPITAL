# Mini-Project — Find Crunch Machine Co.'s Value-Maximizing Debt Level

> Build one clean deliverable that answers the board's question from Challenge 1 with a chart, not just a number: given Crunch Machine Co.'s FY2025 operating profile, what debt level maximizes firm value under the tradeoff model, and how far off is the company's actual capital structure? This is the week's capstone — every formula from Lectures 1–3 feeds into one model.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

A board doesn't want three separate spreadsheets — sorry, three separate SQL queries and Python scripts — for the tax shield, the distress cost, and the leverage-amplification story. They want one model, one chart, and one clear recommendation, with the assumptions stated up front so a skeptical director can push back on the right thing. That's what you're building today.

---

## Deliverable

A directory in your portfolio `c35-week-05/mini-project/` containing:

1. `capital_structure_model.py` — the full model: tax shield, distress cost, tradeoff value function, and the optimizer (both calculus-based and grid-search, reconciled).
2. `value_vs_leverage.png` — a chart of firm value against debt level, with the model's optimum and Crunch Machine Co.'s actual debt both clearly marked.
3. `report.md` — the numbers, the chart, and a written recommendation (structure below).
4. `sensitivity.csv` (or `.sql` output) — the `D*` sensitivity table across at least 3 values of `α`, generated in SQL or Python.

Everything runs against `income_statement` and `balance_sheet` from the [week README](../README.md), on PostgreSQL or SQLite — note which you used.

---

## Part 1 — Pull and confirm the base numbers (30 min)

Using SQL, confirm the FY2025 figures this whole model rests on:

```sql
SELECT i.fiscal_year, i.operating_income AS ebit, i.interest_expense,
       b.short_term_debt + b.long_term_debt AS total_debt,
       b.total_equity,
       ROUND(i.operating_income / i.interest_expense, 2) AS interest_coverage
FROM income_statement i
JOIN balance_sheet b ON b.fiscal_year = i.fiscal_year
WHERE i.fiscal_year = 2025;
```

Confirm: `EBIT = 2,225`, `interest = 1,380`, `total_debt = 19,700`, `total_equity = 19,010`, `coverage ≈ 1.61`. In Python, compute `VU = EBIT × (1 − Tc) / rU` with `Tc = 0.25`, `rU = 0.10` — confirm `VU = 16,687.5`. Put all of this at the top of `report.md` as your "given" data, cited from the query above.

## Part 2 — Build the full model (60 min)

In `capital_structure_model.py`, assemble the complete tradeoff model as a small set of composable functions (reuse and clean up your Exercise 3 and Challenge 1 work — don't start from scratch):

```python
def unlevered_value(ebit: float, tax_rate: float, r_unlevered: float) -> float: ...
def prob_distress(D: float, VU: float) -> float: ...              # capped at 1, per Challenge 2's fix
def expected_distress_cost(D: float, VU: float, alpha: float) -> float: ...
def levered_value(D: float, VU: float, tax_rate: float, alpha: float) -> float: ...
def optimal_debt_calculus(VU: float, tax_rate: float, alpha: float) -> float: ...
def optimal_debt_grid(VU: float, tax_rate: float, alpha: float,
                       d_max: float = 30000, step: float = 10) -> float: ...
```

`prob_distress` **must cap at 1** — apply Challenge 2's fix here as the permanent, correct version of the function, not the earlier uncapped one from Exercise 3. Run both optimizer functions with `VU = 16,687.5`, `Tc = 0.25`, `α = 0.30` and confirm they agree (within the grid step).

## Part 3 — Chart value vs. leverage (45 min)

Build a DataFrame sweeping `D` from `0` to `30,000` in steps of `500` (61 rows), computing `levered_value` at every step using your **capped** `prob_distress`. Plot it with `matplotlib`:

- x-axis: debt level `D`.
- y-axis: `levered_value`.
- A marked point (different color, labeled) at the model's optimal `D*`.
- A marked point (different color, labeled) at Crunch Machine Co.'s actual `D = 19,700`.
- Title, axis labels, and a legend. No unlabeled chart is a finished chart.

Save as `value_vs_leverage.png`.

## Part 4 — Sensitivity table (30 min)

Reuse Challenge 1, Part 3's sensitivity work: compute `D*` for `α ∈ {0.10, 0.15, 0.20, 0.25, 0.30, 0.40, 0.50}`. Save the resulting table (`alpha`, `D_star`, `D_star_as_pct_of_actual_debt`) as `sensitivity.csv`.

## Part 5 — Write the report

In `report.md`:

- The confirmed "given" numbers from Part 1.
- The chart from Part 3, embedded or referenced, with 3–4 sentences describing its shape and where the two marked points sit relative to each other.
- The sensitivity table from Part 4, with one sentence on what range of `α` would need to be true for Crunch Machine Co.'s **actual** debt level to be roughly optimal.
- A final recommendation, addressed to the board: should Crunch Machine Co. reduce debt, hold steady, or increase it? State your confidence level and name the one assumption (`α`, `rU`, or the quadratic shape of `p(D)` itself) that would most change your recommendation if it turned out to be wrong.

---

## Rules

- **The model functions must be genuinely reusable** — no hardcoded `19700` or `16687.5` baked into `levered_value`, `prob_distress`, or the optimizers. Every number specific to Crunch Machine Co. FY2025 should be passed in as an argument, not hardcoded inside a function body. Prove this by running your Part 4 sensitivity sweep and confirming it works by *calling* your functions with different `α` values, not by hand-editing the function each time.
- **`prob_distress` must be capped at 1** everywhere in this deliverable — this is the corrected version from Challenge 2, not the uncapped Exercise 3 version. If you didn't do Challenge 2, implement the cap now; it's a one-line `min(1.0, (D/VU)**2)`.
- **No spreadsheets**, per this week's data tooling rule — every number and every chart comes from SQL and Python code you can re-run.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 30% | Model functions match Lecture 2's formulas exactly; both optimizers agree with each other |
| Chart quality | 20% | Clear inverted-U shape, both markers correctly placed and labeled, proper axis labels |
| Sensitivity analysis | 20% | Full `α` sweep, correctly showing `D*` falling as `α` rises |
| Reusability | 15% | No hardcoded Crunch-specific numbers inside the core functions |
| Recommendation | 15% | Clear board-level answer, with a named, honest confidence caveat |

---

## Why this matters

This is the shape of real capital-structure advisory work: a company doesn't need a lecture on Modigliani-Miller, it needs a chart with two dots on it — "here's where you are, here's where the model says you should be" — and an honest account of how much to trust the gap between them. Keep `capital_structure_model.py`; Week 6's financing-choice comparisons (debt vs. equity for a specific new financing need) build directly on the same `unlevered_value` and tax-shield machinery you just wrote.

When done: take the [quiz](../quiz.md) and start [Week 6 — Equity vs. debt financing — dilution, coupons, covenants](../../week-06-equity-vs-debt-financing/).
