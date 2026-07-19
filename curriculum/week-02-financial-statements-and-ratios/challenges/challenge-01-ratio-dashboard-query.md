# Challenge 1 — Ship a Ratio Dashboard Query

**Time:** ~60 minutes. **Difficulty:** Medium.

## The scenario

Your manager doesn't want twelve separate queries emailed to her one at a time. She wants **one query** she can run — or hand to a BI tool, or paste into a scheduled report — that returns a complete ratio dashboard for Crunch Machine Co.: one row per fiscal year, every major ratio as a column, ready to scan top to bottom and see the whole five-or-six-year story at a glance. That's what you're shipping.

## Your task

Write **one** SQL query, against `income_statement`, `balance_sheet`, and `cash_flow_statement`, that returns exactly one row per `fiscal_year` with the following columns, in this order:

**Liquidity**
- `current_ratio`
- `quick_ratio`
- `cash_ratio`

**Leverage**
- `debt_to_equity`
- `debt_to_assets`
- `interest_coverage`

**Profitability**
- `gross_margin_pct`
- `operating_margin_pct`
- `net_margin_pct`
- `roa_pct`
- `roe_pct`

**Efficiency**
- `dso_days`
- `dio_days`
- `dpo_days`
- `cash_conversion_cycle_days`

Round every ratio to a sensible precision (2 decimals for ratios expressed as multiples, 1 decimal for percentages and day-counts). Order the result by `fiscal_year` ascending.

## Constraints

- **One query.** You may use CTEs (`WITH`) freely — that's expected, and the cleanest way to build this without repeating the same joins fourteen times — but the final result must come from a single `SELECT` statement, not fourteen separate queries pasted together.
- It must run against **all six years** (FY2020–FY2025) once Exercise 1's row is loaded, with no year silently dropped by a bad join.
- Every column must be independently correct — spot-check at least three columns per year against your Exercise 3 answers before you call this done.
- Alias every column exactly as named above (`current_ratio`, not `curr_ratio` or `CurrentRatio`) — a real dashboard consumer (a BI tool, a teammate's script) depends on stable column names.

## Hints

<details>
<summary>On structuring the CTEs</summary>

A clean shape: one CTE that joins all three tables together on `fiscal_year` (call it `base`), then a final `SELECT` off that CTE computing every ratio as a derived column. You don't need a separate CTE per ratio family — one wide `base` CTE with every raw figure you need, then one arithmetic-only final `SELECT`, is usually the most readable version.

</details>

<details>
<summary>On rounding without breaking readability</summary>

Wrapping every single expression in `ROUND(...)` makes the query hard to read. Consider computing the raw (unrounded) ratio in the `base` CTE or an intermediate CTE, then rounding only in the final `SELECT` — it keeps the arithmetic and the presentation cleanly separated.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Completeness | Missing ratio families or years | All 14 ratios, all 6 years, no gaps |
| Correctness | A ratio or two silently wrong (bad join, wrong sign) | Every column verified against Exercise 3 |
| Structure | Copy-pasted joins repeated per ratio | One clean `base` CTE, arithmetic-only final `SELECT` |
| Readability | Cryptic aliases, inconsistent rounding | Exact column names as specified, consistent precision |
| Usability | Query only works with values you hardcoded | Genuinely reusable — would work unchanged if a 7th year were added |

## Submission

Commit your query as `dashboard.sql` to your portfolio under `c35-week-02/challenge-01/`. Also save its output — run it and paste the result as a Markdown table into `dashboard-output.md` in the same folder, so your dashboard's *answer*, not just its SQL, is reviewable at a glance.
