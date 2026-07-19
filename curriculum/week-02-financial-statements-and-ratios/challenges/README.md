# Week 2 Challenges — Overview

Two challenges, both harder and more open-ended than the exercises. Where the exercises drilled individual ratios one at a time, the challenges ask you to ship something closer to what an actual analyst produces: one dashboard query, and one automated distress-detection query. Do them after all three exercises — both assume the full six-year dataset (FY2020–FY2025) is loaded and correctly ties out.

| # | Challenge | Difficulty | What you ship |
|--:|-----------|------------|----------------|
| 1 | [challenge-01-ratio-dashboard-query.md](./challenge-01-ratio-dashboard-query.md) | Medium | One query, one row per year, every ratio family as columns |
| 2 | [challenge-02-detect-distress-signals.md](./challenge-02-detect-distress-signals.md) | Hard | SQL that automatically flags which years show real warning signs, with a defensible threshold for each |

## How to work them

- These are graded on the *quality and defensibility of your judgment calls* as much as on correct SQL. Where a challenge asks you to pick a threshold ("what counts as dangerously low?"), write down your reasoning — a query with no stated rationale is an incomplete answer even if it runs.
- Reuse queries you already wrote in the exercises rather than starting from scratch — Challenge 1 in particular is largely an assembly job over Exercise 3's individual ratio queries.
- Test your final query against the real, uncorrupted data (don't leave `cash_flow_statement_corrupted` lying around from Exercise 2).

## Submission

Commit each challenge's file(s) to your portfolio under `c35-week-02/challenge-0N/`.
