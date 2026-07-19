# Exercise 2 — Build a Cash-Flow Table in SQL

**Goal:** Get fluent querying a real cash-flow ledger — filtering by project, summing by category, sorting by size and time, and spotting `NULL`s — then extend the table with a project of your own.

**Estimated time:** 45–60 minutes.

## Setup

You already ran the seed (see the [week README](../README.md)). Confirm you're connected and the table is there:

```sql
SELECT COUNT(*) FROM cash_flows;   -- must print 38
```

In `psql`, run `\d cash_flows` to see the columns; in SQLite, `.schema cash_flows`. Keep that column list handy.

Create a file `solutions.sql` and put each answer under a `-- Task N` comment.

## Part A — Query the seed data

1. **All projects.** Select the distinct `project_id`, `project_name`, `category`, `risk_tier`, and `hurdle_rate` — one row per project, not per cash flow. *(Hint: `DISTINCT`. Expected: 6 rows.)*
2. **One project's stream.** Select every row for `project_id = 3` (AI Quality-Control R&D), ordered by `period`. *(Expected: 6 rows, starting with the −300,000 capex at period 0.)*
3. **Total raw cash, unadjusted for time.** For each project, sum `amount` (this is the *un-discounted* total — deliberately naive, so you have a "wrong" baseline to compare against once you compute NPV in Lecture 2's style). *(Hint: `SUM(amount)`, filtered per project or one query per project this week — `GROUP BY` arrives formally in Week 2's ratio work, but you may use it here if you already know it.)*
4. **Biggest single outflow.** Which single row has the most negative `amount`? Show `project_name`, `period`, `amount`. *(Expected: the European Market Entry capex, −900,000.)*
5. **High-risk projects only.** Select all rows where `risk_tier = 'High'`, sorted by `project_name` then `period`. *(Expected: 12 rows across 2 projects.)*
6. **Rows with no note.** Select `project_name`, `period`, `flow_type` for every row where `note IS NULL`. *(This is most of the table — confirm your count is well over half of 38.)*
7. **Rows WITH a note.** The complement — every row where `note IS NOT NULL`. Select `project_name`, `note`. *(Expected: 10 rows. Read them — several carry real modeling context, like the Europe project's tariff assumption.)*
8. **Capex only.** Select every row where `flow_type = 'Capex'`, sorted by `amount` ascending (most negative first). *(Expected: 6 rows — one per project, all at period 0.)*
9. **Opex rows.** Which projects have an `Opex` line, and in which period? *(Expected: 2 rows — R&D's tuning cost in period 1, Europe's setup cost in period 1.)*
10. **Salvage value.** Select every `Salvage` row. Which two projects have one, and how much is each worth? *(Expected: 2 rows.)*
11. **Hurdle rate ranking.** Select the 6 distinct `(project_name, hurdle_rate)` pairs, sorted by `hurdle_rate` descending. Which project has the highest required return, and which has the lowest?
12. **A judgment filter.** Select every row belonging to a project whose `hurdle_rate` is **10% or higher** — a rough proxy for "the riskier half of the portfolio." How many distinct projects does that include?

## Part B — Extend the table

13. **Add your own project.** Write `INSERT` statements adding a 7th project of your invention — pick a `project_name`, `category`, `sponsor`, `risk_tier`, and `hurdle_rate`, and give it at least 4 cash-flow rows: one period-0 outflow and at least three later inflows. Use `project_id = 7` and continue the `cash_flow_id` sequence from 39.
14. **Query your addition.** Write a query proving your new project loaded correctly (its full stream, ordered by period).
15. **Clean up.** Write the `DELETE FROM cash_flows WHERE project_id = 7;` statement that would remove it, so the seed table is restorable to exactly 38 rows. *(You don't have to run it — just have it ready and correct.)*

## Expected result (spot checks)

- Task 1 → 6 rows.
- Task 4 → European Market Entry, period 0, −900,000.
- Task 5 → 12 rows (AI Quality-Control R&D's 6 + European Market Entry's 6).
- Task 7 → 10 rows.
- Task 9 → 2 rows.
- Task 10 → 2 rows: New CNC Machine's 15,000 salvage, Solar Rooftop Retrofit's 10,000 salvage.

## Done when…

- [ ] `solutions.sql` has all 15 tasks, each under a `-- Task N` comment.
- [ ] Task 3's per-project totals are signed correctly (capex/opex reduce the total, revenue/salvage increase it).
- [ ] Task 6 and Task 7's row counts add to exactly 38 (28 with no note + 10 with one).
- [ ] Your Task 13 insert includes at least one negative (outflow) and three positive (inflow) rows.
- [ ] You can explain, in one sentence, why Task 3's "raw total" is a misleading way to compare projects — this sets up Lecture 2's discounted version of the same question.

## Stretch

- Write a query that returns each project's **raw total cash** (Task 3's number) next to its **period-0 investment** (the capex), as two columns, so you can eyeball a crude "total return multiple" per project. Which project has the highest raw multiple? Does your intuition say that's automatically the *best* project — and why might discounting change the answer?

## Submission

Commit `solutions.sql` to your portfolio under `c35-week-01/exercise-02/`.
