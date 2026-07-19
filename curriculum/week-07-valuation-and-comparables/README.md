# Week 7 — Valuation & comparables

> **Goal:** by Sunday you can value a real company two independent ways — a discounted cash flow (DCF) built from your own revenue and margin assumptions, and a trading/precedent-comparables analysis built from peer market data — bridge each to a per-share number, and reconcile the two into one defensible valuation range you can argue for in a room.

Welcome back to **C35 · Crunch Capital**. Every week so far has been building toward this one. Week 1 taught you to discount a cash flow. Week 3 taught you to judge a project with NPV. Week 4 taught you where the discount rate itself comes from. This week you point all three at the biggest question in corporate finance: **what is a whole company worth?** There are exactly two honest ways to answer that question, and they start from opposite ends of the evidence. A **DCF** builds the value up from scratch — project every future cash flow the business will generate, discount each one back to today at the company's cost of capital, and sum them. A **comparables analysis** skips the forecasting entirely and instead asks what the market is *already* paying for similar businesses, then applies that same pricing to your company. Neither method is "the answer." Used together, and reconciled honestly when they disagree, they're how real valuation work gets done — at a bank, at a private equity fund, or in your own head before you buy a stock.

This week's case is **Crunch Machine Co.** (ticker `CRNM`), the manufacturer you've been modeling since Week 1 — now the target of a full valuation exercise as of **December 31, 2025**. You'll build a five-year unlevered free cash flow forecast from a set of explicit assumptions (not a wide dump of pre-computed numbers — you derive revenue, EBITDA, and cash flow yourself), terminate it two different ways, discount it to an enterprise value, then build an independent trading-comps and precedent-transactions analysis from six peer companies and five real-feeling M&A deals. By Friday you'll have three or four different answers to "what's CRNM worth per share?" — and the actual skill this week teaches is what to do with that disagreement.

## Learning objectives

By the end of this week, you will be able to:

- **Project unlevered free cash flow** from a set of driver assumptions — revenue growth, EBITDA margin, D&A, capex, and net working capital — computed in SQL and Python, not handed to you pre-built.
- **Compute a terminal value two independent ways** — the perpetuity growth (Gordon growth) method and the exit-multiple method — and cross-check each one against the other's *implied* parameter.
- **Discount a full unlevered cash-flow stream to an enterprise value**, using a given WACC, and understand exactly what "unlevered" and "enterprise value" mean and why the discount rate must match the cash flow.
- **Build a trading-comparables set** — select peers, compute EV/Revenue and EV/EBITDA multiples from raw market data, and apply summary statistics (median, mean, a trimmed range) to a target company.
- **Build a precedent-transactions set** — the same multiple mechanics applied to real M&A deals instead of public trading prices — and explain why transaction multiples run higher than trading multiples (the control premium).
- **Bridge enterprise value to equity value and to a per-share price** — the mechanics of the EV-to-equity bridge (net debt, and what else belongs in it).
- **Triangulate a defensible valuation range** — reconcile a DCF and a comps-based answer that disagree, explain *why* they disagree in specific, quantified terms, and land on one number (or a tight range) you can defend.

## Prerequisites

