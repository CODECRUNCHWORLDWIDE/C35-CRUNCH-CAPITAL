# Week 3 — Challenges

Two open-ended problems. Unlike the exercises, these have **no single right answer** — the skill is constructing a real case and defending a position with numbers, not recall. Do them after the three exercises.

1. **[Challenge 1 — Find the case where IRR lies](challenge-01-irr-lies-case.md)** — construct a pair of mutually exclusive projects where ranking by IRR picks the wrong one, solve for the exact crossover rate, and prove it with a full NPV profile. *(~90 min.)*
2. **[Challenge 2 — Stress the hurdle rate](challenge-02-sensitivity-on-hurdle-rate.md)** — sweep the hurdle rate across a realistic range for a project from `cash_flows`, find the exact rate where its accept/reject decision flips, and argue whether the company's actual hurdle rate is safely on the right side of that line. *(~90 min.)*

## How these are judged

There's no answer key with exact numbers to match — your constructed cases will have different numbers from anyone else's. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of the core mechanics** — do your NPV, IRR, and crossover-rate calculations actually check out when someone else runs your numbers?
- **A genuinely constructed case, not a restated example.** Reusing this week's lecture numbers verbatim doesn't demonstrate you understand *why* the trap occurs — build your own.
- **Explicit reasoning** — show your work (the full NPV profile, the root-finding call, the sensitivity sweep), and write the argument in plain English a non-technical stakeholder could follow.

Keep your work in `challenge-01/` (code + a short writeup) and `challenge-02/` (code + a short writeup). The reasoning is the point — a bare number with no explanation earns partial credit at best.
