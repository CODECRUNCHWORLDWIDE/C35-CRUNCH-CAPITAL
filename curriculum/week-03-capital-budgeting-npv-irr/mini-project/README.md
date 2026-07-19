# Mini-Project — Rank Crunch Machine Co.'s Project Slate

> Take all six projects in `cash_flows`, compute every metric from this week, screen the slate, rank the survivors, fund a subset under a real budget constraint, and defend the cutoff in writing — exactly what a capital-allocation committee does before it signs off on next year's spending.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's the closest thing this course has to an actual job task. Crunch Machine Co.'s CFO has six project proposals on her desk and a fixed amount of capital to allocate this cycle. She doesn't want a spreadsheet of vibes — she wants a defensible, numbers-first recommendation: which projects get funded, which get rejected, and *why*, in language a non-finance board member can follow.

---

## Deliverable

A directory in your portfolio `c35-week-03/mini-project/` containing:

1. `analysis.sql` — every SQL query used, each under a `-- Q<n>: <what this answers>` comment.
2. `analysis.py` — every Python computation used (NPV, IRR, MIRR, payback, PI, the budget-funding function, the brute-force check), organized in the same order as the report below.
3. `report.md` — the actual deliverable: a memo to the CFO (template in Part E).

Everything runs against the seed table from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used.

---

## Part A — full metric table (45 min)

For all six projects in `cash_flows`, compute and tabulate:

- NPV (at the project's own `hurdle_rate`)
- IRR
- MIRR (using the hurdle rate as both finance rate and reinvest rate)
- Simple payback period
- Discounted payback period
- Profitability Index (PI)

One row per project, one column per metric. This should mostly be assembling work you already did across Exercises 1–3 — the mini-project's job is putting it all in one place and treating it as a finished artifact, not a scratch calculation.

## Part B — screen and rank (30 min)

1. Filter to projects with **NPV > 0**. State, in `report.md`, exactly which projects survive and which don't, and for each rejected project, give the one-sentence reason it was rejected (its NPV, stated plainly).
2. Rank the survivors by **PI**, descending.
3. Cross-check: does the IRR ranking of the survivors agree with the PI ranking? (For this particular slate, it should — but *say so explicitly*, and explain in one sentence why that's not guaranteed to always be true, per Lecture 2's timing trap.)

## Part C — fund under a budget, twice (45 min)

Crunch Machine Co.'s capital committee is choosing between two budget scenarios this cycle. For **each** scenario:

1. State which project(s) get funded, using the PI-ranked greedy approach from Lecture 3, Section 5.
2. State the total capital spent and the total NPV captured.
3. Run the brute-force check (Exercise 3, Part D) and confirm the greedy answer is optimal for that budget.

**Scenario 1 — Tight budget: $250,000.**
**Scenario 2 — Generous budget: $450,000.**

Report both outcomes side by side in a table.

## Part D — sensitivity check (30 min)

Pick the project on your "funded" list that has the **smallest gap** between its hurdle rate and its IRR (i.e., the least cushion). Run a hurdle-rate sweep on it (reuse Challenge 2's method if you did that challenge; otherwise do a lighter version — 5 rate points spanning ±3 percentage points from the stated rate is enough for this deliverable). State, in one or two sentences, whether this project's "fund it" recommendation is robust to a plausible estimation error in its hurdle rate.

## Part E — the memo (30 min)

Write `report.md` as a memo, addressed to "The Capital Committee," under 600 words, structured like this:

1. **Recommendation** (2–3 sentences, up front — don't bury the lede): which projects to fund at each budget level.
2. **The screen** (1 short paragraph): which projects were rejected outright and why, referencing NPV.
3. **The ranking** (1 short paragraph + your table from Part B): how you chose among survivors, and why PI rather than IRR or raw NPV size drove the ranking under a budget constraint.
4. **The two budget scenarios** (your table from Part C).
5. **One risk callout** (from Part D): which recommendation has the least cushion, and what would have to go wrong for it to flip.

No SQL, no raw code, no jargon dump in the memo itself — that all lives in `analysis.sql`/`analysis.py`. The memo is what the CFO actually reads.

---

## Milestones

- **Milestone 1 (45 min):** Part A — the full metric table, all six projects, all six metrics.
- **Milestone 2 (30 min):** Part B — screen and rank.
- **Milestone 3 (45 min):** Part C — both budget scenarios, both brute-force-checked.
- **Milestone 4 (60 min):** Parts D and E — the sensitivity check and the memo.

---

## Rules

- **Every number in the memo must trace back to a query or a computation in `analysis.sql`/`analysis.py`.** No numbers invented for the writeup.
- **State every assumption.** If you round a rate, note it. If you pick a project for Part D, say why you picked it.
- **The memo is for a non-finance reader.** If a sentence needs a finance degree to parse, rewrite it.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 35% | NPV/IRR/MIRR/payback/PI all match verified values; budget funding is correct for both scenarios |
| Screening logic | 15% | Negative-NPV projects rejected unconditionally, regardless of budget |
| Ranking logic | 15% | PI used correctly to rank survivors under a budget constraint; brute-force check performed |
| Sensitivity awareness | 15% | Part D's risk callout is specific and numeric, not vague |
| Memo quality | 20% | Recommendation stated up front; readable by a non-finance stakeholder; every claim traceable to the analysis files |

---

## Stretch goals

- Redo Part C with a **third** budget scenario of your choosing, chosen specifically to make the greedy PI ranking and a brute-force optimum potentially disagree (hint: you'll need the two survivors' combined cost to sit awkwardly relative to your budget — with only two survivors this may not be possible; if it isn't, explain in one paragraph what would have to be true about the slate for a disagreement to arise, referencing Lecture 3's synthetic X/Y/Z example).
- Add a seventh, **hypothetical** project of your own design to the slate (own numbers, own hurdle rate) and redo the full analysis with it included — does it change either budget scenario's recommendation?

---

## Reflection (in `report.md`, ~150 words, after the memo)

1. Which metric changed your mind about a project's attractiveness the most — NPV, IRR, or PI — and why?
2. Where did the "screen first, then rank" order matter? Would ranking-then-screening have given a different (wrong) answer for this slate?
3. What would you want to know about the *rejected* projects before writing them off completely — is there a version of any of them (different scope, different timing) that might have cleared the hurdle rate?

---

## Why this matters

This is the entire discipline of capital budgeting compressed into one exercise: screen ruthlessly, rank the survivors by the metric that actually maximizes value under a constraint, and communicate the result to someone who will act on it without re-deriving your math. Every later week in this course — cost of capital, capital structure, valuation, M&A — assumes you can do exactly this, fast and correctly, on demand.

When done: push, then take the [quiz](../quiz.md) and start [Week 4 — Cost of capital](../../week-04-cost-of-capital-capm-wacc/).
