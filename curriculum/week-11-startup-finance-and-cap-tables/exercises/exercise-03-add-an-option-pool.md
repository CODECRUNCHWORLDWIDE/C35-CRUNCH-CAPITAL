# Exercise 3 — Add an Option Pool

**Goal:** Solve the option-pool-shuffle algebra yourself (both by closed-form and by iteration), insert the pool and Meridian's Series A shares, and land the complete post-Series-A cap table that Lecture 2 walked through.

**Estimated time:** 60 minutes.

## Setup

Continue in `solstice_captable`. Create `exercise-03.sql` and, for Task 4, a short `solve_pool.py`.

## Tasks

1. **Existing fully diluted shares before the pool.** Sum founders (10,000,000) + advisor (100,000) + the two capped SAFE conversions from Exercise 2 (500,000 + 100,000). *(Expected: 10,700,000. Note this is 100,000 higher than Lecture 2's 10,600,000 — this exercise's cap table includes Sam Okafor's advisor grant from Exercise 1, which the lecture's simplified walkthrough didn't carry forward. Carry your own number through the rest of this exercise; that's the correct, more-complete answer.)*

2. **Set up the pool-shuffle equation.** Meridian's term sheet wants the pool at 20% of pre-money fully diluted shares, measured *after* the pool is created. Write out the equation `Pool = 0.20 × (existing_shares + Pool)` using your Task 1 number, and solve it algebraically by hand. Show your work in a comment.

3. **Verify with SQL.** Write a recursive CTE (Postgres) or a manual iteration (SQLite has no `WITH RECURSIVE` support the same way — use a Python loop instead if you're on SQLite) that converges to the same pool size. *(Expected: ≈ 2,675,000 shares — check your Task 2 arithmetic against this.)*

4. **Verify with Python.** In `solve_pool.py`, solve the same equation with plain algebra (`pool = 0.20 * existing / 0.80`) and print the result. Confirm it matches Task 3 to the nearest whole share.

5. **Price the round and issue shares.** Using your pool size from Task 3/4, compute: (a) pre-money fully diluted shares (existing + pool), (b) Series A price per share (`$20,000,000 / pre-money FD`), (c) Meridian's new shares (`$5,000,000 / price per share`), (d) Tom Fischer's discount-SAFE conversion shares (`$64,000 / (price per share × 0.80)`).

6. **Insert everything.** Insert rows into `cap_table_events` for: the two converted capped SAFEs (from Exercise 2), Tom's converted discount SAFE, the option pool, and Meridian's Series A shares. Use `round_name = 'Series-A'` and `event_date = '2024-02-01'` for all five.

7. **Full post-round ownership table.** Re-run the ownership-percentage query from Exercise 1 against the now-complete table. Report every holder's shares and percentage.

## Expected result (spot checks)

- Task 1 → 10,700,000.
- Task 3/4 → pool ≈ 2,675,000 shares (20% of `10,700,000 + pool`).
- Task 5(b) → price per share ≈ $1.4953/share (slightly different from Lecture 2's $1.509434, because your existing-shares base — and therefore the pool — is larger, given Sam's grant carried forward from Exercise 1).
- Task 6 → `SELECT COUNT(*) FROM cap_table_events WHERE round_name = 'Series-A';` returns 5.
- Task 7 → percentages sum to 100.00%, and the option pool sits at almost exactly 20%.

## Done when…

- [ ] `exercise-03.sql` and `solve_pool.py` both exist and agree on the pool size to the nearest share.
- [ ] Your Task 7 output includes every holder from Exercises 1–3: Maya, Diego, Sam, Basecamp, Priya, Tom, Option Pool, Meridian.
- [ ] You can explain, in your own words, why your pool size (≈2,675,000) differs slightly from Lecture 2's worked example (2,650,000) even though both use the same 20% target.
- [ ] Total fully diluted shares after Task 6 matches `existing (10,700,000) + pool + Meridian's new shares + Tom's converted shares`.

## Stretch

- Recompute the whole exercise with a 15% pool target instead of 20%. Does Meridian's post-money ownership percentage change? Should it? *(Hint: revisit the "whose ownership absorbs the pool" section of Lecture 2.)*
- Founders sometimes negotiate the pool size *down* after seeing this math. If Solstice's founders successfully negotiated the pool to 12% instead of 20%, recompute Maya's and Diego's combined post-Series-A ownership percentage and state, in dollars at a hypothetical $25,000,000 exit (assume everyone converts to common, no preferences), how much that negotiation was worth to them.

## Submission

Commit `exercise-03.sql` and `solve_pool.py` to your portfolio under `c35-week-11/exercise-03/`.
