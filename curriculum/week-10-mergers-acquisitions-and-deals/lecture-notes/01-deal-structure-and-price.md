# Lecture 1 — Deal structure and price

> **Duration:** ~2 hours. **Outcome:** You can turn an offer price per share into a full purchase price, a control premium over the unaffected market price, an implied enterprise value and EV/EBITDA multiple, and a split between cash and newly issued stock — and explain what each number means to the people signing the deal.

Every acquisition, no matter how large or small, answers the same three questions in the same order: **what are we paying, what does that price imply about the whole business, and how are we paying for it?** This lecture builds all three for **Crunch Machine Co.'s (CRNM)** acquisition of **Solstice Precision Systems, Inc. (SPSY)**, using the `acquirer_financials`, `target_financials`, and `deal_terms` tables from this week's [README](../README.md).

## 1. The unaffected price, and why it's the anchor for everything

Before any deal talk, SPSY trades at **$15.00/share** — its `share_price` in `target_financials`. This is called the **unaffected price**: the price the market assigned before any acquisition rumor or announcement could move it. Every premium, every "how generous is this offer" conversation, is measured against this one number, not against whatever the stock happens to be trading at the day the deal is announced (which, once rumors leak, is already inflated by speculation).

With 10,000,000 shares outstanding, SPSY's unaffected **equity value** (its market cap) is:

```
Equity value = share_price × shares_outstanding
             = 15.00 × 10,000,000
             = $150,000,000
```

That is what the market thinks SPSY's equity is worth with no deal on the table. Everything from here is about how much *more* than that CRNM has to offer to actually get the deal done — and why.

## 2. The control premium

Acquirers virtually never pay the unaffected price. To convince a board and a majority of shareholders to give up independent control of their company, an acquirer has to offer a **control premium** — extra value that compensates target shareholders for the synergies, strategic value, and loss of optionality the deal represents. Real-world control premiums in strategic (non-distressed) deals typically run **20%–40%** over the unaffected price.

CRNM's offer, from `deal_terms`, is **$19.50/share**. The premium:

```
Control premium = (offer_price − unaffected_price) / unaffected_price
                 = (19.50 − 15.00) / 15.00
                 = 4.50 / 15.00
                 = 30.0%
```

A clean, textbook-typical premium. Two things worth internalizing about this number:

- **It is not free money.** CRNM is explicitly telling its own shareholders "I believe this business plus what we can do together is worth at least 30% more than the market currently prices SPSY standalone." Lecture 2 is where that claim gets tested against a number.
- **It is a negotiating outcome, not a formula output.** Bankers build DCFs and comps ranges (exactly like Week 7) to find a *defensible band*, but the actual number in `deal_terms` is what the target's board agreed to accept — informed by that analysis, not dictated by it.

## 3. Total equity purchase price

The offer price applies to every outstanding share, so:

```
Equity purchase price = offer_price_per_share × target_shares_outstanding
                       = 19.50 × 10,000,000
                       = $195,000,000
```

This is the check CRNM writes (in cash, stock, or both) to SPSY's shareholders in exchange for their shares. It is **not** the enterprise value of the deal — it says nothing yet about SPSY's debt and cash, which is exactly what Section 4 fixes.

## 4. From equity purchase price to implied enterprise value

Recall from Week 7: **Enterprise value = Equity value + Net debt**, where `Net debt = total debt − cash`. The same bridge runs in reverse here. When CRNM buys SPSY's equity, it also effectively takes on SPSY's debt and picks up SPSY's cash — so the *implied* price CRNM is paying for the whole business (debt and equity together) is:

```
Target net debt = existing_debt − cash
                = 20,000,000 − 8,000,000
                = $12,000,000

Implied enterprise value = Equity purchase price + Target net debt
                          = 195,000,000 + 12,000,000
                          = $207,000,000
```

CRNM is implicitly paying **$207,000,000** for SPSY's whole operating business — $12M more than the headline $195M equity check, because it's also absorbing SPSY's net debt.

