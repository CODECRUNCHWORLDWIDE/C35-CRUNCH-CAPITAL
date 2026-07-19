# Exercise 1 — Discount a Single Cash Flow

**Goal:** Get the present value and future value formulas into your fingers — by hand first, then in SQL, then in Python — so all three ways of computing the same number feel interchangeable.

**Estimated time:** 45 minutes.

## Setup

You don't need the seed table for this one — just a calculator (or paper), your SQL shell, and a Python REPL or script. Have Lecture 2's formulas open beside you:

```
FV = PV × (1 + r)^n
PV = FV / (1 + r)^n
```

Create two files: `solutions.sql` and `solutions.py`. Put each answer under a numbered comment (`-- Task N` in SQL, `# Task N` in Python).

## Part A — By hand (show your work)

Write the arithmetic out step by step — don't skip to the calculator. In a `hand-work.md` file, show each step.

1. **FV, one year.** You invest $5,000 today at 6% annual interest. What is it worth in 1 year? Show the multiplication.
2. **FV, five years.** Same $5,000 at 6%, but left for 5 years. Show `(1.06)^5` computed step by step (square it, then multiply by itself again, etc.) before multiplying by 5,000.
3. **PV, single flow.** You're promised $10,000 in 3 years. Your discount rate is 9%. What is that promise worth today? Show the denominator `(1.09)^3` computed first, then the division.
4. **PV, compare two rates.** The same $10,000-in-3-years promise, but now compare a 4% discount rate to a 15% discount rate. Compute both present values. Which is bigger, and does that make sense given what a *higher* discount rate should do to a future cash flow?

## Part B — In SQL

Translate each of Part A's calculations into a single `SELECT`. Use `POWER(base, exponent)`.

5. **Task 5.** Rewrite Task 1 (FV of $5,000 at 6% for 1 year) as a SQL query returning one column named `future_value`.
6. **Task 6.** Rewrite Task 2 (FV of $5,000 at 6% for 5 years) the same way.
7. **Task 7.** Rewrite Task 3 (PV of $10,000 in 3 years at 9%) as a query returning one column named `present_value`, rounded to 2 decimal places with `ROUND(...)`.
8. **Task 8.** Write one query that returns **both** present values from Task 4 (4% and 15%) side by side as two columns, `pv_at_4pct` and `pv_at_15pct`.

## Part C — In Python

Write and use the two functions from Lecture 2, section 6:

```python
def present_value(cash_flow: float, rate: float, period: int) -> float:
    return cash_flow / (1 + rate) ** period

def future_value(present: float, rate: float, period: int) -> float:
    return present * (1 + rate) ** period
```

9. **Task 9.** Call `future_value` to reproduce Task 1's answer. Print it rounded to 2 decimals.
10. **Task 10.** Call `present_value` to reproduce Task 3's answer. Print it rounded to 2 decimals.
11. **Task 11.** Write a loop that prints the present value of a fixed $10,000 cash flow arriving in year `n`, for every `n` from 1 to 10, at a 9% discount rate. Watch how fast the present value shrinks as `n` grows — that's the compounding effect from Lecture 2 made visible.

## Expected results (spot checks)

- Task 1 → `5,300.00`
- Task 2 → `6,691.13` (approximately)
- Task 3 → `7,721.83` (approximately)
- Task 4 → the 4% present value (~`8,889.96`) is larger than the 15%-discounted one (~`6,575.16`) — a *higher* discount rate always produces a *lower* present value for the same future cash flow. If your numbers came out the other way, re-check which rate went in which formula.
- Task 11 → the first value (`n=1`) should be the largest; the value should shrink every step and be noticeably smaller than half its starting value by around `n=8`.

## Done when…

- [ ] `hand-work.md` shows the actual arithmetic for Tasks 1–4, not just final answers.
- [ ] `solutions.sql` has Tasks 5–8, each returning a correctly named, rounded column.
- [ ] `solutions.py` has Tasks 9–11, each printing a clearly labeled result.
- [ ] You can explain, out loud, why a higher discount rate produces a *lower* present value — not just that it does.
- [ ] Your Task 11 loop shows the present values shrinking, and you can point to *why* — same formula as always, growing exponent in the denominator.

## Stretch

- Add a Task 12: at what rate does $5,000 double in exactly 12 years? You don't have the formula for this yet (it's the **Rule of 72** territory) — try to reason your way to an approximate answer using `future_value` and trial and error, then look up the Rule of 72 and see how close your guess was.
- Add a Task 13: write a Python function `effective_annual_rate(nominal_rate, m)` implementing the EAR formula from Lecture 2, section 4. Confirm it reproduces the 12%-nominal, monthly-compounding example (should return ≈ `0.1268`).

## Submission

Commit `hand-work.md`, `solutions.sql`, and `solutions.py` to your portfolio under `c35-week-01/exercise-01/`.
