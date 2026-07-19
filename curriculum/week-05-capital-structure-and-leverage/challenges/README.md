# Week 5 — Challenges

Two open-ended problems. Unlike the exercises, these ask you to **solve** a model, not just evaluate one at given inputs, and to **stress-test** a real financial position — the skill is finding a specific number under constraints and defending it. Do them after the three exercises.

1. **[Challenge 1 — Optimal debt under tradeoff](challenge-01-optimal-debt-under-tradeoff.md)** — solve for the value-maximizing debt level two independent ways (calculus and grid search), confirm they agree, and test how sensitive the answer is to the model's assumptions. *(~90 min.)*
2. **[Challenge 2 — Leverage blow-up scenario](challenge-02-leverage-blowup-scenario.md)** — find the exact EBIT shock that breaches a debt covenant and the one that wipes out pretax income, for Crunch Machine Co.'s actual FY2025 capital structure. *(~90 min.)*

## How these are judged

There's no single numeric answer key for either challenge — the model's assumptions (`α`, `rU`, covenant thresholds) are inputs you're told to reason about, not values handed to you unquestioned. You're being graded on:

- **Correctness of the core mechanics** — does your calculus derivation actually match your numerical grid search? Does your covenant-breach EBIT figure actually produce the stated coverage ratio when you plug it back in?
- **Sensitivity awareness** — do you show how the answer moves when an assumption changes, not just report one static number?
- **Explicit reasoning** — do you state your assumptions and show your work, not just present a finished number?

Keep your work in `challenge-01/` and `challenge-02/` with your code **and** your written reasoning. The reasoning is the point.
