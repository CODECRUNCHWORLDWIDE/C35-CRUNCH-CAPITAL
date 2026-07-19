# Week 12 — Capstone — allocate & defend

> **Goal:** by Sunday you can stand in front of an Investment Committee (IC) with one target company, one proposed position size, one quantified risk picture, and a reproducible memo — every number backed by a query or a script you can re-run live if someone asks "show me" — and defend the whole case for 20 minutes of questions without ever opening a spreadsheet.

Welcome to the last week of **C35 · Crunch Capital**. Every prior week built one piece of the machine: Week 1 taught you to discount a cash flow, Week 4 gave you a WACC, Week 7 valued a company by DCF and comps, Week 8 built a diversified core allocation, Week 9 taught you to backtest a rule instead of trusting a hunch, Week 10 modeled a deal, Week 11 built a cap table. This week nothing is new in kind — it is all **assembly**. You are an analyst at **Crunch Capital Advisors**, the fictional multi-asset shop from Week 8. The IC already runs the six-fund core allocation you optimized that week. Now they're considering a **satellite position** in **Crunch Machine Co.** (`CRNM`) — the same company you valued in Week 7 and whose peer group you backtested in Week 9. Your job, in order: (1) assemble a defensible valuation from pieces you already built, (2) size a position and show what it would have done to the portfolio historically, (3) put a number on how much you could lose and under what conditions, and (4) write the memo that survives an IC member trying to poke a hole in every line of it — including the uncomfortable question of whether your own incentives, or the target's management's, are pointing the same direction as the thesis.

That last part is the part self-taught quants skip. A valuation is not "done" when the DCF spits out a number — it's done when you can say *why* that number, *what would make it wrong*, and *who benefits if it's wrong and nobody notices*. This week is a full dress rehearsal for the job itself: allocate capital, know your risk, and defend the number to people whose job is to find the hole in it.

## Learning objectives

By the end of this week, you will be able to:

- **Assemble an end-to-end investment case** from separately-built pieces — a DCF, a comps range, a market price, and a proposed position size — into one coherent recommendation, in SQL and Python, with every intermediate number traceable to a query.
- **Size a position** inside an existing portfolio and quantify its effect on expected return and volatility using the covariance/correlation machinery from Week 8.
- **Compute historical and parametric Value at Risk (VaR)** for a single position and for a portfolio, and explain precisely what a 95% or 99% VaR number does and does not promise you.
- **Run scenario analysis and stress tests** — apply named shocks (recession, margin compression, multiple re-rating, credit crunch) to your valuation and compute the probability-weighted range of outcomes, not just the base case.
- **Reason about governance and incentive risk** — recognize the incentive structures and control failures that turn a good thesis into a bad outcome, and quantify (not just narrate) how a poorly-designed bonus plan would have behaved across your own scenarios.
- **Write and defend a reproducible risk memo** — a document where every claimed number has a query or script attached, structured the way a real IC memo is structured, and able to survive a red-team session against your own thesis.

## Prerequisites

