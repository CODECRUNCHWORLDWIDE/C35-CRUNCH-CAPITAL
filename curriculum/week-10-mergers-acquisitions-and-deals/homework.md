# Week 10 — Homework

Fifteen practice problems across three sections, spread across the week (~5 hours total). These use a **fresh scenario** — not CRNM/SPSY — so you can't pattern-match the exercise answers; you have to apply the actual formulas from the lectures. Work with pen, a calculator, or a Python REPL — a full SQL seed isn't required for homework, though you're welcome to build one.

**Scenario:** **Vantage Holdings Inc. (VNTG)** is acquiring **Bramwell Systems Corp. (BRWL)**. A separate, unrelated sponsor deal — **Castleton Manufacturing Ltd.** — is used for the LBO section.

## Section A — Deal structure and price (Lecture 1)

1. Vantage offers **$50.00/share** for Bramwell, which has an unaffected price of **$38.00/share** and **5,000,000** shares outstanding. Compute the control premium and the total equity purchase price.
2. Bramwell carries **$30,000,000** of debt and **$5,000,000** of cash. Compute the implied enterprise value from Q1's equity purchase price.
3. Bramwell's LTM EBITDA is **$40,000,000**. Compute the implied EV/EBITDA multiple from Q2's implied EV.
4. The deal is financed **70% cash / 30% stock**. Vantage's own share price is **$25.00**. Compute the cash consideration, the stock consideration, and the number of new Vantage shares issued.
5. Vantage had **40,000,000** shares outstanding before the deal. Compute the pro forma combined share count and the percentage of the combined company that Bramwell's former shareholders now own.

## Section B — Accretion/dilution and synergies (Lecture 2)

