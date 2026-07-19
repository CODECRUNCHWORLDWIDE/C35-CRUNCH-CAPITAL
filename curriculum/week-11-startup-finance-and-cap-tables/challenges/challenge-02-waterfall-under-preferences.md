# Challenge 2 — Model a Preference Waterfall

**Time:** ~60 minutes. **Difficulty:** Medium-hard.

## The scenario

Continue from Challenge 1's three-round cap table (Founding → Series A → Series B, 20,769,375 fully diluted shares). Series B's term sheet includes its own liquidation preference:

- **Series B Preferred (Alpine Growth Capital):** 1x, **non-participating**, $15,000,000 preference, **most senior** (last money in, first out — senior to both Series A and Series Seed).
- Series A and Series Seed keep the terms from Lecture 3 ($5,000,000 and $664,000, in that seniority order beneath Series B).

Full seniority stack, most senior first: **Series B → Series A → Series Seed → Common.**

## Your task

### Part A — three-round crossover thresholds

1. Compute each preferred class's **fully diluted percentage** of the post-Series-B cap table (20,769,375 shares).
2. Compute each class's **crossover exit price** — the exit value above which that class is better off converting to common than taking its fixed preference. Use the same heuristic from Lecture 3 (preference ÷ fully diluted %).
3. One of the three crossover thresholds comes out to a suspiciously round number. Find it, and explain algebraically why it's clean — the same kind of reasoning you used for Challenge 1's "clean percentage" question.

### Part B — run the waterfall at two exit prices

Compute the full cascading payout (who takes their preference, who converts, and the exact dollar amount to every holder) at:

4. **Exit = $30,000,000.** *(This should produce a genuinely three-way mixed outcome — one class converts, two take their preference. State which is which and why, using your Part A thresholds.)*
5. **Exit = $80,000,000.** *(Every threshold from Part A should be cleared here — confirm all three preferred classes convert, and that the payout collapses to a pure fully-diluted pro-rata split.)*

### Part C — participating vs. non-participating (the expensive comparison)

6. Recompute the **$30,000,000** waterfall under a hypothetical where **Series B is participating** instead of non-participating — meaning Alpine takes its $15,000,000 preference *and* also shares pro-rata in whatever remains, alongside every class that converted (or was always common).
7. Report, in dollars: how much *more* Alpine receives under participating terms, and how much *less* the Series Seed holders and the common holders (combined) receive as a result. These two numbers should be closely related — explain why.
8. In one paragraph: why do experienced founders and their lawyers fight hard against participating preferred, even when the headline preference amount and percentage ownership are identical to a non-participating deal?

## Constraints

- Use the two-step method from Lecture 3 throughout: decide (heuristic against total exit value), then pay out (cascading, seniority order, remaining pool split pro-rata among everyone who didn't take a preference).
- Every scenario's total payout (sum across all holders) must equal the exit price, within a few dollars of rounding. State that check explicitly in your write-up for both Part B computations.
- For Part C, only Series B's participation status changes — Series A and Seed stay non-participating in every version.

## Hints

<details>
<summary>On the clean crossover threshold (Part A.3)</summary>

You already know from Challenge 1 that Series B's stake is exactly 20% of the post-money cap table for algebraic reasons tied to "no pool top-up." A preference of $15,000,000 against an exactly-20% stake produces a crossover threshold that's a very round number — work it out and you'll see why 1x preference ÷ (a clean 1/5 fraction) can't help but be clean.

</details>

<details>
<summary>On Part C's "closely related" numbers</summary>

Participation doesn't create money — the $30,000,000 pie is the same size either way. Every extra dollar Alpine gets from participating comes directly out of the pool that Seed-converters and common would otherwise have split. If you've computed both scenarios correctly, Alpine's gain and (Seed + common)'s combined loss should match almost exactly, modulo rounding.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Thresholds | Computed for one class only, or arithmetic errors | All three, correctly derived, with the clean one explained algebraically |
| Waterfall mechanics | Skips straight to a guessed split | Shows the decide-then-cascade method explicitly, seniority order respected |
| Reconciliation | Payouts don't sum to the exit price | Both Part B scenarios sum to the exit price (within rounding) |
| Participating comparison | States "participating is worse for founders" without numbers | Quantifies Alpine's gain and Seed+common's loss, shows they're linked |

## Submission

Commit `challenge-02.md` (queries + written analysis) to your portfolio under `c35-week-11/challenge-02/`.