## 5. Implied multiples — sanity-checking the price against Week 7's tools

An implied enterprise value only means something in context. Convert it to multiples of SPSY's own financials, exactly like Week 7's trading comps:

```
Implied EV/EBITDA = 207,000,000 / 20,000,000  = 10.35×
Implied EV/Revenue = 207,000,000 / 100,000,000 =  2.07×
```

A real deal team would immediately hold these against a comps set and a set of precedent transactions — exactly the tables Week 7 built — to answer "is 10.35x EBITDA a fair, a cheap, or an expensive price for this kind of business?" This week doesn't rebuild that comps table from scratch, but the muscle memory is the same: **a price is only meaningful relative to what similar businesses have sold for**, and precedent-transaction multiples (Week 7, `precedent_transactions`) typically run *higher* than trading multiples precisely because they embed a control premium like the one you just computed.

## 6. Cash vs. stock — the two currencies of an acquisition

CRNM can pay the $195,000,000 purchase price in two fundamentally different currencies, and `deal_terms` specifies a blend of both:

- **Cash** — CRNM raises new debt (or draws down existing cash) and pays SPSY's shareholders in dollars. They receive a fixed, certain amount and walk away with no further stake in the combined company.
- **Stock** — CRNM issues brand-new shares of its own stock and gives them to SPSY's shareholders instead of cash. They become CRNM shareholders, with a claim that rises and falls with CRNM's stock price going forward — and CRNM's own share count grows to accommodate them.

`deal_terms` specifies **55% cash / 45% stock**:

```
Cash consideration  = 0.55 × 195,000,000 = $107,250,000
Stock consideration = 0.45 × 195,000,000 = $ 87,750,000
```

## 7. Sizing the cash and stock legs

**The cash leg** is funded with new acquisition debt at the rate in `deal_terms` (7.0%):

```
New acquisition debt          = $107,250,000
New acquisition debt interest = 107,250,000 × 0.070 = $7,507,500 / year
```

That $7,507,500 is a new, permanent expense line on the combined company's income statement — it is exactly what Section 6 of Lecture 2 has to account for.

**The stock leg** is paid in newly issued CRNM shares, priced at CRNM's own unaffected share price ($20.00 from `acquirer_financials`):

```
New CRNM shares issued = Stock consideration / CRNM share price
                        = 87,750,000 / 20.00
                        = 4,387,500 new shares
```

Those 4,387,500 shares did not exist before the deal. They dilute every existing CRNM shareholder's ownership percentage — and they are the second half of what Lecture 2's accretion/dilution math has to work through.

## 8. Pro forma share count and ownership split

Add the new shares to CRNM's existing count to get the **pro forma (combined) share count**:

```
Pro forma shares = CRNM existing shares + New shares issued
                  = 24,000,000 + 4,387,500
                  = 28,387,500
```

Former SPSY shareholders now own:

```
SPSY holders' ownership % = New shares issued / Pro forma shares
                           = 4,387,500 / 28,387,500
                           ≈ 15.5%
```

That 15.5% figure matters beyond bookkeeping — it's how much of the *combined* company's future upside (and risk) SPSY's former owners now hold, purely because part of the deal was paid in stock instead of cash. A board deciding the cash/stock mix is implicitly deciding how much of the company's future they're willing to hand to the people they're buying out today.

## 9. Computing this in SQL

```sql
SELECT
    d.offer_price_per_share,
    t.share_price                                       AS unaffected_price,
    ROUND((d.offer_price_per_share - t.share_price)
           / t.share_price, 4)                            AS control_premium,
    d.offer_price_per_share * t.shares_outstanding         AS equity_purchase_price,
    (t.existing_debt - t.cash)                             AS target_net_debt,
    d.offer_price_per_share * t.shares_outstanding
        + (t.existing_debt - t.cash)                       AS implied_ev,
    ROUND((d.offer_price_per_share * t.shares_outstanding
        + (t.existing_debt - t.cash)) / t.ebitda, 2)        AS implied_ev_ebitda,
    d.cash_financing_pct * d.offer_price_per_share
        * t.shares_outstanding                              AS cash_consideration,
    d.stock_financing_pct * d.offer_price_per_share
        * t.shares_outstanding                              AS stock_consideration,
    ROUND(d.stock_financing_pct * d.offer_price_per_share
        * t.shares_outstanding / a.share_price, 0)          AS new_shares_issued
FROM deal_terms d
JOIN target_financials t   ON t.company = d.target
JOIN acquirer_financials a ON a.company = d.acquirer;
```

