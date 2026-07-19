# Challenge 1 — Track Dilution Through Series B

**Time:** ~60 minutes. **Difficulty:** Medium.

## The scenario

Eighteen months after Series A, Solstice is growing fast and raises a Series B led by **Alpine Growth Capital**:

- Pre-money valuation: **$60,000,000**
- New money: **$15,000,000** (post-money: $75,000,000)
- Close date: `2025-08-01`
- **No option pool top-up this round** — the 20%-ish pool from Series A still has enough headroom that Alpine doesn't require a refresh. *(This is realistic: not every round needs a pool top-up. Whether one is needed depends on how much of the existing pool is left unallocated and what the incoming investor negotiates.)*

Your job: build the Series B round into the cap table, then produce a **three-round ownership history** — Founding, Series A, Series B — for every holder, and explain what happened to each one's percentage along the way.

## Your task

1. **Extend `cap_table_events`** with a Series B round (`round_name = 'Series-B'`, `event_date = '2025-08-01'`) using Lecture 2's post-Series-A cap table as your starting fully diluted share count (16,615,500 shares).

2. **Price the round.** Because there's no pool top-up, Series B's pre-money fully diluted share count is simply the existing 16,615,500. Compute the price per share and Alpine's new share count.

3. **Insert Alpine's row** into `cap_table_events`.

4. **Build the three-round ownership table.** Write one query that returns every holder's shares and ownership percentage **as of each round's close** — Founding, Series A, Series B — as three columns or three stacked rows per holder (your choice; state which you picked). *(Hint: a `SUM(shares) OVER (...)` with a filter on `event_date <=` the round's close date, computed once per round, is one clean way to do this — a window function scoped by a `WHERE`/subquery per round, or three separate queries `UNION ALL`'d together with a `round_label` column.)*

5. **Written analysis (in `challenge-01.md`).** Answer, with numbers from your query:
   - What was Maya's ownership percentage after Founding, after Series A, and after Series B?
   - Alpine's Series B stake works out to a clean, round percentage of the post-money cap table. Compute it and explain algebraically *why* it comes out clean when a round has no pool top-up — tie your explanation back to the definition of post-money valuation.
   - Every other holder's percentage from right after Series A to right after Series B changes by the exact same multiplicative factor. Find that factor, verify it against two different holders' before/after percentages, and explain in one sentence why a no-pool-topup round dilutes every existing holder *uniformly*.
   - The option pool never received any new shares in Series B, yet its ownership percentage still dropped. Why?

## Constraints

- Don't touch or "correct" any Series A rows — Series B is purely additive, same as every round has been all week.
- Round share counts to the nearest whole share; state your rounding in the write-up if it changes a downstream number by more than a few shares.

## Hints

<details>
<summary>On the "clean percentage" question</summary>

Post-money valuation is pre-money + new money by definition. If there's no pool top-up, price per share is set purely by pre-money valuation over the existing share count — meaning the new investor's shares (new money ÷ price per share) as a fraction of the new total (existing + new) reduces algebraically to new money ÷ post-money valuation, with the existing share count cancelling out entirely. Try the algebra with variables instead of numbers and you'll see it.

</details>

<details>
<summary>On the uniform dilution factor</summary>

If nothing else changes and only new shares are added, every existing holder's share count is untouched — the multiplier that changes their *percentage* is simply (pre-money FD shares) ÷ (post-money FD shares), the same ratio for every existing holder, since the numerator of their fraction (their own share count) doesn't move and the denominator (total shares) grows by the same amount for everyone.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Mechanics | Series B priced inconsistently with Lecture 2's method | Same pre-money/price-per-share method, correctly applied with no pool term |
| Reconciliation | Percentages don't sum to 100% at any round | All three rounds' percentages independently sum to 100.00% |
| The "clean %" derivation | States the number without deriving why it's clean | Shows the algebra, generalized with variables, not just plugged numbers |
| The uniform-dilution insight | Notices the factor but can't explain why it's the same for everyone | Explains it from first principles (share count fixed, denominator grows) |

## Submission

Commit `challenge-01.md` (with your queries and written analysis) to your portfolio under `c35-week-11/challenge-01/`.
