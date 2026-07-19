# Week 12 — Homework

Five problems, ~5 hours total, spread across the week. These extend the lectures and exercises with more reps at valuation assembly, risk quantification, and governance reasoning — and one problem that asks you to apply the whole week to a target of your own choosing, which is genuinely good practice for using this framework after the course ends.

All work runs against `crunch_capital` and this week's `capstone_*` tables unless a problem says otherwise.

---

## Problem 1 — A second exit multiple (60 min)

`capstone_deal_snapshot.exit_multiple_ev_ebitda` is 8.0x. Recompute the exit-multiple DCF (Lecture 1, Section 2–3) at exit multiples of **6.0x**, **8.0x** (base), **10.0x**, and **12.0x**. Plot (by hand, in a small markdown table — or with `matplotlib` if you want the practice) per-share value against exit multiple.

**Deliver** `problem-01.sql` (or `.py`) and a one-sentence answer: is the exit-multiple DCF's per-share value roughly linear in the exit multiple, or does it curve? Why does that make sense given the formula?

---

## Problem 2 — VaR at a different confidence level and horizon (60 min)

Exercise 2 computed 95%/99% VaR at 1-day and 10-day horizons. Compute the **90%** and **99.9%** 1-day parametric VaR for the CRNM position (`z(90%) ≈ 1.282`, `z(99.9%) ≈ 3.090`). Then compute the **20-day** 95% VaR using the square-root-of-time rule.

**Deliver** `problem-02.py` with all four new figures, plus two sentences: how much does VaR grow from 90% to 99.9% confidence (same horizon), and how much does it grow from 1-day to 20-day (same confidence)? Which lever — confidence or horizon — moves the number more, proportionally?

---

## Problem 3 — Rewrite the scenario set (75 min)

`capstone_scenarios` has six named scenarios. Design **two new scenarios** of your own — give each a name, a probability, and shocks to growth/margin/multiple/WACC — such that all eight probabilities (six original, minus a small amount from each to make room, plus your two new ones) still sum to exactly 1.00.

Justify each new scenario in 2–3 sentences: what real-world situation does it represent, and why did you set the shocks and probability the way you did?

**Deliver** `problem-03.md` with your two new scenarios, your justification, and the recomputed probability-weighted expected per-share value across all eight scenarios (compare it to Exercise 2's six-scenario answer).

---

## Problem 4 — A governance case of your own (75 min)

Using the same structured format as `capstone_governance_cases` (company alias, industry, incentive structure, failure mode, outcome, lesson), write **one new row** based on a real, public governance failure you research yourself (a real company, but you should still use a fictional alias in your write-up per the course's convention of not naming real companies in coursework).

Use a reputable source — a company's own proxy statement or 8-K filing on [SEC EDGAR](https://www.sec.gov/cgi-bin/browse-edgar), a major business publication's reporting, or an academic case study. Cite your source.

**Deliver** `problem-04.md`: your new row in the same 6-field format, a 150-word summary of what actually happened (with your source cited), and one sentence on which of this week's `capstone_governance_cases` rows it most resembles and why.

---

## Problem 5 — Run the framework on a different target (90 min)

Pick a **different** fictional target company — you can invent one from scratch, or lightly adapt CRNM's numbers (different revenue, margin, capital structure, or WACC). Build your own version of `capstone_deal_snapshot` and `capstone_cash_flows` for it (five years of drivers, your choice of growth/margin path).

Run a condensed version of this week's process: (1) a DCF per-share value by both terminal-value methods, (2) one named stress scenario of your choosing, (3) one governance risk you'd flag for a company in that industry.

**Deliver** `problem-05.md` with your invented company's assumptions, your DCF result, your stress scenario result, and your governance flag. This is deliberately open-ended — the point is proving to yourself that the framework generalizes past CRNM, which is the actual test of whether you learned the method or just this week's specific numbers.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 60 min |
| 2 | 60 min |
| 3 | 75 min |
| 4 | 75 min |
| 5 | 90 min |
| **Total** | **~6 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
