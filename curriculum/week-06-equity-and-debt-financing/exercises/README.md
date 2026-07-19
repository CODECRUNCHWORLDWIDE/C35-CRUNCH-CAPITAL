# Week 6 — Exercises

Three guided exercises, ~45–60 min each. **Do every calculation yourself** — in SQL first where noted, then in Python. The point isn't the final number; it's building the reflex to price a real financing term sheet the way an analyst does, not the way a textbook problem set does.

1. **[Exercise 1 — Model a share issuance](exercise-01-model-a-share-issuance.md)** — price the follow-on offering and compute every shareholder's new ownership stake, in SQL.
2. **[Exercise 2 — Build a bond schedule](exercise-02-build-a-bond-schedule.md)** — build and store the term loan's full amortization schedule, and query it.
3. **[Exercise 3 — Cost of each path](exercise-03-cost-of-each-path.md)** — compute the true, all-in, normalized cost of the equity raise and the term loan, apples to apples.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md), and `SELECT COUNT(*) FROM shareholders;` returns **6** with `SUM(shares_held) = 10000000`.
- You have a Postgres or SQLite shell open, and a Python virtual environment with `pandas`, `numpy`, `sqlalchemy`, and `psycopg[binary]` installed and activated.

## Suggested workflow

- Open the exercise file beside your terminal and editor.
- For SQL tasks, write the query before running it, then check your output against the "Expected" note.
- For Python tasks, wrap your logic in a named function first — every function you write this week is reused, unmodified, in the mini-project.
- Save your work: `solutions.sql` and `solutions.py`, one section per exercise under a numbered comment. You'll commit these.

## Note on rounding

Share counts must always be whole numbers (`CEIL()` in SQL, `math.ceil()` in Python) — never truncate or round to the nearest integer, which can under-fund the raise. Dollar amounts should be rounded to the cent (`ROUND(x, 2)`) only at the point you display or report them — keep full precision through intermediate calculations, or small rounding errors compound across a 5-year schedule.
