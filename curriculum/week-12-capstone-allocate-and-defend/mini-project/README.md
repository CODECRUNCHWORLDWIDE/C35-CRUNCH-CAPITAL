# Mini-Project — Run a Full Deal End-to-End and Defend the Number

> Value Crunch Machine Co. by DCF and comps, size a position in the Crunch Capital Advisors portfolio, quantify the risk with VaR and scenarios, factor in a governance read, and deliver one IC-ready memo where every number cites the query or script that produced it. This is the capstone of the whole 12-week course — everything you've built comes together here.

**Estimated time:** 3 hours, best done Saturday after all three exercises and both challenges are complete. This mini-project is not new work — it is the **final assembly** of Exercises 1–3 plus your Challenge 1 sensitivity findings and your Challenge 2 governance mechanism, into one deliverable that could genuinely go in front of a real investment committee.

---

## The brief

You are the analyst who ran this week's exercises. Crunch Capital Advisors' IC meets Monday. You have the floor for 20 minutes to present the CRNM satellite position and answer questions. Your deliverable is the memo you'll present from — and a live database/script setup you can query on the spot if someone asks "show me."

## Deliverable

A directory in your portfolio `c35-week-12/mini-project/` containing:

1. **`memo.md`** — the final IC memo. Extends your Exercise 3 draft; do not restart from scratch. Must include, at minimum:
   - Recommendation, Valuation, Risk, Governance, and "What would change our mind" (Exercise 3's five sections).
   - A new **Sensitivity** section incorporating your Challenge 1 findings: the break-even DCF/comps weighting, the terminal-growth sensitivity table, and the correlation break-even.
   - At least **15** numeric claims, each citing a specific script/query and task/step.
2. **`valuation.sql`** and **`valuation.py`** — your final, cleaned-up Exercise 1 work (fix anything Challenge 1 exposed as fragile).
3. **`risk.py`** — your final Exercise 2 VaR and scenario work, extended to include the correlation-sensitivity table from Challenge 1, Attack 3.
4. **`governance.py`** — your Exercise 3 / Challenge 2 governance quantification (the incentive-payout scenario table).
5. **`sensitivity.py`** — the three attacks from Challenge 1, packaged as one runnable script that prints all three tables.
6. **`presentation_notes.md`** — a one-page (~300 word) speaker's outline for your 20-minute IC presentation: what you'd say in the first 2 minutes (the recommendation, cold), and the 3 hardest questions you expect and your prepared answer to each.

Every script must run **cleanly against a fresh copy of `crunch_capital`** seeded from this week's README — a grader (or a real IC member) should be able to clone your repo, run the setup SQL, run your scripts, and get your numbers.

---

## Requirements

Your final recommendation must answer all of the following, each with a number and a citation:

1. **What is CRNM worth, and by which methods?** (DCF perpetuity, DCF exit multiple, comps, blended.)
2. **What position size do you recommend**, and what does it do to the whole Crunch Capital Advisors portfolio's expected return and volatility?
3. **What is the 1-day and 10-day, 95% and 99% VaR** — for the position alone and for the portfolio with it added?
4. **What does the probability-weighted scenario valuation say**, and how does it compare to the point-estimate target?
5. **Which single assumption is the thesis most sensitive to**, quantified (Challenge 1, Attack 2's terminal-growth finding, unless your own analysis found something more sensitive — argue for whichever you found)?
6. **What is the single biggest governance risk** for a business like this, and what specific control mitigates it?
7. **What would change your mind?** — concrete, observable triggers.

## Milestones

Pace yourself; this is a full assembly job, not new analysis from scratch.

- **Milestone 1 (45 min):** Pull Exercises 1–3 and Challenges 1–2 into one working directory. Re-run everything against a fresh `crunch_capital` seed to confirm it's all still reproducible — fix anything that broke.
- **Milestone 2 (45 min):** Write the Sensitivity section and `sensitivity.py`, folding in Challenge 1's three attacks.
- **Milestone 3 (45 min):** Finalize `memo.md` end to end. Count your citations — you need 15+.
- **Milestone 4 (45 min):** Write `presentation_notes.md`. Say your 2-minute recommendation out loud, timed. Trim until it fits.

## Rules

- **Every number in `memo.md` cites its source.** This is the single hardest requirement to satisfy honestly and the single most important skill this course has tried to teach — treat it as non-negotiable.
- **The recommendation must be a specific position size** (a percentage of the book, or a dollar amount against your stated book size), not a vague "we like this name."
- **The governance section must name a specific control**, not a general principle.
- **No spreadsheets, anywhere**, per the course's [data tooling rule](../../SYLLABUS.md#data-tooling-rule) — every table, every calculation, every scenario, every sensitivity grid lives in SQL and Python, exactly as it has for eleven weeks before this one.

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Valuation correctness | 20% | DCF and comps figures match Exercise 1's expected results (or a well-justified deviation) |
| Risk quantification | 20% | VaR (both methods), scenario, and portfolio-level figures all present and internally consistent |
| Sensitivity / red-team integration | 15% | Challenge 1's three attacks are folded in, not just referenced |
| Governance specificity | 15% | A named case, a quantified incentive model, and a specific, implementable control |
| Reproducibility | 20% | Every script runs clean on a fresh seed; every memo claim has a real citation |
| Presentation | 10% | `presentation_notes.md` has a tight 2-minute open and genuinely hard anticipated questions with real answers |

## Reflection (`notes.md`, ~250 words)

1. Where in this week's work did a number you were confident in turn out to be more fragile than you expected, once you red-teamed it?
2. If you had to defend this memo live and could only bring **one** script to answer follow-up questions, which one, and why?
3. Looking back across all 12 weeks — which single skill from an earlier week did you lean on hardest this week, and why?
4. What's one thing a real IC memo would need that this capstone didn't ask you to build (be specific — e.g., a full legal/regulatory review, a management-team reference-check process, a liquidity/exit-timeline analysis)?

---

## Why this matters

This is the shape of the actual job: someone with capital to allocate needs a defensible answer, under time pressure, that survives someone actively trying to poke holes in it. Every earlier week gave you one clean tool used in isolation. This week you used all of them together, on one target, under scrutiny — which is the only way any of these tools actually get used for real. Keep this project. It's the single best piece of evidence in your portfolio that you can do the job, not just recite the formulas.

When done: take the [quiz](../quiz.md). You've finished C35 · Crunch Capital.
