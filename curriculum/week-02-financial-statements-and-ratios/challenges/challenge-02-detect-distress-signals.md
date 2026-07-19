# Challenge 2 — Flag a Company Heading for Distress

**Time:** ~60 minutes. **Difficulty:** Hard. **Thresholds require judgment — defend yours.**

## The scenario

A ratio dashboard (Challenge 1) is useful, but it still requires a human to scan fourteen columns across six years and notice the pattern. Real credit and equity analysts don't do that by eye at scale — they build **rule-based flags**: a query that automatically marks which years cross a danger threshold, so the *dashboard tells you where to look* instead of you hunting for it. That's what you're building this time: not a fresh set of ratios, but a layer of judgment on top of the ratios you already have.

## Your task

Working from the dashboard you already built (or the individual queries from Exercise 3), write SQL that produces **one row per fiscal year** with a boolean (or `0`/`1`) flag column for each of the following five distress signals, plus a final `red_flag_count` column summing how many fired that year:

1. **`flag_liquidity_declining`** — the current ratio *and* the quick ratio have both fallen for **three consecutive years** as of this year (i.e., this year's value is lower than last year's, which was lower than the year before's, for both ratios at once).
2. **`flag_low_interest_coverage`** — interest coverage has fallen below **3.0×**.
3. **`flag_high_leverage`** — debt-to-equity has crossed above **0.75**.
4. **`flag_unsustainable_payout`** — the dividend payout ratio (dividends paid, as a positive number, ÷ net income) exceeds **100%** — the company is paying out more in dividends than it earned that year.
5. **`flag_working_capital_deterioration`** — the cash conversion cycle has grown by more than **10 days** compared to two years prior.

Then add:

6. **`red_flag_count`** — an integer, `0` to `5`, counting how many of the five flags are `true` for that year.

Order by `fiscal_year`. Every flag must be computed from a query — no hardcoded year-to-flag mapping.

## Constraints

- Every threshold above (`3.0×`, `0.75`, `100%`, `10 days`, `3 consecutive years`) is a judgment call the challenge is handing you pre-made, but you still need to **defend it**: in a short `rationale.md`, explain in one or two sentences per flag why that specific threshold is a reasonable line between "concerning" and "not yet concerning" for a manufacturing company like Crunch Machine Co. (Real analysts vary these by industry — a bank runs on far more leverage than a manufacturer, for instance. State that context for at least one flag.)
- Use window functions (`LAG()`, possibly nested two periods back) rather than self-joins or hardcoded year comparisons — this is exactly the pattern Lecture 3 and Exercise 3 built toward.
- Your query must work unchanged if a 7th year of clean data were appended — no `WHERE fiscal_year = 2025` shortcuts.

## What the data should show you

Run your finished query and look at the `red_flag_count` column across the six years. You should see a company that starts with **zero or one** flag firing and ends with **most or all five** firing — and the *year each individual flag first turns on* should roughly line up with the order things went wrong: leverage and payout signals typically fire before the multi-year liquidity-decline flag can even be evaluated (it needs three years of history to trigger at all). If your query shows every flag firing in FY2020, or none ever firing, something in your `LAG()` logic is likely off — go back and check it against your Exercise 3 numbers.

## Hints

<details>
<summary>On the three-consecutive-years flag</summary>

You need two levels of `LAG()`: `LAG(current_ratio, 1) OVER (...)` for last year and `LAG(current_ratio, 2) OVER (...)` for two years back. The flag is true when `current_ratio < lag_1 AND lag_1 < lag_2` (strictly decreasing for three points), evaluated for **both** current ratio and quick ratio in the same row, `AND`ed together. The first two years in your dataset can't have this flag evaluate at all — decide explicitly whether that should read as `false` or `NULL`, and say which you chose and why in `rationale.md`.

</details>

<details>
<summary>On the payout ratio sign</summary>

`dividends_paid` is stored as a negative number in `cash_flow_statement`. Take its absolute value (`ABS(dividends_paid)`) before dividing by `net_income`, or the ratio comes out negative and the `> 100%` comparison is meaningless.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Correctness | A flag fires on the wrong year, or never fires | Every flag's timing matches a hand-checked spot value |
| Window function use | Hardcoded per-year `CASE WHEN fiscal_year = ...` | Genuine `LAG()`-based logic that generalizes to new years |
| Judgment | Thresholds copied with no explanation | `rationale.md` defends each threshold with a real reason |
| Synthesis | Five flags computed but never combined | `red_flag_count` correctly sums all five per year |
| Honesty about limits | Claims certainty a 6-year, 1-company dataset can't support | Notes, in `rationale.md`, that these thresholds are illustrative and a real credit analysis would calibrate them against industry peers, not just one company's own history |

## Submission

Commit `distress-flags.sql` and `rationale.md` to your portfolio under `c35-week-02/challenge-02/`.
