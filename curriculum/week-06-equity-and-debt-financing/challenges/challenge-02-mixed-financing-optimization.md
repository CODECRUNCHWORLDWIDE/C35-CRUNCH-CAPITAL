# Challenge 2 — Mixed Financing Optimization

**Time:** ~90 minutes. **Difficulty:** Medium-high. **No single right implementation.**

## The scenario

The board comes back with a complication: the lead lender's credit committee will only underwrite a term loan that keeps Crunch Machine Co.'s **total** leverage at or below **2.00× Debt/EBITDA** — a more conservative internal risk policy than the loan's own 3.00× default covenant, and a common real-world gap between "what triggers default" and "what a lender is actually willing to originate." At the company's current $22,000,000 EBITDA and $30,000,000 of existing debt, that caps how much *new* debt the lender will provide — and it's less than the full $20,000,000 the Midwest hub needs. **The company has to fund the shortfall with equity, whether it wants to or not.** Your job: figure out exactly how much of each, and prove it's the cheapest way to close the gap.

## What "optimizing" the mix means here

Lecture 3 showed debt is cheaper than equity, dollar for dollar. Left unconstrained, the cheapest financing plan is "borrow the entire $20,000,000." The lender's cap makes that impossible. The optimization problem this challenge asks you to solve is: **use as much of the cheap instrument (debt) as the constraint allows, and fund only the unavoidable remainder with the expensive one (equity)** — and then prove, with numbers, that this is actually cheaper than other mixes a less careful analyst might propose (an even split, or leaning further toward equity "for safety").

## Your task

### Part 1 — Find the maximum debt the lender will provide

Using `company_snapshot`'s `ebitda` and `existing_debt`, and the lender's 2.00× total-leverage cap:

```
Max total debt = 2.00 × EBITDA
Max new debt   = Max total debt − Existing debt
```

Write this as a function `max_new_debt(ebitda, existing_debt, leverage_cap)` and confirm it returns **$14,000,000** for this week's numbers.

### Part 2 — Split the $20,000,000 need

The remainder must come from equity:

```
Debt portion   = max_new_debt(...)             # $14,000,000
Equity portion = Total need − Debt portion       # $6,000,000 (net)
```

Confirm your split sums back to $20,000,000.

### Part 3 — Price each piece

Reuse your Exercise 1 (`price_offering`) and Exercise 2 (`loan_schedule`) functions, unmodified, on the new amounts:

1. Price a $14,000,000 term loan at the same 7.5% coupon, 5-year term, 1% origination fee. Build its full amortization schedule and report total interest paid over 5 years.
2. Price a $6,000,000 (net) equity offering at the same $19.00 offer price and 5% underwriting fee as Lecture 1. Report the exact share count issued (whole shares, rounded up) and the actual net proceeds.
3. **Verify the covenant math still holds** at the new $14,000,000 debt level: recompute leverage (should now be well under both the lender's 2.00× cap and the loan's 3.00× default covenant) and interest coverage (should still clear the 4.00× minimum). If either check fails, you've made an arithmetic error upstream — find it before continuing.

### Part 4 — Compute the blended true cost, and compare

Using Exercise 3's `after_tax_cost_of_debt` and `annual_cost_of_equity` functions (reused, not rewritten):

1. Compute the 5-year after-tax cost of the $14,000,000 debt piece (interest + origination fee, tax-shielded per Lecture 3).
2. Compute the 5-year cost of the $6,000,000 equity piece (`5 × annual_cost_of_equity(...)`).
3. Sum them for the **blended 5-year true cost** of this mixed raise.
4. Compare that blended cost to two reference points: the **all-equity** 5-year cost from Exercise 3 (funding the full $20,000,000 with equity alone), and the **unconstrained all-debt** 5-year cost from Exercise 3 (what it *would* cost if the lender allowed the full $20,000,000 as debt — which they don't, but the number is a useful lower bound). Where does the mixed plan's cost fall relative to those two?

### Part 5 — Prove it's actually optimal, not just feasible

A skeptical board member asks: "why not split it 50/50 instead, or lean more on equity to be conservative?" Test **two alternative splits** — a $10,000,000 debt / $10,000,000 (net) equity split, and a $6,000,000 debt / $14,000,000 (net) equity split — through the same Part 3–4 pipeline. For each, confirm the leverage covenant still holds (it will — both use less debt than the $14M cap), and compute each alternative's blended 5-year true cost. Rank all three splits (the maximum-debt plan from Parts 1–4, the 50/50 split, and the debt-light split) from cheapest to most expensive.

## Constraints

- Every function must be **reused from Exercises 1–3, unmodified** — if you find yourself rewriting `price_offering` or `loan_schedule` with different logic for this challenge, stop; the whole point is proving those functions generalize to a new deal size without changes.
- No spreadsheet, per this course's data tooling rule.
- Show all three splits' blended costs side by side in your writeup, not just the winning one — the comparison is the proof, not the final number alone.

## Hints

<details>
<summary>On why the maximum-debt plan should win</summary>

If debt's true annual cost (5.625% after tax) is lower than equity's (11%) at every dollar amount — and nothing in this week's model says that relationship changes with size — then any dollar moved from the debt piece to the equity piece, versus the maximum-debt plan, strictly increases the blended cost. The three-way ranking in Part 5 should confirm this pattern; if it doesn't, look for an arithmetic error before concluding the theory is wrong.

</details>

<details>
<summary>On the lender's 2.00× cap vs. the loan's own 3.00× covenant</summary>

These are two different numbers for a reason: 3.00× is the **default trigger** written into the loan agreement (breach it after the loan is signed, and you're in technical default, per Challenge 1). 2.00× is the lender's **own comfort level** for originating the loan in the first place — a more conservative number, since the lender wants headroom between "what I'm willing to lend into" and "what would actually blow up the deal." Real credit committees routinely underwrite well inside their own covenant thresholds for exactly this reason.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Correctness | Split doesn't sum to $20,000,000, or covenant check fails | All splits sum correctly; all covenant checks explicitly shown passing |
| Reuse | Functions rewritten or duplicated for the new amounts | Exercise 1–3 functions called unmodified on new inputs |
| Comparison | Only the winning split's cost is shown | All three splits' costs shown side by side, correctly ranked |
| Reasoning | "Debt is cheaper" asserted without proof | Part 5's three-way comparison explicitly demonstrates why the maximum-debt-within-constraint plan wins |

## Submission

Commit your split-and-price engine (`mixed_raise.py`, importing from your Exercise 1–3 modules), the three-way comparison table, and a short writeup answering the skeptical board member's question, to your portfolio under `c35-week-06/challenge-02/`.
