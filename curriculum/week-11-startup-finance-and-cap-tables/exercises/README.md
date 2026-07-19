# Week 11 — Exercises

Three guided exercises, ~1 hour each. **Type every query and script yourself** — don't copy-paste from the lectures. You're rebuilding Solstice Data's cap table from scratch, one financing event at a time, and by the third exercise it needs to match Lecture 2's numbers exactly.

1. **[Exercise 1 — Build a founding cap table](exercise-01-build-a-founding-cap-table.md)** — schema, founder grants, vesting, ownership % query.
2. **[Exercise 2 — Convert a SAFE](exercise-02-convert-a-safe.md)** — take three SAFEs to their Series A conversion using the cap/discount rule.
3. **[Exercise 3 — Add an option pool](exercise-03-add-an-option-pool.md)** — solve the pool-shuffle algebra and land the full post-round cap table.

## Before you start

- You've completed all three lectures.
- You have a working `solstice_captable` database (Postgres) or `solstice_captable.db` (SQLite) — see the [week README](../README.md) if you haven't created it.
- You have Python with `pandas` available for the exercises that ask you to cross-check a SQL result with a script.

## Suggested workflow

- Work each exercise in its own SQL file (`exercise-01.sql`, etc.) plus a short Python script where asked.
- Compute every number **twice** — once by hand/on paper for the smallest example, once in code — before trusting the code on the full table. A cap table is exactly the kind of thing where an off-by-one-row bug silently overstates someone's stake by a few points, and you want to catch that habit now, on toy numbers, not on a real deal.
- Save your work; Exercise 3's output cap table is what Challenges 1–2 and the mini-project build on.