- **Week 4 of this course** (cost of capital & WACC) — this week's DCF takes a WACC as a given input and does not re-derive it; you should already understand what WACC represents and why it's the correct discount rate for an *unlevered* cash-flow stream.
- **Week 3 of this course** (NPV/IRR) — the discounting mechanics (`PV = CF / (1+r)^t`, summing a stream of present values) are assumed automatic. This week is that same mechanic applied to five years of a whole company instead of one project.
- Comfortable with `SELECT`, `GROUP BY`, aggregates (`SUM`, `AVG`, `MIN`, `MAX`), and window/statistical functions in SQL (`PERCENTILE_CONT` for medians is introduced from scratch in Lecture 2 if it's new to you).
- Python with `pandas` and `numpy` — no new libraries required beyond prior weeks.
- **No spreadsheets.** Same [data tooling rule](../../SYLLABUS.md#data-tooling-rule) as every week: every assumption, every projected cash flow, every comp, and every multiple lives in SQL and Python. A comps set is a table with a `GROUP BY`, not a workbook tab with fifteen peer columns dragged across.

## Set up this week's data (do this first)

You need the same PostgreSQL 16+ (or SQLite 3.35+) database and Python 3.10+/pandas environment from prior weeks. This week adds four new tables to `crunch_capital`: `valuation_assumptions`, `revenue_forecast`, `trading_comps`, and `precedent_transactions`.

```bash
psql crunch_capital              # PostgreSQL
# or
sqlite3 crunch_capital.db        # SQLite fallback
```

Paste this into your shell — it works unchanged on both engines:

```sql
-- 1. A one-row snapshot of everything needed to run the DCF and the EV-to-equity
--    bridge: trailing financials, capital structure, and the discounting inputs.
CREATE TABLE valuation_assumptions (
    company                  TEXT    PRIMARY KEY,
    valuation_date           DATE    NOT NULL,
    ltm_revenue              NUMERIC NOT NULL,   -- last twelve months, FY2025 actual
    ltm_ebitda               NUMERIC NOT NULL,
    shares_outstanding       BIGINT  NOT NULL,
    total_debt               NUMERIC NOT NULL,
    cash_and_equivalents     NUMERIC NOT NULL,
    wacc                     NUMERIC NOT NULL,   -- carried forward from Week 4's methodology
    terminal_growth_rate     NUMERIC NOT NULL,   -- perpetuity-growth method, as a decimal
    exit_multiple_ev_ebitda  NUMERIC NOT NULL,   -- exit-multiple method cross-check
    tax_rate                 NUMERIC NOT NULL
);

INSERT INTO valuation_assumptions VALUES
('Crunch Machine Co.', '2025-12-31', 210000000, 43050000, 24000000,
 110000000, 35000000, 0.092, 0.025, 8.0, 0.25);

-- 2. Five years (FY2026–FY2030) of forecast DRIVERS, not pre-computed cash flows.
--    You derive revenue, EBITDA, and unlevered FCF from these in Exercise 1.
CREATE TABLE revenue_forecast (
    fiscal_year                 INTEGER PRIMARY KEY,
    revenue_growth_rate         NUMERIC NOT NULL,  -- year-over-year, decimal
    ebitda_margin               NUMERIC NOT NULL,  -- decimal, of that year's revenue
    da_pct_revenue               NUMERIC NOT NULL,  -- depreciation & amortization, % of revenue
    capex_pct_revenue            NUMERIC NOT NULL,  -- capital expenditure, % of revenue
    nwc_pct_incremental_revenue  NUMERIC NOT NULL   -- Δ net working capital, % of the YEAR'S revenue increase
);

INSERT INTO revenue_forecast VALUES
(2026, 0.08, 0.210, 0.04, 0.055, 0.15),
(2027, 0.07, 0.220, 0.04, 0.050, 0.15),
(2028, 0.06, 0.225, 0.04, 0.045, 0.15),
(2029, 0.05, 0.230, 0.04, 0.040, 0.15),
(2030, 0.04, 0.235, 0.04, 0.035, 0.15);

-- 3. Six trading comps — public peers in the same industrial-equipment space,
--    raw market data (you compute EV and multiples yourself in Exercise 3).
CREATE TABLE trading_comps (
    comp_id               INTEGER PRIMARY KEY,
    company                TEXT    NOT NULL,
    ticker                 TEXT    NOT NULL,
    revenue_ntm            NUMERIC NOT NULL,   -- next-twelve-months consensus estimate
    ebitda_ntm             NUMERIC NOT NULL,
    share_price             NUMERIC NOT NULL,
    shares_outstanding      BIGINT  NOT NULL,
    total_debt               NUMERIC NOT NULL,
    cash_and_equivalents      NUMERIC NOT NULL
);

INSERT INTO trading_comps VALUES
(1, 'TitanForge Industrial',     'TFI', 285000000, 62700000, 23.58, 18000000, 182026400, 17000000),
(2, 'Meridian Motion Systems',   'MMS', 198000000, 40590000, 17.69, 14500000,  84331380, 12000000),
(3, 'Anchor Fabrication Corp',   'AFC', 410000000, 79950000, 15.19, 26000000, 237667000, 25000000),
(4, 'Vantage Drive Technologies','VDT', 165000000, 39600000, 32.44, 11000000,  72964000, 10000000),
(5, 'Falcon Precision Group',    'FPG', 320000000, 68800000, 19.94, 21500000, 202696000, 19000000),
(6, 'Ironclad Machine Works',    'IMW', 245000000, 46550000, 10.58, 19000000, 149064000, 15000000);

-- 4. Five precedent M&A transactions in the same sector — deal-level EV and the
--    target's trailing financials at announcement, so you can compute deal multiples.
CREATE TABLE precedent_transactions (
    deal_id             INTEGER PRIMARY KEY,
    acquirer             TEXT    NOT NULL,
    target                TEXT    NOT NULL,
    announce_date          DATE    NOT NULL,
    buyer_type              TEXT    NOT NULL,   -- 'Strategic' or 'Financial Sponsor'
    deal_ev                  NUMERIC NOT NULL,
    target_revenue_ltm        NUMERIC NOT NULL,
    target_ebitda_ltm         NUMERIC NOT NULL
);

INSERT INTO precedent_transactions VALUES
(1, 'Praxis Industrial Group',      'Summit Gear Corp',            '2025-03-14', 'Strategic',          612000000, 265000000, 54000000),
(2, 'Vertex Capital Partners',      'Ridgeline Fabrication',       '2024-11-02', 'Financial Sponsor',  340000000, 158000000, 31000000),
(3, 'Continental Machine Holdings', 'Bantam Tool & Die',           '2024-06-20', 'Strategic',          210000000,  98000000, 21000000),
(4, 'Atlas Industrial Partners',    'Crestpoint Manufacturing',    '2023-09-08', 'Financial Sponsor',  890000000, 410000000, 76000000),
(5, 'Sentinel Equity Group',        'Northgate Precision Systems', '2023-02-27', 'Financial Sponsor',  155000000,  78000000, 16500000);
```

Sanity checks — these should print `1`, `5`, `6`, and `5`:

```sql
SELECT COUNT(*) FROM valuation_assumptions;
SELECT COUNT(*) FROM revenue_forecast;
SELECT COUNT(*) FROM trading_comps;
SELECT COUNT(*) FROM precedent_transactions;
```

Notice what you were **not** given: a `cash_flows` table like Week 1's, with the answer already computed. `revenue_forecast` holds *drivers* — growth rates, margins, percentages — because that's what real projection work looks like: assumptions in, a model that derives the numbers out. If a driver changes (say, EBITDA margin comes in 100bps lower), every downstream number should recompute correctly on the next query run — that's the entire argument for building this in SQL and Python instead of a spreadsheet, made concrete for the highest-stakes calculation in the course.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed the data; building a DCF | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Free cash flow projection + terminal value | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Trading and transaction comps | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Triangulating a range; EV-to-equity bridge | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Sensitivity + reconciliation challenges | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (value CRNM two ways) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-building-a-dcf.md](./lecture-notes/01-building-a-dcf.md) | Free cash flow projection, terminal value (two methods), discounting to enterprise value | 2h |
| 2 | [lecture-notes/02-trading-and-transaction-comps.md](./lecture-notes/02-trading-and-transaction-comps.md) | Selecting a comp set, EV multiples, precedent transactions, and where comps beat a DCF | 2h |
| 3 | [lecture-notes/03-triangulating-a-range.md](./lecture-notes/03-triangulating-a-range.md) | Reconciling DCF vs. comps, the EV-to-equity bridge, and defending the gap | 2h |
| 4 | [exercises/exercise-01-project-free-cash-flows.md](./exercises/exercise-01-project-free-cash-flows.md) | Derive revenue, EBITDA, and unlevered FCF from `revenue_forecast`, in SQL and Python | 1.5h |
| 5 | [exercises/exercise-02-terminal-value-methods.md](./exercises/exercise-02-terminal-value-methods.md) | Compute terminal value by perpetuity growth and by exit multiple; cross-check both | 1.5h |
| 6 | [exercises/exercise-03-build-a-comp-set.md](./exercises/exercise-03-build-a-comp-set.md) | Compute EV and multiples from `trading_comps`; apply them to Crunch Machine Co. | 1.5h |
| 7 | [challenges/challenge-01-dcf-sensitivity-football-field.md](./challenges/challenge-01-dcf-sensitivity-football-field.md) | Build a WACC × growth sensitivity grid, then a full valuation football field | 1.5h |
| 8 | [challenges/challenge-02-reconcile-dcf-vs-comps.md](./challenges/challenge-02-reconcile-dcf-vs-comps.md) | Quantify exactly why the DCF and comps answers disagree, and defend a final range | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Value Crunch Machine Co. by DCF and by trading comps; reconcile to one range | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, free data sources, and further reading | — |

## By the end of this week you can…

- Turn a set of growth, margin, capex, and working-capital assumptions into a full five-year unlevered free cash flow forecast, in SQL and in pandas.
- Terminate that forecast two different, independent ways, and use each method's *implied* parameter to sanity-check the other.
- Discount the whole stream to an enterprise value, and bridge that enterprise value to a per-share price.
- Build a trading-comps and a precedent-transactions analysis from raw market data, compute the multiples yourself, and apply them defensibly to a target.
- Look at two valuation methods that disagree by tens of millions of dollars, explain specifically why, and land on one number you can defend under questioning.

## Up next

Week 8 — Portfolio theory (mean-variance, the efficient frontier, and the Sharpe ratio) — once you can price one company correctly, we turn to pricing a *collection* of them held together, where the same discounting instincts meet a genuinely new idea: diversification.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
