# Exercise 1 — CAPM Cost of Equity

**Goal:** Get the CAPM formula into your fingers — compute cost of equity by hand, in SQL, and in Python, and see exactly how much each input moves the output.

**Estimated time:** 1 hour.

## Setup

Confirm the reference table is seeded:

```sql
SELECT * FROM capital_structure WHERE company = 'Crunch Machine Co.';
```

You should see one row: `risk_free_rate = 0.0425`, `equity_risk_premium = 0.0500`, `vendor_beta = 1.30`, `tax_rate = 0.25`.

Create `solutions.sql` and `solutions.py`. Put each task's work under a comment with the task number.

## Tasks

### Part A — By hand (write these out, then verify in code)

1. Using Crunch Machine Co.'s three CAPM inputs from `capital_structure`, compute its cost of equity by hand. Show your arithmetic. *(Expected: 10.75%.)*

2. A close industry peer has $\beta = 0.85$ instead of 1.30, with the same $R_f$ and $ERP$. Compute its cost of equity by hand. *(Expected: 8.5%.)*

3. Explain, in one sentence, why peer #2's cost of equity is lower even though it operates in the same industry as Crunch Machine Co.

### Part B — In SQL

4. Write a single `SELECT` against `capital_structure` that computes Crunch Machine Co.'s cost of equity inline, aliased as `cost_of_equity`. *(Expected: matches Task 1.)*

5. Extend Task 4's query to also compute what the cost of equity **would be** if the equity risk premium rose to 6.5% instead of 5.0%, aliased as `cost_of_equity_stressed_erp`, in the *same* query (two computed columns, one query). *(Expected: 12.70%.)*

### Part C — In Python

6. Connect to `crunch_capital` with SQLAlchemy, pull the `capital_structure` row into a `pandas.Series`, and compute the cost of equity in Python. Print it formatted as a percentage with 2 decimal places.

7. Write a Python function `cost_of_equity(rf, beta, erp)` that returns the CAPM cost of equity. Call it three times: once with Crunch Machine Co.'s own inputs, once with $\beta = 0.55$ (a defensive utility-like company), and once with $\beta = 1.80$ (an aggressive growth company) — same $R_f$ and $ERP$ each time. Print all three, labeled.

8. Using the function from Task 7, build a small `pandas.DataFrame` with columns `scenario`, `beta`, `cost_of_equity`, covering at least 5 beta values from 0.5 to 2.0 in steps of 0.375, holding $R_f$ and $ERP$ fixed at Crunch Machine Co.'s values. Print the table sorted by beta ascending.

## Expected results (spot checks)

- Task 1 → 10.75%.
- Task 2 → 8.50%.
- Task 5's stressed column → 12.70%.
- Task 8's table has exactly 5 rows, cost of equity strictly increasing as beta increases (holding $R_f$/$ERP$ fixed — this should be self-evidently true from the formula's shape; if your table isn't monotonic, you have a bug).

## Done when…

- [ ] `solutions.sql` has Tasks 4–5, each under a `-- Task N` comment.
- [ ] `solutions.py` has Tasks 6–8, each in its own clearly labeled section.
- [ ] Your Task 1 and Task 4 answers match to the fourth decimal place.
- [ ] You can explain, out loud, why beta is the *only* company-specific input in Part A/B's formula — $R_f$ and $ERP$ are the same for every company on a given day.

## Stretch

- Modify Task 7's function to accept an optional `size_premium` keyword argument (default `0`), added on top of the standard CAPM output — a common real-world adjustment analysts make for small, thinly-traded companies whose risk CAPM's single-factor model may understate. Recompute Crunch Machine Co.'s cost of equity with a `size_premium=0.015` (150 basis points) and compare to the unadjusted number.

## Submission

Commit `solutions.sql` and `solutions.py` to your portfolio under `c35-week-04/exercise-01/`.
