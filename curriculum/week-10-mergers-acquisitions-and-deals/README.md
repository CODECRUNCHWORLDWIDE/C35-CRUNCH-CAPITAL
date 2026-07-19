# Week 10 — M&A & deal analysis

> **Goal:** by Sunday you can price an acquisition, structure how it's paid for, prove — with your own numbers — whether it helps or hurts the acquirer's earnings per share, put a defensible dollar value on the synergies the deal is supposed to create, and build a full leveraged buyout of the same target from entry to exit, with a real sponsor return.

Welcome back to **C35 · Crunch Capital**. Every prior week has been building a single skill toward this one: Week 3 taught you to judge a project with NPV. Week 4 gave you a discount rate. Week 5 and 6 taught you how a company chooses between debt and equity. Week 7 taught you to price a whole company two independent ways. This week, all of it converges on the biggest, highest-stakes decision in corporate finance: **should Company A buy Company B, how should it pay for it, and — if a private equity sponsor bought the same target instead — what would *they* need to believe to make money?**

This week's case has two buyers chasing one target. **Crunch Machine Co.** (`CRNM`) — the manufacturer you've been modeling since Week 1 — has identified **Solstice Precision Systems, Inc.** (`SPSY`), a smaller, complementary precision-components maker, as an acquisition target. At the same time, a private equity sponsor is independently evaluating SPSY for a leveraged buyout. You'll build **both** deals from the same target financials: first CRNM's strategic acquisition — a purchase price, a control premium, a cash/stock financing mix, and the accretion/dilution math that tells the board whether the deal helps or hurts CRNM's own shareholders — then the sponsor's LBO — debt sized off EBITDA, five years of cash sweep paying that debt down, and an exit that produces (or doesn't) an attractive return. By Friday you'll be able to answer, with real numbers, the question every M&A banker gets asked in every deal: **"why is this price right, and who should actually buy this company?"**

## Learning objectives

By the end of this week, you will be able to:

- **Model an acquisition's purchase price and structure** — compute a control premium over an unaffected share price, size the total equity purchase price and implied enterprise value, and split the consideration between cash and newly issued acquirer stock.
- **Run accretion/dilution on combined EPS** — build a full pro forma income statement for the combined company (combined EBIT, blended interest expense, pro forma share count) and prove, with a number, whether the deal is accretive or dilutive to the acquirer's standalone EPS.
- **Estimate and value synergies** — separate cost synergies from revenue synergies, phase each in realistically over time, apply a risk discount to the less certain revenue synergies, and compute the net present value of both using the perpetuity method from Week 7.
- **Model an LBO with debt paydown and returns** — size acquisition debt off a leverage multiple, build a 100% cash-sweep debt schedule across a multi-year hold, and compute the sponsor's exit equity, MOIC, and IRR.
- **Reason about the financing mix and deal risk** — compare cash vs. stock financing on EPS impact, pro forma leverage, and ownership dilution, and decompose an LBO's return into the three levers that actually drive it: EBITDA growth, multiple change, and debt paydown.

## Prerequisites

- **Week 7 of this course** (valuation & comparables) — this week reuses the DCF/perpetuity-value mechanic from Week 7 to value synergies, and assumes you're comfortable computing an enterprise-value-to-equity-value bridge (net debt in, net debt out).
- **Week 5 of this course** (capital structure & leverage) — the LBO is, at its core, a leverage decision: you should already be comfortable with the idea that debt changes a business's risk and its owners' return without changing the business itself.
- **Week 4 of this course** (cost of capital & WACC) — this week's synergy valuation discounts at CRNM's WACC (9.2%, carried forward unchanged from Weeks 4 and 7).
- Comfortable with `SELECT`, `JOIN`, `GROUP BY`, aggregates, and recursive CTEs (`WITH RECURSIVE`) in SQL — Lecture 3 uses one to build the LBO's year-over-year debt schedule, the same pattern Week 7 used to compound revenue.
- Python with `pandas` and `numpy`. No new libraries this week.
- **No spreadsheets.** Same data tooling rule as every week in this course: purchase price, pro forma income statements, synergy schedules, and the LBO's debt-paydown schedule all live in SQL and Python. A cash-sweep schedule is a recursive query or a Python loop over rows, not a workbook tab with a debt balance dragged down 60 columns and a formula nobody can audit.

## Set up this week's data (do this first)

You need the same PostgreSQL 16+ (or SQLite 3.35+) database and Python 3.10+/pandas environment from prior weeks. This week adds five tables to `crunch_capital`: `acquirer_financials`, `target_financials`, `deal_terms`, `synergies`, and `lbo_assumptions`.

```bash
psql crunch_capital              # PostgreSQL
# or
sqlite3 crunch_capital.db        # SQLite fallback
```

Paste this into your shell — it works unchanged on both engines:

```sql
-- 1. The acquirer: Crunch Machine Co. (CRNM), FY2025 LTM actuals, standalone.
CREATE TABLE acquirer_financials (
    company              TEXT    PRIMARY KEY,
    ticker               TEXT    NOT NULL,
    revenue              NUMERIC NOT NULL,
    ebitda               NUMERIC NOT NULL,
    da                   NUMERIC NOT NULL,   -- depreciation & amortization
    existing_debt        NUMERIC NOT NULL,
    existing_debt_rate   NUMERIC NOT NULL,   -- weighted average coupon, decimal
    cash                 NUMERIC NOT NULL,
    shares_outstanding   BIGINT  NOT NULL,
    share_price          NUMERIC NOT NULL,   -- unaffected, pre-announcement
    tax_rate             NUMERIC NOT NULL
);

INSERT INTO acquirer_financials VALUES
('Crunch Machine Co.', 'CRNM', 230000000, 43000000, 9200000,
 110000000, 0.060, 35000000, 24000000, 20.00, 0.25);

-- 2. The target: Solstice Precision Systems, Inc. (SPSY), FY2025 LTM actuals,
--    unaffected by any deal speculation.
CREATE TABLE target_financials (
    company              TEXT    PRIMARY KEY,
    ticker               TEXT    NOT NULL,
    revenue              NUMERIC NOT NULL,
    ebitda               NUMERIC NOT NULL,
    da                   NUMERIC NOT NULL,
    existing_debt        NUMERIC NOT NULL,
    existing_debt_rate   NUMERIC NOT NULL,
    cash                 NUMERIC NOT NULL,
    shares_outstanding   BIGINT  NOT NULL,
    share_price          NUMERIC NOT NULL,   -- unaffected trading price
    tax_rate             NUMERIC NOT NULL
);

INSERT INTO target_financials VALUES
('Solstice Precision Systems, Inc.', 'SPSY', 100000000, 20000000, 4000000,
 20000000, 0.060, 8000000, 10000000, 15.00, 0.25);

-- 3. CRNM's strategic deal terms — the base-case offer for SPSY.
CREATE TABLE deal_terms (
    deal_id                    INTEGER PRIMARY KEY,
    acquirer                   TEXT    NOT NULL,
    target                     TEXT    NOT NULL,
    offer_price_per_share      NUMERIC NOT NULL,
    cash_financing_pct         NUMERIC NOT NULL,  -- of total equity purchase price
    stock_financing_pct        NUMERIC NOT NULL,
    new_acquisition_debt_rate  NUMERIC NOT NULL   -- rate on new debt raised to fund the cash portion
);

INSERT INTO deal_terms VALUES
(1, 'Crunch Machine Co.', 'Solstice Precision Systems, Inc.', 19.50, 0.55, 0.45, 0.070);

-- 4. Estimated synergies from combining CRNM and SPSY.
CREATE TABLE synergies (
    synergy_id           INTEGER PRIMARY KEY,
    synergy_type         TEXT    NOT NULL,   -- 'Cost' or 'Revenue'
    annual_run_rate      NUMERIC NOT NULL,   -- fully phased-in, pre-tax, pre-risk-discount
    risk_discount_pct    NUMERIC NOT NULL,   -- haircut applied to the run-rate before valuing (0 for cost)
    year1_phase_in_pct   NUMERIC NOT NULL,
    year2_phase_in_pct   NUMERIC NOT NULL,
    year3_phase_in_pct   NUMERIC NOT NULL    -- 1.00 = fully phased in from this year forward
);

INSERT INTO synergies VALUES
(1, 'Cost',    3000000, 0.00, 0.50, 1.00, 1.00),
(2, 'Revenue', 1000000, 0.50, 0.33, 0.67, 1.00);

-- 5. The sponsor's LBO assumptions for the same target, SPSY.
CREATE TABLE lbo_assumptions (
    target                     TEXT    PRIMARY KEY,
    entry_multiple_ev_ebitda   NUMERIC NOT NULL,
    transaction_fees_pct       NUMERIC NOT NULL,  -- % of entry EV
    term_loan_multiple_ebitda  NUMERIC NOT NULL,
    term_loan_rate             NUMERIC NOT NULL,
    hold_period_years          INTEGER NOT NULL,
    revenue_growth_rate        NUMERIC NOT NULL,  -- flat annual, applied to SPSY's own revenue
    da_pct_revenue             NUMERIC NOT NULL,
    capex_pct_revenue          NUMERIC NOT NULL,
    nwc_pct_incremental_revenue NUMERIC NOT NULL,
    cash_sweep_pct             NUMERIC NOT NULL,  -- % of free cash flow applied to debt paydown
    exit_multiple_ev_ebitda    NUMERIC NOT NULL,
    tax_rate                   NUMERIC NOT NULL
);

INSERT INTO lbo_assumptions VALUES
('Solstice Precision Systems, Inc.', 10.0, 0.02, 5.0, 0.080, 5,
 0.05, 0.04, 0.03, 0.10, 1.00, 10.0, 0.25);
```

Sanity checks — these should each print `1`, except `synergies`, which prints `2`:

```sql
SELECT COUNT(*) FROM acquirer_financials;
SELECT COUNT(*) FROM target_financials;
SELECT COUNT(*) FROM deal_terms;
SELECT COUNT(*) FROM synergies;
SELECT COUNT(*) FROM lbo_assumptions;
```

Notice the target's EBITDA margin (20%, `20,000,000 / 100,000,000`) is higher than the acquirer's (~18.7%, `43,000,000 / 230,000,000`), and SPSY trades at a much lower earnings multiple than CRNM does — that gap is not an accident. It is the entire reason this deal is interesting, and Lecture 2 has you prove exactly why with a number.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed the data; deal structure and price | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Accretion/dilution mechanics | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Synergies; re-run accretion/dilution | 1.5h | 1.5h | 1h | 0.5h | 1h | 0h | 5.5h |
| Thursday | The LBO — sizing debt, the cash sweep | 2h | 1.5h | 0h | 0.5h | 1h | 1h | 6h |
| Friday | Stock vs. cash, decomposing LBO returns | 0h | 0h | 2h | 0.5h | 1h | 1.5h | 5h |
| Saturday | Mini-project (price CRNM's deal + run the LBO) | 0h | 0h | 0h | 0h | 0h | 2h | 2h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **5.5h** | **4.5h** | **3h** | **3.5h** | **5h** | **4.5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-deal-structure-and-price.md](./lecture-notes/01-deal-structure-and-price.md) | Purchase price, control premium, implied EV, and cash-vs-stock deal structure | 2h |
| 2 | [lecture-notes/02-accretion-dilution-and-synergies.md](./lecture-notes/02-accretion-dilution-and-synergies.md) | Combined EPS, the accretion/dilution math, and valuing cost and revenue synergies | 2h |
| 3 | [lecture-notes/03-the-lbo.md](./lecture-notes/03-the-lbo.md) | LBO mechanics — sources & uses, debt sizing, the cash-sweep schedule, the sponsor's return | 2h |
| 4 | [exercises/exercise-01-accretion-dilution-model.md](./exercises/exercise-01-accretion-dilution-model.md) | Build the base-case pro forma income statement and accretion/dilution model, no synergies | 1.5h |
| 5 | [exercises/exercise-02-value-synergies.md](./exercises/exercise-02-value-synergies.md) | Value cost and revenue synergies; re-run accretion/dilution with them included | 1.5h |
| 6 | [exercises/exercise-03-simple-lbo.md](./exercises/exercise-03-simple-lbo.md) | Build SPSY's 5-year LBO debt schedule and compute MOIC and IRR | 1.5h |
| 7 | [challenges/challenge-01-stock-vs-cash-tradeoff.md](./challenges/challenge-01-stock-vs-cash-tradeoff.md) | Compare all-cash, all-stock, and mixed financing on EPS, leverage, and dilution | 1.5h |
| 8 | [challenges/challenge-02-lbo-return-drivers.md](./challenges/challenge-02-lbo-return-drivers.md) | Decompose the LBO's return into EBITDA growth, multiple change, and debt paydown | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Model CRNM's full acquisition of SPSY and a sponsor LBO of the same target | 2h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, free data sources, and further reading | — |

## By the end of this week you can…

- Turn an offer price per share into a full purchase price, a control premium, and an implied enterprise value and EV/EBITDA multiple.
- Build a pro forma combined income statement from two standalone companies and a financing structure, and prove whether the resulting EPS is accretive or dilutive.
- Separate cost synergies from revenue synergies, phase each in over a realistic timeline, risk-adjust the less certain ones, and value both with the same perpetuity-discounting method from Week 7.
- Size acquisition debt off a leverage multiple, build a multi-year cash-sweep debt schedule, and compute a sponsor's MOIC and IRR at exit.
- Decompose a deal's outcome — whether an accretion/dilution result or an LBO return — into its named, separable drivers, instead of reporting one blended number nobody can act on.

## Up next

Week 11 — Startup finance & cap tables (rounds, SAFEs, and the exit waterfall) — this week you valued and financed a mature company; next week the same discounting and ownership-math instincts get applied to a company with no earnings yet, where every financing round rewrites who owns what.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
