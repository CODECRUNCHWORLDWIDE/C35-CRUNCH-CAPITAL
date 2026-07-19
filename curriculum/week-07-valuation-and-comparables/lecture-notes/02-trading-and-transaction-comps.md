# Lecture 2 — Trading and Transaction Comps

> **Duration:** ~2 hours. **Outcome:** You can build a trading-comparables set and a precedent-transactions set from raw market data, compute EV multiples correctly, summarize them defensibly, apply them to a target company, and explain specifically where a comps analysis is more trustworthy than a DCF — and where it isn't.

Lecture 1 built Crunch Machine Co.'s value from the inside out — forecast its own cash flows, discount them at its own cost of capital. This lecture builds a second, independent answer from the outside in: **what is the market already paying for businesses like this one?** If five public companies that look a lot like Crunch Machine Co. all trade at roughly 8–9 times EBITDA, that's real evidence about what CRNM might be worth — evidence that doesn't depend on anyone's 5-year revenue forecast being right.

## 1. Why enterprise-value multiples, not price-to-earnings

A "multiple" is just a ratio: some measure of value, divided by some measure of the underlying business's size or earning power. The most common and most useful multiples in company valuation are **EV/Revenue** and **EV/EBITDA** — enterprise value divided by a business metric, not P/E (price per share ÷ earnings per share).

Why EV-based, not equity-based (like P/E)? Because **enterprise value is capital-structure-neutral** — it measures the value of the whole business, regardless of how much of that business is financed with debt versus equity. P/E, by contrast, is distorted by leverage: two operationally identical companies with different amounts of debt will show *different* P/E ratios even though their businesses are worth exactly the same, because interest expense (a financing choice, not an operating result) flows through to the earnings-per-share denominator. EV/EBITDA sidesteps this entirely — EBITDA sits *above* interest expense on the income statement, so it's unaffected by financing choices, and EV is already capital-structure-neutral by construction (Lecture 1, Section 8). That's why comps work, almost always, is done in EV multiples, and why this week's `trading_comps` table gives you the raw pieces to compute EV yourself rather than a pre-built P/E column.

```
Market Capitalization = Share Price × Shares Outstanding
Enterprise Value (EV)  = Market Capitalization + Total Debt − Cash & Equivalents
```

Read the EV formula in plain English: **EV is what it would cost to buy the entire business outright** — you'd have to buy every share (market cap) *and* pay off or assume all of its debt, but you'd immediately get the cash on its balance sheet back, which nets against the price. That's exactly the same "whole business, before any financing structure" concept the DCF's enterprise value represents in Lecture 1 — which is precisely why the two are comparable to each other at all.

## 2. Selecting a comp set — the criteria that matter

A comp set is only as good as its selection discipline. A defensible trading-comps set is chosen on:

- **Industry and end market.** Same or closely adjacent — industrial equipment manufacturers serving similar customers, not "any company with similar revenue."
- **Size.** Similar revenue and EBITDA scale. A $2B revenue conglomerate and a $200M niche manufacturer rarely trade at comparable multiples, even in the same industry — scale changes growth expectations, financing access, and buyer pools.
- **Growth and margin profile.** Comps growing at wildly different rates, or with structurally different margins, will carry different multiples for good reason — a 6%-growth, 20%-margin business and a 25%-growth, 35%-margin business are not really "comparable" just because both make industrial equipment.
- **Capital intensity and business model.** A company that leases its equipment out versus one that sells it outright has a fundamentally different cash-conversion profile, even selling the same physical product.

This week's `trading_comps` table has already been screened to six companies that plausibly clear these bars against Crunch Machine Co. — similar industrial-equipment end markets, revenue in the $165M–$410M range (CRNM's LTM revenue is $210M), and EBITDA margins clustered in the high-teens to low-20s%. Real comp selection is a judgment call an analyst defends in a footnote; this week, treat the six as pre-screened and focus on the mechanics of what to do with them (Challenge 2 asks you to argue whether any of the six actually belong).

## 3. Computing multiples in SQL

Every multiple starts with EV, computed from the raw balance-sheet and market fields in `trading_comps`:

```sql
SELECT
    ticker,
    company,
    revenue_ntm,
    ebitda_ntm,
    ROUND(share_price * shares_outstanding, 2)                                    AS market_cap,
    ROUND(share_price * shares_outstanding + total_debt - cash_and_equivalents, 2) AS enterprise_value,
    ROUND((share_price * shares_outstanding + total_debt - cash_and_equivalents)
          / revenue_ntm, 2)                                                       AS ev_revenue,
    ROUND((share_price * shares_outstanding + total_debt - cash_and_equivalents)
          / ebitda_ntm, 2)                                                        AS ev_ebitda
FROM trading_comps
ORDER BY ev_ebitda DESC;
```

Running this over the seed data gives:

