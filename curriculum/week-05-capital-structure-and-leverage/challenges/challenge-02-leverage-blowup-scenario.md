# Challenge 2 — Model a Leverage Blow-Up Scenario

**Time:** ~90 minutes. **Difficulty:** Medium-high. **Requires combining all three lectures.**

## The scenario

It's the start of FY2026. Crunch Machine Co.'s FY2025 interest coverage closed at **1.61x** — already in the "high risk" band from Lecture 2. The company's loan agreement contains a standard covenant: **interest coverage must not fall below 1.5x**, measured quarterly, or the loan is in technical default (the lender can demand immediate repayment, restrict dividends, or force a renegotiation on much worse terms — regardless of whether the company can actually pay). Your job: quantify exactly how close to the edge Crunch Machine Co. already is, and what a plausible demand shock would do.

Every part of this challenge uses Crunch Machine Co.'s **actual FY2025** figures as the starting point: `EBIT = 2,225`, `interest expense = 1,380`, `Tc = 0.25`, `VU = 16,687.5`, `α = 0.30` (Lecture 2's Crunch Machine Co. standard).

## Part 1 — The covenant-breach threshold

1. Solve directly (algebra, then confirm numerically): at what `EBIT` does interest coverage hit **exactly 1.5x**? *(`EBIT_floor = covenant × interest`.)*
2. Express that `EBIT_floor` as a **percentage decline** from the FY2025 actual `EBIT = 2,225`.
3. In one sentence: how large a demand shock — in plain terms, "a bad quarter," "a mild recession," "a major customer loss" — would you associate with a decline of that size? Is this a remote tail risk or an uncomfortably plausible near-term scenario for a company already at 1.61x coverage?

## Part 2 — The breakeven threshold

4. Solve for the `EBIT` at which **pretax income hits exactly zero** (`EBIT = interest`), and express it as a percentage decline from `2,225`.
5. Compute DFL (Lecture 3's formula) at the FY2025 actual, and explain — using the DFL number itself, not just the two threshold percentages — why the gap between "covenant breach" and "pretax income wipeout" is smaller in dollar terms than it might first appear, even though the percentage-decline gap between Part 1's answer and Part 4's answer looks sizable.

## Part 3 — Pricing the blow-up with the distress-cost model

6. Compute the **maximum possible** distress cost — what Crunch Machine Co. loses if distress is a dead certainty (`p = 1`): `max_cost = α × VU`.
7. Now compute the model's "expected" distress cost at the **actual** FY2025 debt level using Lecture 2 and Exercise 3's raw formula, unmodified: `expected_cost = p(D) × α × VU`, with `p(D) = (D/VU)²` and `D = 19,700`. Compare it to your Part 6 answer. You should find something that shouldn't be possible for a genuine expected value: **`expected_cost` comes out *larger* than `max_cost`** — an expected cost can never exceed the worst case it's an average of. That contradiction is the model telling you something concrete, not a bug in your arithmetic: confirm it by checking `p(D)` itself — is it still a valid probability (between 0 and 1)?
8. Fix it the obvious way: **cap `p(D)` at 1** whenever the raw formula exceeds it, and recompute `expected_cost` and `levered_value` at `D = 19,700` with the capped probability. How much does capping change `levered_value` compared to the raw (uncapped) number Exercise 3 would have reported at this same debt level?
9. In `blowup-report.md` (≤ 350 words): explain what it means, in plain terms, that Crunch Machine Co.'s actual debt level pushes the stylized `p(D) = (D/VU)²` formula past its valid range (`D > VU`). Then walk the shock chain in dollars — a demand shock of the size from Part 1 pushes coverage below covenant → triggers technical default → risks the (now correctly capped) `max_cost` from Part 6 — and state, honestly, which single link in that chain you're **least** confident about as a real-world assumption.

## Constraints

- Show the algebra for both thresholds (Parts 1 and 4) — don't just state the answer.
- Part 3's "breach triggers full distress probability" is a deliberate simplification for this challenge, not a claim that real covenant breaches always cause instant, total distress. Say so explicitly in your report — a covenant breach is often waived, renegotiated, or cured without full distress costs materializing; you are pricing a **worst-case** chain reaction, and the report should say that plainly.
- Use exact figures, not rounded-off approximations, in Parts 1, 4, and 6–7 — this challenge is about precision, since the whole point is that Crunch Machine Co. is closer to the edge than a rough glance at "still profitable" would suggest.

## Hints

<details>
<summary>On Part 2's "smaller than it looks" question</summary>

The covenant-breach decline (≈ 7%) and the breakeven decline (≈ 38%) look far apart as percentages. But DFL tells you the company's earnings are *already* amplifying small EBIT moves heavily — a DFL of 2.63 at the FY2025 actual means the company doesn't get much warning between "still comfortably profitable" and "in real trouble." Frame your Part 5 answer around how fast pretax income is falling *per point of EBIT decline*, not just the two isolated thresholds.

</details>

<details>
<summary>On Part 3's dollar figures</summary>

`p(D)` at the actual FY2025 debt level (`D = 19,700`, `VU = 16,687.5`) is `(19,700 / 16,687.5)² ≈ 1.393` — which is **greater than 1**, a nonsensical "probability." This is a real limitation of the stylized quadratic model from Lecture 2: it was calibrated to behave sensibly for `D` up to roughly `VU`, and Crunch Machine Co.'s actual debt already exceeds `VU` itself. Left uncapped, the raw formula reports an "expected" cost of about `6,976.9` — bigger than the `5,006.25` maximum possible cost from Part 6, which is the impossible result Part 7 asks you to notice. Capping `p` at `1` fixes it: the corrected expected cost equals the Part 6 maximum exactly, and the corrected `levered_value` at `D = 19,700` comes out *higher* than the raw, uncapped number — meaning the raw formula was actually **overstating** how bad Crunch Machine Co.'s situation is at this specific debt level, precisely because it was being extrapolated past where it's mathematically valid. Report that finding plainly — it's a genuine, useful correction, not just cleanup.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Algebra | Thresholds stated without derivation | Both thresholds solved step by step and verified by plugging back in |
| Precision | Rounded, hand-wavy dollar figures | Exact figures throughout, correctly catching and capping the `p > 1` edge case in Part 3 |
| Narrative | Disconnected numbers | Part 3's report links shock → breach → distress cost as one causal chain, in dollars |
| Honesty | Presents the worst-case chain as certain fact | Explicitly flags which assumption in the chain is weakest |

## Submission

Commit your derivations, code, and `blowup-report.md` to your portfolio under `c35-week-05/challenge-02/`.
