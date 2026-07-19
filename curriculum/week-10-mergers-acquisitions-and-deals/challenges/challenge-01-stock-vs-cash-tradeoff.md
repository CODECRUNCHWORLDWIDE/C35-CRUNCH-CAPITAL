# Challenge 1 — Choose stock vs. cash

**Time:** ~90 minutes. **Builds on:** Exercise 1 and Lecture 2, Section 5 (why the deal is accretive at all).

## The setup

CRNM's board has approved the **$19.50/share** offer price and the **$195,000,000** total purchase price from Lecture 1 — that part isn't in play. What's still open is *how to pay for it*. The board has asked you to model three financing structures for the identical purchase price and present the tradeoffs, not just a single recommended answer:

1. **100% cash** — the entire $195,000,000 funded with new acquisition debt at 7.0%.
2. **100% stock** — the entire $195,000,000 paid in newly issued CRNM shares at $20.00.
3. **The base case, 55% cash / 45% stock** — from Exercise 1, for comparison.

## Part A — Rebuild the pro forma for each structure

For each of the three structures, rebuild Exercise 1's pro forma combined income statement (no synergies — isolate the financing effect on its own) and compute:

- Combined interest expense.
- Pro forma share count.
- Pro forma EPS.
- Accretion/(dilution) % vs. CRNM's $0.85 standalone EPS.

Do this for all three before reading further — guessing the ranking first, then checking it, is the point of this challenge.

## Part B — Pro forma leverage

For each structure, compute the **combined pro forma debt** (CRNM's existing $110,000,000 + SPSY's existing $20,000,000 + any new acquisition debt) and divide by **combined EBITDA** (`$43,000,000 + $20,000,000 = $63,000,000`, no synergies) to get pro forma Debt/EBITDA.

## Part C — Ownership dilution

For each structure, compute what percentage of the pro forma combined company former SPSY shareholders would own (Lecture 1, Section 8's method). For the 100% cash structure, this should be an easy one-line answer — think about why before you compute anything.

## Part D — The memo

Write a short recommendation (a paragraph or two, in a markdown file or code comments) that a CFO could actually read, covering:

1. **Which structure is most accretive to EPS, and why** — connect it back to Lecture 2, Section 5's multiple-arbitrage explanation. Does more stock or more cash produce more accretion here, and does that match or contradict the common assumption that "cash deals are always more accretive"?
2. **Which structure carries the most leverage risk**, and whether that risk level looks prudent for a company CRNM's size, given the leverage benchmarks discussed in Week 5.
3. **A specific recommended mix** — it doesn't have to be one of the three you modeled; you can propose a different split and justify it with the same math.

## Expected results

| Structure | Pro forma EPS (no synergy) | Accretion | Pro forma Debt/EBITDA | SPSY holders' ownership |
|---|---:|---:|---:|---:|
| 100% cash | ≈ $0.8859 | ≈ +4.2% | ≈ 5.16x | 0% |
| 55% cash / 45% stock (base case) | ≈ $0.9113 | ≈ +7.2% | ≈ 3.77x | ≈ 15.5% |
| 100% stock | ≈ $0.9333 | ≈ +9.8% | ≈ 2.06x | ≈ 28.9% |

## The twist worth noticing

If you were told "cash deals are always more accretive than stock deals," this challenge's numbers directly contradict that folk wisdom — **here, the all-stock structure is the *most* accretive of the three**, precisely because CRNM's earnings multiple (~23.5x) is so much richer than the effective cost of issuing new debt. The folk wisdom is only true when the acquirer's own P/E is *lower* than the after-tax cost of debt implies — it is a case-by-case result of the specific multiples involved, never a rule you can apply without computing it. That's the real lesson this challenge is testing: **don't reason about deal financing from a rule of thumb when you have the tables in front of you to just compute the answer.**

## Stretch

CRNM's board is (understandably) uncomfortable with a pro forma Debt/EBITDA above 4.0x for a strategic acquirer without an investment-grade balance sheet. Find the cash/stock split that gets pro forma leverage as close to — but not above — 4.0x as possible, and report the resulting accretion percentage at that split.
