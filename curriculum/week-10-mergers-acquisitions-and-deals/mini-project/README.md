# Mini-Project — Two buyers, one target

**Time:** ~2 hours (Saturday's block), building directly on all three exercises and both challenges.

## The brief

You are the analyst supporting **Solstice Precision Systems, Inc.'s (SPSY)** board of directors. Two buyers want the company:

1. **Crunch Machine Co. (CRNM)** — a strategic acquirer offering $19.50/share, financed 55% cash / 45% stock, promising cost and revenue synergies from combining operations.
2. **An unnamed private equity sponsor** — planning a leveraged buyout at a 10.0x EBITDA entry multiple, a $100,000,000 term loan, and a 5-year hold with a 100% cash sweep.

The board has asked you for one thing: **a single memo that prices both offers, models both outcomes, and recommends which one the board should pursue** — not because one is definitionally "better," but because they create fundamentally different outcomes for SPSY's current shareholders, and the board needs to understand exactly what each path means before voting.

## Requirements

Your deliverable is a `solutions.sql` file, a `solutions.py` file, and a markdown memo (`memo.md`) in this directory. Every number in the memo must trace back to a query or a script in those two files — no number should appear in the memo that you can't point to the exact line of code that produced it.

### Part 1 — Price CRNM's offer (uses Lecture 1 + Exercise 1, Parts A–B)

- Compute the control premium, the equity purchase price, the implied enterprise value, and the implied EV/EBITDA multiple.
- Compute the cash and stock consideration, new shares issued, and pro forma share count for the base-case 55/45 financing mix.

### Part 2 — Run CRNM's accretion/dilution, with and without synergies (uses Exercise 1 + Exercise 2)

- Build the pro forma combined income statement without synergies, and compute the accretion/(dilution) percentage.
- Value both synergy streams (cost and revenue) using the perpetuity method, and re-run the pro forma **with** the full run-rate synergies included.
- Report both accretion percentages (with and without synergies) and the total synergy NPV.

### Part 3 — Build the sponsor's LBO of SPSY (uses Exercise 3)

- Build the full sources & uses table.
- Build the 5-year cash-sweep debt-paydown schedule as a Python loop.
- Compute exit EV, exit equity value, MOIC, and IRR.

### Part 4 — The financing-mix sensitivity (uses Challenge 1)

- Report the accretion percentage and pro forma Debt/EBITDA at 100% cash, 100% stock, and the 55/45 base case, so the board can see the full range CRNM could plausibly offer.

### Part 5 — The memo

Write `memo.md`, addressed to SPSY's board, covering:

1. **What each buyer is offering, in plain terms** — headline price, form of consideration, and what SPSY's current shareholders end up holding in each case (cash in hand vs. CRNM stock vs. being fully cashed out by the sponsor).
2. **What each deal is worth to the *other* side** — is CRNM's deal accretive to its own shareholders (and by how much, with and without synergies)? What return is the sponsor underwriting, and is it strong enough to justify the price they're implicitly paying (their 10.0x entry multiple)?
3. **What's uncertain in each case** — CRNM's synergy estimate carries real execution risk (Lecture 2, Section 7); the sponsor's LBO return is exposed to multiple compression at exit (Challenge 2, Part D). Name the single biggest risk in each deal.
4. **Your recommendation** — which offer should SPSY's board pursue, or is there a case for running a competitive process between the two? Justify it with numbers from Parts 1–4, not just judgment.

## Deliverables checklist

- [ ] `solutions.sql` — all SQL used across Parts 1–4, one labeled section per part.
- [ ] `solutions.py` — all Python used across Parts 1–4, one labeled section per part (the LBO cash-sweep loop belongs here).
- [ ] `memo.md` — the board memo, following the four sections above.
- [ ] Every number in `memo.md` matches a number your code actually produces — no hand-typed figures.

## Expected headline results (yours should match within ±0.5%)

| Metric | Expected value |
|---|---:|
| CRNM implied EV | $207,000,000 |
| CRNM implied EV/EBITDA | 10.35x |
| Accretion, no synergies | ≈ +7.2% |
| Total synergy NPV | ≈ $27,172,580 |
| Accretion, with synergies | ≈ +16.5% |
| Sponsor entry equity | $104,000,000 |
| Sponsor exit equity value | ≈ $202,890,131 |
| Sponsor MOIC / IRR | ≈ 1.95x / ≈14.3% |

## Stretch goals

- **A blended "walk-away" price.** If SPSY's board wanted to run a competitive auction, what offer price would make the sponsor's IRR fall to a level roughly matching CRNM's accretion economics (i.e., the price at which both buyers are, loosely, paying an equivalent "fair" amount)? There's no single correct method here — propose one and defend it.
- **A collar.** Real stock deals often include a "collar" that adjusts the exchange ratio if the acquirer's stock price moves before closing, protecting the target from a falling acquirer stock price. Sketch (in the memo, no code required) how you'd structure one for CRNM's offer and what problem it solves for SPSY's shareholders.
- **A second sponsor bid at higher leverage.** Re-run Part 3 with `term_loan_multiple_ebitda = 6.0` instead of 5.0 (same mechanic as Challenge 2's stretch) and add a third row to your comparison table — does more leverage make the sponsor's implicit bid more or less competitive with CRNM's?
