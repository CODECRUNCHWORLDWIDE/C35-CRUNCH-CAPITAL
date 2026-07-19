# Exercise 3 — Compute the True Cost of Each Financing Path

**Goal:** Turn Lecture 3's side-by-side cost comparison into reusable Python functions, then use them to answer a question the lecture didn't: at what cost-of-equity assumption would the two paths cost the *same* amount per year?

**Estimated time:** 60 minutes.

## Setup

You'll need your `loan_schedule` function from Exercise 2 and the deal terms from Exercise 1. If you haven't saved them as importable functions yet, do that first — copy them into a `financing.py` module you can `import` from here and from the mini-project.

```python
from financing import loan_schedule, price_offering  # your Exercise 1 & 2 functions
```

Create a file `solutions.py`. Put each task under a `# Task N` comment.

## Tasks

1. **After-tax cost of debt, as a function.** Write `after_tax_cost_of_debt(coupon_rate, tax_rate)`, implementing Lecture 3, Section 2's formula. Confirm `after_tax_cost_of_debt(0.075, 0.25)` returns `0.05625`.

2. **Total after-tax dollar cost of the loan.** Using your `loan_schedule` function from Exercise 2, sum `interest_paid` across all 5 years, apply the tax shield, and add the $200,000 origination fee (untaxed, per Lecture 3's simplification). Confirm you land on approximately **$3,737,353.84** total, over the loan's life.

3. **Annual cost of equity capital, as a function.** Write `annual_cost_of_equity(net_proceeds, cost_of_equity_rate)`. Using `net_proceeds` from your Exercise 1 `price_offering` function and a cost of equity of **11%**, confirm you get approximately **$2,200,001.51** per year.

4. **Normalize to a 5-year horizon.** Debt's cost ends after 5 years; equity's doesn't. To compare them honestly, compute **total cost over exactly 5 years** for both paths: debt's is your Task 2 total (already a 5-year total); equity's is `5 × annual_cost_of_equity(...)` from Task 3. Which is bigger, and by how much, in dollars, over those same 5 years?

5. **Extend the horizon to 10 years.** Debt is fully repaid after year 5 — assume the company simply pays no further financing cost on this loan after that (a real company would likely refinance or take a new loan, but ignore that for this exercise). Equity's cost keeps accruing at 11%/year for the full 10 years. Recompute total 5-year-debt-cost vs. total 10-year-equity-cost. Does debt's cost advantage over equity get bigger or smaller as the horizon lengthens, and why does that make sense given that debt is temporary and equity is permanent?

6. **Solve for the break-even cost of equity.** At what `cost_of_equity_rate` would the 5-year total cost of equity (Task 4's equity side) exactly equal the 5-year total cost of debt (Task 2's answer)? You can solve this algebraically (rearrange `5 × rate × net_proceeds = debt_total_cost` for `rate`) or by trying values in a small loop until you're within a few dollars. Report the break-even rate as a percentage.

## Expected results (spot checks)

- Task 1 → `0.05625`.
- Task 2 → total after-tax cost ≈ `$3,737,353.84`.
- Task 3 → ≈ `$2,200,001.51`/year.
- Task 4 → 5-year equity cost (≈ `$11,000,007.55`) is substantially **larger** than 5-year debt cost (≈ `$3,737,353.84`) — debt is the cheaper path over this horizon, by a wide margin.
- Task 6 → the break-even cost of equity should come out well **below** 11% (work the algebra — it's under 4%), meaning: at Crunch Machine Co.'s actual 11% cost of equity, debt is decisively cheaper on cost alone. Equity would only tie debt on cost if shareholders required a dramatically lower return than they actually do.

## Done when…

- [ ] `solutions.py` runs top to bottom with no errors and prints all 6 tasks' results.
- [ ] `after_tax_cost_of_debt` and `annual_cost_of_equity` are genuinely reusable — no hardcoded Crunch Machine Co. numbers inside either function body.
- [ ] You can state, in one sentence, why extending the horizon (Task 5) widens rather than narrows debt's cost advantage.
- [ ] You can explain, in your own words, why the answer to "which is cheaper" alone still isn't a complete recommendation — tie this back to Lecture 3, Section 8's four-part framework.

## Stretch

- Add a `Task 7`: what cost-of-equity rate would make the two paths cost the *same* over a **10-year** horizon instead of 5? Is the break-even rate higher or lower than Task 6's 5-year break-even, and does that match your intuition from Task 5?

## Submission

Commit `solutions.py` (and your shared `financing.py` module) to your portfolio under `c35-week-06/exercise-03/`.
