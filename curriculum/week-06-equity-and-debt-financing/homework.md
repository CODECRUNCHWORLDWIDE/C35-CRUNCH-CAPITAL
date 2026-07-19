# Week 6 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of SQL, Python, hand calculation, and written reasoning. Commit each.

All queries and calculations run against this week's `company_snapshot`, `shareholders`, and `financing_terms` seed from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Twenty equity and debt drills (90 min)

Mix of SQL, Python, and hand calculation — your choice per question, but do at least 5 by hand first with the arithmetic shown, and confirm the rest with code. Put everything in `drills.md` with the question number, method used, and answer.

1. Gross proceeds needed to net $15,000,000 at a 4% underwriting fee.
2. Shares issued to raise that $15,000,000 (net) at an $18.00 offer price, rounded up to a whole share.
3. Using this week's actual 10,000,000-share, $20.00 cap table, what percentage of the company would a **new** $15,000,000 raise (from Q1–Q2) dilute existing holders by, in total?
4. The Employee Option Pool's ownership percentage before and after this week's actual $20,000,000 raise (1,108,034 shares issued).
5. Annual payment on a $12,000,000, 6%, 4-year, annually-amortizing loan.
6. Total interest paid over that loan's full life.
7. Remaining balance on that loan after 2 of its 4 years.
8. After-tax cost of debt at an 8% coupon and a 21% tax rate (a US federal corporate rate, for contrast with this week's 25% company-specific rate).
9. Annual cost of equity capital on $10,000,000 of net proceeds at a 9% cost of equity.
10. Total 4-year cost of Q5–Q7's loan (after-tax interest, no fee) versus total 4-year cost of Q9's equity ($10,000,000 at 9%, over 4 years) — which is cheaper, and by how much?
11. Leverage ratio for a company with $40,000,000 total debt and $16,000,000 EBITDA.
12. Is that Q11 leverage ratio inside or outside this week's 3.00× covenant threshold?
13. Interest coverage ratio for a company with $9,000,000 EBIT and $2,250,000 total interest expense.
14. Is that Q13 coverage ratio inside or outside this week's 4.00× minimum?
15. Using this week's actual numbers, what total debt would push Crunch Machine Co.'s leverage ratio to exactly 3.00×, holding EBITDA at $22,000,000?
16. What EBITDA would breach that same 3.00× covenant, holding total debt at this week's post-raise $50,000,000?
17. If the new-issue discount on this week's equity deal widened from 5% to 15%, how many *more* shares would need to be issued to net the same $20,000,000?
18. At what cost of equity would $20,000,013.70 of equity cost exactly $1,500,000/year — the same as the first year's interest on this week's actual $20,000,000 loan?
19. A company issues $5,000,000 of subordinated debt at 11% instead of senior debt at 7.5% — compute the after-tax cost of each at a 25% tax rate, and explain in one sentence why subordinated debt carries a higher coupon for the same borrower.
20. In one sentence: across today's drills, was debt cheaper than equity in every single comparison you computed, or were there exceptions — and if there were exceptions, what changed to cause them?

---

## Problem 2 — Explain dilution and covenants to a non-finance colleague (45 min)

In `explain-writeup.md`, write no more than 400 words, in plain language (imagine a colleague from product or engineering who's never seen a cap table or a loan covenant):

1. Using this week's actual numbers, explain why the Founder & CEO's ownership drops from 30% to about 27% after the raise, even though they didn't sell a single share.
2. Explain what a "covenant" is, using the leverage covenant specifically, without using the word "default."
3. If your colleague asked "so why doesn't the company just always use debt, since it's cheaper?" — answer in three sentences, using at least one idea from Lecture 3 beyond cost.
4. In one sentence, explain why a loan's "coupon rate" understates how expensive debt actually looks to a company's shareholders once covenants are factored in.

---

## Problem 3 — Query practice across both engines (45 min)

The same tasks, run on **both** Postgres and SQLite, in `cross-engine.sql`:

1. The full pre/post dilution table for all six shareholders, on both engines — confirm identical results.
2. A query using `CEIL()` to round a fractional share count up, run on both engines and confirm the same integer.
3. **Postgres only:** the leverage-ratio query from Lecture 2, Section 6.1, rewritten using a `CASE WHEN` to also return `'Breach'` or `'OK'` as a label based on the 3.00× threshold.
4. Note in a comment: does SQLite's `CEIL()` support work the same way as Postgres's, or did you need a workaround? *(Check the docs if unsure — don't guess.)*

---

## Problem 4 — Build a second term sheet (60 min)

Crunch Machine Co.'s banker comes back with an **alternative** debt term sheet: same $20,000,000, same 5-year term, but **8.25%** coupon (higher — the bank says it's because this version drops the origination fee to 0%, and is unsecured rather than secured, hence the higher rate for the lender's added risk).

1. Build the full amortization schedule for this alternative loan, store it in a new table `term_loan_schedule_alt`, and report total interest paid over 5 years.
2. Compute its after-tax total cost (interest + $0 fee) over 5 years, at the 25% tax rate.
3. Compare it to the original 7.5%-with-1%-fee loan's 5-year after-tax cost from Lecture 3. Which term sheet is actually cheaper once the fee is accounted for?
4. In 3–4 sentences, explain why an **unsecured** loan (Section 5 of Lecture 2) commanding a higher coupon than a **secured** one, for the same borrower, is exactly what the seniority/collateral discussion predicts — and whether the fee difference changes which one you'd actually recommend.

**Deliver** `alt-term-sheet.sql` and `alt-term-sheet-writeup.md`.

---

## Problem 5 — Model a financing decision you understand (60 min)

Pick a real financing decision you actually understand — a personal one is fine (a car loan vs. paying cash and giving up an investment return, a friend proposing to "invest" in a side project you're running in exchange for a cut) or a business one you've read about.

1. Frame it explicitly as an equity-vs-debt choice: what would "selling ownership" look like in your example, and what would "borrowing with a fixed repayment" look like?
2. Price both, with real or realistic numbers, as `INSERT` statements into new tables `my_financing_equity` and `my_financing_debt` (any reasonable shape, documented).
3. Compute the true cost of each path using this week's functions (or the formulas by hand if the example doesn't fit the exact function signatures).
4. In 3–4 sentences, say which you'd choose and why, using at least two of Lecture 3's four axes (cost, capacity, control, flexibility/signaling) — not cost alone.

**Deliver** `my-financing-decision.sql` and `my-financing-writeup.md`.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 90 min |
| 2 | 45 min |
| 3 | 45 min |
| 4 | 60 min |
| 5 | 60 min |
| **Total** | **~5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
