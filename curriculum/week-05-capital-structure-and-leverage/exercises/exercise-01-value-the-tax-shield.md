# Exercise 1 — Value the Interest Tax Shield

**Goal:** Get the tax-shield formulas into your fingers — the annual shield, the perpetual-debt present value, and the finite-life sum — computed both in SQL and in Python, and reconciled against each other.

**Estimated time:** 60 minutes.

## Setup

Confirm you're connected and both tables are loaded:

```sql
SELECT COUNT(*) FROM income_statement;   -- must print 5
SELECT COUNT(*) FROM balance_sheet;      -- must print 5
```

Create `solutions.sql` and `solutions.py`. Put each answer under a numbered comment (`-- Task N` in SQL, `# Task N` in Python).

## Part A — In SQL

1. **Task 1.** Write one query returning `fiscal_year`, `interest_expense`, and `annual_tax_shield` (`= 0.25 × interest_expense`, rounded to 1 decimal) for all five years.

2. **Task 2.** Write one query joining `income_statement` and `balance_sheet` on `fiscal_year`, returning `fiscal_year`, `total_debt` (`short_term_debt + long_term_debt`), and `pv_tax_shield_perpetual` (`= 0.25 × total_debt`, rounded to 1 decimal) for all five years.

3. **Task 3.** Using a `WITH` clause (a common table expression), compute the **cumulative** annual tax shield across all five years — i.e., a running total of Task 1's `annual_tax_shield`, ordered by `fiscal_year`. *(Hint: window function `SUM(...) OVER (ORDER BY fiscal_year)`.)*

## Part B — In Python

Write these two functions, then call each against all five years pulled from the database (or hardcode the five `(fiscal_year, interest_expense, total_debt)` tuples if you haven't connected Python to SQL yet — but note in a comment that you did so).

```python
def annual_tax_shield(interest_expense: float, tax_rate: float = 0.25) -> float:
    ...

def pv_tax_shield_perpetual(total_debt: float, tax_rate: float = 0.25) -> float:
    ...
```

4. **Task 4.** Call both functions for all five years and print a table (a `pandas` DataFrame is fine) with columns `fiscal_year`, `annual_tax_shield`, `pv_tax_shield_perpetual`. Confirm every value matches your SQL answers from Part A exactly.

5. **Task 5.** Compute `VU` (unlevered firm value, `= EBIT × (1 − Tc) / rU` with `rU = 0.10`) for all five years, and add a `VL` column (`= VU + pv_tax_shield_perpetual`). Which year has the highest `VL`? Is it the same year that has the highest `pv_tax_shield_perpetual` alone? If not, explain in one sentence why not.

## Part C — Finite-life shield (the harder case)

6. **Task 6.** Suppose, hypothetically, Crunch Machine Co.'s FY2025 total debt (`19,700`) is actually a single 5-year term loan taken out at the start of FY2025 at an 8% annual rate, fully amortizing with **level annual payments** (use the annuity payment formula from Week 1's Challenge 1: `Payment = P × [r(1+r)^n] / [(1+r)^n − 1]`, with `P = 19700`, `r = 0.08`, `n = 5`). Build the 5-period amortization schedule (period, payment, interest_paid, principal_paid, remaining_balance) in Python.

7. **Task 7.** Compute the finite-life present value of the tax shield over that 5-year schedule, discounting each period's shield at 8%:

```python
def pv_finite_tax_shield(schedule, tax_rate: float, discount_rate: float) -> float:
    ...
```

8. **Task 8.** Compare your Task 7 answer to the perpetual-debt shortcut (`Tc × 19,700 = 4,925.0`, from Task 2). Which is bigger? Explain in one sentence *why* the finite-life number must always be smaller than the perpetual shortcut.

## Expected results (spot checks)

- Task 1, FY2025 → `annual_tax_shield = 345.0`.
- Task 2, FY2025 → `total_debt = 19,700`, `pv_tax_shield_perpetual = 4,925.0`.
- Task 5, FY2025 → `VU = 16,687.5`, `VL = 21,612.5`.
- Task 5 — the highest `VL` should be **2021** (`VU = 53,550.0`, `VL = 56,375.0`), not 2025, even though 2025 has the largest tax shield alone — because `VU` fell far more than the shield grew (Lecture 1, Section 7).
- Task 6 — the level annual payment should be roughly `4,934.0` (verify with a reference amortization calculator: principal 19,700, 8%, 5 years, annual payments).
- Task 8 — the finite-life PV should land around `1,038` — noticeably smaller than the perpetual shortcut's `4,925.0` (a 5-year loan's shield ends after 5 years; the perpetual shortcut assumes it never does).

## Done when…

- [ ] `solutions.sql` has Tasks 1–3, each returning correctly named, rounded columns.
- [ ] `solutions.py` has Tasks 4–8, with a printed table for Task 4 and a printed comparison for Task 8.
- [ ] Your SQL and Python answers for the same quantity (annual shield, perpetual PV) match exactly.
- [ ] You can explain, out loud, why the finite-life tax shield is always smaller than the `Tc × D` perpetual shortcut.

## Stretch

- Add a Task 9: recompute the perpetual `pv_tax_shield_perpetual` for FY2025 using `Tc = 0.21` (the flat US federal statutory corporate rate) instead of `0.25`. How much smaller is it? What does this tell you about how sensitive the tax shield's value is to a company's actual effective tax rate versus a generic statutory assumption?

## Submission

Commit `solutions.sql` and `solutions.py` to your portfolio under `c35-week-05/exercise-01/`.
