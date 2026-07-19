# Mini-Project — Three Rounds to an Exit

> Build Solstice Data's complete cap table from incorporation through a Series B, entirely in SQL and Python, then run a full liquidation-preference waterfall to compute exactly what every shareholder walks away with at an acquisition. One database, three financing rounds, one exit — proof that you can model a startup's ownership history end to end, not just answer isolated textbook questions about it.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone. Nobody hands you a finished cap table — you build it, event by event, exactly the way it actually happens: incorporation, then a seed round nobody can price yet, then a Series A that finally prices the company and simultaneously dilutes the founders with an option pool they didn't ask for the exact size of, then a Series B, and then — the moment every one of those earlier decisions was made in service of — an exit, where a liquidation preference stack decides who actually gets paid what. Do this once, correctly, in a database, and you'll never again read a term sheet without knowing exactly which line to worry about.

---

## Deliverable

A directory in your portfolio `c35-week-11/mini-project/` containing:

1. `schema.sql` — every `CREATE TABLE` statement, run in order, that builds the full data model (cap table events, SAFEs, financing rounds, liquidation terms).
2. `build_captable.sql` — every `INSERT`/`UPDATE` that takes Solstice from two founders to a fully diluted post-Series-B cap table.
3. `waterfall.sql` (and/or `waterfall.py`) — the exit waterfall computation, parameterized so it can be re-run at any exit price.
4. `report.md` — for each of the questions below: the answer (the actual numbers), and one to two sentences of interpretation.
5. `notes.md` — a short reflection (see the end).

Use the canonical numbers from the three lectures as your starting point (Founding: 10,000,000 founder shares; Series A: $20M pre-money, $5M new money, 20% pool target, three SAFEs; Series B: $60M pre-money, $15M new money, no pool top-up). Works on PostgreSQL or SQLite; note which you used.

---

## The build (do this first, in order)

### Part 1 — Founding

1. Create the schema: `cap_table_events`, `safes`, `financing_rounds`, `liquidation_terms`.
2. Insert Maya Chen (5,500,000 shares, 55%) and Diego Ruiz (4,500,000 shares, 45%), founding date `2023-02-01`, standard 4yr/1yr-cliff vesting.

### Part 2 — Seed SAFEs and Series A

3. Insert the three SAFEs (Basecamp $500,000/$10M cap, Priya $100,000/$10M cap, Tom $64,000/20% discount).
4. Lock the capped SAFEs' conversion price against the 10,000,000-share founding basis.
5. Solve the option-pool-shuffle equation for a 20% pre-money (post-topup) target against the 10,600,000-share existing base (founders + capped SAFEs). Confirm you land on 2,650,000 pool shares.
6. Price Series A, issue Meridian Ventures' shares, convert Tom's discount SAFE.
7. Confirm your post-Series-A fully diluted total is **16,615,500 shares**, matching Lecture 2.

### Part 3 — Series B

8. Insert a Series B round: Alpine Growth Capital, $60,000,000 pre-money, $15,000,000 new money, no pool top-up, close date `2025-08-01`.
9. Price the round and issue Alpine's shares. Confirm your post-Series-B fully diluted total is **20,769,375 shares**, and that Alpine's stake is exactly **20.00%**.

### Part 4 — The exit waterfall

10. Build `liquidation_terms` with the full three-round seniority stack: Series B ($15,000,000, most senior) → Series A ($5,000,000) → Series Seed ($664,000) → Common (no preference, residual).
11. Compute each preferred class's crossover exit price (the threshold above which it converts rather than takes its preference).
12. Write a parameterized waterfall query/script — given any exit price, it should return every holder's exact payout, using the decide-then-cascade method from Lecture 3.
13. Run it at **three exit prices of your own choosing**, spanning at least one scenario where every preferred class takes its preference, one genuinely mixed scenario, and one scenario where every class converts. Report all three in `report.md`.

---

## Report questions (answer all in `report.md`)

1. What is Maya Chen's fully diluted ownership percentage after Founding, after Series A, and after Series B?
2. What is the option pool's size in shares and percentage right after it's created at Series A? Does that percentage change at Series B, and if so why?
3. List each preferred class's crossover exit price, and explain in one sentence why Series B's is the highest of the three.
4. For your three chosen exit prices: which preferred classes took their preference and which converted, at each price?
5. At your lowest exit-price scenario, how much (in dollars) do Maya and Diego combined receive, and what fraction of the total exit proceeds does that represent? Compare that fraction to their fully diluted ownership percentage and explain the gap.
6. At your highest exit-price scenario, verify that total payouts across all holders sum to the exit price (within rounding), and state the per-share value everyone effectively received.
7. If Series B's $15,000,000 preference were instead **2x non-participating** (a much more aggressive, though not unheard-of, term) instead of 1x, recompute your lowest-exit-price scenario. How much less do the founders receive, in dollars?