| Ticker | Revenue (NTM) | EBITDA (NTM) | EV | EV/Revenue | EV/EBITDA |
|---|---:|---:|---:|---:|---:|
| VDT | 165,000,000 | 39,600,000 | ≈419,800,000 | 2.54× | 10.60× |
| TFI | 285,000,000 | 62,700,000 | ≈589,500,000 | 2.07× | 9.40× |
| FPG | 320,000,000 | 68,800,000 | ≈612,400,000 | 1.91× | 8.90× |
| MMS | 198,000,000 | 40,590,000 | ≈328,800,000 | 1.66× | 8.10× |
| AFC | 410,000,000 | 79,950,000 | ≈607,600,000 | 1.48× | 7.60× |
| IMW | 245,000,000 | 46,550,000 | ≈335,100,000 | 1.37× | 7.20× |

**NTM** means "next twelve months" — a forward-looking consensus estimate, not trailing actuals. Comps are conventionally applied on forward figures when they're available and reliable, because the market is pricing the business on what it will earn *next*, not what it already earned. (Precedent transactions, in Section 5, use LTM instead — deal announcements are anchored to the target's most recent actual financials, not a forecast the buyer and seller may not agree on.)

## 4. Summarizing a multiple set — median over mean, and why

Six data points, two useful summary statistics, and a judgment call about which one to trust more:

```sql
SELECT
    ROUND(AVG(ev_ebitda_calc), 2)                                             AS mean_ev_ebitda,
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY ev_ebitda_calc), 2)     AS median_ev_ebitda,
    ROUND(MIN(ev_ebitda_calc), 2)                                             AS min_ev_ebitda,
    ROUND(MAX(ev_ebitda_calc), 2)                                             AS max_ev_ebitda
FROM (
    SELECT (share_price * shares_outstanding + total_debt - cash_and_equivalents)
           / ebitda_ntm AS ev_ebitda_calc
    FROM trading_comps
) t;
```

`PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY ...)` is standard SQL for a **median** and works in PostgreSQL directly. SQLite has no built-in median function — compute it in pandas instead (`df["ev_ebitda"].median()`), or fall back to `AVG` of the two middle values in a manually ordered subquery. On this six-company set: **mean EV/EBITDA ≈ 8.63×, median ≈ 8.50×.**

**Prefer the median over the mean for small comp sets.** With only six data points, one outlier (VDT's 10.60× — see Exercise 3 for why it might not belong) pulls the mean upward more than it should. The median is robust to that single outlier in a way an average of six numbers isn't. A common practice that goes one step further: drop the single highest and single lowest multiple entirely and average what's left — a **trimmed range** — which both reduces outlier sensitivity and gives you a defensible *low–high* band instead of one point estimate. This week's trimmed range (dropping VDT high and AFC/IMW low) is **7.6×–9.4×** — Exercise 3 has you compute this and apply it.

## 5. Precedent transactions — the same math, a different question

A **precedent-transactions analysis** asks a related but distinct question: not "what does the market pay for a *minority* stake in a similar public company," but "what have acquirers actually paid to buy *entire* similar companies outright." Mechanically it's identical — `deal_ev / target_ebitda_ltm`, `deal_ev / target_revenue_ltm` — but computed from `precedent_transactions` instead of `trading_comps`, and using the target's trailing (LTM) financials at the time of the deal, since that's what both sides were actually looking at when they agreed on price.

```sql
SELECT
    target,
    buyer_type,
    deal_ev,
    target_revenue_ltm,
    target_ebitda_ltm,
    ROUND(deal_ev / target_revenue_ltm, 2)  AS ev_revenue,
    ROUND(deal_ev / target_ebitda_ltm, 2)   AS ev_ebitda
FROM precedent_transactions
ORDER BY ev_ebitda DESC;
```

| Target | Buyer type | EV | Revenue (LTM) | EBITDA (LTM) | EV/Revenue | EV/EBITDA |
|---|---|---:|---:|---:|---:|---:|
| Crestpoint Manufacturing | Financial Sponsor | 890,000,000 | 410,000,000 | 76,000,000 | 2.17× | 11.71× |
| Summit Gear Corp | Strategic | 612,000,000 | 265,000,000 | 54,000,000 | 2.31× | 11.33× |
| Ridgeline Fabrication | Financial Sponsor | 340,000,000 | 158,000,000 | 31,000,000 | 2.15× | 10.97× |
| Bantam Tool & Die | Strategic | 210,000,000 | 98,000,000 | 21,000,000 | 2.14× | 10.00× |
| Northgate Precision Systems | Financial Sponsor | 155,000,000 | 78,000,000 | 16,500,000 | 1.99× | 9.39× |

**Median EV/EBITDA here is 10.97×** — noticeably higher than trading comps' 8.50× median. That gap (**≈29% higher**, `(10.97 − 8.50) / 8.50`) is not noise — it's the **control premium**: an acquirer buying 100% of a company and taking control of its decisions will typically pay more per dollar of EBITDA than a public-market investor buying a few hundred shares, because control comes with the ability to change strategy, extract synergies with the buyer's existing operations, and eliminate a competitor. This is why trading comps and precedent transactions are treated as **two different, both-legitimate answers to two different questions** — "what would a small stake trade for" versus "what would the whole company sell for" — not two attempts at the same number that should agree.

## 6. Applying multiples to a target

Once you trust a multiple (or a range), applying it to Crunch Machine Co. is one multiplication:

```
Implied EV = Target's own metric × Comp-set multiple
```

Using the trading-comps trimmed range (7.6×–9.4× EV/EBITDA) against CRNM's own NTM EBITDA (**$47,628,000**, from Lecture 1's forecast — the *forward-looking* figure, matching how trading comps are conventionally applied):

