# Week 11 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 12. A mix of multiple-choice and short "compute this" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** A company's charter authorizes 10,000,000 shares. It has issued 8,000,000 to founders and reserved (but not yet granted) a 1,500,000-share option pool. What is the company's **fully diluted** share count?

- A) 8,000,000
- B) 9,500,000
- C) 10,000,000
- D) 1,500,000

---

**Q2.** A founder's grant vests on a standard 4-year schedule with a 1-year cliff. At exactly 10 months after the vest start date, what fraction of the grant has vested?

- A) 10/48
- B) 0 — nothing vests until the 1-year cliff passes
- C) 25%, because the cliff always vests a quarter immediately
- D) 10/12

---

**Q3.** Founder A holds 5,000,000 shares out of 10,000,000 total (50%). The company grants an advisor 200,000 new shares from a previously unissued reserve. What happens to Founder A's **share count** and **percentage**?

- A) Both decrease.
- B) Share count stays the same; percentage decreases.
- C) Share count decreases; percentage stays the same.
- D) Both stay the same.

---

**Q4.** A SAFE has a $5,000,000 valuation cap and no discount. Using this week's "at signing" convention, if the company's fully diluted share count at signing was 5,000,000 shares, what is the SAFE's locked cap price per share?

- A) $0.50
- B) $1.00
- C) $2.00
- D) It cannot be determined until the priced round happens.

---

**Q5.** A SAFE has both a $10,000,000 cap and a 20% discount. At the priced round, the cap implies a $2.00/share conversion price and the discount implies a $1.50/share conversion price. Which price does the SAFE convert at?

- A) $2.00 — the cap always wins when both are present.
- B) $1.50 — the lower price, because it gives the investor more shares for the same money.
- C) The average of the two, $1.75.
- D) Whichever the company chooses.

---

**Q6.** Why does a discount-only SAFE (no cap) necessarily convert at a price that depends on the priced round's own price per share, while a capped SAFE's conversion price (under this week's convention) does not?

- A) It doesn't — both always depend on the round's price.
- B) The discount is defined as a percentage *of* the round's price; the cap is a fixed dollar figure divided by a fixed, already-known share count.
- C) Discount-only SAFEs always convert last, after all capped SAFEs.
- D) There is no real difference; this is a common misconception.

---

**Q7.** An investor's term sheet requires a 20% option pool, created **pre-money**. Existing fully diluted shares (before the pool) are 8,000,000. Solve `Pool = 0.20 × (8,000,000 + Pool)` for the pool size.

- A) 1,600,000
- B) 2,000,000
- C) 2,500,000
- D) 1,000,000

---

**Q8.** Why does an investor-mandated, pre-money option pool dilute the existing shareholders (founders, prior investors) but leave the *incoming* investor's post-money percentage unaffected by the pool's size?

- A) It doesn't — the incoming investor is diluted by the pool exactly like everyone else.
- B) The pool is counted as part of the pre-money share base used to set the price per share, before the new investor's own shares are calculated on top of it.
- C) Investors are contractually exempt from all dilution by law.
- D) The pool only dilutes employees, never investors.

---

**Q9.** A Series A investor puts in $5,000,000 at a $25,000,000 post-money valuation, with no additional pool changes beyond what's already priced in. What is a quick, reliable estimate of their resulting ownership percentage?

- A) $5,000,000 ÷ $20,000,000 (pre-money) = 25%
- B) $5,000,000 ÷ $25,000,000 (post-money) = 20%
- C) It cannot be estimated without knowing the exact share count.
- D) Always exactly 10%, by convention.

---

**Q10.** What does "1x non-participating liquidation preference" mean?

- A) The holder always gets 1x their investment back, plus their pro-rata share of everything else.
- B) The holder gets the **greater of** 1x their investment back, or their as-converted pro-rata share — never both.
- C) The holder gets 1x their investment back only if the company converts to an LLC.
- D) The holder gets 1x the company's total valuation.

---

**Q11.** In a seniority stack of Series B (senior) → Series A → Series Seed → Common, at a modest exit where every preferred class takes its fixed preference, in what order are those preferences paid?

- A) Series Seed first, then Series A, then Series B — juniors get paid first to protect small investors.
- B) All classes are paid simultaneously and proportionally.
- C) Series B first, then Series A, then Series Seed — most senior (typically the most recent money) paid first.
- D) Whichever class has the most shares is paid first.

---

