# Week 2 Exercises — Overview

Three exercises, worked in order, all against the `income_statement`, `balance_sheet`, and `cash_flow_statement` tables you seeded in the [week README](../README.md). Each one builds on the one before it — do them in sequence.

| # | Exercise | Skill drilled |
|--:|----------|----------------|
| 1 | [exercise-01-load-filings-into-sql.md](./exercise-01-load-filings-into-sql.md) | Parsing a raw filing into normalized `INSERT` statements |
| 2 | [exercise-02-reconcile-the-statements.md](./exercise-02-reconcile-the-statements.md) | Proving articulation in SQL — cash and retained earnings tying out |
| 3 | [exercise-03-compute-core-ratios.md](./exercise-03-compute-core-ratios.md) | Computing the full liquidity/leverage/profitability/efficiency ratio set |

## How to work them

- Create a working file per exercise (`solutions.sql` or a `.py` script, as each exercise specifies) and keep every query under a labeled comment (`-- Task N`), the same convention as every other week in this course.
- Run every query against the live database — don't just read the SQL and assume it works. Numbers matter this week; a typo in a `JOIN` condition silently drops rows and every ratio downstream goes wrong.
- Exercise 1 changes the data (adds a sixth year, FY2020) — do it **first**, and confirm your row counts, before starting Exercise 2 or 3. Exercises 2 and 3 assume FY2020 is loaded.
- Where an exercise asks you to reconcile or compute something, check your own answer against the "Expected" hint before moving on. If your number doesn't match, don't skip it — track down why. Financial data with a hidden error is worse than no data, because you'll trust the ratio built on top of it.

## Submission

Commit your working files to your portfolio under `c35-week-02/exercise-0N/` as each exercise specifies.
