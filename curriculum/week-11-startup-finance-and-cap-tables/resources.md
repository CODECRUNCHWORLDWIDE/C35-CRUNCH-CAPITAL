# Week 11 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+ and SQLite 3.35+** — same requirement as every prior week; see [Week 1's resources](../week-01-finance-foundations-and-tooling/resources.md) if you haven't already set these up.
- **Python 3.10+ with `pandas` and `numpy`** — nothing new beyond prior weeks. This week's `solve_pool.py` and any waterfall scripts are plain Python; no specialized cap-table library is required or used.

## Required reading (this week's core)

- **Y Combinator — "Cap Table 101":** <https://www.ycombinator.com/library/6a-cap-tables-101>
  *Why: the clearest short primer on what a cap table actually is, from the source that also wrote the standard SAFE.*
- **Y Combinator — Standard SAFE documents & explainer:** <https://www.ycombinator.com/documents>
  *Why: the actual legal templates (post-money SAFE, priced-round docs) behind the mechanics this week teaches in simplified form.*
- **Carta — "What is a liquidation preference?":** <https://carta.com/learn/startups/equity-management/liquidation-preference/>
  *Why: the clearest treatment of participating vs. non-participating preferred you'll find outside a law firm's client memo.*
- **Carta — "The option pool shuffle explained":** <https://carta.com/learn/startups/equity-management/option-pool/>
  *Why: the exact mechanic from Lecture 2, explained from the investor-negotiation side rather than the math side — read both for the full picture.*

## Reference (keep in tabs)

- **NVCA — Model Legal Documents:** <https://nvca.org/model-legal-documents/>
  *Why: the actual industry-standard term sheet, certificate of incorporation, and investors' rights agreement templates that define real vesting, preference, and pool language. Skim the term sheet template once — it's shorter than you'd think.*
- **Y Combinator — "A guide to seed fundraising":** <https://www.ycombinator.com/library/4A-a-guide-to-seed-fundraising>
  *Why: the practical, founder-facing walkthrough of how a seed round actually gets negotiated and closed.*
- **Carta — "Post-money vs. pre-money SAFEs":** <https://carta.com/learn/startups/equity-management/safe-notes/>
  *Why: this week uses a simplified "at signing" conversion basis for teaching clarity — this article explains the real, fully circular post-money SAFE formula that production cap-table software actually solves, and why YC moved to the post-money SAFE in 2018.*
- **Cooley GO — "Understanding Anti-Dilution Provisions":** <https://www.cooleygo.com/understanding-anti-dilution-provisions/>
  *Why: covers full-ratchet and weighted-average anti-dilution — the protection a down round can trigger, mentioned in the mini-project's stretch goals but not modeled this week.*

## Practice beyond the lecture scenario

- **Carta's free cap table calculator (illustrative, not a data tool):** <https://carta.com/> (product tour/demo)
  *Why: seeing a commercial cap-table product's UI after you've built the same logic in SQL makes clear exactly what it's automating — and what simplifications this week made that real software doesn't.*
- **Index Ventures — "The Option Pool Shuffle" (deep dive with worked examples):** <https://www.indexventures.com/perspectives/>
  *Why: more worked examples of the exact algebra from Lecture 2, from an investor's perspective.*

## Deeper background (optional this week)

- **Fenwick & West — "Venture Capital Survey" (quarterly, terms benchmarking):** <https://www.fenwick.com/insights>
  *Why: real, current market data on what preference multiples, participation rates, and pool sizes actually look like across live deals — useful context for how aggressive (or standard) Solstice's terms are.*
- **Paul Graham — "Equity Equation":** <http://paulgraham.com/equity.html>
  *Why: a short, classic essay on how to think about the *value* of a percentage point of equity at different company stages — useful judgment alongside this week's mechanics.*

## Glossary

| Term | Definition |
|------|------------|
| **Cap table** | The complete ledger of every security ever issued by a company and who holds it. |
| **Authorized shares** | The maximum shares a company's charter permits it to ever issue. |
| **Fully diluted shares** | Outstanding shares plus every security that could convert into shares — options, pool, unconverted SAFEs. |
| **Vesting** | Earning equity over time (typically 4 years) rather than receiving it all at grant. |
| **Cliff** | A vesting delay (typically 1 year) before which nothing vests, followed by a lump-sum jump. |
| **SAFE** | Simple Agreement for Future Equity — a contract for shares later, at a price set by a future priced round. |
| **Valuation cap** | The maximum company valuation a SAFE's conversion price is calculated against. |
| **Discount** | A percentage reduction off the priced round's per-share price, applied to a SAFE's conversion. |
| **Pre-money valuation** | What the company is worth immediately before new investment. |
| **Post-money valuation** | Pre-money valuation plus new money raised in the round. |
| **Option pool** | Shares reserved for future employee/advisor grants, often expanded pre-money at an investor's request. |
| **Option pool shuffle** | The mechanic by which a pre-money pool expansion dilutes existing holders but not the incoming investor. |
| **Liquidation preference** | A preferred holder's contractual right to a set payout before common stock, at exit. |
| **Non-participating preferred** | Gets the *greater of* its preference or its as-converted pro-rata share — never both. |
| **Participating preferred** | Gets its preference *and* also shares pro-rata in remaining proceeds. |
| **Seniority** | The order preferred classes are paid in at exit; typically last-money-in, first-money-out. |
| **Waterfall** | The full computation of how exit proceeds cascade to every shareholder, in seniority order. |

---

*Broken link? Open an issue or PR.*
