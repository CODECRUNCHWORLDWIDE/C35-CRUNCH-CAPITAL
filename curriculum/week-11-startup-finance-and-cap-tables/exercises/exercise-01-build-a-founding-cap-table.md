# Exercise 1 — Build a Founding Cap Table

**Goal:** Stand up the `cap_table_events` schema, insert Solstice's founding grants, add vesting terms, and write the ownership-percentage query you'll reuse all week.

**Estimated time:** 60 minutes.

## Setup

Create (or reuse) `solstice_captable`:

```bash
createdb solstice_captable   # PostgreSQL
psql solstice_captable
# or
sqlite3 solstice_captable.db
```

Create a file `exercise-01.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Create the schema.** Create `cap_table_events` with these columns: `event_id` (PK), `holder_name`, `holder_type`, `security_type`, `shares`, `invested_amount`, `price_per_share`, `event_date`, `round_name`, `notes`. *(Copy the `CREATE TABLE` from the week README if you want the exact types — but type it yourself, don't paste.)*

2. **Insert the founding grants.** Two rows: Maya Chen, 5,500,000 common shares, 55%; Diego Ruiz, 4,500,000 common shares, 45%. Both `event_date = '2023-02-01'`, `round_name = 'Founding'`, `price_per_share = 0.0001`. *(Expected: `SELECT SUM(shares) FROM cap_table_events;` returns 10,000,000.)*

3. **Add vesting columns.** `ALTER TABLE` to add `vest_start DATE`, `cliff_months INTEGER`, `vest_months INTEGER`. `UPDATE` both founder rows: `vest_start = '2023-02-01'`, `cliff_months = 12`, `vest_months = 48`.

4. **Add the advisor grant.** Insert Sam Okafor, Advisor, 100,000 common shares, `event_date = '2023-05-15'`, `round_name = 'Founding'`, `price_per_share = 0.05`, 2-year vest (`cliff_months = 0`, `vest_months = 24`). *(Expected: total shares now 10,100,000.)*

5. **Ownership percentage query.** Write the `SUM(shares) OVER ()` query from Lecture 1 that returns every holder with their fully diluted ownership %, sorted by shares descending. *(Expected: Maya ≈ 54.46%, Diego ≈ 44.55%, Sam ≈ 0.99%.)*

6. **Vested shares as of a date.** Write a query computing each founder's **vested** share count as of `2024-08-01` (18 months after `vest_start`), using the cliff/vesting logic from Lecture 1. *(Expected: both founders are past their 1-year cliff and have vested `18/48` ≈ 37.5% of their grant — Maya ≈ 2,062,500 vested, Diego ≈ 1,687,500 vested.)*

7. **A pre-cliff check.** Write the same vested-shares query but for `2023-08-01` (6 months in — before the 1-year cliff). *(Expected: 0 vested shares for both founders. This is the point of a cliff — nothing vests until month 12, then a quarter vests all at once.)*

## Expected result (spot checks)

- Task 2 → `SUM(shares)` = 10,000,000.
- Task 4 → `SUM(shares)` = 10,100,000.
- Task 5 → percentages sum to 100.00 (allow for rounding to the last cent).
- Task 6 → Maya's vested shares ≈ 2,062,500; Diego's ≈ 1,687,500.
- Task 7 → 0 vested shares for both founders.

## Done when…

- [ ] `exercise-01.sql` has all 7 tasks, each under a `-- Task N` comment.
- [ ] Task 5's percentages sum to 100.00%.
- [ ] Task 6 and Task 7 show the cliff working correctly — zero before month 12, a jump at month 12, then smooth monthly vesting after.
- [ ] You can explain in one sentence why Sam's grant reduced Maya's and Diego's *percentage* but not their *share count*.

## Stretch

- Add a query that returns, for every holder, both their **granted** shares and their **vested-as-of-today** shares side by side, using `CURRENT_DATE` instead of a hardcoded date.
- What happens to Sam's vested percentage on `2023-11-15` (6 months in, no cliff)? Write the query and confirm it's exactly 25% of his grant (6/24 months).

## Submission

Commit `exercise-01.sql` to your portfolio under `c35-week-11/exercise-01/`.