- **Week 7 of this course** (valuation & comparables) — this week's valuation assembly reuses the DCF and comps mechanics from that week; you should be comfortable with unlevered FCF, terminal value (both methods), and the EV-to-equity bridge. If it's rusty, skim Week 7's README before Monday.
- **Week 8 of this course** (portfolio theory & allocation) — you should be comfortable with expected return, volatility, covariance, and portfolio-variance arithmetic (`w'Σw`) for a multi-asset book; this week sizes one more position into that same book.
- **Week 9 of this course** (investment analytics & backtesting) — comfortable computing a return series' volatility, Sharpe ratio, and drawdown from daily prices; this week reuses that machinery for VaR.
- Working SQL (`SELECT`, `JOIN`, `GROUP BY`, window functions) and Python with `pandas`/`numpy` — no new libraries required this week.
- **No spreadsheets.** Same [data tooling rule](../../SYLLABUS.md#data-tooling-rule) as every week: the valuation, the price history, the scenarios, the portfolio weights, and the governance cases all live in SQL and Python — an IC memo built on a spreadsheet model nobody else can audit is exactly the failure mode this course exists to prevent.

## Set up this week's data (do this first)

You need the same PostgreSQL 16+ (or SQLite 3.35+) `crunch_capital` database and Python 3.10+/`pandas`/`numpy` environment from prior weeks. This week adds five new tables: `capstone_deal_snapshot`, `capstone_cash_flows`, `capstone_scenarios`, `capstone_price_history`, `capstone_portfolio_context`, and `capstone_governance_cases`.

```bash
psql crunch_capital              # PostgreSQL
# or
sqlite3 crunch_capital.db        # SQLite fallback
```

**Step 1 — the deal snapshot and the cash-flow stream.** This is the assembled output of Week 7's methodology applied fresh, as of today's valuation date, so the whole week works from one consistent set of numbers.

```sql
-- 1. One-row snapshot: everything needed to bridge enterprise value to a
--    per-share price, plus the DCF and comps outputs you'll assemble
--    in Exercise 1 (given here so every later step can reference them).
CREATE TABLE capstone_deal_snapshot (
    company                TEXT    PRIMARY KEY,
    ticker                  TEXT    NOT NULL,
    as_of_date               DATE    NOT NULL,
    ltm_revenue               NUMERIC NOT NULL,
    ltm_ebitda                 NUMERIC NOT NULL,
    shares_outstanding          BIGINT  NOT NULL,
    total_debt                   NUMERIC NOT NULL,
    cash_and_equivalents          NUMERIC NOT NULL,
    wacc                           NUMERIC NOT NULL,
    terminal_growth_rate            NUMERIC NOT NULL,   -- perpetuity-growth method
    exit_multiple_ev_ebitda           NUMERIC NOT NULL,   -- exit-multiple method
    tax_rate                           NUMERIC NOT NULL,
    comps_median_ev_ebitda               NUMERIC NOT NULL,  -- carried forward from Week 7-style peer set
    current_share_price                   NUMERIC NOT NULL   -- today's market price
);

INSERT INTO capstone_deal_snapshot VALUES
('Crunch Machine Co.', 'CRNM', '2026-01-05',
 210000000, 43050000, 24000000, 110000000, 35000000,
 0.092, 0.025, 8.0, 0.25, 8.50, 12.50);

-- 2. The five-year unlevered free cash flow stream, already derived from
--    Week-7-style driver assumptions (revenue growth, EBITDA margin, D&A,
--    capex, NWC). You are NOT re-deriving drivers this week — that was
--    Week 7's job. This week you ASSEMBLE: discount this stream, add a
--    terminal value, and bridge to a per-share price.
CREATE TABLE capstone_cash_flows (
    fiscal_year       INTEGER PRIMARY KEY,
    revenue             NUMERIC NOT NULL,
    ebitda                NUMERIC NOT NULL,
    da                      NUMERIC NOT NULL,   -- depreciation & amortization
    ebit                     NUMERIC NOT NULL,
    nopat                     NUMERIC NOT NULL,  -- net operating profit after tax
    capex                      NUMERIC NOT NULL,
    delta_nwc                   NUMERIC NOT NULL,  -- change in net working capital
    unlevered_fcf                 NUMERIC NOT NULL
);

INSERT INTO capstone_cash_flows VALUES
(2026, 226800000, 47628000,  9072000, 38556000, 28917000, 12474000, 2520000, 22995000),
(2027, 242676000, 53388720,  9707040, 43681680, 32761260, 12133800, 2381400, 27953100),
(2028, 257236560, 57878226, 10289462, 47588764, 35691573, 11575645, 2184084, 32221306),
(2029, 270098388, 62122629, 10803935, 51318694, 38489021, 10803935, 1929274, 36559746),
(2030, 280902324, 66012046, 11236093, 54775953, 41081965,  9831581, 1620590, 40865886);

-- 3. Six named scenarios with probability weights (they sum to 1.00) and
--    the shocks each applies to the base-case drivers above. You compute
--    each scenario's resulting valuation in Exercise 2 — this table is
--    the assumption set, not the answer.
CREATE TABLE capstone_scenarios (
    scenario_id             INTEGER PRIMARY KEY,
    scenario_name             TEXT    NOT NULL,
    probability                 NUMERIC NOT NULL,
    revenue_growth_shock          NUMERIC NOT NULL,  -- added to EACH year's growth rate, decimal
    ebitda_margin_shock             NUMERIC NOT NULL,  -- added to EACH year's margin, decimal
    exit_multiple_shock               NUMERIC NOT NULL,  -- added to the exit EV/EBITDA multiple
    wacc_shock                          NUMERIC NOT NULL,  -- added to WACC, decimal
    notes                                 TEXT
);

INSERT INTO capstone_scenarios VALUES
(1, 'Base Case',           0.45,  0.000,  0.000,  0.0, 0.000, 'Week 7-style assumptions, unchanged'),
(2, 'Recession',            0.20, -0.040, -0.020, -1.5, 0.000, 'Demand shock; margins compress with volume'),
(3, 'Margin Compression',    0.15, -0.010, -0.030, -1.0, 0.000, 'Input-cost / tariff shock; growth roughly holds'),
(4, 'Multiple Re-Rating',     0.10,  0.010,  0.005,  1.5, 0.000, 'Sector re-rates up on a peer take-out'),
(5, 'Stagflation',              0.07, -0.020, -0.025, -2.0, 0.010, 'Growth AND discount rate both move against you'),
(6, 'Credit Crunch',              0.03, -0.015, -0.010, -1.0, 0.015, 'Refinancing risk repriced; WACC and multiple both hit');

-- 4. The satellite-position context: Crunch Capital Advisors' existing
--    six-fund core allocation (Week 8) plus the proposed CRNM satellite
--    weight, with each asset's own annual return/volatility and its
--    correlation to CRNM specifically. (We do NOT re-derive the full 7x7
--    covariance matrix here — that is a deliberate simplification you are
--    expected to name and defend in this week's memo; see Challenge 1.)
CREATE TABLE capstone_portfolio_context (
    ticker             TEXT PRIMARY KEY,
    role                 TEXT    NOT NULL,   -- 'core' or 'satellite'
    target_weight           NUMERIC NOT NULL,  -- proposed weight, decimal, sums to 1.00
    annual_return             NUMERIC NOT NULL,  -- decimal
    annual_volatility           NUMERIC NOT NULL,  -- decimal
    corr_with_crnm                 NUMERIC NOT NULL   -- correlation of this asset's returns with CRNM's
);

INSERT INTO capstone_portfolio_context VALUES
('CRNQ', 'core',      0.22, 0.095, 0.170, 0.55),
('CRNI', 'core',      0.12, 0.075, 0.190, 0.45),
('CRNB', 'core',      0.25, 0.035, 0.060, 0.05),
('CRNH', 'core',      0.10, 0.065, 0.110,  0.25),
('CRNG', 'core',      0.08, 0.045, 0.150, -0.10),
('CRNR', 'core',      0.15, 0.080, 0.180,  0.30),
('CRNM', 'satellite', 0.08, 0.130, 0.280,  1.00);

-- 5. Six anonymized-but-structured governance/incentive case studies used
--    in Lecture 3 and Challenge 2. Real patterns, fictional company names.
CREATE TABLE capstone_governance_cases (
    case_id            INTEGER PRIMARY KEY,
    company_alias         TEXT    NOT NULL,
    industry                TEXT    NOT NULL,
    incentive_structure       TEXT    NOT NULL,
    failure_mode                 TEXT    NOT NULL,
    outcome                        TEXT    NOT NULL,
    lesson                           TEXT    NOT NULL
);

INSERT INTO capstone_governance_cases VALUES
(1, 'Northbridge Retail Holdings',  'Retail',          'Bonuses tied to revenue growth only',
    'Revenue-metric gaming',    'Channel-stuffed sales; revenue up, cash flow and margin down for 3 years',
    'Tie incentive pay to a cash or return metric (FCF, ROIC), never a top-line metric alone'),
(2, 'Falkirk Energy Partners',      'Energy',          'CEO equity vests entirely on stock price at exit',
    'Short-termism / underinvestment', 'Maintenance capex deferred to hit a price target; asset failures followed',
    'Vesting on a single price-based trigger encourages harvesting the balance sheet, not building it'),
(3, 'Ashworth Diagnostics',          'Healthcare',      'Board audit committee met once a year',
    'Weak oversight cadence',      'A material accounting error went undetected for two full quarters',
    'Match monitoring cadence to the actual pace of the business, not a calendar minimum'),
(4, 'Solmark Components',            'Manufacturing',   'Founder held both CEO and Chair roles, no lead independent director',
    'Concentration of power',       'Related-party supplier contract approved with no independent review',
    'Separate CEO/Chair, or appoint a genuinely empowered lead independent director, when founder control is high'),
(5, 'Verity Consumer Brands',         'Consumer',        'Sales bonus paid on bookings at time of shipment',
    'Channel-stuffing at quarter end', 'Returns spiked the following quarter; reported growth was partly reversed',
    'Recognize and reward revenue net of expected returns, not gross bookings'),
(6, 'Ironway Logistics',               'Transportation',  'Safety incidents excluded from all bonus scorecards',
    'Unmeasured externality',           'Deferred fleet maintenance cut costs short-term; accident rate rose',
    'Anything you do not put a number on and put in the scorecard will get cut when budgets tighten');
```

**Step 2 — generate two years of daily prices for `CRNM`.** Same reproducible geometric Brownian motion approach as Week 9, seeded so every student gets identical numbers, priced off today's `current_share_price`. Save as `generate_crnm_prices.py`:

```python
import numpy as np
import pandas as pd

# One seed for the whole cohort = everyone gets the same 504 rows.
rng = np.random.default_rng(35120)

TICKER = "CRNM"
START_PRICE = 12.50     # matches capstone_deal_snapshot.current_share_price
ANNUAL_DRIFT = 0.06     # modest positive drift; the thesis is in the VALUATION gap, not a rocket-ship price path
ANNUAL_VOL = 0.28       # a levered, cyclical industrial name — deliberately higher vol than the Week 8 core funds

dates = pd.bdate_range("2024-01-02", "2025-12-31")   # business days only, 504 of them
n = len(dates)

daily_mu = ANNUAL_DRIFT / 252
daily_sigma = ANNUAL_VOL / np.sqrt(252)
z = rng.standard_normal(n)
log_returns = daily_mu - 0.5 * daily_sigma**2 + daily_sigma * z
price_path = START_PRICE * np.exp(np.cumsum(log_returns))
volume = rng.lognormal(mean=13.3, sigma=0.30, size=n).astype(int)

rows = [(TICKER, d.date().isoformat(), round(float(p), 2), int(v))
        for d, p, v in zip(dates, price_path, volume)]

prices = pd.DataFrame(rows, columns=["ticker", "price_date", "close_price", "volume"])
prices.to_csv("crnm_prices_seed.csv", index=False)
print(f"generated {len(prices)} rows for {TICKER}")
```

**Step 3 — create the table and load the CSV.**

```sql
CREATE TABLE capstone_price_history (
    ticker       TEXT    NOT NULL,
    price_date   DATE    NOT NULL,
    close_price  NUMERIC NOT NULL,
    volume       BIGINT  NOT NULL,
    PRIMARY KEY (ticker, price_date)
);
```

```python
# PostgreSQL (sqlalchemy + psycopg2)
from sqlalchemy import create_engine
import pandas as pd

prices = pd.read_csv("crnm_prices_seed.csv", parse_dates=["price_date"])
engine = create_engine("postgresql+psycopg2://localhost/crunch_capital")
prices.to_sql("capstone_price_history", engine, if_exists="append", index=False, method="multi", chunksize=1000)
```

```python
# SQLite fallback
import sqlite3
import pandas as pd

prices = pd.read_csv("crnm_prices_seed.csv")
conn = sqlite3.connect("crunch_capital.db")
prices.to_sql("capstone_price_history", conn, if_exists="append", index=False)
conn.close()
```

**Sanity checks** — these should print `1`, `5`, `6`, `504`, `7`, and `6`:

```sql
SELECT COUNT(*) FROM capstone_deal_snapshot;
SELECT COUNT(*) FROM capstone_cash_flows;
SELECT COUNT(*) FROM capstone_scenarios;
SELECT COUNT(*) FROM capstone_price_history;
SELECT COUNT(*) FROM capstone_portfolio_context;
SELECT COUNT(*) FROM capstone_governance_cases;
```

```sql
-- Probabilities should sum to 1.00, weights should sum to 1.00
SELECT ROUND(SUM(probability), 4) FROM capstone_scenarios;
SELECT ROUND(SUM(target_weight), 4) FROM capstone_portfolio_context;
```

Notice what's given and what isn't. You're given the assembled DCF inputs (`capstone_cash_flows`), not a pre-computed enterprise value — Exercise 1 has you discount it yourself. You're given a `comps_median_ev_ebitda` (the *result* of a Week-7-style comps exercise, since re-running that full process isn't this week's point). You're given a `current_share_price` **below** every reasonable estimate of intrinsic value — that gap is the entire investment case, and your first job this week is to quantify exactly how big it is and how confident you should be in it.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Capstone weeks run a little front-loaded — there's more to assemble before Wednesday, and Friday/Saturday lean hard into the mini-project.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed the data; assembling the end-to-end deal | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Tuesday | Sizing the position; VaR mechanics | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Scenario analysis and stress testing | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Governance, incentives, and control failures | 2h | 1.5h | 1h | 0.5h | 1h | 1h | 6h |
| Friday | Drafting and red-teaming the memo | 0h | 0h | 1h | 0.5h | 1h | 2h | 4.5h |
| Saturday | Mini-project (full deal end-to-end) | 0h | 0h | 0h | 0h | 0h | 3h | 3h |
| Sunday | Quiz + course review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **4.5h** | **3h** | **3.5h** | **5h** | **6h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-the-end-to-end-deal.md](./lecture-notes/01-the-end-to-end-deal.md) | Stitching the DCF, comps, and market price into one investment case; sizing the position | 2h |
| 2 | [lecture-notes/02-quantifying-risk.md](./lecture-notes/02-quantifying-risk.md) | Historical and parametric VaR, scenario analysis, and stress testing the thesis | 2h |
| 3 | [lecture-notes/03-governance-and-defending-the-number.md](./lecture-notes/03-governance-and-defending-the-number.md) | Incentives, controls, and building a reproducible memo that survives scrutiny | 2h |
| 4 | [exercises/exercise-01-assemble-the-valuation.md](./exercises/exercise-01-assemble-the-valuation.md) | Discount `capstone_cash_flows`, bridge to a per-share range, compare to the market price | 1.5h |
| 5 | [exercises/exercise-02-var-and-scenarios.md](./exercises/exercise-02-var-and-scenarios.md) | Compute historical + parametric VaR, and a probability-weighted scenario valuation | 1.5h |
| 6 | [exercises/exercise-03-draft-the-risk-memo.md](./exercises/exercise-03-draft-the-risk-memo.md) | Assemble the valuation, sizing, risk, and governance sections into one memo | 1.5h |
| 7 | [challenges/challenge-01-red-team-the-thesis.md](./challenges/challenge-01-red-team-the-thesis.md) | Attack your own base case's weakest assumptions with numbers, not vibes | 1.5h |
| 8 | [challenges/challenge-02-governance-failure-case.md](./challenges/challenge-02-governance-failure-case.md) | Diagnose a governance case from `capstone_governance_cases` and design the fix | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Run the full deal end-to-end and deliver an IC-ready risk memo | 3h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, risk-management references, and further reading | — |

## By the end of this week you can…

- Take a target company from "here is a market price" to "here is our estimate of value, our position size, and our confidence interval," entirely in SQL and Python.
- Compute VaR two independent ways, know which one to trust when they disagree, and explain the difference between "5% of the time we lose more than $X" and "the worst we could ever lose is $X" (they are not the same claim).
- Turn a set of macro/company scenarios into a probability-weighted range of outcomes instead of a single point estimate, and use that range to size a position responsibly.
- Recognize the incentive and governance patterns that turn a good thesis into a bad outcome, and say specifically what control would have caught each one.
- Write a memo where an IC member can pick any number and get the query or script that produced it — the actual, portable skill this entire course has been building toward.

## Course complete

You've now run every stage of a real capital-allocation job — from a single cash flow (Week 1) to a full deal, priced, sized, risk-managed, and defended (Week 12) — without opening a spreadsheet once. Take the [quiz](./quiz.md), ship the [mini-project](./mini-project/README.md), and go build something with it.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