## 10. Computing this in Python

```python
import pandas as pd

deal = pd.read_sql("SELECT * FROM deal_terms;", engine).iloc[0]
target = pd.read_sql("SELECT * FROM target_financials;", engine).iloc[0]
acquirer = pd.read_sql("SELECT * FROM acquirer_financials;", engine).iloc[0]

control_premium = (deal.offer_price_per_share - target.share_price) / target.share_price
equity_purchase_price = deal.offer_price_per_share * target.shares_outstanding
target_net_debt = target.existing_debt - target.cash
implied_ev = equity_purchase_price + target_net_debt
implied_ev_ebitda = implied_ev / target.ebitda

cash_consideration = deal.cash_financing_pct * equity_purchase_price
stock_consideration = deal.stock_financing_pct * equity_purchase_price
new_shares_issued = stock_consideration / acquirer.share_price
pro_forma_shares = acquirer.shares_outstanding + new_shares_issued

print(f"Control premium:      {control_premium:.1%}")
print(f"Equity purchase price: ${equity_purchase_price:,.0f}")
print(f"Implied EV:             ${implied_ev:,.0f}")
print(f"Implied EV/EBITDA:      {implied_ev_ebitda:.2f}x")
print(f"New shares issued:      {new_shares_issued:,.0f}")
print(f"Pro forma shares:       {pro_forma_shares:,.0f}")
```

Both should print: control premium 30.0%, equity purchase price $195,000,000, implied EV $207,000,000, implied EV/EBITDA 10.35x, new shares issued 4,387,500, pro forma shares 28,387,500.

## 11. Common mistakes

- **Quoting the equity purchase price as if it were the enterprise value.** They differ by exactly the target's net debt — forgetting this step understates what the acquirer is really paying for the operating business by millions of dollars.
- **Computing the control premium against the deal-announcement price instead of the unaffected price.** Once a deal rumor leaks, the market price already embeds speculation — using it as the base understates the true premium being paid.
- **Sizing new shares off the acquirer's post-announcement price instead of its unaffected price.** The number of shares issued is fixed at signing based on an agreed price (or a fixed exchange ratio) — it does not float with the stock price after the announcement moves it.
- **Forgetting that the cash leg isn't free.** New acquisition debt carries a real interest rate that becomes a permanent cost on the combined income statement — that cost is exactly what determines whether the deal turns out to be accretive or dilutive.

## 12. Check yourself

- SPSY's unaffected price is $15.00 and CRNM's offer is $19.50. Compute the control premium without looking back at Section 2.
- Why does the implied enterprise value differ from the $195,000,000 equity purchase price, and in which direction (higher or lower)?
- If the deal were 100% stock instead of 55/45, how many new CRNM shares would be issued? (Use Section 7's method with `stock_financing_pct = 1.00`.)
- What two things does a board give up, and what does it gain, by paying more of a deal in stock instead of cash?

Lecture 2 takes this exact price and structure and asks the question the board actually cares about: does this deal make CRNM's own earnings per share go up or down?

## Further reading

- **Investopedia — "Control Premium":** <https://www.investopedia.com/terms/c/control-premium.asp>
- **Investopedia — "Cash and Stock Merger":** <https://www.investopedia.com/terms/c/cashandstockmerger.asp>
- **Damodaran (NYU Stern) — "Acquisitions and Takeovers":** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/invemgmt/acqhist.htm>
- **PostgreSQL — Numeric types (for exact-precision deal math):** <https://www.postgresql.org/docs/current/datatype-numeric.html>
