# Week 10 — Resources

Curated free and official references for this week's topics: deal structure and price, accretion/dilution, synergies, and the LBO. Everything here is free to access; no paid data terminals required for this course.

## Deal structure and price

- **Investopedia — "Control Premium"** — <https://www.investopedia.com/terms/c/control-premium.asp>
- **Investopedia — "Cash and Stock Merger"** — <https://www.investopedia.com/terms/c/cashandstockmerger.asp>
- **Investopedia — "Tender Offer"** — <https://www.investopedia.com/terms/t/tenderoffer.asp> — the mechanism many public-company acquisitions use to actually buy shares once a price is agreed
- **U.S. SEC — EDGAR full-text search** — <https://www.sec.gov/cgi-bin/srqsb?text=&first=1&last=40> (or the modern search at <https://www.sec.gov/edgar/search/>) — search any real public-company merger agreement (`8-K` filings following an announcement) to see actual deal terms, premiums, and financing structures

## Accretion/dilution and synergies

- **Investopedia — "Accretion/Dilution Analysis"** — <https://www.investopedia.com/terms/a/accretiondilution.asp>
- **Investopedia — "Synergy"** — <https://www.investopedia.com/terms/s/synergy.asp>
- **Damodaran (NYU Stern) — "Valuing Synergy"** — <https://pages.stern.nyu.edu/~adamodar/pdfiles/papers/synergy.pdf> — a rigorous academic treatment of why synergy estimates in announced deals are so often too optimistic
- **Harvard Business Review — "The Big Idea: The New M&A Playbook"** — search HBR's free-preview archive for coverage of post-merger integration and synergy realization rates

## The LBO

- **Investopedia — "Leveraged Buyout (LBO)"** — <https://www.investopedia.com/terms/l/leveragedbuyout.asp>
- **Investopedia — "Cash Sweep"** — <https://www.investopedia.com/terms/c/cash-sweep.asp>
- **Investopedia — "Internal Rate of Return (IRR)"** — <https://www.investopedia.com/terms/i/irr.asp>
- **Investopedia — "Multiple on Invested Capital (MOIC)"** — <https://www.investopedia.com/terms/m/multiple-on-invested-capital.asp>

## Tools to install (if you haven't already, from prior weeks)

- **PostgreSQL 16+** — <https://www.postgresql.org/download/> — this week's `WITH RECURSIVE` cash-sweep query needs it, or SQLite 3.35+ as the course's fallback.
- **Python 3.10+** with `pandas`, `numpy`, and `sqlalchemy` — `pip install pandas numpy sqlalchemy` (add `psycopg2-binary` for Postgres, unnecessary for SQLite).
- A SQL client of your choice — `psql`/`sqlite3` on the command line work fine; DBeaver or TablePlus are free GUI options if you prefer one.

## Recap of internal references this week depends on

- **Week 4** — cost of capital & WACC (this week's synergy NPV reuses CRNM's 9.2% WACC unchanged).
- **Week 5** — capital structure & leverage (the LBO's use of debt to amplify equity returns is the same mechanism Week 5 introduced for a corporate balance sheet, deliberately maximized here).
- **Week 7** — valuation & comparables (the perpetuity-value method used for synergy NPV, and the EV-to-equity bridge used throughout this week's deal pricing, are both built in Week 7).
