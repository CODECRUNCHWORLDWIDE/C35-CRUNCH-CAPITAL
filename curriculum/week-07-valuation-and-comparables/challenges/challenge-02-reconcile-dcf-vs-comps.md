# Challenge 2 — Reconcile DCF vs. Comps

**Time:** ~90 minutes. **Difficulty:** Medium-high. **No single right implementation.**

## The scenario

You've now built four valuation answers for Crunch Machine Co.: two DCF terminal-value flavors, a trading-comps range, and a precedent-transactions range. They don't agree, and — per Lecture 3, Section 2 — "they're all just estimates" is not an acceptable explanation to give whoever is deciding whether to buy, sell, or invest based on your work. This challenge makes you **quantify** the disagreement instead of gesturing at it, using the actual mechanism connecting a DCF and a market multiple: they're both, underneath, claims about a growth rate.

## Part 1 — Convert the market's multiple into a growth rate the DCF can be compared against

Lecture 1's exit-multiple terminal value and Exercise 2's implied-growth-rate algebra give you a tool: any EV/EBITDA multiple, applied as a terminal exit multiple, implies *some* constant perpetual growth rate once you solve `TV = FCF_(n+1)/(WACC − g)` for `g`.

Use this tool on the **trading comps' median EV/EBITDA multiple (8.50×)** instead of the 8.0× directly assumed in Lecture 1:

1. Compute the terminal value using **8.50×** applied to Crunch Machine Co.'s own FY2030 EBITDA, exactly as Lecture 1, Section 7 did with 8.0×.
2. Discount it back to today (same `t=5` discount factor) and compute the resulting enterprise value.
3. Solve for the **implied perpetual growth rate** this terminal value corresponds to, using Exercise 2 Task 11's rearranged formula.
4. Compare that implied growth rate to the DCF's own **explicit** terminal growth assumption (2.5%). Report the gap in percentage points.

Repeat the same four steps using the **precedent transactions' median EV/EBITDA multiple (10.97×)** instead.

## Part 2 — Interpret what the two gaps mean

In `reconciliation.md`, answer directly:

1. Is the growth rate implied by the trading-comps multiple **higher or lower** than the DCF's own 2.5% assumption? What does that tell you about whether the public market is pricing in *more* or *less* optimism about Crunch Machine Co.'s long-run growth than your own DCF assumes?
2. Is the growth rate implied by the precedent-transactions multiple higher or lower than 2.5%? Given that precedent multiples already embed a control premium (Lecture 2, Section 5), is it fair to read this gap the same way as the trading-comps gap in question 1 — i.e., is it *purely* about growth expectations, or is some of it explained by something else entirely? Be specific about what that "something else" is.
3. Using both gaps together: does the evidence suggest Crunch Machine Co.'s own DCF assumptions (21.0% → 23.5% margin expansion, 2.5% terminal growth) are more aggressive than, roughly in line with, or more conservative than what the market is currently paying for similar businesses? Defend your answer with the specific numbers from Part 1, not just an impression.

## Part 3 — Build a weighted final recommendation

Lecture 3, Section 3 lays out a four-step process for landing on one number: rule out broken assumptions, look for convergence, weight by trust in the evidence, and report a range with a base case. Apply it explicitly:

1. Based on Part 1–2's analysis, state — with reasoning, not just a number — how much weight (as rough percentages that sum to 100%, e.g. "50% DCF, 30% trading comps, 20% precedent transactions") you would personally assign to each method for Crunch Machine Co. specifically.
2. Compute a weighted-average per-share value using your weights and each method's **midpoint** per-share value (use the midpoint of each method's low–high range from Lecture 3, Section 1 / Challenge 1's football field).
3. Compare your weighted number to the simple "overlap zone" ($15.50–$17.20) Lecture 3, Section 3 identified by eyeballing the ranges. Are they close? If your weighted number falls outside that zone, explain specifically which method's weight is pulling it there and whether you think that's justified.

## Constraints

- Every number in Part 1 must come from a real calculation shown in your code — not eyeballed off a chart.
- Your weights in Part 3 must be **justified in writing**, referencing specifics from Part 1–2 (e.g., "I weight trading comps lower because VDT's inclusion — flagged in Exercise 3 — makes the set's high end less trustworthy") — a set of weights with no stated reasoning is not an acceptable answer for this challenge.
- No spreadsheets, per this week's data tooling rule.

## Hints

<details>
<summary>On the implied-growth-rate formula giving a negative or nonsensical number</summary>

Double-check you're using the **undiscounted** terminal value (`TV`, before applying `1/(1+WACC)^5`) in the implied-growth formula, not the discounted present value — mixing the two produces a meaningless result. This is the same distinction Exercise 2's Tasks 4–5 and 7–8 are careful to keep separate.

</details>

<details>
<summary>On the precedent-transactions gap being hard to interpret cleanly</summary>

That difficulty is the point of Part 2, question 2 — a precedent multiple mixes two effects (the market's growth expectations for the sector, *and* the control premium a buyer pays on top). A trading-comps multiple isolates the first effect more cleanly since there's no control premium in a public-market trade. If your writeup for question 2 doesn't mention this distinction explicitly, it's missing the actual point of comparing the two.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Correctness | Implied-growth-rate calculation reuses the wrong TV (discounted instead of undiscounted) | Both Part 1 calculations are correct and clearly shown |
| Quantification | "The DCF and comps disagree because assumptions are different" | States the exact percentage-point gap for both the trading-comps and precedent-transaction comparisons |
| Reasoning | Weights in Part 3 look arbitrary or unstated | Weights are explicitly justified, referencing specific evidence from Part 1–2 and Exercise 3 |
| Synthesis | Final number reported with no comparison to the football field | Explicitly compares the weighted number to the eyeballed overlap zone and explains any divergence |

## Submission

Commit `reconciliation.py` (or `.sql`+`.py`) and `reconciliation.md` to your portfolio under `c35-week-07/challenge-02/`.
