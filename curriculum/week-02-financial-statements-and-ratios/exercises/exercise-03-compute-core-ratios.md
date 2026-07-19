# Exercise 3 — Compute the Core Ratio Set

**Goal:** Compute every ratio from [Lecture 3](../lecture-notes/03-ratios-that-predict-trouble.md) — liquidity, leverage, profitability, and efficiency — across all six years of Crunch Machine Co.'s history, and practice reading a trend instead of a single number.

**Estimated time:** 90 minutes.

## Setup

Confirm all three tables hold six years (FY2020–FY2025). Create a working file `solutions.sql`.

## Tasks

### Liquidity

1. **Current ratio**, all six years, rounded to 2 decimals.
2. **Quick ratio**, all six years, rounded to 2 decimals.
3. **Cash ratio**, all six years, rounded to 2 decimals.
4. **One combined query** returning `fiscal_year, current_ratio, quick_ratio, cash_ratio` in a single result set, all six years, one row per year.

### Leverage

5. **Debt-to-equity**, all six years, rounded to 2 decimals. (`total_debt` = `short_term_debt + long_term_debt`.)
6. **Debt-to-assets**, all six years, rounded to 3 decimals.
7. **Interest coverage** (`operating_income / interest_expense`), all six years, rounded to 1 decimal. *(Note: FY2020's interest expense is on file from Exercise 1 — this ratio should compute for all six years, not just five.)*

### Profitability

8. **Gross, operating, and net margin**, all six years, each as a percentage rounded to 1 decimal, in one query.
9. **ROA and ROE**, all six years, each as a percentage rounded to 1 decimal, in one query.

### Efficiency

10. **DSO, DIO, and DPO**, all six years, each rounded to 1 decimal, in one query.
11. **Cash conversion cycle** (`DSO + DIO - DPO`), all six years, rounded to 1 decimal.

### Reading the trend

12. **Year-over-year change.** Pick **any one** ratio you computed above and write a query using `LAG()` that shows, for each year, the ratio's value **and** its change from the prior year, the same pattern Lecture 3 Section 6 demonstrated for the current ratio.
13. **The steepest single-year drop.** Across the ratio you chose in Task 12, which single year-over-year change was the largest (in absolute value)? Write a query that identifies it — don't just eyeball the output.

## Expected result (spot checks)

- Task 1 → FY2020 current ratio ≈ `2.89`; FY2025 ≈ `1.73`.
- Task 3 → FY2020 cash ratio ≈ `1.25`; FY2025 ≈ `0.13`.
- Task 5 → FY2020 debt-to-equity ≈ `0.36`; FY2025 ≈ `1.04`.
- Task 7 → FY2020 interest coverage ≈ `13.6×`; FY2025 ≈ `1.6×`.
- Task 8 → FY2020 net margin ≈ `13.2%`; FY2025 net margin ≈ `1.4%`.
- Task 10 → FY2021 DSO ≈ `40.0` days; FY2025 DSO ≈ `62.3` days.

## Done when…

- [ ] All 13 tasks return correct, rounded results across all six years (or five, where a `LAG()`-based query legitimately drops the first year).
- [ ] Task 4's combined liquidity query and Task 8/9/10's combined queries each return exactly one row per fiscal year, not one row per ratio.
- [ ] You can state, from memory and without looking it up, which of the four ratio families (liquidity, leverage, profitability, efficiency) each of your 11 computed ratios belongs to.
- [ ] Task 13 correctly identifies the single steepest year-over-year move for your chosen ratio, with a query — not a guess.

## Stretch

- Compute the **dividend payout ratio** (`dividends_paid` — take its absolute value — ÷ `net_income`, expressed as a percentage) for all six years using data from `cash_flow_statement` joined to `income_statement`. Watch what happens to this ratio in the final two years — it's the single loudest number in this entire dataset, and you'll use it directly in Challenge 2.
- In Python (pandas), pull the three tables with `pd.read_sql`, compute the same current ratio and interest coverage series as pandas Series, and plot neither of them — just print the six values in order and confirm they match your SQL output exactly. (You'll do real plotting with `matplotlib` starting in a later week; this week the goal is trusting that SQL and pandas agree, not visualization.)

## Submission

Commit `solutions.sql` (and, if you did the Python stretch, `ratios.py`) to your portfolio under `c35-week-02/exercise-03/`.
