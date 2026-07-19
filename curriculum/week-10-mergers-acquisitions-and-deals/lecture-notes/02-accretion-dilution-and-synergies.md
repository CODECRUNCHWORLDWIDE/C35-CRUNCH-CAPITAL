# Lecture 2 — Accretion/dilution and synergies

> **Duration:** ~2 hours. **Outcome:** You can build a pro forma combined income statement, prove whether a deal is accretive or dilutive to the acquirer's EPS, and separately value cost and revenue synergies with the same discounting method Week 7 used for terminal value.

Every acquisition Lecture 1 priced now has to clear one more bar before a CFO signs it: **does it help or hurt the acquirer's own shareholders, measured in earnings per share?** This is the single most-quoted number in any public-company M&A announcement — "the deal is expected to be accretive to EPS within the first full year" — and it is entirely computable from the tables you already have.

## 1. Accretive vs. dilutive, defined precisely

- **Accretive**: the combined company's pro forma EPS is *higher* than the acquirer's standalone EPS. The deal adds more earnings per share than it costs in new shares and new interest.
- **Dilutive**: the combined company's pro forma EPS is *lower* than the acquirer's standalone EPS. The deal's cost (new shares, new interest) outweighs the earnings it brings in.

Neither word is a verdict on whether the deal is a *good* one — a dilutive deal can still be strategically correct (paying up for a capability you can't build yourself), and an accretive one can still be a bad idea (buying a shrinking, low-quality business cheap). Accretion/dilution measures one specific thing: **the deal's mechanical effect on next year's reported EPS**, given the price and financing structure Lecture 1 built. Treat it as a required check, not the whole verdict.

## 2. CRNM's standalone EPS — the baseline everything else is measured against

From `acquirer_financials`, build CRNM's own income statement first:

| Line | Calculation | Value |
|---|---|---:|
| EBITDA | given | 43,000,000 |
| D&A | given | 9,200,000 |
| EBIT | `43,000,000 − 9,200,000` | 33,800,000 |
| Interest expense | `110,000,000 × 6.0%` | 6,600,000 |
| EBT | `33,800,000 − 6,600,000` | 27,200,000 |
| Tax (25%) | `27,200,000 × 0.25` | 6,800,000 |
| Net income | `27,200,000 − 6,800,000` | 20,400,000 |
| **Standalone EPS** | `20,400,000 / 24,000,000` | **$0.85** |

$0.85 is the number every pro forma EPS in this lecture gets compared against. CRNM trades at $20.00/share, so its P/E is `20.00 / 0.85 ≈ 23.5x` — remember that multiple; Section 5 explains why it's the whole reason this deal works.

## 3. Building the pro forma combined income statement

The pro forma statement combines both companies' operations and layers on the new financing from Lecture 1. Four pieces get combined:

1. **Combined EBIT** = CRNM's EBIT + SPSY's EBIT (before any synergies — added separately in Section 6).
2. **Combined interest expense** = CRNM's existing interest + SPSY's existing interest (assumed to survive the deal, unchanged) + the *new* interest on the acquisition debt from Lecture 1's cash leg.
3. **Combined tax** = combined EBT × the (assumed shared) 25% tax rate.
4. **Pro forma share count** = CRNM's existing shares + the new shares issued in the stock leg.

SPSY's own standalone income statement, for reference (same method as Section 2):

| Line | Calculation | Value |
|---|---|---:|
| EBIT | `20,000,000 − 4,000,000` | 16,000,000 |
| Interest expense | `20,000,000 × 6.0%` | 1,200,000 |
| EBT | `16,000,000 − 1,200,000` | 14,800,000 |
| Tax (25%) | `14,800,000 × 0.25` | 3,700,000 |
| Net income | | 11,100,000 |

Now assemble the pro forma, **no synergies yet**:

| Line | Calculation | Value |
|---|---|---:|
| Combined EBIT | `33,800,000 + 16,000,000` | 49,800,000 |
| Combined interest | `6,600,000 + 1,200,000 + 7,507,500` | 15,307,500 |
| Combined EBT | `49,800,000 − 15,307,500` | 34,492,500 |
| Tax (25%) | `34,492,500 × 0.25` | 8,623,125 |
| **Combined net income** | | **25,869,375** |
| Pro forma shares | `24,000,000 + 4,387,500` | 28,387,500 |
| **Pro forma EPS** | `25,869,375 / 28,387,500` | **$0.9113** |

## 4. The accretion/dilution verdict

```
Accretion/(dilution) % = (Pro forma EPS − Standalone EPS) / Standalone EPS
                        = (0.9113 − 0.85) / 0.85
                        ≈ +7.2%
```

Even before a single dollar of synergy, the deal is **accretive by about 7.2%** — CRNM's own shareholders end next year with more earnings per share than if the deal never happened. This is not a coincidence; it follows directly from a pattern explained in Section 5.

## 5. Why this deal is accretive before synergies — multiple arbitrage

CRNM trades at a P/E of ~23.5x. SPSY, at the $19.50 offer price, has an implied P/E of only `19.50 / (11,100,000/10,000,000) = 19.50/1.11 ≈ 17.6x`. **CRNM is buying earnings at a lower multiple than the market pays for its own earnings.** This gap — buying "cheap" earnings with "expensive" currency — is the single most common reason a strategic stock-for-stock or cash deal turns out accretive, and it works through two separate channels:

- **The stock leg** effectively trades CRNM shares (worth 23.5x their own earnings) for SPSY earnings (bought at 17.6x) — CRNM gives up less earnings-equivalent value than it receives.
- **The cash leg** works whenever SPSY's earnings yield (`E/P = 11,100,000/195,000,000 ≈ 5.7%`) exceeds the after-tax cost of the new debt (`7.0% × (1 − 25%) = 5.25%`) — here it does, by about 45 basis points, so even the debt-funded portion adds a sliver of accretion.

This is the mechanical explanation every banker gives for "why is this deal accretive" — and it is exactly why Challenge 1 has you test what happens when you change *only* the financing mix and watch the accretion percentage move.

## 6. Adding synergies — cost synergies

Lecture 1's price assumed two companies operating exactly as they do today. Real deals also promise **synergies** — savings or extra revenue that only exist *because* the two companies combined. The `synergies` table holds two rows: a **cost synergy** (overlapping SG&A, procurement, back-office) and a **revenue synergy** (cross-selling each company's products into the other's customer base).

**Cost synergies** are the more reliable kind — eliminating a duplicate finance department is a decision management directly controls. `synergies` gives a $3,000,000 annual run-rate, phased in 50% in Year 1 and 100% from Year 2 onward (most integration savings take a year to fully execute — severance, system migrations, and lease terminations don't happen on day one).

Adding the full run-rate ($3,000,000, i.e., the Year 2+ steady state) to the pro forma from Section 3:

| Line | Calculation | Value |
|---|---|---:|
| Combined EBIT (Section 3) | | 49,800,000 |
| + Cost synergies (full run-rate) | | 3,000,000 |
| Combined EBIT with synergies | | 52,800,000 |
| Combined interest (unchanged) | | 15,307,500 |
| Combined EBT | `52,800,000 − 15,307,500` | 37,492,500 |
| Tax (25%) | | 9,373,125 |
| **Combined net income** | | **28,119,375** |
| **Pro forma EPS with synergies** | `28,119,375 / 28,387,500` | **$0.9906** |

```
Accretion/(dilution) with synergies = (0.9906 − 0.85) / 0.85 ≈ +16.5%
```

Synergies alone move the deal from **+7.2%** accretive to **+16.5%** accretive — cost synergies contributed roughly 9.3 points of that improvement, more than the underlying multiple-arbitrage effect from Section 5. This is exactly why bankers and boards obsess over synergy estimates: in this deal, synergies are worth more to EPS than the "free" multiple gap.

## 7. Revenue synergies deserve a heavier discount

`synergies` also carries a **revenue synergy**: $1,000,000 of run-rate EBITDA (from $5,000,000 of incremental cross-sell revenue at a 20% margin), phased in over three years, but with a **50% risk discount** applied before anything else. Why the haircut, when cost synergies got none? Because cost synergies are a decision — management chooses to close an office. Revenue synergies are a *forecast* — a bet that customers will actually buy more because two companies merged, which depends on people outside the company's control. Real-world M&A post-mortems consistently find revenue synergies realized at a fraction of the number in the announcement deck; a 50% haircut here is a deliberately conservative, defensible starting point, not an arbitrary penalty.

Applying the risk discount before valuing: `1,000,000 × (1 − 0.50) = $500,000` risk-adjusted run-rate. This week's exercises value this properly with full phase-in and discounting (Exercise 2); the point to internalize here is the *order of operations*: **discount for risk first, phase in second, then value** — never value the optimistic number and apply a haircut to the answer, which hides how much of your valuation is standing on an unproven assumption.

## 8. Valuing a synergy stream — the perpetuity method from Week 7

A synergy, once realized, typically persists indefinitely (the SG&A savings don't reverse; the cross-sell relationship, once established, keeps generating revenue). That makes it a **growing (or flat) perpetuity** — exactly the terminal-value math from Week 7, just applied to a smaller, more specific cash flow stream instead of an entire company.

For the cost synergy, after-tax and phased:

```
After-tax run-rate = 3,000,000 × (1 − 0.25) = $2,250,000
Year 1 (50% phased) = 2,250,000 × 0.50 = $1,125,000
Year 2+ (100% phased) = $2,250,000 forever, starting Year 2
```

Discount each piece at CRNM's WACC (9.2%, unchanged from Weeks 4 and 7):

```
PV(Year 1)              = 1,125,000 / 1.092                  ≈ $1,030,220
Perpetuity value at end of Year 1 (i.e., "as of the start of Year 2")
                         = 2,250,000 / 0.092                 ≈ $24,456,522
PV of that perpetuity    = 24,456,522 / 1.092                ≈ $22,399,745
NPV of cost synergies    = 1,030,220 + 22,399,745             ≈ $23,429,965
```

Roughly **$23.4M** — about 12% of the $195M equity purchase price — is sitting in cost synergies alone. Exercise 2 has you run the same method on the risk-adjusted revenue synergy (which comes out much smaller, given the haircut and slower phase-in) and combine both into a single synergy NPV you can weigh against the control premium from Lecture 1.

## 9. Computing the pro forma in SQL

```sql
WITH pf AS (
    SELECT
        (a.ebitda - a.da) + (t.ebitda - t.da)                         AS combined_ebit,
        (a.existing_debt * a.existing_debt_rate)
          + (t.existing_debt * t.existing_debt_rate)
          + (d.cash_financing_pct * d.offer_price_per_share
             * t.shares_outstanding * d.new_acquisition_debt_rate)     AS combined_interest,
        a.shares_outstanding
          + ROUND(d.stock_financing_pct * d.offer_price_per_share
             * t.shares_outstanding / a.share_price, 0)                AS pro_forma_shares,
        a.tax_rate,
        (a.ebitda - a.da - (a.existing_debt * a.existing_debt_rate))
          * (1 - a.tax_rate) / a.shares_outstanding                    AS standalone_eps
    FROM acquirer_financials a
    JOIN target_financials t ON t.company = 'Solstice Precision Systems, Inc.'
    JOIN deal_terms d        ON d.acquirer = a.company AND d.target = t.company
)
SELECT
    combined_ebit,
    combined_interest,
    (combined_ebit - combined_interest) * (1 - tax_rate)               AS combined_net_income,
    pro_forma_shares,
    ROUND((combined_ebit - combined_interest) * (1 - tax_rate)
          / pro_forma_shares, 4)                                        AS pro_forma_eps,
    ROUND(standalone_eps, 4)                                            AS standalone_eps,
    ROUND((((combined_ebit - combined_interest) * (1 - tax_rate)
          / pro_forma_shares) - standalone_eps) / standalone_eps, 4)    AS accretion_dilution_pct
FROM pf;
```

## 10. Common mistakes

- **Comparing pro forma net income to standalone net income instead of EPS.** Combined net income is almost always higher (it's two companies' earnings added together) — that tells you nothing. The entire point of accretion/dilution is *per-share* — dividing by the new, larger share count is what can turn a bigger combined profit into a smaller number per share.
- **Adding synergies to net income after tax instead of to EBIT before tax.** Synergies are operating savings or revenue — they flow through the tax line like every other dollar of EBIT. Adding them post-tax overstates their EPS benefit by ignoring the tax the company would owe on them.
- **Using the full, un-phased synergy run-rate in Year 1.** Real integrations take time; using the steady-state number immediately overstates near-term accretion and sets up a credibility gap when Year 1 results miss the promise.
- **Applying the same risk discount (or none) to both synergy types.** Cost and revenue synergies carry genuinely different execution risk — treating them identically hides exactly the distinction a skeptical board member will ask about first.

## 11. Check yourself

- In your own words, what does "accretive" mean, and why is it not the same thing as "a good deal"?
- Walk through, without looking back, why buying a company at a lower P/E than your own can make a stock-funded deal accretive even with zero synergies.
- Why does the cost synergy get a 0% risk discount while the revenue synergy gets 50%?
- If CRNM's own P/E were *lower* than SPSY's implied deal P/E instead of higher, what would you expect to happen to the accretion/dilution result, and why?

Lecture 3 leaves CRNM's strategic math behind entirely and rebuilds SPSY as an LBO target instead — same company, a completely different buyer, and a completely different question: not "does this help our EPS," but "can debt and growth alone make this investment pay off."

## Further reading

- **Investopedia — "Accretion/Dilution Analysis":** <https://www.investopedia.com/terms/a/accretiondilution.asp>
- **Investopedia — "Synergy" (in M&A):** <https://www.investopedia.com/terms/s/synergy.asp>
- **Damodaran (NYU Stern) — "Valuing Synergy":** <https://pages.stern.nyu.edu/~adamodar/pdfiles/papers/synergy.pdf>
- **PostgreSQL — Common Table Expressions:** <https://www.postgresql.org/docs/current/queries-with.html>