6. Vantage's standalone EBIT is **$80,000,000**, its existing debt is **$200,000,000** at **5.0%**, and its tax rate is **25%**. Using its **40,000,000** shares, compute its standalone EPS.
7. Bramwell's standalone EBIT is **$35,000,000**, its existing debt is **$30,000,000** at **6.0%**, and its tax rate is **25%**. Compute Bramwell's standalone net income (you won't need Bramwell's own EPS for this section).
8. The cash leg of the deal (Q4's cash consideration) is funded with new acquisition debt at **7.0%**. Compute the new annual interest expense this creates.
9. Using Q1–Q8's results, build the full pro forma combined income statement (no synergies) and compute the pro forma EPS and the accretion/(dilution) percentage vs. Vantage's standalone EPS from Q6.
10. Vantage's deal team estimates **$8,000,000/year** of run-rate cost synergies, assumed to be **fully phased in from Year 1** (no ramp), a **10.0%** discount rate, and a **25%** tax rate. Compute the after-tax NPV of this synergy stream as a flat (unphased) perpetuity.
11. Now assume the same $8,000,000 run-rate instead phases in at **60%** in Year 1 and **100%** from Year 2 onward, same 10.0% discount rate and 25% tax rate. Compute the NPV using the two-piece method from Lecture 2, Section 8. How much smaller is this than Q10's answer, and why does that make sense?

## Section C — The LBO (Lecture 3)

12. A sponsor buys **Castleton Manufacturing Ltd.** at **8.0x** EBITDA, where Castleton's EBITDA is **$25,000,000**. Transaction fees are **1.5%** of entry EV. The term loan is sized at **4.5x** EBITDA, at a **9.0%** rate. Compute entry EV, fees, total uses, the term loan, and the sponsor's equity check.
13. In Year 1, Castleton's EBIT is **$18,000,000**, D&A is **$5,000,000**, capex is **$4,000,000**, and the increase in net working capital is **$1,000,000**. Tax rate is **25%**, and the cash sweep is **100%**. Compute Year 1 interest expense (on Q12's term loan), net income, free cash flow, and the ending debt balance.
14. The sponsor exits after this single year at the **same 8.0x entry multiple**, with Year 1 EBITDA of **$26,000,000**. Compute the exit EV, the exit equity value (using Q13's ending debt), and the resulting MOIC.
15. **Conceptual.** In 2–3 sentences: why can a sponsor's IRR and MOIC tell two different stories about the same deal when the holding period changes, even if the exit equity value in dollars stays exactly the same?

## Answers

<details>
<summary>Click to reveal — work the problems first</summary>

1. Premium = `(50 − 38) / 38 ≈ 31.6%`. Equity purchase price = `50 × 5,000,000 = $250,000,000`.
2. Net debt = `30,000,000 − 5,000,000 = $25,000,000`. Implied EV = `250,000,000 + 25,000,000 = $275,000,000`.
3. `275,000,000 / 40,000,000 ≈ 6.875x`.
4. Cash = `0.70 × 250,000,000 = $175,000,000`. Stock = `0.30 × 250,000,000 = $75,000,000`. New shares = `75,000,000 / 25.00 = 3,000,000`.
5. Pro forma shares = `40,000,000 + 3,000,000 = 43,000,000`. Bramwell ownership = `3,000,000 / 43,000,000 ≈ 7.0%`.
6. Interest = `200,000,000 × 0.05 = 10,000,000`. EBT = `80,000,000 − 10,000,000 = 70,000,000`. Tax = `17,500,000`. NI = `52,500,000`. EPS = `52,500,000 / 40,000,000 = $1.3125`.
7. Interest = `30,000,000 × 0.06 = 1,800,000`. EBT = `35,000,000 − 1,800,000 = 33,200,000`. Tax = `8,300,000`. NI = `$24,900,000`.
8. `175,000,000 × 0.07 = $12,250,000`.
9. Combined EBIT = `80,000,000 + 35,000,000 = 115,000,000`. Combined interest = `10,000,000 + 1,800,000 + 12,250,000 = 24,050,000`. EBT = `90,950,000`. Tax = `22,737,500`. NI = `68,212,500`. Pro forma EPS = `68,212,500 / 43,000,000 ≈ $1.5863`. Accretion = `(1.5863 − 1.3125) / 1.3125 ≈ +20.9%`.
10. After-tax run-rate = `8,000,000 × 0.75 = 6,000,000`. Flat perpetuity NPV = `6,000,000 / 0.10 = $60,000,000`.
11. Year 1 CF = `6,000,000 × 0.60 = 3,600,000`; PV = `3,600,000 / 1.10 ≈ 3,272,727`. Perpetuity from Year 2 (valued at end of Year 1) = `6,000,000 / 0.10 = 60,000,000`; PV = `60,000,000 / 1.10 ≈ 54,545,455`. Total ≈ `$57,818,182` — smaller than Q10 because the Year-1 ramp delays part of the cash flow by a year, and a delayed dollar is always worth less than an immediate one.
12. Entry EV = `8.0 × 25,000,000 = 200,000,000`. Fees = `0.015 × 200,000,000 = 3,000,000`. Total uses = `203,000,000`. Term loan = `4.5 × 25,000,000 = 112,500,000`. Sponsor equity = `203,000,000 − 112,500,000 = $90,500,000`.
13. Interest = `112,500,000 × 0.09 = 10,125,000`. EBT = `18,000,000 − 10,125,000 = 7,875,000`. Tax = `1,968,750`. NI = `5,906,250`. FCF = `5,906,250 + 5,000,000 − 4,000,000 − 1,000,000 = 5,906,250`. Ending debt = `112,500,000 − 5,906,250 = $106,593,750`.
14. Exit EV = `8.0 × 26,000,000 = 208,000,000`. Exit equity = `208,000,000 − 106,593,750 = 101,406,250`. MOIC = `101,406,250 / 90,500,000 ≈ 1.12x`.
15. MOIC ignores time entirely — it only compares the dollars out to the dollars in, regardless of how long the money was tied up. IRR annualizes that same dollar return over the actual holding period, so the identical MOIC produces a *higher* IRR over a shorter hold and a *lower* IRR over a longer one — a 1.5x MOIC in 2 years is a much stronger annualized result than the same 1.5x MOIC in 8 years.

</details>
