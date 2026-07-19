# Week 4 — Cost of capital & WACC

> **Goal:** by Sunday you can look at any company and produce a defensible, single number — its weighted average cost of capital — built from the ground up: a CAPM cost of equity, a beta you estimated yourself from real price data in pandas, an after-tax cost of debt from its actual bonds, and market-value capital weights. You'll compute it three different ways for the same company and be able to explain why the three numbers don't match.

Welcome back to **C35 · Crunch Capital**. Week 3 taught you to judge a *project* with NPV and IRR — but every one of those calculations quietly assumed you already had a discount rate (the "hurdle rate" sitting in the `cash_flows` table). This week you stop assuming it and start *building* it. The discount rate a company should use is not a policy choice or a round number pulled from the air — it's the blended return that its investors, in aggregate, require for the risk they're taking on. That blended rate is the **weighted average cost of capital (WACC)**, and it is arguably the single most consequential number in corporate finance: raise it half a point and profitable projects start looking unprofitable; lower it half a point and you start funding things that will destroy value. Get WACC wrong and every NPV downstream of it is wrong too, no matter how carefully you built the cash flow forecast.

This week we build WACC for one real-feeling company: **Crunch Machine Co.**, the fictional manufacturer from Week 1, now recast as a publicly traded firm with five years of monthly stock price history, a market benchmark to regress it against, and three outstanding bonds. You'll compute its cost of equity with CAPM, estimate its beta yourself by regressing returns in pandas (rather than trusting a vendor's black-box number), price its debt from real bond terms, and assemble all of it — with market-value weights — into one WACC. Then you'll do it two more ways (a vendor beta, and a peer-comparables beta) and see, concretely, why "the WACC" is really a range an analyst has to reason about, not a single fact to look up.

## Learning objectives

By the end of this week, you will be able to:

- **Estimate cost of equity with CAPM** — state the formula, source each input (risk-free rate, beta, equity risk premium) correctly, and compute a required return on equity by hand and in code.
- **Estimate a beta from price data in Python** — pull a stock's and a benchmark's price history out of SQL, compute monthly returns in pandas, and regress one on the other to get beta, alpha, and R².
- **Distinguish levered from unlevered beta** — unlever a peer's beta to strip out its financing choice, average across peers, and relever at a target capital structure (the Hamada approach to a "pure-play" beta).
- **Compute after-tax cost of debt** — approximate a bond's yield to maturity from its price and coupon, blend multiple tranches by market value, and apply the tax shield correctly.
- **Combine into a full weighted average cost of capital** — use market-value (not book-value) weights for debt and equity and assemble `WACC = (E/V)×Re + (D/V)×Rd×(1−t)`.
- **Explain what moves each component of WACC** — rates, risk premium, beta, leverage, credit spread, and tax policy — and reason about which of those a company actually controls.

## Prerequisites

