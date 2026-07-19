# Week 1 — Exercises

Three guided exercises, ~45–60 min each. **Do every calculation yourself** — by hand first where noted, then in code. The point isn't the final number; it's building the reflex to move between "formula on paper," "query in SQL," and "function in Python" without losing precision or confidence.

1. **[Exercise 1 — Discount a single cash flow](exercise-01-discount-a-single-cash-flow.md)** — present value and future value of one cash flow, by hand, in SQL, and in Python.
2. **[Exercise 2 — Build a cash-flow table in SQL](exercise-02-build-a-cash-flow-table.md)** — query, filter, and extend the `cash_flows` seed table.
3. **[Exercise 3 — PV and FV in pandas](exercise-03-pv-fv-in-pandas.md)** — reusable discounting functions over data pulled from SQL.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and `SELECT COUNT(*) FROM cash_flows;` returns **38**.
- You have a Postgres or SQLite shell open, and a Python virtual environment with `pandas`, `numpy`, `sqlalchemy`, and `psycopg[binary]` installed and activated.

## Suggested workflow

- Open the exercise file beside your terminal and editor.
- For hand-calculation tasks, **write the arithmetic out** before reaching for a calculator or code — the goal is that the formula becomes automatic, not that you get an answer fast.
- For SQL/Python tasks, write the query or function before running it, then check your output against the "Expected" note.
- If a result surprises you, stop and figure out *why* before moving on — that confusion is the lesson.
- Save your work: `solutions.sql` for SQL tasks, `solutions.py` for Python tasks, one file per exercise, each answer under a numbered comment. You'll commit these.

## Note on the two engines

Every SQL task works on both PostgreSQL and SQLite. Postgres supports both `POWER(x, y)` and the `^` operator for exponents; SQLite only has `POWER()` (available in modern SQLite builds) — if yours lacks it, use repeated multiplication or compute the exponent in Python instead and pass it in. Where it matters, the exercise says so.
