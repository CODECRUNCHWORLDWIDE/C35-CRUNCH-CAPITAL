# Exercise 2 — Convert a SAFE at a Priced Round

**Goal:** Build the `safes` table, lock each SAFE's cap price, and convert all three into Series A shares using the "greater deal for the investor" rule — cap or discount, whichever gives more shares.

**Estimated time:** 60 minutes.

## Setup

Continue in `solstice_captable` from Exercise 1. Create `exercise-02.sql`.

## Tasks

1. **Create the `safes` table.** Columns: `safe_id` (PK), `investor_name`, `invested_amount`, `valuation_cap` (nullable), `discount_pct` (nullable), `signed_date`.

2. **Insert the three SAFEs.** Basecamp Seed Fund: $500,000, $10,000,000 cap, no discount, `2023-06-01`. Priya Anand: $100,000, $10,000,000 cap, no discount, `2023-06-15`. Tom Fischer: $64,000, no cap, 20% discount, `2023-07-01`.

3. **Lock the capped SAFEs' price per share.** Using the "at signing" convention from Lecture 2, the cap price is `valuation_cap / 10000000.0` (Solstice's founding fully diluted share count — from before the advisor grant; SAFE caps in this course reference the *priced round's own baseline*, not later dilution events like Sam's advisor grant). Write a query returning each capped SAFE's `cap_price_per_share`. *(Expected: exactly $1.00/share for both Basecamp and Priya.)*

4. **Convert the capped SAFEs.** Compute each capped SAFE's conversion share count: `invested_amount / cap_price_per_share`. *(Expected: Basecamp → 500,000 shares; Priya → 100,000 shares.)*

5. **Price the Series A round.** Given pre-money valuation $20,000,000 and a pre-money fully diluted share count of 13,250,000 (10,600,000 existing + 2,650,000 option pool — you'll derive that pool number yourself in Exercise 3; for now, take it as given), compute the price per share. *(Expected: ≈ $1.509434/share.)*

6. **Convert Tom's discount-only SAFE.** His conversion price is 80% of the Series A price per share from Task 5. Compute his conversion share count. *(Expected: 53,000 shares exactly. If you don't get a whole number, re-check Task 5's price — Tom's numbers were designed to land clean.)*

7. **A discount-vs-cap comparison (the "greater deal" rule).** Suppose, hypothetically, Tom's SAFE *also* had a $12,000,000 cap. Using the founding-basis convention from Task 3, what would his cap price per share be, and how many shares would that buy for $64,000? Compare that share count to Task 6's answer and state which term — the (hypothetical) cap or the actual discount — would give Tom more shares, and therefore which one his SAFE would convert under.

## Expected result (spot checks)

- Task 3 → both capped SAFEs price at exactly $1.00/share.
- Task 4 → 500,000 and 100,000 shares.
- Task 5 → ≈ $1.509434/share.
- Task 6 → 53,000 shares exactly.
- Task 7 → a $12M cap at the founding-basis convention prices at $1.20/share ($12,000,000 / 10,000,000), which buys `64000/1.20 ≈ 53,333` shares — *more* than the 53,000 from the discount, so in that hypothetical, the cap would win and the SAFE would convert under the cap terms instead.

## Done when…

- [ ] `exercise-02.sql` has all 7 tasks.
- [ ] You can state, in one sentence, why a capped SAFE's conversion price doesn't depend on the Series A price per share at all, while a discount-only SAFE's conversion price depends on nothing else.
- [ ] Task 7's comparison is correct and you can explain why more shares is always the better outcome for the SAFE investor (same dollars invested, bigger stake).

## Stretch

- Write a single SQL query using a `CASE` expression that computes the conversion share count for *any* SAFE row — capped, discount-only, or (hypothetically) both — picking whichever price is lower (i.e., whichever share count is higher), given a Series A price per share as an input value.
- What would happen to Basecamp's and Priya's conversion share counts if their cap had instead been $20,000,000 (equal to the Series A pre-money valuation)? Compute it and explain, in one sentence, why a cap set equal to (or above) the priced round's valuation makes the cap essentially worthless to the investor.

## Submission

Commit `exercise-02.sql` to your portfolio under `c35-week-11/exercise-02/`.
