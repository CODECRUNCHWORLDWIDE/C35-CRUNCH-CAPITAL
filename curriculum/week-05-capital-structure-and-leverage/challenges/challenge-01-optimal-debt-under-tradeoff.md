# Challenge 1 — Solve for Optimal Debt

**Time:** ~90 minutes. **Difficulty:** Medium-high. **Answer is a range, defended, not one static number.**

## The scenario

Crunch Machine Co.'s board has asked you, the newly hired analyst, one direct question: *"Our banker says we should refinance and take on more debt to save on taxes. Our auditor says we're already too levered. Who's right, and what number should we actually be targeting?"*

You have the tools from Lecture 2. This challenge asks you to actually deliver a defensible number — not just the formula — and to be honest about how much that number depends on assumptions nobody in the room can verify with certainty.

## Part 1 — Solve by calculus

Using `VU = 16,687.5`, `Tc = 0.25`, `α = 0.30` (this week's standard Crunch Machine Co. parameters), derive `D*` from the tradeoff-value equation:

$$V_L(D) = V_U + T_c D - \frac{\alpha}{V_U}D^2$$

1. Take `dV_L/dD`, set it to zero, and solve for `D` symbolically (by hand — show the algebra in a markdown file, `derivation.md`).
2. Confirm your `D*` matches Lecture 2's worked answer (`≈ 6,953.1`).
3. Take the **second derivative** and confirm it's negative — proving this is a maximum, not a minimum or an inflection point. *(This is a one-line check: `d²V_L/dD² = −2α/VU`, which is negative for any positive `α` and `VU` — but write out why that guarantees a maximum, referencing the second-derivative test.)*

## Part 2 — Solve by grid search, independently

Without looking at your Part 1 answer, write a Python function that finds the value-maximizing `D` numerically:

```python
def find_optimal_debt(VU: float, tax_rate: float, alpha: float,
                       d_min: float = 0, d_max: float = 30000, step: float = 10) -> float:
    """Grid-search for the D that maximizes levered_value(D, VU, tax_rate, alpha)."""
    ...
```

4. Run it with `step=10` (a fine grid) over the same `VU`, `Tc`, `α` as Part 1. How close is the grid-search answer to your calculus answer? *(It should agree to within the step size — if it's off by more than that, one of your two methods has a bug; find it.)*

## Part 3 — Sensitivity: how much does `α` matter?

The distress-cost fraction `α` is the model's shakiest input — it's a stylized assumption, not a number anyone measured for Crunch Machine Co. specifically.

5. Re-run **both** methods (calculus formula, grid search) for `α ∈ {0.10, 0.20, 0.30, 0.40, 0.50}`, holding `VU` and `Tc` fixed. Build a table: `alpha`, `D_star`, `D_star_as_pct_of_VU`.
6. Plot `D*` (y-axis) against `α` (x-axis). Does `D*` fall as `α` rises, as economic intuition predicts (higher distress cost → less debt is optimal)? Does the relationship look linear, or curved? *(Hint: look at the formula — `D* = Tc·VU / (2α)`. What kind of function of `α` is that?)*

## Part 4 — Reconcile with reality

7. At **every** `α` value in your Part 3 table, is Crunch Machine Co.'s actual FY2025 debt (`19,700`) above or below `D*`? Is there **any** value of `α` in a defensible range (say, `0.05` to `0.60`) at which `19,700` would actually be the value-maximizing level, or close to it? Show the `α` (if any) that would make `D* ≈ 19,700`, by solving `α = Tc × VU / (2 × 19,700)` directly.
8. In `verdict.md` (≤ 300 words), answer the board's original question: is Crunch Machine Co. over-levered, and how confident are you in that conclusion given the sensitivity analysis you just ran? Name the single assumption you'd most want real data on before making a $10M+ refinancing recommendation.

## Constraints

- Part 1's algebra must be shown step by step, not just asserted — a bare "`D* = 6,953.1`" with no derivation is an incomplete answer.
- Parts 1 and 2 must be genuinely independent — don't derive Part 2's grid search by simply evaluating your Part 1 formula at many points; it should evaluate `levered_value(D)` directly and search, so a bug in one method wouldn't automatically appear in the other too.
- Every plot needs axis labels and a title. An unlabeled chart is not a finished deliverable.

## Hints

<details>
<summary>On the algebra in Part 1</summary>

`V_L(D) = VU + Tc·D − (α/VU)·D²` is a standard quadratic in `D`. Its derivative is `Tc − 2(α/VU)D`. Setting that to zero and isolating `D` gives `D = Tc·VU / (2α)` directly — no need for the quadratic formula, since there's no constant term complicating things once you're working with the *derivative*, not the original equation.

</details>

<details>
<summary>On Part 3's curve shape</summary>

`D*(α) = (Tc·VU/2) × (1/α)` — everything except `α` is a fixed constant here, so `D*` is directly proportional to `1/α`. That's a hyperbola, not a straight line: `D*` falls fast as `α` rises from a small value, then flattens out as `α` gets larger. Doubling `α` from 0.10 to 0.20 cuts `D*` in half; doubling it again from 0.20 to 0.40 cuts it in half again — same proportional effect, but the *absolute* drop is much bigger the first time.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Algebra | Formula stated with no derivation | Full step-by-step derivative shown, second-derivative check included |
| Cross-validation | Only one method attempted | Calculus and grid search both run, shown to agree |
| Sensitivity | One `α` value only | Full table + chart across 5 `α` values, correct interpretation of the `1/α` shape |
| Judgment | No verdict, or a verdict with no caveats | Clear recommendation in `verdict.md`, honest about which assumption drives the most uncertainty |

## Submission

Commit `derivation.md`, your Python code, your Part 3 chart, and `verdict.md` to your portfolio under `c35-week-05/challenge-01/`.
