# Week 9 — Challenges

Two open-ended problems. Unlike the exercises, these have **no single right answer** — the skill is judgment and rigor, not recall. Do them after the three exercises, once your backtest engine is working end to end.

1. **[Challenge 1 — Walk-forward validate a strategy](challenge-01-walk-forward-validation.md)** — roll a train/test split forward through 2024–2025 and report whether the momentum + value strategy's edge survives out-of-sample, or was mostly noise. *(~90 min.)*
2. **[Challenge 2 — Expose a look-ahead bug](challenge-02-expose-a-lookahead-bug.md)** — a backtest script that reports a suspiciously great result contains a real, realistic look-ahead bug. Find it, prove it's there, fix it, and report the honest (worse) number. *(~90 min.)*

## How these are judged

There's no answer key with a single correct Sharpe ratio. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **Rigor of the test design** — did you actually hold out data you didn't peek at, or did the "test" quietly become another training run?
- **Honesty about the result** — did you report the number you got, even when it was worse than you hoped, or did you keep adjusting until it looked good (which defeats the entire exercise)?
- **Diagnosis, not just detection** — for Challenge 2 specifically, can you explain *exactly* what information leaked backward in time and *why* it inflated the result, not just point at the buggy line?
- **Clear written reasoning** — a number with no explanation is not a finding.

Keep your work in `challenge-01.md` / `challenge-02.md` with your code, your results, **and** your written reasoning. The reasoning is the point — this is the week where "it ran and gave a number" stops being good enough.
