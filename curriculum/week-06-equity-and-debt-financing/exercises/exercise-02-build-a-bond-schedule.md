# Exercise 2 — Build a Bond/Loan Cash-Flow Schedule

**Goal:** Build Lecture 2's full amortization schedule yourself in Python, load it into SQL, and answer real questions against it with queries — then extend the engine to a loan shape it hasn't seen yet.

**Estimated time:** 60 minutes.

## Setup

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://localhost/crunch_capital")
# SQLite users: engine = create_engine("sqlite:///crunch_capital.db")

terms = pd.read_sql("SELECT * FROM financing_terms WHERE instrument_type = 'Debt'", engine)
print(terms)
```

Create a file `solutions.py`. Put each task under a `# Task N` comment.

## Tasks

1. **The payment function.** Write `annual_payment(principal, annual_rate, years)` implementing the annuity formula from Lecture 2, Section 2. Confirm it returns `4943294.36` (rounded to the cent) for `annual_payment(20_000_000, 0.075, 5)`.

2. **The full schedule.** Write `loan_schedule(principal, annual_rate, years)` returning a list of dicts (or a DataFrame), one row per year, with `period`, `beginning_balance`, `payment`, `interest_paid`, `principal_paid`, `ending_balance`. Reproduce Lecture 2, Section 3's five-row table exactly (within a cent per cell).

3. **Verify it, don't just trust it.** Print three checks:
   - The final row's `ending_balance` is `0` (or within a fraction of a cent).
   - The sum of every row's `principal_paid` equals the original `$20,000,000`.
   - The sum of every row's `payment` equals `5 × annual_payment(...)`.

   If any check fails, find the bug before moving on.

4. **Load it into SQL.** Using `pandas.DataFrame(rows).to_sql("term_loan_schedule", engine, if_exists="replace", index=False)`, persist your Task 2 schedule. Confirm with `SELECT * FROM term_loan_schedule ORDER BY period;` from your SQL shell.

5. **Query it — total interest.** In SQL, sum `interest_paid` across all 5 rows. Confirm it matches Lecture 2's **$4,716,471.78** (within a cent).

6. **Query it — front-loading.** In SQL, compare total interest paid in **year 1** to total interest paid in **year 5**. By what multiple is year 1's interest bigger? Does this match Lecture 2, Section 3's claim of "more than 4×"?

7. **Query it — balance after 2 years.** In SQL, find the `ending_balance` after period 2. What fraction of the original $20,000,000 has been paid down after 2 of the loan's 5 years? Is it more or less than the naive "2/5 = 40%" a non-finance colleague might guess, and why?

8. **A different loan shape — monthly instead of annual.** Real term loans are often paid **monthly**, not annually. Extend your `loan_schedule` function (or write a new one) to accept a `payments_per_year` parameter, converting the annual rate to a periodic rate (`annual_rate / payments_per_year`) and the term to a period count (`years * payments_per_year`), matching the mechanics from Week 1's amortization challenge. Run the same $20,000,000, 7.5%, 5-year loan with **monthly** payments instead of annual. Report the monthly payment amount and the *total* interest paid over the full term — is total interest higher or lower than the annual-payment version, and can you explain why in one sentence (hint: think about how quickly the balance shrinks under each schedule)?

## Expected results (spot checks)

- Task 1 → `4943294.36`.
- Task 3 → all three checks pass; final balance is `0` (or a fraction of a cent).
- Task 5 → `4716471.78` total interest.
- Task 6 → year 1's interest (`$1,500,000.00`) is a little over **4.35×** year 5's (`$344,881.00`).
- Task 8 → monthly payments produce **lower** total interest than annual payments, for the same nominal rate and term, because the balance is paid down in smaller increments more frequently, so less balance sits outstanding accruing interest at any given moment.

## Done when…

- [ ] `solutions.py` runs top to bottom with no errors and prints all 8 tasks' results.
- [ ] `loan_schedule` works for **any** `principal`, `annual_rate`, and `years` — prove it by also running it on a small sanity-check loan ($100,000 at 5% for 3 years) and manually verifying period 1's interest by hand (`100000 × 0.05 = 5000.00`).
- [ ] Task 4's `term_loan_schedule` table exists in your database and matches your Python output exactly.
- [ ] You can state, in one sentence, why monthly payments produce less total interest than annual payments on the same loan.

## Stretch

- Add a `Task 9`: what if the company could make an **extra $1,000,000 principal-only payment** at the end of year 2 (on top of the scheduled payment), on the original annual schedule? Recompute the remaining schedule for years 3–5 with that extra paydown applied to the balance before year 3's interest accrues, and report the new total interest paid over the loan's life. How much interest did the extra payment save?

## Submission

Commit `solutions.py` and a `schema.sql` documenting the `term_loan_schedule` table to your portfolio under `c35-week-06/exercise-02/`.
