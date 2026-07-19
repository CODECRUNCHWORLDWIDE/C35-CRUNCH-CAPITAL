# Challenge 1 — Relever Beta Across Structures

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **Method matters more than the final number.**

## The scenario

Crunch Machine Co.'s board is evaluating a spin-off: a new, wholly-owned subsidiary — **Crunch Automation Systems** — that would carry its own capital structure, separate from the parent. It has no trading history of its own (it doesn't exist yet as a public entity), so you can't regress a beta for it the way you did for CRNM in Exercise 2. This is the single most common real-world reason analysts reach for the **pure-play / comparables method**: value something with no price history by borrowing the *business risk* of similar public companies, stripping out *their* financing choices, and relevering at the structure the new entity will actually carry.

Your job: build that estimate.

## The data

Four public companies in the same industrial-automation space as the proposed subsidiary. None of them are perfect matches, and part of this challenge is judging how much that matters. Create this table and load it yourself — it's new this challenge, not part of the week's seed:

```sql
CREATE TABLE peer_betas (
    peer_id      INTEGER PRIMARY KEY,
    company      TEXT    NOT NULL,
    levered_beta NUMERIC NOT NULL,
    debt_to_equity NUMERIC NOT NULL,   -- market-value D/E
    tax_rate     NUMERIC NOT NULL,
    note         TEXT
);

INSERT INTO peer_betas VALUES
(1, 'Anvex Industrial Corp',   1.15, 0.35, 0.25, 'Closest business match; conservative balance sheet'),
(2, 'Forgeline Motors',        1.45, 0.70, 0.24, 'Broader auto-parts exposure than pure automation'),
(3, 'Ridgeway Tool & Die',     0.95, 0.20, 0.26, 'Smaller, niche, very low leverage'),
(4, 'Steelcrest Automation',   1.60, 0.90, 0.25, 'Closest name match, but aggressively levered');
```

## Your task

Work through this in `challenge-01.py`, and write your reasoning in `challenge-01.md` as you go — every judgment call below needs a stated justification, not just a number.

1. **Unlever each peer.** For each of the four peers, apply the Hamada formula:
   $$\beta_U = \frac{\beta_L}{1 + (1-t)(D/E)}$$
   Print a table of `company`, `levered_beta`, `debt_to_equity`, `tax_rate`, `unlevered_beta`.

2. **Decide how to average.** You have four unlevered betas. A simple mean is the obvious first move — compute it. Then consider: should every peer count equally? `Steelcrest Automation` is named as the closest match but is also the most aggressively levered (meaning its *unlevered* beta required the biggest adjustment, which is also where estimation error compounds the most). Compute a second average that **excludes** Steelcrest, and a third that's a simple average of just `Anvex` and `Ridgeway` (the two peers the notes describe most favorably). Report all three averages side by side. State, in `challenge-01.md`, which one you'd actually use and why — there's no single correct choice here, but an unstated one is a weak answer.

3. **Determine the target capital structure.** Crunch Machine Co.'s board has proposed that Crunch Automation Systems launch at the *parent's own* current market-value $D/E$ (you computed this in Exercise 3 — recompute or reuse it: $D/E \approx 0.2623$) and the same 25% tax rate. Relever your chosen average unlevered beta at this target structure:
   $$\beta_L^{\text{target}} = \beta_U \times (1 + (1-t)(D/E)_{\text{target}})$$
   Report the resulting relevered beta.

4. **Compare to the other two beta estimates from this week.** Build a small summary table with three rows — `vendor` (1.30), `regression` (≈1.2429, from Exercise 2), and `comps-relevered` (your Task 3 answer) — each with its beta and the cost of equity it implies (using Crunch Machine Co.'s $R_f$ and $ERP$ from `capital_structure`). Print the table.

5. **Write the recommendation.** In `challenge-01.md`, answer in a short paragraph: which of the three betas would you recommend the board use for **Crunch Automation Systems specifically** (not the parent company) — and explicitly justify why a *new subsidiary with no trading history* should or shouldn't use the *parent's own* regression beta at all.

## Constraints

- Show every intermediate unlevered beta — don't just report the final relevered number. A reviewer should be able to check your arithmetic at each step.
- Every averaging decision (Task 2) and every structural assumption (Task 3's target $D/E$) needs one written sentence of justification in `challenge-01.md`.
- No spreadsheets. `peer_betas` lives in SQL; every average and the relevering calculation happens in pandas.

## Hints

<details>
<summary>On why unlevered betas cluster more tightly than levered betas</summary>

Look at the spread of the four `levered_beta` values (0.95 to 1.60, a range of 0.65) versus the spread of the four `unlevered_beta` values you compute in Task 1. If the unlevered spread is meaningfully tighter, that's a good sign the four companies really do share similar *business* risk and most of their levered-beta differences are just financing choices — which is exactly the assumption the whole comparables method depends on. If the unlevered spread were just as wide as the levered spread, that would be a red flag that these aren't good comparables after all.

</details>

<details>
<summary>On Task 5</summary>

A brand-new subsidiary with no trading history has, by definition, no company-specific price history to regress — so the parent's *own* regression beta (1.2429) reflects the parent's *entire* consolidated business, which may include operations quite different from the new subsidiary's narrower focus. The comps-based beta is arguably the *more* correct answer for a truly distinct new business line, even though it's built from proxies rather than the company's own data — this is precisely the situation the pure-play method exists to solve.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|---------------|
| Mechanics | Unlevers/relevers correctly for at least one peer | All four correctly unlevered, relevering formula applied correctly |
| Judgment | Reports only a simple mean, no alternatives considered | Computes multiple averaging approaches and states a reasoned preference |
| Structural assumption | Uses the target $D/E$ without comment | Explicitly justifies using the parent's $D/E$ as the subsidiary's target (or argues for a different one) |
| Comparison | Reports the relevered beta alone | Places it side by side with the vendor and regression betas and interprets the gap |
| Recommendation | No clear final answer | Clear recommendation for *this specific decision* (a new subsidiary), reasoned, not just restating "it depends" |

## Submission

Commit `challenge-01.md` and `challenge-01.py` to your portfolio under `c35-week-04/challenge-01/`.
