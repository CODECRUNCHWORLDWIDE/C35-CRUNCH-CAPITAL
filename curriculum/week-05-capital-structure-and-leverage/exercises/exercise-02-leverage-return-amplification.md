# Exercise 2 — Leverage Return Amplification

**Goal:** Build a complete DFL and ROE-amplification table across a grid of scenarios in pandas, and confirm the mechanical relationship between EBIT swings and equity-return swings holds exactly as Lecture 3 derived it.

**Estimated time:** 90 minutes.

## Setup

Have Lecture 3 open beside you — you'll need the DFL formula and the unlevered-vs-levered ROE worked example (Section 5). Create `amplification.py`.

## Part A — DFL for Crunch Machine Co., all five years

1. **Task 1.** Write a function `dfl(ebit: float, interest: float) -> float` returning `EBIT / (EBIT − interest)`. Guard against division by zero (raise a clear `ValueError` if `EBIT == interest`, since that's the point where DFL is undefined/infinite).

2. **Task 2.** Pull (or hardcode, with a comment noting you did so) Crunch Machine Co.'s five years of `(fiscal_year, operating_income, interest_expense)`. Build a DataFrame with a `dfl` column computed by your Task 1 function for every year.

3. **Task 3.** Add a column `pct_ebit_swing_needed_for_100pct_ni_swing` — the percentage EBIT change that, at each year's DFL, would produce exactly a 100% swing in net income (`= 1 / dfl`, expressed as a percent). What happened to this number between 2021 and 2025? What does a *smaller* number mean about how fragile the company's earnings became?

## Part B — Unlevered vs. levered scenario table (general engine)

Build a reusable version of Lecture 3, Section 5's worked example — one that works for **any** asset base, debt level, and EBIT scenario, not just the lecture's specific numbers.

4. **Task 4.** Write a function:

```python
def scenario_table(total_assets: float, debt: float, cost_of_debt: float,
                    tax_rate: float, ebit_scenarios: dict[str, float]) -> "pd.DataFrame":
    """
    For each named EBIT scenario, compute net income and ROE for both an
    all-equity (unlevered) version of the firm and the actual levered
    version, holding total_assets fixed across both.
    Returns a DataFrame with columns: scenario, ebit, roe_unlevered, roe_levered.
    """
    ...
```

5. **Task 5.** Call `scenario_table` with `total_assets=20000`, `debt=10000`, `cost_of_debt=0.07`, `tax_rate=0.25`, and the three scenarios from Lecture 3 (`EBIT` = 2000, 2500, 3000). Confirm your output matches Lecture 3's printed table exactly.

6. **Task 6.** Call `scenario_table` again with the **same** parameters except `debt=15000` (more levered — equity is now only `5000`). Compare the ROE range (max minus min across the three scenarios) to Task 5's range. Is the spread bigger or smaller with more debt? By roughly what factor?

## Part C — Crunch Machine Co. itself, EBIT-shock scenarios

7. **Task 7.** Using Crunch Machine Co.'s **actual FY2025** balance sheet (`total assets = 46,350`, `total debt = 19,700`, embedded `rD = interest_expense / total_debt ≈ 7.0%`, `Tc = 0.25`), build a 5-row scenario table with EBIT at `−30%`, `−15%`, `0%` (base = 2,225), `+15%`, and `+30%` relative to the FY2025 actual. Show `ebit`, `pretax_income`, `net_income`, and `roe` (using FY2025's actual `total_equity = 19,010` as the equity base) for every row.

8. **Task 8.** At what EBIT percentage change (to the nearest 1%) does `pretax_income` first turn **negative** in your Task 7 table? *(You may need to add finer-grained scenarios between your existing rows to narrow it down, or solve it directly: `pretax = 0` when `EBIT = interest = 1,380`, so what percentage decline from `2,225` is that?)*

## Expected results (spot checks)

- Task 1/2, FY2025 → `dfl ≈ 2.63`; FY2021 → `dfl ≈ 1.10`.
- Task 5 → `roe_levered` values of `0.0975`, `0.1350`, `0.1725` for the three scenarios, matching Lecture 3 exactly.
- Task 6 → the `debt=15000` version should show a **wider** ROE range than the `debt=10000` version — more leverage, more amplification.
- Task 8 → the breakeven EBIT decline should be very close to **38%** (`(2225 − 1380) / 2225 ≈ 0.3798`).

## Done when…

- [ ] `dfl()` correctly raises on the undefined case (`EBIT == interest`).
- [ ] `scenario_table()` works for **any** inputs you pass it — proven by running it twice with different debt levels and getting sensible, different output both times.
- [ ] Task 7's table is printed in full, all 5 rows, all 4 columns.
- [ ] You can state, in one sentence, the approximate EBIT decline that would zero out Crunch Machine Co.'s FY2025 pretax income.

## Stretch

- Add a Task 9: extend `scenario_table` to also return a `dfl` column per scenario (DFL technically changes slightly as EBIT changes, since `pretax income` is the denominator). How much does DFL itself vary across your Task 7 EBIT range? Is the variation large enough to matter for a rough estimate, or is treating DFL as roughly constant across a plausible EBIT range a reasonable simplification?

## Submission

Commit `amplification.py` to your portfolio under `c35-week-05/exercise-02/`.
