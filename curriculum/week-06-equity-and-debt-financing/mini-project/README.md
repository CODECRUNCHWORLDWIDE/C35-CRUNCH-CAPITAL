# Mini-Project — Model a Financing Round Two Ways, and Recommend One

> Model Crunch Machine Co.'s $20,000,000 financing need as a full equity term sheet **and** a full debt term sheet, in SQL and Python, then put a real, numbers-backed recommendation in front of the board: which one should they sign, and why. This is the week's capstone — proof that the dilution math, the amortization engine, and the cost/control/flexibility framework from all three lectures compose into an actual financing decision, not three separate homework answers.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

A CFO doesn't get to hand the board "well, it's complicated" — they get one meeting, real numbers, and a recommendation they can defend when someone on the board pushes back. That's what you're building today.

---

## Deliverable

A directory in your portfolio `c35-week-06/mini-project/` containing:

1. `financing.py` — a single module with every reusable function from the week (share-issuance pricing, dilution table, loan amortization schedule, after-tax cost of debt, annual cost of equity), consolidated and importable.
2. `model_tests.py` — a set of checks proving the module is correct (see Part 4).
3. `board_memo.md` — the actual recommendation, written as if for Crunch Machine Co.'s board (see Part 5).
4. `schema.sql` — every table this project reads from or writes to: `company_snapshot`, `shareholders`, `financing_terms` (given), plus any new tables you create (`term_loan_schedule`, a dilution table, a cost-comparison table).

Everything runs against the seed tables from the [week README](../README.md), on PostgreSQL or SQLite — note which you used.

---

## Part 1 — The equity path, fully modeled

Using `financing.py`:

1. Price the follow-on offering exactly as in Lecture 1 / Exercise 1: whole-share count, actual gross, fee, and net proceeds.
2. Build the complete pre/post ownership table for all six existing shareholders **plus** the new investors, with `pct_before`, `pct_after`, and `pct_point_dilution` for each.
3. Store the post-raise cap table in a new SQL table `post_raise_cap_table_equity` (columns: `shareholder_name`, `shares_held`, `pct_before`, `pct_after`).

## Part 2 — The debt path, fully modeled

1. Build the complete 5-year amortization schedule for the $20,000,000, 7.5% term loan, and store it in `term_loan_schedule` (same shape as Exercise 2).
2. Compute both covenant ratios (leverage, interest coverage) at closing, and confirm both clear their thresholds with room to spare.
3. Report total interest paid over the loan's life, and the total after-tax cost including the origination fee.

## Part 3 — The true cost comparison

1. Compute the after-tax cost of debt (%) and the annual + 5-year total dollar cost of the debt path.
2. Compute the annual + 5-year total dollar cost of the equity path, using an 11% cost of equity.
3. Store both in a table `financing_cost_comparison` (columns: `option_name`, `annual_cost_pct`, `annual_cost_dollars`, `five_year_total_cost`, `is_permanent`) — `is_permanent` should be `TRUE` for equity, `FALSE` for debt, capturing Lecture 3's point that the two costs aren't the same *kind* of obligation even when they're expressed in the same units.

## Part 4 — Prove it's correct

In `model_tests.py`, verify your module against at least these five known cases:

1. `price_offering(20_000_000, 19.00, 0.05)` should return `shares_issued == 1108034` and `net_proceeds` within a cent of `20000013.70`.
2. `loan_schedule(20_000_000, 0.075, 5)`'s final row should have `ending_balance` within a fraction of a cent of `0`, and total `interest_paid` within a cent of `4716471.78`.
3. `after_tax_cost_of_debt(0.075, 0.25)` should equal `0.05625` exactly.
4. The sum of every shareholder's (including new investors') `pct_after` in your Part 1 dilution table should equal `100.0` within a hundredth of a percentage point.
5. `leverage_ratio` and `interest_coverage_ratio`, run on the post-raise debt scenario, should match Lecture 2's baseline numbers (`≈2.273` and `≈5.455`).

Print a clear pass/fail for each check.

## Part 5 — Write the board memo

In `board_memo.md`, write the actual recommendation — treat this as the real deliverable, not an afterthought:

- **The ask:** one paragraph stating the $20,000,000 need and what it funds (the Midwest distribution hub).
- **Two term sheets, side by side:** a table (built from your `financing_cost_comparison`) showing net proceeds, upfront fee, annual cost, 5-year total cost, whether the obligation is permanent or finite, and whether it dilutes control.
- **The dilution cost, made concrete:** report the Founder & CEO's and the combined founders' ownership before and after the equity path, in both percentage and (using the simplified post-raise valuation approach from Lecture 1, Section 5) approximate dollar terms.
- **The covenant cost, made concrete:** report both covenant ratios at closing for the debt path, and how much cushion each has before breaching (tie back to Challenge 1 if you did it — otherwise, state it directly from Part 2).
- **Your recommendation:** pick one path (or defend a specific mixed split, if you did Challenge 2), and justify it using **all four** of Lecture 3, Section 8's axes — cost, capacity, control, and flexibility/signaling — not just whichever number is smallest. A recommendation that only cites cost is an incomplete recommendation, even if it's the "obvious" answer.
- **One risk you'd flag to the board** that this simplified model doesn't capture (e.g., market reaction to the equity announcement, refinancing risk if rates rise before the loan term ends, the possibility the hub project underperforms and strains debt service).

---

## Rules

- **Every function must be genuinely reusable** — no hardcoded Crunch Machine Co. constants inside `financing.py`'s function bodies (the constants belong in the calling code, not the function). If you can't run `price_offering` or `loan_schedule` on a different company's numbers without editing the function, they're not done.
- **No spreadsheets**, per this week's data tooling rule — every table lives in SQL, every calculation in Python.
- **Show your verification.** A financing model you haven't tested against known answers is not one a board should trust — Part 4 isn't optional polish, it's the proof the memo depends on.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 30% | All five Part 4 checks pass; both paths' numbers match the lectures' worked examples |
| Reusability | 20% | `financing.py` functions work on any valid inputs, not just Crunch Machine Co.'s numbers |
| Database integration | 15% | Every derived table (`term_loan_schedule`, cap table, cost comparison) is genuinely stored in SQL and queryable |
| Memo quality | 25% | Recommendation explicitly addresses all four of cost, capacity, control, and flexibility/signaling — not cost alone |
| Risk awareness | 10% | The named model limitation in Part 5 is specific and real, not generic hedging |

---

## Why this matters

This is the shape of a real financing decision: two fully-priced term sheets, a cost comparison that accounts for taxes and time horizon, and a recommendation that weighs more than the cheapest number on the page. Every remaining week of this course that touches financing — M&A financing mixes in Week 10, startup rounds in Week 11 — reuses exactly this pattern: model each option completely, compare on multiple axes, and write the recommendation down where someone else can check your reasoning, not just your arithmetic.

When done: take the [quiz](../quiz.md) and start [Week 7 — Valuation: DCF and trading comparables](../../week-07-valuation-dcf-and-comps/).
