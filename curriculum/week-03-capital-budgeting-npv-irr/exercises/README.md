# Week 3 — Exercises

Three guided exercises, ~45–60 min each. **Compute every number yourself** before checking it against the "Expected" note — the goal is that writing an NPV/IRR/PI function becomes automatic, not that you copy a formula once and move on.

1. **[Exercise 1 — NPV over a project stream](exercise-01-npv-in-python.md)** — write an NPV function in Python, cross-check it in SQL, and run it over all six projects in `cash_flows`.
2. **[Exercise 2 — IRR and MIRR side by side](exercise-02-irr-and-mirr.md)** — compute both for every project, check each for multiple roots, and compare the gap between them.
3. **[Exercise 3 — Rank projects under a budget](exercise-03-rank-under-a-budget.md)** — screen the slate by NPV, rank the survivors by PI, and fund a subset under two different budgets.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and `SELECT COUNT(*) FROM cash_flows;` returns **38**.
- You have a Postgres or SQLite shell open, and a Python virtual environment with `pandas`, `numpy`, `numpy-financial`, `scipy`, `sqlalchemy`, and `psycopg[binary]` installed and activated.

## Suggested workflow

- Open the exercise file beside your terminal and editor.
- Write each formula from memory first, then check it against the lecture notes if you get stuck — don't paste code you haven't read.
- For SQL tasks, run the query and compare the output row-for-row against the "Expected" table before moving on.
- For Python tasks, print intermediate values (each discounted cash flow, not just the final sum) the first time through — it's the fastest way to catch a sign error or an off-by-one period.
- Save your work: `solutions.sql` for SQL tasks, `solutions.py` for Python tasks, one file per exercise, each answer under a numbered comment. You'll reuse both files in the mini-project.

## A heads-up about this week's numbers

Not every project in `cash_flows` has positive NPV at its own hurdle rate — you'll discover this directly in Exercise 1, and it's the entire premise of Exercise 3. That's intentional, and it mirrors real capital-budgeting slates: some projects that sound reasonable in a pitch don't survive contact with a discounted cash-flow model. Don't "fix" a negative NPV by second-guessing your arithmetic — verify your formula against the CNC Machine (Exercise 1's known-good answer), and if your formula is right, trust the negative number.
