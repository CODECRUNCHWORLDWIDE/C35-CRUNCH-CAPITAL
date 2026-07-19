# Exercise 3 — Draft the Risk Memo

**Goal:** Assemble Exercises 1 and 2, plus one governance read from `capstone_governance_cases`, into a single reproducible memo — the direct first draft of the mini-project's deliverable.

**Estimated time:** 90 minutes.

## Setup

You need the outputs of Exercise 1 (valuation) and Exercise 2 (VaR + scenarios) in front of you. Create `exercise-03/memo-draft.md`.

## The memo structure

Every real IC memo follows roughly this shape. Use these five headers, in this order.

### 1. Recommendation (3–4 sentences)

State the call plainly: buy/hold/pass, at what size, why. This section is read by people who will read nothing else — it has to stand alone.

### 2. Valuation (from Exercise 1)

- The three independent estimates (DCF perpetuity, DCF exit multiple, comps) and the blended target.
- Your stated weighting and its one-sentence justification.
- The implied upside vs. the market price.
- **Every number here must be labeled with the query/script that produced it** — e.g., "(see `exercise-01/valuation.sql`, Task 5)".

### 3. Risk (from Exercise 2)

- 1-day and 10-day VaR at 95% and 99%, both for the standalone position and for the whole portfolio with it added.
- The probability-weighted scenario value and how it compares to the point-estimate target.
- One sentence naming the single scenario that would hurt the most if it happened, and its probability.

### 4. Governance (new this exercise)

1. Pick **one** row from `capstone_governance_cases` whose `failure_mode` you judge to be the most plausible risk for a company like CRNM (an industrial manufacturer with sales-driven commission structures — recall Week 1's `employees` seed table had commission-based sales roles; that pattern generalizes here).
2. Write two sentences: why that failure mode is plausible for this kind of business, and what specific control or metric-design change would mitigate it.
3. State whether this governance read should change your position **size** from Exercise 2 (smaller, same, or — rarely — larger), and why.

### 5. What would change our mind (2–3 bullets)

Name the specific, observable things — a quarterly result, a disclosure, a macro print — that would make you revisit the recommendation. A memo with no exit criteria is a memo nobody can be held accountable to.

## Tasks

1. Write all five sections in `memo-draft.md`, following the structure above.
2. Every quantitative claim gets a query/script citation in parentheses, per Lecture 3, Section 4's reproducibility standard. Count them when you're done — a memo this size should have at least 10 such citations.
3. Read your own memo out loud once. Any sentence that states a number with no citation attached — fix it before moving on.

## Done when…

- [ ] All five sections are present, in order.
- [ ] At least 10 numeric claims each cite a specific query or script + task/step.
- [ ] The governance section names one specific case, one specific mitigation, and one explicit sizing conclusion.
- [ ] The "what would change our mind" section has concrete, observable triggers — not vague statements like "if things get worse."

## Stretch

- Have someone else (a classmate, or re-read it yourself after 24 hours) try to find one number in your memo with no attached query. If they find one, that's the bug this exercise exists to catch.
- Rewrite the Recommendation section to fit in a single tweet-length sentence (≤280 characters) without losing the position size or the target price.

## Submission

Commit `exercise-03/memo-draft.md` to your portfolio under `c35-week-12/exercise-03/`. This draft becomes the backbone of the [mini-project](../mini-project/README.md) — you'll extend, not restart, it on Saturday.