**Q12.** A preferred class's liquidation preference is $2,000,000 and its fully diluted ownership is 10%. Approximately what exit price is its "convert vs. take the preference" crossover?

- A) $2,000,000
- B) $10,000,000
- C) $20,000,000
- D) $200,000

---

**Q13.** At an exit price well above every preferred class's crossover threshold, what does the waterfall payout collapse to?

- A) Every class still takes its fixed preference first.
- B) A pure fully-diluted pro-rata split — liquidation preferences stop mattering.
- C) Common stock receives nothing.
- D) The company must liquidate in seniority order regardless of conversion.

---

**Q14.** How does a **participating** preferred holder's payout differ from a non-participating holder's, given the same preference amount and the same percentage ownership?

- A) They're identical — "participating" only affects voting rights, not economics.
- B) The participating holder takes their fixed preference **and** also shares pro-rata in the remaining proceeds; the non-participating holder gets the *greater of* the two, never both.
- C) The participating holder always gets less, since they must share with more people.
- D) Participating preferred only applies to common stock, never to preferred.

---

**Q15.** Why does adding a new priced round with **no option pool top-up** dilute every existing shareholder's percentage by the exact same multiplicative factor?

- A) It doesn't — dilution always varies by holder based on their security type.
- B) Because every existing holder's share count is unchanged, and the same new total share count (the same denominator) applies to all of them.
- C) Because the new investor's shares are split evenly among all existing holders.
- D) Only true if every existing holder is common stock.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **C** — 10,000,000. Fully diluted counts every share that *could* exist, including the reserved-but-unallocated pool, capped at what's authorized (which happens to equal the fully diluted total here).
2. **B** — nothing vests before the 1-year cliff, full stop. At exactly the cliff, 25% vests all at once (12/48), then monthly after.
3. **B** — Founder A's 5,000,000 shares don't move; the denominator (total shares) grew, so the *percentage* shrinks. This is the core distinction the whole week is built on.
4. **B** — $5,000,000 ÷ 5,000,000 shares = $1.00/share, fixed at signing under this week's convention, independent of anything that happens later.
5. **B** — $1.50, the lower conversion price. Lower price per share means more shares for the same invested dollars — always the better deal for the SAFE holder, and SAFEs are contractually defined to convert at whichever is better for the investor.
6. **B** — the discount is a percentage *of the round's own price*, which doesn't exist until the round is priced; the cap is a fixed dollar amount divided by an already-known historical share count, so it's fully determined the moment the SAFE is signed.
7. **B** — 0.20 × (8,000,000 + Pool) = Pool → 1,600,000 + 0.20×Pool = Pool → 1,600,000 = 0.80×Pool → Pool = 2,000,000.
8. **B** — the pool is folded into the pre-money share base that sets the price per share; the new investor's shares are calculated afterward, on top of that already-diluted base, so their percentage is unaffected by how big the pool was.
9. **B** — $5,000,000 ÷ $25,000,000 = 20%. This shortcut works exactly when there's no pool top-up absorbed differently by the new investor — new money ÷ post-money valuation is always the new investor's clean percentage in that case (see Lecture 2 and Challenge 1).
10. **B** — the defining feature of non-participating preferred: greater of the fixed preference or the as-converted amount, never both. ("Participating" is the term for getting both — Q14.)
11. **C** — most senior first. "Last money in, first money out" is the standard convention; the most recent (usually largest, most senior-negotiated) round gets paid off the top before anyone junior sees a dollar.
12. **C** — $2,000,000 ÷ 0.10 = $20,000,000. The crossover heuristic is preference amount divided by fully diluted percentage.
13. **B** — at a large enough exit, every class is better off converting, so the waterfall becomes a pure pro-rata split across the whole fully diluted table; preferences never trigger.
14. **B** — this is the entire economic difference between the two shapes. Non-participating: greater-of, pick one. Participating: preference *plus* pro-rata — the "double dip" founders and their counsel push back on.
15. **B** — since no new pool shares are added, every existing holder's numerator (their own share count) is unchanged, and the same new, larger total (denominator) divides everyone's stake — producing an identical dilution multiplier for every existing holder, regardless of how many shares they held going in.

</details>

**Scoring:** 12+ → start Week 12. 9–11 → re-read the lecture sections behind your misses, especially the pool shuffle (Lecture 2) and the waterfall decision test (Lecture 3). <9 → re-read all three lectures from the top; this week's ideas compound directly into next week's capstone deal model.