---

## Milestones

- **Milestone 1 (45 min):** Part 1 + 2 — founding through Series A, matching the lecture's canonical 16,615,500-share total exactly before moving on.
- **Milestone 2 (30 min):** Part 3 — Series B, matching the 20,769,375-share total and Alpine's clean 20.00% stake.
- **Milestone 3 (60 min):** Part 4 — the waterfall query/script, tested against all three of Lecture 3's original single-round scenarios ($8M/$20M/$100M against the *post-Series-A* table) before you trust it on the harder three-round, three-preference-class version.
- **Milestone 4 (30–45 min):** Run your three chosen exit scenarios, write `report.md`, and the 2x-preference variant in question 7.

---

## Rules

- **Build additively.** Every part inserts new rows; nothing from an earlier part gets rewritten or deleted. If you find a mistake in Part 1 after finishing Part 3, add a correcting insert or a clear note — don't quietly edit history.
- **Verify each milestone's fully diluted total before moving on.** A wrong Series A total silently poisons every later number; catch it at 16,615,500, not at the waterfall.
- **The waterfall must reconcile.** Total payout across all holders equals the exit price (allow a few dollars of rounding, no more). If it doesn't, you have a bug — find it before reporting numbers.
- **State every assumption.** Rounding conventions, how you handled fractional shares, which convention you used for "pre-money" pool sizing — write it down in `report.md` wherever it could plausibly be done a different way.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness through Series A | 25% | Matches Lecture 2's 16,615,500-share total and every individual holder's shares exactly |
| Correctness through Series B | 20% | Matches the 20,769,375-share total; Alpine's stake is exactly 20.00% |
| Waterfall mechanics | 30% | Decide-then-cascade method correctly applied at all three chosen exit prices; every scenario reconciles to the exit price |
| Interpretation | 15% | `report.md` states answers in plain language with real numbers, not just raw query output |
| Stated assumptions | 10% | Every judgment call (rounding, conventions) is written down |

---

## Reflection (`notes.md`, ~200 words)

1. Which part of the build was hardest to get exactly right — the SAFE conversions, the pool shuffle, or the waterfall — and why?
2. Where did a small early number (a pool size, a share count) compound into a bigger discrepancy later, and how did you catch it?
3. If you were advising Solstice's founders before they signed the Series A term sheet, what's the one clause (pool size, preference multiple, participation) you'd push back on hardest, based on what you now know it costs them numerically?
4. What would change about this model if Solstice had to run this waterfall in real life, with actual legal documents instead of a simplified teaching convention? Name one specific simplification from this week you'd need to unwind.

---

## Stretch goals

- Model a **down round** — a hypothetical Series C priced *below* Series B's valuation — and show what happens to the SAFE-holders' and Series A/B investors' percentages, plus whether any **anti-dilution** protection (full-ratchet or weighted-average, briefly mentioned in `resources.md`) would apply. This is genuinely one of the more painful real-world cap-table scenarios and worth a first look even if you don't fully model it.
- Add a **secondary sale** — a founder selling 500,000 of their vested shares to a new investor at the Series B price, with no change to the company's total share count. Show how this differs from a primary issuance (a new-money round) in its effect on everyone else's percentage.
- Build a single Python function `waterfall(exit_price, cap_table_df, liquidation_terms_df) -> dict[holder, payout]` that reproduces every scenario in this project and Challenge 2 from one reusable piece of code, then unit-test it against the lecture's known answers.

---

## Why this matters

This mini-project is the shape of real startup finance work: nobody sits down and directly asks "what's the exit waterfall?" — you get there by faithfully modeling every event that happened before it, in order, and trusting the math to tell you the answer instead of guessing. Do this once end to end and a term sheet stops being a wall of legalese and starts being a set of parameters you can plug into a model you already understand.

When done: push, then take the [quiz](../quiz.md) and move on to [Week 12 — Capstone: full deal, risk, and governance](../../week-12-capstone-deal-risk-and-governance/).
