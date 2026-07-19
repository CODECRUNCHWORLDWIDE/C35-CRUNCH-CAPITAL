# Challenge 2 — Stress the Hurdle Rate

**Time:** ~90 minutes. **Builds on:** Lecture 1 Section 4, Lecture 1 Section 9 (NPV profile), Exercise 1.

## The brief

Every accept/reject decision this week has depended on a hurdle rate you were handed as a given number in the `hurdle_rate` column. In the real world, that number is an estimate — usually built from a cost-of-capital model (Week 4's subject) that involves its own assumptions and forecast error. A capital committee that treats a hurdle rate as exact to the basis point is fooling itself. Your job this challenge: pick one of this week's six projects, and find out **how wrong the hurdle rate would have to be before the accept/reject decision flips.**

## Requirements

1. **Pick one project** from `cash_flows`. You may pick a project that currently passes (New CNC Machine or Brand Relaunch Campaign) or one that currently fails (any of the other four) — either makes for a valid analysis, but say which you picked and why.
2. **Sweep the hurdle rate** across a realistic range — at minimum, from 3 percentage points below the project's stated hurdle rate to 8 percentage points above it, in increments of no more than 0.5 percentage points. Compute NPV at every point (reuse your Exercise 1 NPV function).
3. **Find the exact breakeven rate** — the discount rate where NPV crosses zero for this project — using a numerical root-finder (`scipy.optimize.brentq` or `numpy_financial.irr()`, since for a single project this breakeven rate *is* the project's IRR; state that connection explicitly in your writeup).
4. **Quantify the margin of safety (or margin of danger).** How many percentage points of "wrongness" in the stated hurdle rate would it take to flip the decision? Express this both as a raw percentage-point gap and as a **relative** measure (e.g., "the hurdle rate would need to be about 35% higher/lower than stated" — relative framing matters because a 2-point gap means something very different for a 6% hurdle rate than for an 18% one).
5. **Build a small table or chart**: hurdle rate on one axis, NPV on the other, with the current stated rate and the breakeven rate both marked clearly.
6. **Write 200–300 words arguing whether the company should be comfortable** with its current accept/reject call on this project, given how much cushion (or how little) exists between the stated hurdle rate and the point where the decision flips. Reference the project's `risk_tier` column from the seed table — a High-risk project sitting close to its breakeven line should worry you more than a Low-risk one in the same position, because risk-tier estimates themselves carry more uncertainty.

## What "strong" looks like

- The breakeven rate is solved precisely, not read off a coarse table.
- The relative-margin framing in step 4 is actually computed, not just asserted qualitatively.
- The writeup in step 6 makes a specific, falsifiable claim ("this project has almost no cushion — a 1.5-point miss on the hurdle rate flips the call" or "this project is robust — the hurdle rate would need to nearly double before the decision changes") backed directly by the numbers from steps 3–4.

## Stretch goal

Do the same sensitivity sweep for **two** projects — one currently passing, one currently failing — and compare their margins of safety side by side. Which project's accept/reject call is more fragile to a hurdle-rate estimation error, and does that match your intuition from their `risk_tier` values?

## Deliverable

A short writeup (`challenge-02/README.md` or similar) with your sweep table/chart, the solved breakeven rate, the margin-of-safety figures, and the 200–300 word argument from step 6.
