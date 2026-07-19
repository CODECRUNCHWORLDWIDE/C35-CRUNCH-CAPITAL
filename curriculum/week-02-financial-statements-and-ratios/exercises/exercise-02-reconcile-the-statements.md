# Exercise 2 — Reconcile Cash Across the Statements

**Goal:** Turn Lecture 2's articulation checks into a reusable SQL toolkit, run it across all six years of Crunch Machine Co.'s history (now that FY2020 is loaded), and use it to catch a planted error the way a real analyst catches a bad number before it reaches a report.

**Estimated time:** 45 minutes.

## Setup

Confirm all three tables now hold six years (FY2020–FY2025) — this exercise assumes you completed Exercise 1:

```sql
SELECT MIN(fiscal_year), MAX(fiscal_year), COUNT(*) FROM cash_flow_statement;
-- expect: 2020, 2025, 6
```

Create a working file `solutions.sql`.

## Tasks

1. **Retained-earnings tie-out, all years.** Write the `re_check` query from [Lecture 2](../lecture-notes/02-how-the-statements-link.md) and run it across all six years. With FY2020 loaded, FY2021 should now resolve — confirm every row (FY2021–FY2025; FY2020 has no prior year to check against and is expected to be excluded) reads `ties_out = true`.

2. **Cash tie-out, all years.** Write the cash-flow-to-balance-sheet tie-out query and run it across all six years, including FY2020. Confirm every row reads `ties_out = true`.

3. **The combined articulation query.** Write the single combined query from Lecture 2 (`statements_articulate`) and confirm it returns `true` for every year from FY2021 onward.

4. **Prove the balance sheet balances, independently.** Separately from Tasks 1–3, write a query confirming `total_assets = total_liabilities + total_equity` for all six years — this is the third articulation check (Lecture 1's core identity), and it's worth checking on its own since a broken balance sheet would break every other check downstream too.

5. **Plant and catch an error.** Run this — it corrupts a *copy* of the data, not your real table, so your original data is unaffected:

   ```sql
   CREATE TABLE cash_flow_statement_corrupted AS
   SELECT * FROM cash_flow_statement;

   UPDATE cash_flow_statement_corrupted
   SET cash_ending = cash_ending - 500
   WHERE fiscal_year = 2023;
   ```

   Re-run your Task 2 query against `cash_flow_statement_corrupted` (joined to the real `balance_sheet`) instead of the original `cash_flow_statement`. Report which `fiscal_year` now fails, and by how much (write a query that computes the actual dollar difference, not just `true`/`false`).

6. **Explain the downstream damage.** In one or two sentences in a comment, explain what would happen to FY2023's `net_change_in_cash` figure and to FY2024's `cash_beginning` figure if this $500 error were never caught and the corrupted table were used for the rest of the course. (Hint: `cash_beginning` for one year should always equal `cash_ending` for the year before it — is that still true here?)

7. **Clean up.** Drop the scratch table: `DROP TABLE cash_flow_statement_corrupted;`

## Expected result (spot checks)

- Task 1 → 5 rows returned (FY2021–FY2025), all `ties_out = true`.
- Task 2 → 6 rows returned (FY2020–FY2025), all `ties_out = true`.
- Task 4 → all 6 years show assets exactly equal to liabilities plus equity.
- Task 5 → FY2023 fails, off by exactly `500` (in the direction matching how you subtracted it).

## Done when…

- [ ] All four articulation checks (Tasks 1, 2, 3, 4) pass cleanly against the real, uncorrupted tables.
- [ ] Task 5 correctly isolates the one corrupted year and reports the exact dollar gap.
- [ ] Your Task 6 explanation names the *specific* downstream field that breaks next, not just "other things would be wrong."
- [ ] `cash_flow_statement_corrupted` no longer exists in your database (Task 7).

## Stretch

- Write a single query that runs *all four* articulation checks (RE tie-out, cash tie-out, balance-sheet identity, and a check that `cash_beginning` for each year equals `cash_ending` for the prior year) and returns one row per year with four boolean columns plus a fifth `all_checks_pass` column that is `true` only if all four are.
- In Python (pandas), pull all three tables with `pd.read_sql`, replicate the Task 4 balance-sheet-identity check as a DataFrame comparison (`assert (df['total_assets'] == df['total_liabilities'] + df['total_equity']).all()`), and confirm it passes silently. This is the shape of an automated data-quality test you'd actually run in a real pipeline, not just a one-off query.

## Submission

Commit `solutions.sql` (and, if you did the Python stretch, a `check_articulation.py`) to your portfolio under `c35-week-02/exercise-02/`.
