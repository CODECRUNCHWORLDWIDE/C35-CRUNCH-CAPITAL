# Week 10 — Exercises

Three guided exercises, ~60–90 min each. Together they build the full pipeline this week's mini-project depends on: a base-case accretion/dilution model (Exercise 1), a synergy valuation layered on top of it (Exercise 2), and an independent LBO of the same target (Exercise 3). Do them in order — Exercise 2 re-runs Exercise 1's model with an extra line item, and the mini-project assembles all three.

1. **[Exercise 1 — Accretion/dilution model](exercise-01-accretion-dilution-model.md)** — build CRNM's pro forma combined income statement from `acquirer_financials`, `target_financials`, and `deal_terms`, and prove the base-case (no-synergy) deal is accretive.
2. **[Exercise 2 — Value synergies](exercise-02-value-synergies.md)** — compute the NPV of both synergy streams in `synergies`, then re-run Exercise 1's accretion/dilution model with the full run-rate synergies included.
3. **[Exercise 3 — Simple LBO](exercise-03-simple-lbo.md)** — build SPSY's 5-year cash-sweep debt schedule from `lbo_assumptions` and compute the sponsor's MOIC and IRR at exit.

## Before you start

- You've read all three lectures.
- You ran this week's seed from the [README](../README.md) and all five sanity-check counts (`1`, `1`, `1`, `2`, `1`) came back correct.
- You have a Postgres or SQLite shell open, and a Python environment with `pandas`, `numpy`, and `sqlalchemy` installed and activated.

## Suggested workflow

- Work Exercise 1 fully before starting Exercise 2 — Exercise 2 adds exactly one line item (synergies) to Exercise 1's pro forma statement, so a bug in Exercise 1 will silently propagate into Exercise 2's "accretion with synergies" answer.
- For every calculation, write the formula before you run it, then check your output against each exercise's "Expected results" section — a deal model that "looks about right" but doesn't match a known-correct check is not a model you can defend in front of a board.
- Save your work: `solutions.sql` for SQL tasks, `solutions.py` for Python tasks, one section per exercise under a numbered comment. You'll reuse and extend these files in the mini-project.

## Note on precision

Deal math compounds rounding fast — a purchase price, a share count, an interest rate, and a tax rate all multiply together, and a one-share rounding choice on `new_shares_issued` can move pro forma EPS in the fourth decimal place. The "Expected results" in each exercise give a value with enough tolerance (usually ±0.5%) to accommodate reasonable rounding-order differences — if you're within that band, you're right. If you're off by more, you have a real bug, not a rounding artifact.
