# Challenge 2 — Diagnose a Governance Failure

**Time:** ~90 minutes. **Difficulty:** Medium-Hard. **No single right answer.**

## The scenario

`capstone_governance_cases` gives you six compressed vignettes — a company alias, an incentive structure, a failure mode, an outcome, and a one-line lesson. That compression is deliberate: real governance case studies (proxy statements, 8-Ks, board minutes, post-mortem reports) run to hundreds of pages, and the skill this challenge builds is **reconstructing the missing middle** — the specific chain of decisions between "here's the incentive" and "here's the bad outcome" — from a short description, the way you'd have to when a real case only gives you fragments (a news article, a earnings-call admission, a departed-executive's LinkedIn timeline).

## Your task

Pick **one** case from `capstone_governance_cases` — not the one you used in Exercise 3's memo; pick a different one, so you get a second rep.

```sql
SELECT * FROM capstone_governance_cases WHERE case_id = <your choice>;
```

### Part 1 — Reconstruct the chain (write ~300 words)

Walk the causal chain in specific, concrete steps: incentive structure → what a rational manager does in response → why existing controls didn't catch it → the outcome. Do not just restate the row's fields in sentence form — add the *mechanism*. For example, for Case 1 (revenue-only bonuses, channel-stuffing), a weak answer says "they gamed revenue." A strong answer explains the specific mechanism: sales reps pulled next-quarter orders forward with extended payment terms or generous return rights to book revenue in the current period, which the revenue-growth bonus metric couldn't distinguish from organic demand, and which only became visible when the returns hit the following quarter's financials — by which point the bonus had already been paid and (typically) couldn't be clawed back.

### Part 2 — Quantify the incentive, the way Lecture 3 did

Using the same technique as Lecture 3, Section 2, build a small `pandas` model: given your chosen case's incentive structure, construct 3–4 simple scenarios (you can invent plausible numbers — state them) and show, with a table, in how many/what fraction of scenarios the flawed incentive pays out **despite** the outcome being value-destroying. This is the same move as the revenue-bonus-vs-quality-gate example — reproduce it for your case, not the one from the lecture.

### Part 3 — Design the fix

Propose a **specific, implementable** control or incentive redesign. "Better oversight" is not specific. "Require any related-party transaction over $1M to be approved by a majority of independent directors with no reporting relationship to the CEO, disclosed within 5 business days" is specific. Your fix should:

1. Directly address the mechanism you identified in Part 1 (not just the symptom in the `outcome` column).
2. Be something a real board could vote on and implement in a single meeting — no "change the culture" answers.
3. Include one sentence on a **cost or friction** your fix introduces (every control has a cost — slower decisions, more paperwork, a director's time — a fix with no acknowledged cost usually hasn't been thought through).

### Part 4 — Would this control have caught it earlier?

Estimate, in your own judgment and stated as such, how much earlier your proposed control would likely have surfaced the problem relative to when it actually surfaced in the case's `outcome` description. A quarter earlier? A year earlier? Never (i.e., your fix addresses the incentive but wouldn't have changed detection timing)? Be honest if the answer is the last one — not every good fix is also a faster detector, and conflating the two is a common mistake.

## Constraints

- You may invent specific numbers for Part 2's scenarios (dollar figures, percentages) as long as you state them as assumptions — this challenge is about the *mechanism* and the *quantification method*, not about having access to the real company's real numbers (which don't exist; these are fictional aliases).
- Part 3's fix must be a **change**, not a restatement of "don't do the bad thing" — "stop paying revenue-only bonuses" is not a fix; "replace the metric with a 70/30 blend of free cash flow growth and revenue growth, both measured net of returns and with a 2-quarter payout lag" is.

## Hints

<details>
<summary>On reconstructing the chain (Part 1)</summary>

Ask three questions in order for any governance case: (1) What specific action maximizes the flawed metric fastest? (2) What existing control should have caught that action, and specifically why didn't it (too infrequent, too narrow in scope, conflicted reviewer, no one was looking)? (3) What made the outcome visible when it finally became visible — was it a scheduled audit, a whistleblower, a macro shock that removed the cover, or something else? Case studies that only answer "what happened" without answering these three are descriptions, not diagnoses.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|---------------|
| Mechanism | Restates the table's `failure_mode` column | Adds a specific, plausible causal chain with concrete actions |
| Quantification | Skips Part 2 or hand-waves it | A real small model, table of scenarios, honest payout comparison |
| Fix specificity | "More oversight" / "better culture" | A rule specific enough that a board could vote on it verbatim |
| Cost awareness | No acknowledged tradeoff | States a real cost or friction the fix introduces |
| Honesty (Part 4) | Claims the fix catches everything faster | Honestly separates "fixes the incentive" from "speeds up detection" when they're not the same thing |

## Submission

Commit `challenge-02.md` (all four parts, plus your `pandas` model code) to your portfolio under `c35-week-12/challenge-02/`.