```
Low:  47,628,000 × 7.6  ≈ 362,000,000
High: 47,628,000 × 9.4  ≈ 447,700,000
```

Using the precedent-transactions trimmed range (10.0×–11.33× EV/EBITDA) against CRNM's own LTM EBITDA (**$43,050,000**, matching how precedent deals are conventionally applied — trailing, not forward):

```
Low:  43,050,000 × 10.0  ≈ 430,500,000
High: 43,050,000 × 11.33 ≈ 487,800,000
```

Two ranges, from two independent sources, both plausible, both defensible — and both noticeably below Lecture 1's DCF enterprise value of **≈$461M–$524M**. Lecture 3 takes these numbers (and the DCF's) all the way to a per-share price and reconciles the gap.

## 7. Where comps beat a DCF — and where they don't

Neither method is strictly better; each has a specific failure mode the other doesn't share.

**Comps are stronger when:**
- A good comp set genuinely exists — several truly similar public companies or recent deals.
- You distrust your own ability to forecast 5+ years of growth and margins with any precision (which, honestly, is most of the time for most businesses).
- You want a fast, market-grounded sanity check on a DCF that might be running on overly optimistic assumptions.

**DCF is stronger when:**
- No good comp set exists — a genuinely unique business, a new industry, or a company going through a transition (turnaround, major strategic shift) that makes historical peer pricing irrelevant.
- The whole market is temporarily mispricing the sector (a sector-wide bubble or a sector-wide panic) — comps just repeat that mispricing back to you, while a DCF, built from the company's *own* fundamentals, doesn't.
- You need to understand *which specific assumption* drives value — a DCF's explicit assumptions can be stress-tested one at a time (Challenge 1); a multiple is a single opaque number that bundles growth, margin, risk, and market sentiment together with no way to separate them.

The honest professional answer, and the one this course takes: **build both, every time you can.** A DCF without a comps sanity check risks defending an assumption-driven number nobody else believes. Comps without a DCF risks just repeating whatever the market currently believes, discounting risk correctly by accident if at all. Lecture 3 is what to do once you have both.

## 8. Common mistakes

- **Using equity multiples (P/E) instead of enterprise multiples** when comparing companies with different leverage — P/E can make an identical business look cheaper or more expensive purely because of its capital structure.
- **Mixing LTM and NTM inconsistently** — applying a multiple computed on peers' forward estimates to the target's trailing financials (or vice versa) silently biases the answer.
- **Trusting the mean over the median on a small comp set** — a single outlier swings a six-company average far more than it should.
- **Treating trading comps and precedent transactions as competing estimates of the same number.** They answer different questions (minority stake vs. control) and *should* differ by roughly a control premium — a small gap between them is more suspicious than a ~20–30% one.

## 9. Check yourself

- Why is EV/EBITDA preferred over P/E when comparing companies with different amounts of debt?
- Write the enterprise value formula from memory and explain, in one sentence, what each term represents in an "acquiring the whole company" story.
- Why might you use median instead of mean on a six-company comp set? What does a trimmed range add on top of the median?
- What is a control premium, concretely — name the two contributors to it discussed in Section 5.
- Name one situation where a DCF is more trustworthy than comps, and one where the reverse is true.

Lecture 3 takes every number this lecture and Lecture 1 produced — DCF enterprise values, trading-comps EV range, precedent-transactions EV range — and bridges all of them to per-share prices, then reconciles the disagreement into one defensible answer.

## Further reading

- **Damodaran (NYU Stern) — "Relative Valuation":** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/valuation/relval.htm>
- **Investopedia — "Enterprise Value (EV)":** <https://www.investopedia.com/terms/e/enterprisevalue.asp>
- **Investopedia — "Comparable Company Analysis":** <https://www.investopedia.com/terms/c/comparable-company-analysis.asp>
- **Investopedia — "Precedent Transaction Analysis":** <https://www.investopedia.com/terms/p/precedenttransactionanalysis.asp>
- **PostgreSQL — `PERCENTILE_CONT` (ordered-set aggregates):** <https://www.postgresql.org/docs/current/functions-aggregate.html#FUNCTIONS-ORDEREDSET-TABLE>