- **Week 3 of this course** (capital budgeting, NPV/IRR) — you should already be fluent with discounting a cash-flow stream; this week supplies the discount rate that feeds that machinery.
- **Week 1 of this course** — the PostgreSQL + pandas workspace (`crunch_capital` database, `psql`/`sqlite3`, a Python virtual environment with `pandas`, `numpy`, `sqlalchemy`) must already be set up. If it isn't, go back to [Week 1](../week-01-finance-foundations-and-tooling/) first.
- Comfort with basic algebra and reading a formula (this week leans on `y = a + bx`-style regression more than any week so far).
- Working SQL (`SELECT`, `WHERE`, `ORDER BY`, joins across two tables by a shared key) — Week 3 already had you join `cash_flows` against a second table; this week does the same shape of join between price history and reference tables.
- **No spreadsheets**, per the [course's data tooling rule](../../SYLLABUS.md#data-tooling-rule). Every price series, bond schedule, and capital-structure snapshot this week lives in SQL and gets modeled in pandas — never Excel.

## Set up this week's data (do this first)

You need the PostgreSQL 16+ (or SQLite 3.35+) database from Week 1, plus your Python environment with `pandas`, `numpy`, and `sqlalchemy` installed. This week adds three new tables to the same `crunch_capital` database: `stock_prices`, `debt_tranches`, and `capital_structure`.

```bash
psql crunch_capital              # PostgreSQL
# or
sqlite3 crunch_capital.db        # SQLite fallback
```

Paste this into your shell — it works unchanged on both engines:

```sql
-- 1. Five years of monthly closing prices for Crunch Machine Co. (ticker CRNM)
--    and its benchmark, the fictional "Crunch Market Composite" (ticker MKT).
CREATE TABLE stock_prices (
    price_id    INTEGER PRIMARY KEY,
    ticker      TEXT    NOT NULL,   -- 'CRNM' (the stock) or 'MKT' (the market benchmark)
    price_date  DATE    NOT NULL,
    adj_close   NUMERIC NOT NULL
);

INSERT INTO stock_prices (price_id, ticker, price_date, adj_close) VALUES
(1, 'CRNM', '2020-12-31', 42.00),
(2, 'MKT',  '2020-12-31', 4000.00),
(3, 'CRNM', '2021-01-31', 35.79),
(4, 'MKT',  '2021-01-31', 3827.13),
(5, 'CRNM', '2021-02-28', 36.30),
(6, 'MKT',  '2021-02-28', 3990.57),
(7, 'CRNM', '2021-03-31', 39.77),
(8, 'MKT',  '2021-03-31', 4229.02),
(9, 'CRNM', '2021-04-30', 43.27),
(10, 'MKT',  '2021-04-30', 4402.82),
(11, 'CRNM', '2021-05-31', 49.01),
(12, 'MKT',  '2021-05-31', 4724.92),
(13, 'CRNM', '2021-06-30', 51.89),
(14, 'MKT',  '2021-06-30', 4763.42),
(15, 'CRNM', '2021-07-31', 46.56),
(16, 'MKT',  '2021-07-31', 4496.50),
(17, 'CRNM', '2021-08-31', 45.84),
(18, 'MKT',  '2021-08-31', 4520.86),
(19, 'CRNM', '2021-09-30', 51.29),
(20, 'MKT',  '2021-09-30', 4655.21),
(21, 'CRNM', '2021-10-31', 52.79),
(22, 'MKT',  '2021-10-31', 4779.43),
(23, 'CRNM', '2021-11-30', 52.26),
(24, 'MKT',  '2021-11-30', 4688.45),
(25, 'CRNM', '2021-12-31', 43.85),
(26, 'MKT',  '2021-12-31', 4291.38),
(27, 'CRNM', '2022-01-31', 50.93),
(28, 'MKT',  '2022-01-31', 4730.26),
(29, 'CRNM', '2022-02-28', 48.28),
(30, 'MKT',  '2022-02-28', 4767.51),
(31, 'CRNM', '2022-03-31', 49.03),
(32, 'MKT',  '2022-03-31', 4905.55),
(33, 'CRNM', '2022-04-30', 47.00),
(34, 'MKT',  '2022-04-30', 4804.02),
(35, 'CRNM', '2022-05-31', 44.60),
(36, 'MKT',  '2022-05-31', 4664.35),
(37, 'CRNM', '2022-06-30', 48.12),
(38, 'MKT',  '2022-06-30', 4836.77),
(39, 'CRNM', '2022-07-31', 53.23),
(40, 'MKT',  '2022-07-31', 5082.57),
(41, 'CRNM', '2022-08-31', 53.11),
(42, 'MKT',  '2022-08-31', 4907.63),
(43, 'CRNM', '2022-09-30', 51.90),
(44, 'MKT',  '2022-09-30', 4822.70),
(45, 'CRNM', '2022-10-31', 48.53),
(46, 'MKT',  '2022-10-31', 4708.86),
(47, 'CRNM', '2022-11-30', 45.52),
(48, 'MKT',  '2022-11-30', 4515.56),
(49, 'CRNM', '2022-12-31', 42.58),
(50, 'MKT',  '2022-12-31', 4382.59),
(51, 'CRNM', '2023-01-31', 40.81),
(52, 'MKT',  '2023-01-31', 4393.98),
(53, 'CRNM', '2023-02-28', 40.21),
(54, 'MKT',  '2023-02-28', 4083.80),
(55, 'CRNM', '2023-03-31', 43.34),
(56, 'MKT',  '2023-03-31', 4510.50),
(57, 'CRNM', '2023-04-30', 41.47),
(58, 'MKT',  '2023-04-30', 4255.55),
(59, 'CRNM', '2023-05-31', 39.80),
(60, 'MKT',  '2023-05-31', 4214.18),
(61, 'CRNM', '2023-06-30', 40.85),
(62, 'MKT',  '2023-06-30', 4585.71),
(63, 'CRNM', '2023-07-31', 44.30),
(64, 'MKT',  '2023-07-31', 4535.34),
(65, 'CRNM', '2023-08-31', 43.84),
(66, 'MKT',  '2023-08-31', 4580.58),
(67, 'CRNM', '2023-09-30', 43.35),
(68, 'MKT',  '2023-09-30', 4745.78),
(69, 'CRNM', '2023-10-31', 53.18),
(70, 'MKT',  '2023-10-31', 5146.41),
(71, 'CRNM', '2023-11-30', 49.83),
(72, 'MKT',  '2023-11-30', 4979.08),
(73, 'CRNM', '2023-12-31', 55.08),
(74, 'MKT',  '2023-12-31', 5359.03),
(75, 'CRNM', '2024-01-31', 56.12),
(76, 'MKT',  '2024-01-31', 5133.51),
(77, 'CRNM', '2024-02-29', 60.17),
(78, 'MKT',  '2024-02-29', 5612.87),
(79, 'CRNM', '2024-03-31', 61.81),
(80, 'MKT',  '2024-03-31', 5946.31),
(81, 'CRNM', '2024-04-30', 66.57),
(82, 'MKT',  '2024-04-30', 6142.06),
(83, 'CRNM', '2024-05-31', 67.27),
(84, 'MKT',  '2024-05-31', 6106.35),
(85, 'CRNM', '2024-06-30', 71.41),
(86, 'MKT',  '2024-06-30', 6593.15),
(87, 'CRNM', '2024-07-31', 70.58),
(88, 'MKT',  '2024-07-31', 6922.26),
(89, 'CRNM', '2024-08-31', 71.68),
(90, 'MKT',  '2024-08-31', 7377.32),
(91, 'CRNM', '2024-09-30', 68.52),
(92, 'MKT',  '2024-09-30', 7006.51),
(93, 'CRNM', '2024-10-31', 73.70),
(94, 'MKT',  '2024-10-31', 7070.10),
(95, 'CRNM', '2024-11-30', 80.49),
(96, 'MKT',  '2024-11-30', 7235.80),
(97, 'CRNM', '2024-12-31', 86.10),
(98, 'MKT',  '2024-12-31', 7493.36),
(99, 'CRNM', '2025-01-31', 80.21),
(100, 'MKT',  '2025-01-31', 7002.53),
(101, 'CRNM', '2025-02-28', 64.19),
(102, 'MKT',  '2025-02-28', 6552.45),
(103, 'CRNM', '2025-03-31', 66.87),
(104, 'MKT',  '2025-03-31', 6762.47),
(105, 'CRNM', '2025-04-30', 64.27),
(106, 'MKT',  '2025-04-30', 6700.20),
(107, 'CRNM', '2025-05-31', 70.40),
(108, 'MKT',  '2025-05-31', 6828.07),
(109, 'CRNM', '2025-06-30', 65.68),
(110, 'MKT',  '2025-06-30', 6542.48),
(111, 'CRNM', '2025-07-31', 67.36),
(112, 'MKT',  '2025-07-31', 6633.24),
(113, 'CRNM', '2025-08-31', 65.10),
(114, 'MKT',  '2025-08-31', 6421.82),
(115, 'CRNM', '2025-09-30', 62.61),
(116, 'MKT',  '2025-09-30', 6131.71),
(117, 'CRNM', '2025-10-31', 60.65),
(118, 'MKT',  '2025-10-31', 5858.29),
(119, 'CRNM', '2025-11-30', 69.28),
(120, 'MKT',  '2025-11-30', 5995.88),
(121, 'CRNM', '2025-12-31', 68.35),
(122, 'MKT',  '2025-12-31', 5595.20);

-- 2. Crunch Machine Co.'s three outstanding debt tranches, as of 2025-12-31.
CREATE TABLE debt_tranches (
    tranche_id     INTEGER PRIMARY KEY,
    description    TEXT    NOT NULL,
    face_value     NUMERIC NOT NULL,   -- total par value outstanding, in dollars
    coupon_rate    NUMERIC NOT NULL,   -- annual coupon, as a decimal
    price_pct_par  NUMERIC NOT NULL,   -- market price per $100 of face value
    years_to_mat   NUMERIC NOT NULL,   -- years remaining to maturity
    rating         TEXT    NOT NULL
);

INSERT INTO debt_tranches VALUES
(1, 'Senior Notes due 2029', 150000000, 0.0525, 98.50, 3, 'BBB'),
(2, 'Term Loan B due 2031',  100000000, 0.0610, 99.00, 5, 'BBB-'),
(3, 'Green Bond due 2028',    75000000, 0.0475, 101.25, 2, 'A-');

-- 3. A one-row snapshot of everything else you need: shares outstanding,
--    the risk-free rate, the assumed equity risk premium, a vendor-supplied
--    beta, and the tax rate — all as of the same valuation date.
CREATE TABLE capital_structure (
    company             TEXT    PRIMARY KEY,
    valuation_date       DATE    NOT NULL,
    shares_outstanding   BIGINT  NOT NULL,
    risk_free_rate        NUMERIC NOT NULL,   -- 10-year Treasury proxy, as a decimal
    equity_risk_premium   NUMERIC NOT NULL,   -- assumed/supplied ERP, as a decimal
    vendor_beta           NUMERIC NOT NULL,   -- a market-data vendor's published beta
    tax_rate               NUMERIC NOT NULL
);

INSERT INTO capital_structure VALUES
('Crunch Machine Co.', '2025-12-31', 18000000, 0.0425, 0.0500, 1.30, 0.25);
```

Sanity checks — these should print `122`, `3`, and `1`:

```sql
SELECT COUNT(*) FROM stock_prices;
SELECT COUNT(*) FROM debt_tranches;
SELECT COUNT(*) FROM capital_structure;
```

`stock_prices` holds 61 month-end closes (December 2020 through December 2025 — 60 months of *returns*) for both `CRNM` and the `MKT` benchmark, so you can compute real monthly returns and regress one against the other. `debt_tranches` is small and denormalized on purpose, same philosophy as `cash_flows` in Week 1 — one flat table, no joins required, because three bonds don't need a schema. `capital_structure` is a single-row reference table: you'll join or cross-join it against the other two whenever a calculation needs the risk-free rate, the tax rate, or shares outstanding alongside price or bond data.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed the data; CAPM and cost of equity | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Estimating beta from real price data | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Levered vs. unlevered beta; pure-play method | 0h | 1h | 1h | 0.5h | 1h | 0h | 3.5h |
| Thursday | Cost of debt; assembling the full WACC | 2h | 1.5h | 1h | 0.5h | 1h | 1h | 7h |
| Friday | Sensitivity analysis; review | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (a real company's WACC) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5h** | **3h** | **3.5h** | **5h** | **5h** | **27.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-cost-of-equity-capm.md](./lecture-notes/01-cost-of-equity-capm.md) | CAPM, the risk-free rate, beta, the equity risk premium, a first cost-of-equity number | 2h |
| 2 | [lecture-notes/02-estimating-beta-from-data.md](./lecture-notes/02-estimating-beta-from-data.md) | Regressing returns in pandas; levered vs. unlevered beta; the pure-play method | 2h |
| 3 | [lecture-notes/03-cost-of-debt-and-wacc.md](./lecture-notes/03-cost-of-debt-and-wacc.md) | After-tax cost of debt, market-value weights, assembling the full WACC | 2h |
| 4 | [exercises/exercise-01-capm-cost-of-equity.md](./exercises/exercise-01-capm-cost-of-equity.md) | Compute cost of equity by hand and in SQL/Python for several scenarios | 1h |
| 5 | [exercises/exercise-02-regress-a-beta.md](./exercises/exercise-02-regress-a-beta.md) | Pull `stock_prices` into pandas and regress CRNM against MKT | 1.5h |
| 6 | [exercises/exercise-03-assemble-a-wacc.md](./exercises/exercise-03-assemble-a-wacc.md) | Compute cost of debt from `debt_tranches` and assemble Crunch Machine Co.'s WACC | 1.5h |
| 7 | [challenges/challenge-01-relever-beta-across-structures.md](./challenges/challenge-01-relever-beta-across-structures.md) | Unlever four comparable companies' betas and relever at Crunch Machine Co.'s structure | 1.5h |
| 8 | [challenges/challenge-02-wacc-sensitivity-grid.md](./challenges/challenge-02-wacc-sensitivity-grid.md) | Build a beta × leverage WACC sensitivity grid in pandas | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Compute a real company's WACC from real market data | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, free data sources, and further reading | — |

## By the end of this week you can…

- Take a risk-free rate, a beta, and an equity risk premium and produce a cost of equity — and explain what each input is actually measuring.
- Pull a stock's and a benchmark's price history out of SQL, compute returns in pandas, and run a real regression to estimate beta yourself.
- Strip a peer's beta down to its unlevered, business-risk-only form, and relever it at a different company's capital structure.
- Price a company's debt from its actual bonds, blend multiple tranches by market value, and apply the after-tax adjustment correctly.
- Assemble a full WACC using market-value weights — and produce three different, individually defensible WACCs for the same company, then argue for which one you'd actually use and why.

## Up next

Week 5 — Capital structure & leverage — once you can price the cost of capital, we turn to the harder question this week's three-beta exercise foreshadows: how much debt *should* a company carry, and how does that choice feed back into its own WACC?

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
