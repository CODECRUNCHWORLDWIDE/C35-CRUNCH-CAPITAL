# Exercise 1 — Model a Share Issuance

**Goal:** Reproduce Lecture 1's full pricing and dilution pipeline yourself, in SQL, then re-run it under two changed assumptions to see how sensitive the outcome is to the deal terms.

**Estimated time:** 60 minutes.

## Setup

Confirm your database has this week's three seed tables:

```sql
SELECT * FROM company_snapshot;
SELECT * FROM shareholders ORDER BY shares_held DESC;
SELECT * FROM financing_terms WHERE instrument_type = 'Equity';
```

Create a file `solutions.sql`. Put each task under a numbered comment.

## Tasks

1. **Reproduce the deal terms.** Using the numbers from `company_snapshot` and `financing_terms`, write one query that returns `shares_exact`, `shares_issued` (whole-share, rounded up), `actual_gross`, `actual_fee`, and `actual_net`, matching Lecture 1, Section 3's worked numbers exactly.

2. **Pre-raise ownership.** Write a query against `shareholders` that returns every shareholder's name and their `pct_before` (rounded to 3 decimal places), sorted highest to lowest. Confirm the six percentages sum to `100.000`.

3. **Post-raise ownership, per shareholder.** Extend Task 2 into a single query returning `shareholder_name`, `shares_held`, `pct_before`, `pct_after`, and `pct_point_dilution` for every existing shareholder — reproduce Lecture 1, Section 4's table exactly.

4. **The new investors' row.** Using a `UNION ALL` (or a second `SELECT`), add a seventh row to your Task 3 output representing "New Public Investors" — `shares_held` = your Task 1 `shares_issued`, `pct_before` = 0, `pct_after` computed the same way as everyone else. Confirm all seven `pct_after` values sum to `100.000` (within rounding).

5. **A deeper discount.** Re-run Task 1 with the new-issue discount increased from 5% to **10%** (offer price becomes $18.00 instead of $19.00), fee unchanged at 5%. How many more shares does the company have to issue to net the same $20,000,000, and by how many additional percentage points does the Founder & CEO get diluted, compared to the original 5% discount deal?

6. **A smaller underwriting fee.** Starting from the *original* 5% discount, re-run Task 1 with the underwriting fee reduced from 5% to **2%** (a company with strong negotiating leverage or a smaller, simpler deal might get this). How many *fewer* shares does the company need to issue? Which lever — the discount (Task 5) or the fee (Task 6) — moved the share count more, for the same percentage-point change?

## Expected results (spot checks)

- Task 1 → `shares_issued = 1108034`, `actual_net ≈ 20000013.70`.
- Task 2 → Founder & CEO at `30.000%`; six percentages sum to exactly `100.000`.
- Task 3 → Founder & CEO's `pct_after ≈ 27.007`, `pct_point_dilution ≈ 2.993`.
- Task 5 → a 10% discount requires issuing **more** shares than a 5% discount, for the same net-proceeds target — a deeper discount always means giving up more ownership per dollar raised.

## Done when…

- [ ] `solutions.sql` runs top to bottom against your database with no errors.
- [ ] Task 1's whole-share rounding uses `CEIL()`, not truncation or `ROUND()` — a truncated or nearest-rounded share count can under-fund the raise.
- [ ] Task 4's seven-row table sums to `100.000%` (within a hundredth of a point, allowing for rounding).
- [ ] You can state, in one sentence each, which single lever — discount depth or fee percentage — has a bigger effect on dilution, and why that makes sense given what each one actually represents.

## Stretch

- Add a `Task 7`: instead of a single flat new-issue discount, model a **two-tranche** offering — the first $10,000,000 (net) priced at the original 5% discount, the second $10,000,000 (net) priced at a steeper 8% discount (reflecting weaker demand for the second half of a large deal). Compute the total shares issued across both tranches and compare the resulting dilution to a single-tranche $20,000,000 raise at a blended discount — are they the same? Should they be?

## Submission

Commit `solutions.sql` to your portfolio under `c35-week-06/exercise-01/`.
