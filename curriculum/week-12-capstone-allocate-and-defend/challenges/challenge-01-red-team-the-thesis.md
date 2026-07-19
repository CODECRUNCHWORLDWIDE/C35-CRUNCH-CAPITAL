# Challenge 1 — Red-Team Your Own Thesis

**Time:** ~90 minutes. **Difficulty:** Hard. **No single right answer.**

## The scenario

Your Exercise 1–3 memo recommends an 8% CRNM position with a $15.29 blended target against a $12.50 market price. Before this goes to the IC, you have to do the thing almost nobody does voluntarily: **attack your own work as if you were being paid to kill the deal.** A red team's job isn't to be negative for its own sake — it's to find the single assumption that, if wrong, does the most damage to the thesis, and put a number on exactly how much damage.

Professional investors call this "pre-mortem" analysis: instead of asking "why will this work," you ask "it's 12 months from now and this position lost 30% — what happened?" and then work backward to see if that story is plausible given what you actually know today.

## Your task

Attack your own valuation from Exercise 1 along **three** specific lines. For each, quantify the damage — don't just describe it.

### Attack 1 — The weighting

In Exercise 1, Task 8, you chose a weighting between the DCF average and the comps answer (Lecture 1 used 60/40). Recompute the blended target at three alternative weightings: **50/50**, **40/60**, and **0/100** (pure comps — i.e., "the DCF is wrong and the market is right"). At what weighting does the implied upside fall to **zero**? State that break-even weighting explicitly — it tells you exactly how much you'd have to distrust your own DCF before the thesis stops working at all.

### Attack 2 — The terminal growth assumption

The perpetuity-growth DCF assumes 2.5% terminal growth forever. Recompute the perpetuity-method enterprise value and per-share price at terminal growth rates of **1.5%**, **2.0%**, **2.5%** (base), and **3.0%**. How much does a single percentage point of terminal-growth assumption move the per-share value? Is the thesis's upside more sensitive to this single assumption than to anything else in the model? If so, that is the assumption an IC member will (correctly) spend the most time attacking — make sure you can defend the 2.5% figure specifically, not just wave at "reasonable long-run GDP growth."

### Attack 3 — The correlation assumption

Lecture 2/Exercise 2 used a weighted-average correlation of ≈0.27 between the core book and CRNM to compute portfolio VaR — an explicit simplification (README, Section 4, Table 4 comment) that ignores cross-correlations *among* the six core funds. Recompute portfolio volatility at correlation values of **0.10**, **0.27** (base), **0.50**, and **0.80** (a stress case: a systemic shock where "everything correlates to 1" — a real phenomenon in genuine market crises). How much does portfolio VaR change across that range? At what correlation does adding CRNM stop being a diversification benefit and start being a diversification cost (i.e., portfolio volatility exceeds the core-only 8.5%)?

## Constraints

- Every attack must produce a **number**, not just a paragraph. "The correlation could be higher in a crisis" is an observation; "at ρ=0.80, portfolio 1-day 95% VaR rises from $X to $Y, a Z% increase" is a red-team finding.
- After the three attacks, write a **verdict**: does the thesis survive? Would you still recommend the 8% position size, a smaller one, or none at all? You're allowed to conclude "yes, still buy" — a red team that always kills the deal isn't doing useful work either. The point is that your conclusion is now backed by three specific, quantified stress points instead of general confidence.

## Hints

<details>
<summary>On the terminal growth sensitivity (Attack 2)</summary>

The perpetuity formula's denominator is `(WACC − g)`. At WACC=9.2%, moving `g` from 2.5% to 3.0% shrinks the denominator from 6.7% to 6.2% — a small-looking change in the denominator produces a large change in the terminal value, because you're dividing by a small number. This is a well-known DCF fragility: terminal value dominates most DCFs (it should be well over half of total enterprise value here — check), and terminal value is *most* sensitive exactly where the denominator is smallest.

</details>

<details>
<summary>On the correlation break-even (Attack 3)</summary>

Rearrange the two-asset portfolio variance formula to solve for the ρ at which `σ_p = σ_core` (i.e., adding CRNM at its target weight doesn't change portfolio volatility at all) — everything above that ρ makes CRNM a volatility-increasing position at the portfolio level, not just at the position level.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|---------------|
| Quantification | "This assumption is risky" | A specific number: dollars, percentage points, or a break-even value |
| Assumption selection | Attacks an assumption that barely matters | Correctly identifies terminal growth as the highest-leverage assumption |
| Honesty | Only finds problems that don't threaten the recommendation | Willing to report a finding that would shrink or kill the position |
| Verdict | No clear conclusion | States a specific final position size, with the three attacks explicitly cited as the reason |

## Submission

Commit `challenge-01.md` (all three attacks, their numbers, and your verdict) to your portfolio under `c35-week-12/challenge-01/`.
