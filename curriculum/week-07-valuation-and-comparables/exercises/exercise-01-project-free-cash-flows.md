# Exercise 1 — Project Free Cash Flows

**Goal:** Turn `revenue_forecast`'s driver assumptions into a full five-year unlevered free cash flow forecast for Crunch Machine Co. — in SQL and in Python — and get comfortable with the fact that a real forecast is built from assumptions, not handed to you as a finished number.

**Estimated time:** 90 minutes.

## Setup

You need the `valuation_assumptions` and `revenue_forecast` tables from this week's [README](../README.md), and a Python environment with `pandas` and `sqlalchemy`. Have Lecture 1 open beside you — Sections 2–5 walk through exactly this calculation.

Create two files: `solutions.sql` and `solutions.py`. Put each task under a numbered comment (`-- Task N` / `# Task N`).

## Part A — By hand, one year (show your work)

In a `hand-work.md` file, compute **FY2027** the same way Lecture 1, Section 3 computed FY2026 — show every line.

1. **Task 1.** Revenue: compound FY2026's revenue ($226,800,000) forward by FY2027's growth rate.
2. **Task 2.** EBITDA, D&A, and EBIT for FY2027, using FY2027's `ebitda_margin` and `da_pct_revenue`.
3. **Task 3.** NOPAT (25% tax rate), then add back D&A, subtract capex (`capex_pct_revenue`) and the change in NWC (`nwc_pct_incremental_revenue` × the dollar increase over FY2026's revenue). Report the final unlevered FCF.

Your Task 3 answer should be within a few hundred dollars of **$27,953,100** — if it isn't, recheck which year's revenue you used as the "prior" revenue for the NWC calculation (it must be FY2026's, not FY2025's).

## Part B — In SQL, all five years

4. **Task 4.** Write the recursive CTE from Lecture 1, Section 4 (or your own equivalent approach) to compound revenue across all five forecast years, and extend the final `SELECT` to return `fiscal_year`, `revenue`, `ebitda`, `da`, `ebit`, `nopat`, `capex`, `delta_nwc`, and `unlevered_fcf` as named, rounded columns, one row per year.
5. **Task 5.** Add a query that returns the **cumulative** unlevered FCF across all five years (a single number) — the sum of Task 4's `unlevered_fcf` column.
6. **Task 6.** Add a query that computes the **compound annual growth rate (CAGR)** of `unlevered_fcf` from FY2026 to FY2030: `CAGR = (FCF_2030 / FCF_2026)^(1/4) − 1`. Most SQL engines don't have a built-in fractional-power root — use `POWER(FCF_2030 / FCF_2026, 1.0/4)` (Postgres) or the equivalent in your engine.

## Part C — In Python

7. **Task 7.** Write the loop-based forecast function from Lecture 1, Section 5. Pull `revenue_forecast` into a DataFrame with `pandas.read_sql`, run the loop, and print the resulting DataFrame rounded to whole dollars.
8. **Task 8.** Confirm your Python DataFrame's `unlevered_fcf` column matches your SQL Task 4 output for all five years (within $1 of rounding difference). Print a clear pass/fail for each year.
9. **Task 9.** Compute the same CAGR from Task 6, but in Python using your DataFrame instead of SQL. Confirm it matches Task 6's SQL answer.

## Expected results (spot checks)

- Task 1 → Revenue ≈ **$242,676,000**.
- Task 3 → Unlevered FCF ≈ **$27,953,100**.
- Task 4, full table → matches Lecture 1, Section 3's table exactly for all five years; FY2030 unlevered FCF ≈ **$40,865,886**.
- Task 5 → cumulative unlevered FCF across FY2026–2030 ≈ **$160,595,038**.
- Task 6 / Task 9 → CAGR ≈ **15.5%** (unlevered FCF grows meaningfully faster than revenue's own ~5.5% CAGR over the same four years — that's the margin-expansion and capex-tapering story from Lecture 1 compounding into the cash-flow line).

## Done when…

- [ ] `hand-work.md` shows the full FY2027 arithmetic, not just the final answer.
- [ ] `solutions.sql` has Tasks 4–6, each returning correctly named, rounded columns.
- [ ] `solutions.py` has Tasks 7–9, with a clear pass/fail printed for Task 8's cross-check.
- [ ] Your SQL and Python forecasts agree with each other and with Lecture 1's table, for every year.
- [ ] You can explain, out loud, why unlevered FCF's growth rate is so much higher than revenue's — not just that it is.

## Stretch

- Add a Task 10: build a second, more conservative scenario where `ebitda_margin` stays flat at FY2026's 21.0% for all five years instead of expanding, and `capex_pct_revenue` stays flat at 5.5% instead of tapering. Recompute the five-year forecast under this scenario and report the FY2030 unlevered FCF and the cumulative total. How much smaller is the cumulative FCF than the base case? This is the exact kind of "what if the story doesn't play out" question Challenge 1's sensitivity work generalizes.
- Add a Task 11: write a single parameterized Python function `build_forecast(ltm_revenue, drivers_df, tax_rate)` that reproduces Task 7's output but accepts *any* starting revenue and driver table — not hardcoded to Crunch Machine Co.'s numbers. Confirm it reproduces Task 7's results when called with this week's actual inputs.

## Submission

Commit `hand-work.md`, `solutions.sql`, and `solutions.py` to your portfolio under `c35-week-07/exercise-01/`.
