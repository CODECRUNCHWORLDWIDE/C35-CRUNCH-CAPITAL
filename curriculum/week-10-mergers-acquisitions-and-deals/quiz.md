# Week 10 — Quiz

15 questions. Mix of multiple choice and short calculations — no notes needed if you've done the lectures and exercises, but the seed data from this week's [README](./README.md) is fair game for reference on the calculation questions. Answer key is at the bottom; don't peek until you've written down an answer for every question.

1. **(MC)** The "unaffected price" of a target company is:
   - A. The price the target's stock trades at the day the deal is announced
   - B. The price the market assigned before any acquisition rumor or announcement could move it
   - C. The average of the acquirer's and target's share prices
   - D. The target's IPO price

2. **(MC)** A target's unaffected price is $12.00/share and the acquirer's offer is $15.60/share. The control premium is:
   - A. 23.1%
   - B. 30.0%
   - C. 3.6%
   - D. 76.9%

3. **(Short calc)** A target has 8,000,000 shares outstanding, an offer price of $22.00/share, $40,000,000 of debt, and $6,000,000 of cash. Compute the equity purchase price and the implied enterprise value.

4. **(MC)** Which of the following is closest to what a target's implied enterprise value represents?
   - A. The cash portion of the deal only
   - B. What the acquirer is paying for the whole operating business, including debt assumed and cash acquired
   - C. The target's market capitalization before the deal was announced
   - D. The acquirer's own enterprise value after the deal closes

5. **(MC)** A deal is described as "accretive to EPS." This means:
   - A. The deal is definitely a good use of the acquirer's capital
   - B. The combined company's pro forma EPS is lower than the acquirer's standalone EPS
   - C. The combined company's pro forma EPS is higher than the acquirer's standalone EPS
   - D. The target's shareholders received a premium over the unaffected price

6. **(Short calc)** An acquirer's standalone EPS is $1.20. After a deal, the pro forma combined EPS is $1.05. Compute the accretion/(dilution) percentage and state whether the deal is accretive or dilutive.

7. **(MC)** All else equal, a stock-financed deal is more likely to be accretive to the acquirer's EPS when:
   - A. The acquirer's P/E multiple is lower than the target's implied deal P/E
   - B. The acquirer's P/E multiple is higher than the target's implied deal P/E
   - C. The target has no debt
   - D. The acquirer's share price is falling

8. **(MC)** Why does this week's model apply a larger risk discount to revenue synergies than to cost synergies?
   - A. Revenue synergies are always smaller in dollar terms
   - B. Cost synergies are decisions management directly controls; revenue synergies depend on customer behavior outside the company's control
   - C. Tax rules require a different discount for each type
   - D. Revenue synergies are never included in a real deal model

9. **(Short calc)** A cost synergy has an after-tax run-rate of $4,000,000/year, fully phased in from Year 1 (no ramp), discounted at 8.0%. Compute its NPV as a flat perpetuity.

10. **(MC)** In an LBO's sources & uses table, "uses" typically includes:
    - A. The term loan and the sponsor's equity check
    - B. The entry enterprise value and transaction fees
    - C. Only the target's existing debt
    - D. The exit equity value

11. **(MC)** In a cash-sweep LBO debt schedule, each year's interest expense should be computed on:
    - A. The ending debt balance for that year
    - B. The average of the entry and exit debt balances
    - C. The beginning debt balance for that year
    - D. The original entry debt balance, held constant every year

12. **(Short calc)** An LBO's term loan begins the year at $60,000,000, at a rate of 8.0%. That year's free cash flow (before any paydown) is $9,500,000, and the cash sweep is 100%. Compute the interest expense and the ending debt balance.

13. **(MC)** The key difference between MOIC and IRR is:
    - A. MOIC accounts for the holding period; IRR does not
    - B. IRR accounts for the holding period; MOIC does not
    - C. They always produce the same ranking of deals
    - D. MOIC is used only for stock deals, IRR only for LBOs

14. **(Short answer)** An LBO's return can be decomposed into three levers. Name all three, and state which one is exactly $0 when a sponsor holds the entry and exit multiples constant.

15. **(MC)** If a sponsor's exit multiple comes in one full turn *below* the entry multiple (multiple compression), and nothing else about the business changes, the sponsor's MOIC will:
    - A. Rise
    - B. Fall
    - C. Stay exactly the same
    - D. Become negative automatically

---

## Answer key

<details>
<summary>Reveal answers</summary>

1. **B** — the unaffected price is measured before any deal speculation could move the market price; it's the honest baseline every premium is computed against.
2. **B** — `(15.60 − 12.00) / 12.00 = 30.0%`.
3. Equity purchase price = `22.00 × 8,000,000 = $176,000,000`. Net debt = `40,000,000 − 6,000,000 = 34,000,000`. Implied EV = `176,000,000 + 34,000,000 = $210,000,000`.
4. **B** — enterprise value captures the whole operating business, debt and equity together, which is why net debt has to be added to (or subtracted from) the equity purchase price to get there.
5. **C** — "accretive" is a purely mechanical statement about pro forma EPS vs. standalone EPS; it says nothing on its own about whether the deal is strategically wise.
6. `(1.05 − 1.20) / 1.20 = −12.5%` — the deal is **dilutive**.
7. **B** — buying a target at a lower implied multiple than the acquirer's own trading multiple, and paying with stock, is the classic "multiple arbitrage" pattern that produces accretion.
8. **B** — cost synergies are largely a management decision (close a redundant office); revenue synergies are a bet on customer behavior the company doesn't fully control, which is why they warrant a heavier discount.
9. `4,000,000 / 0.08 = $50,000,000`.
10. **B** — "uses" is what the deal costs: the entry enterprise value plus transaction fees. "Sources" (term loan + sponsor equity) is where that money comes from.
11. **C** — interest accrues on the balance outstanding during the year, which is the beginning-of-year balance; using the ending balance understates interest and compounds the error every subsequent year.
12. Interest = `60,000,000 × 0.08 = 4,800,000`. (Note: the $9,500,000 FCF figure given is *before* any paydown — in this problem it's independent of the interest calculation, which only needs the beginning balance and rate.) If the full $9,500,000 FCF is swept, ending debt = `60,000,000 − 9,500,000 = $50,500,000`.
13. **B** — IRR annualizes the return over the actual holding period; MOIC is a simple multiple that ignores time entirely, which is why the same MOIC can represent a great or a mediocre result depending on how long it took.
14. **EBITDA growth, multiple change, and debt paydown.** The **multiple change** lever is exactly $0 when entry and exit multiples are held equal — it only produces a nonzero contribution when the exit multiple differs from the entry multiple.
15. **B** — multiple compression directly reduces exit enterprise value (and therefore exit equity value) with the operating business unchanged, which lowers MOIC.

</details>
