# Week 3 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on computation, a written explanation, and two build-your-own-case tasks. Commit each.

All computations run against the `cash_flows` seed from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Hand-calculate before you code (45 min)

Before touching Python, discount **by hand** (calculator allowed, code not allowed) the Brand Relaunch Campaign's cash-flow stream at its 10% hurdle rate:

```
Period 0: -220,000
Period 1:   80,000
Period 2:   95,000
Period 3:   70,000
Period 4:   40,000
Period 5:   20,000
```

1. Compute the discount factor `1/(1.10)^t` for each period, to 4 decimal places.
2. Multiply each cash flow by its discount factor.
3. Sum the results. You should land within a few cents of **$23,570.67**.

**Deliver** `problem1.md` with your period-by-period table (period, cash flow, discount factor, discounted value) and the final sum. If you don't land close to $23,570.67, find your error before moving on — do not just copy the target number in.

---

## Problem 2 — Full metric sweep on the two rejected-but-closest projects (75 min)

Two projects in this week's table failed to clear their hurdle rate but came reasonably close: **AI Quality-Control R&D** (IRR 13.27% vs. a 15% hurdle) and **Regional Warehouse Expansion** (IRR 8.60% vs. a 10% hurdle).

For each of these two projects, compute and report:

1. NPV at the stated hurdle rate (confirm it's negative).
2. IRR (confirm it's below the hurdle rate).
3. The exact percentage-point gap between IRR and hurdle rate.
4. Simple payback period.
5. Discounted payback period (confirm it's `inf` — the project never recoups its investment on a discounted basis within its 5-year life).

**Deliver** `problem2.sql` and `problem2.py` with all six numbers (two projects × 3 numeric metrics + 2 payback figures... report all 5 metrics per project, 10 numbers total), plus one sentence per project on whether you think a modest re-forecast of its revenue (say, 10% higher in every period) would be enough to flip its NPV positive. You don't have to compute this precisely — an informed guess, stated as a guess, is fine here.

---

## Problem 3 — Explain the screening step (writeup, 45 min)

In `screening-writeup.md`, answer in prose (no more than 400 words total):

1. Explain, in terms a first-year MBA student would understand, why a project can have a large, impressive-looking **raw** cash-flow total and still have negative NPV. Use the Regional Warehouse Expansion ($650,000 invested, $845,000 raw total inflow, yet negative NPV) as your concrete example.
2. Explain why "we have the budget for it" is never, on its own, a valid reason to fund a negative-NPV project.
3. Give a real-world example (not from this course) of a business decision where you suspect the "obviously good idea" framing might be hiding a negative-NPV reality once properly discounted.
4. In one paragraph, explain the difference between rejecting a project because its **NPV is negative** versus rejecting it because your **capital budget ran out** — these are different reasons with different implications for what you tell the project's sponsor.

---

## Problem 4 — Build your own three-project budget puzzle (90 min)

Construct three **independent**, hypothetical projects (your own numbers — 3 to 5 periods each, one initial outflow followed by inflows, plausible dollar amounts) such that:

1. All three have positive NPV at a hurdle rate you choose.
2. A capital budget you choose is enough for **any two** of the three, but not all three.
3. Ranking by raw NPV and ranking by PI select a **different** pair of two projects to fund.

Report all three projects' cash flows, your chosen hurdle rate and budget, their NPV/PI table, and both rankings' funding decisions side by side. Then state, in one paragraph, which pair actually creates more total value, and confirm your PI-based pick is the winner (per Lecture 3, Section 5's Lorie-Savage argument).

**Hint:** work backward from Lecture 3's X/Y/Z illustration — you need one project that's "big NPV, mediocre PI" and two smaller projects that are "smaller NPV each, better PI," where the two smaller ones' combined cost fits the budget alongside (or instead of) the big one.

**Deliver** `problem4.py` (or `.sql` + `.py`) with the full construction and both rankings' outcomes.

---

## Problem 5 — Read the seed table like an analyst (60 min)

Without re-deriving any numbers, just from the `hurdle_rate` and `risk_tier` columns already in `cash_flows`:

1. List the six projects in order from lowest to highest hurdle rate.
2. For each, state whether its risk tier ("Low", "Medium", "High") seems consistent with where it sits in that ordering — do risk tier and hurdle rate move together the way Lecture 1, Section 4 says they should?
3. Now cross-reference this ranking against your NPV results from Exercise 1. Is there any visible relationship between a project's risk tier and whether it actually cleared its hurdle rate, in this particular slate? (There may not be a clean pattern — say so if that's what you find; don't force a conclusion the data doesn't support.)

**Deliver** `problem5.md` with your table and a short (150–250 word) written answer to part 3.

---

## Submission checklist

- [ ] `problem1.md`
- [ ] `problem2.sql` + `problem2.py`
- [ ] `screening-writeup.md`
- [ ] `problem4.py` (or `.sql` + `.py`)
- [ ] `problem5.md`

When all five are done, move on to [`quiz.md`](./quiz.md).
