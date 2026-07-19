# Week 11 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with hands-on SQL/Python, a written explanation, and an extend-the-model task. Commit each.

All problems continue from the cap table you built in the exercises, unless a problem says otherwise.

---

## Problem 1 — Authorized vs. issued vs. fully diluted, three ways (45 min)

In `terms-writeup.md`, write short (2–3 sentence) answers, each grounded in a real number from Solstice's cap table:

1. What was Solstice's authorized share count at founding, and how do you know it (what evidence in the data tells you, versus what you were simply told)?
2. Give a concrete example, using real numbers from the post-Series-A table, of a share count that is **issued and outstanding** but is different from the company's **fully diluted** share count. Which securities make up the gap?
3. Why does a cap table need to track "authorized" separately from "issued" at all — what real corporate action would ever make those two numbers different?

---

## Problem 2 — Ten SAFE conversion drills (75 min)

Ten independent, invented SAFEs (not Solstice's) — write and run each conversion in `safe-drills.sql`. Assume, for all of them, a founding-basis fully diluted share count of **8,000,000 shares** unless stated otherwise, and a priced round at **$4.00/share** unless stated otherwise.

1. $250,000 SAFE, $8,000,000 cap, no discount. Conversion shares?
2. $250,000 SAFE, no cap, 25% discount. Conversion shares (at the $4.00/share round)?
3. $250,000 SAFE, $16,000,000 cap, 20% discount. Which term binds — cap or discount — and how many shares?
4. $1,000,000 SAFE, $4,000,000 cap, no discount, against a founding-basis share count of **10,000,000** shares instead of 8,000,000. Conversion shares?
5. $500,000 SAFE, $32,000,000 cap, no discount, same round ($4.00/share, 8,000,000 basis). What fraction of a cent does this cap price come out to, and what does that imply about whether this cap will ever bind?
6. Two SAFEs from the same investor, same terms as #1, at two different signing dates six months apart. Do they convert to the same number of shares? Why or why not, under this week's "at signing" convention?
7. A SAFE with **both** no cap and no discount (a plain "most-favored-nation" SAFE, uncommon but real). At the priced round, what price does it convert at, and why is that the worst deal of any SAFE shape for the investor?
8. $150,000 SAFE, $6,000,000 cap, 15% discount, against the $4.00/share round. Confirm the cap binds and compute the shares.
9. Same SAFE as #8, but the round instead prices at **$0.50/share** (a down round). Does the cap still bind? Recompute.
10. In one sentence each: name the one term (cap or discount) that protects an early SAFE investor most when the company's valuation **rises sharply**, and the one that matters most when it **barely moves**.

---

## Problem 3 — Explain the option pool shuffle (45 min)

In `pool-writeup.md`, answer in prose (no more than 400 words total):

1. In your own words, why does creating an option pool *before* pricing a round (rather than after) change who bears its dilution?
2. A founder tells you: "the term sheet says 20% pool, but my ownership dropped by way more than 20% of my prior stake — something's wrong." Explain, using Solstice's actual Series A numbers, why this is completely normal and not a mistake.
3. Would a **smaller** pool target (say, 10% instead of 20%) at the *same* pre-money valuation result in a *higher* or *lower* effective valuation for the founders? Justify your answer with the algebra, not just intuition.
4. Real term sheets sometimes specify the pool as a percentage of the **post-money** cap table instead of pre-money. Without doing the full algebra, state whether you'd expect that convention to produce a bigger or smaller pool than the pre-money convention used this week, for the same stated percentage — and why.

---

## Problem 4 — Waterfall stress test (45 min)

In `waterfall-stress.sql`, using Lecture 3's post-Series-A cap table and preference terms:

1. Find the **exact** exit price at which Series Seed is exactly indifferent between converting and taking its preference (i.e., the true crossover, computed precisely rather than rounded).
2. At an exit price $1 above that threshold, confirm Seed's payout from converting is (very slightly) more than $664,000.
3. At an exit price $1 below that threshold, confirm Seed's payout from taking the preference is exactly $664,000 and that converting would have paid less.
4. What happens to total common stock's payout (Maya + Diego + Option Pool combined) as the exit price crosses this same threshold — does it move smoothly, or does something discontinuous happen? Explain why.

---

## Problem 5 — Extend the model with a fourth SAFE (60 min)

Make the seed round your own and re-run the downstream numbers.

1. Write `INSERT` statements adding a **fourth SAFE** to Solstice's seed round: your own invented investor, amount, and terms (cap, discount, or both — your choice, but justify it as realistic in a comment).
2. Recompute the existing-fully-diluted-shares base (Lecture 2's 10,600,000) including your new SAFE's converted shares.
3. Re-solve the option-pool-shuffle equation with your new base and confirm the new pool size.
4. Reprice Series A end to end with your fourth SAFE included, and report the new fully diluted total and Meridian's new ownership percentage.
5. One sentence: did adding a fourth SAFE investor cost the founders more dilution, less, or the same, compared to Lecture 2's three-SAFE version — and why?

**Deliver** `extend-safe.sql` (all the inserts and recomputation queries) plus your one-sentence answer to (5).

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 75 min |
| 3 | 45 min |
| 4 | 45 min |
| 5 | 60 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
