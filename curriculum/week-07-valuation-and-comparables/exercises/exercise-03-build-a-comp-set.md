# Exercise 3 — Build a Comp Set

**Goal:** Compute enterprise value and multiples from `trading_comps`' raw market data, summarize the set with both a mean and a trimmed range, and apply the result to Crunch Machine Co. to get an implied enterprise value and per-share price.

**Estimated time:** 75 minutes.

## Setup

You need the `trading_comps` and `valuation_assumptions` tables from this week's [README](../README.md). Have Lecture 2, Sections 3–6 open.

Create `solutions.sql` and `solutions.py`.

## Part A — Compute EV and multiples per comp (SQL)

1. **Task 1.** Write the query from Lecture 2, Section 3: for every row in `trading_comps`, compute `market_cap`, `enterprise_value`, `ev_revenue`, and `ev_ebitda`, ordered by `ev_ebitda` descending. Confirm your output matches Lecture 2's table exactly (within a rounding dollar or two).
2. **Task 2.** Add a column `implied_leverage` = `total_debt / enterprise_value`, as a percentage. Which comp is the most conservatively financed (lowest ratio), and which is the most aggressively financed (highest)?

## Part B — Summarize the set (SQL + Python)

3. **Task 3.** In SQL, compute `mean_ev_ebitda`, `min_ev_ebitda`, and `max_ev_ebitda` across all six comps (use the subquery pattern from Lecture 2, Section 4). If your engine supports `PERCENTILE_CONT`, add `median_ev_ebitda` too.
4. **Task 4.** In Python (pandas), compute the same four statistics for **both** `ev_ebitda` and `ev_revenue`, using `.mean()`, `.median()`, `.min()`, `.max()`. Confirm your `ev_ebitda` numbers match Task 3.
5. **Task 5.** Compute the **trimmed range** for both multiples: drop the single highest and single lowest value from each of the six-company set, then report the min and max of what's left. (In pandas: sort the column, slice `[1:-1]`, then take `.min()`/`.max()`.)

## Part C — Apply to Crunch Machine Co.

6. **Task 6.** Using the trimmed EV/EBITDA range from Task 5 and Crunch Machine Co.'s **NTM EBITDA** (from Exercise 1's FY2026 forecast — recall trading comps apply to forward figures, per Lecture 2, Section 6), compute the implied enterprise value range: `low = ntm_ebitda × trimmed_min`, `high = ntm_ebitda × trimmed_max`.
7. **Task 7.** Do the same using the trimmed EV/Revenue range and Crunch Machine Co.'s NTM revenue.
8. **Task 8.** Bridge both EV ranges to a per-share range: subtract `valuation_assumptions.total_debt − cash_and_equivalents` (net debt) from each end, then divide by `shares_outstanding`. Report four per-share numbers: EV/EBITDA-low, EV/EBITDA-high, EV/Revenue-low, EV/Revenue-high.

## Part D — Interrogate the comp set

9. **Task 9.** Vantage Drive Technologies (`VDT`) has the highest EV/EBITDA multiple in the set (10.60×) by a meaningful margin over the next-highest (TitanForge at 9.40×), and it's also the smallest comp by revenue ($165M — the smallest in the set). In 3–4 sentences, argue whether VDT genuinely belongs in this comp set or whether its multiple might be driven by something Lecture 2, Section 2's selection criteria would flag (small size relative to the peer group, a possibly different growth or margin profile). You don't have VDT's own growth/margin data beyond what's in the table — reason from what *is* there, and say explicitly what additional data you'd want before deciding.
10. **Task 10.** Recompute Task 3–5's summary statistics **excluding VDT entirely** (a five-company set instead of six). Does the median move much? Does the trimmed range move much? What does that tell you about how much influence one questionable comp can have on a small set?

## Expected results (spot checks)

- Task 1 → `IMW` has the lowest `ev_ebitda` (≈7.20×) and `VDT` has the highest (≈10.60×); mean EV/EBITDA ≈ **8.63×**, median ≈ **8.50×**.
- Task 5 → trimmed EV/EBITDA range ≈ **7.6×–9.4×**; trimmed EV/Revenue range ≈ **1.48×–2.07×**.
- Task 6 → implied EV (EV/EBITDA method) ≈ **$362.0M–$447.7M** (using NTM EBITDA of $47,628,000 from Exercise 1).
- Task 7 → implied EV (EV/Revenue method) ≈ **$335.7M–$469.5M** (using NTM revenue of $226,800,000).
- Task 8 → per-share range roughly **$10.86–$16.44** across the four combinations (net debt $75,000,000, 24,000,000 shares).
- Task 10 → excluding VDT (five comps instead of six): mean EV/EBITDA drops from ≈8.63× to ≈8.24×, median drops from ≈8.50× to ≈8.10×. On a set this small, mean and median move by similar amounts here — the more dramatic effect is on the **trimmed range's high end**, which drops from 9.4× to 8.9× once VDT (the old maximum) is gone entirely rather than just trimmed. That's the real lesson: a single questionable comp doesn't just nudge an average, it can single-handedly set one end of your entire applied range.

## Done when…

- [ ] `solutions.sql` has Tasks 1–3, each returning correctly named, rounded columns.
- [ ] `solutions.py` has Tasks 4–10, with Task 9's argument written out as a comment or short paragraph, not just numbers.
- [ ] Your SQL and Python EV/EBITDA summary statistics agree with each other.
- [ ] You can state, without looking it up, why trading comps use NTM figures rather than LTM (tie back to Lecture 2, Section 3).

## Stretch

- Add a Task 11: build a small football-field-style summary — a DataFrame or printed table with one row per method (`EV/EBITDA trimmed`, `EV/Revenue trimmed`, `EV/EBITDA excl. VDT`, etc.) and columns `implied_ev_low`, `implied_ev_high`, `per_share_low`, `per_share_high`. This is the exact shape of table Challenge 1's football field builds on top of.
- Add a Task 12: instead of a flat trim, compute an EBITDA-margin-weighted average multiple — weight each comp's `ev_ebitda` by how close its own implied EBITDA margin (`ebitda_ntm / revenue_ntm`) is to Crunch Machine Co.'s FY2026 margin (21.0%), giving more weight to genuinely similar peers. Does this change the implied EV range much versus the simple trimmed range?

## Submission

Commit `solutions.sql` and `solutions.py` to your portfolio under `c35-week-07/exercise-03/`.
