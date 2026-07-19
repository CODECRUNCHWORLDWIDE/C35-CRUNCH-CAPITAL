# Week 6 — Equity & debt financing

> **Goal:** by Sunday you can take a real financing need, price it two ways — a follow-on equity raise and a term loan — model the ownership dilution of the first and the full coupon/amortization/covenant structure of the second in SQL and Python, compute the true all-in cost of each, and argue which one a CFO should actually pick and why.

Welcome back to **C35 · Crunch Capital**. Week 4 taught you where a company's cost of capital comes from. Week 5 taught you how the mix of debt and equity on the balance sheet changes that cost, through the tax shield and the rising odds of distress. This week you stop reasoning about capital structure in the abstract and do the thing a CFO actually does when the company needs cash: **raise it**. Every raise is a choice between two families of instrument — sell a slice of the company (**equity**) or promise to pay it back with interest (**debt**) — and the two are not interchangeable. They dilute differently, they cost differently, they signal differently to the market, and they hand differently shaped guns to whoever's on the other side of the table.

We follow one company through the whole week: **Crunch Machine Co.**, the manufacturer you've been modeling since Week 1, needs **$20,000,000** to build a new distribution hub in the Midwest. The board has narrowed it to two term sheets — a follow-on equity offering, and a five-year secured term loan — and you are the analyst who has to model both, all the way down to real numbers, before the board votes. That's this week's mini-project. Everything before it — three lectures, three exercises, two challenges — builds the pieces you need to get there.

## Learning objectives

By the end of this week, you will be able to:

- **Compare** raising equity versus debt for a given financing need, and name the structural tradeoffs (cost, control, dilution, signaling, flexibility) that decide between them.
- **Model a share issuance** — pricing, underwriting fees, and rounding to whole shares — and compute the exact ownership dilution it creates for every existing shareholder, in SQL.
- **Model a bond or term loan** — coupon, amortization schedule, seniority — and build the covenant tests (leverage, interest coverage) a lender attaches to it, in Python and SQL.
- **Compute the true cost of each financing path** — not just the quoted rate or the headline share price, but the all-in cost after fees, dilution, and taxes.
- **Reason about signaling and control** — why the market reads an equity issuance and a debt issuance differently, and why founders and lenders each want different things from the same balance sheet.

## Prerequisites

- Week 4 (cost of capital — CAPM, cost of debt, WACC) and Week 5 (capital structure — leverage, tax shield, distress) of this course, or equivalent. This week assumes you already know *why* debt is usually cheaper than equity (the tax shield) and *why* a company can't just load up on debt forever (distress costs) — we don't re-derive either here, we put them to work.
- Working SQL (`SELECT`, `WHERE`, `GROUP BY`, aggregates, basic arithmetic in a query) and Python with `pandas`/`numpy` — the same stack from every prior week.
- **No spreadsheets.** Every cap table, every amortization schedule, and every covenant test lives in SQL and Python this week, per the [data tooling rule](../../SYLLABUS.md#data-tooling-rule). A cap table with seven rows is a `SELECT`, not a workbook tab.

## Set up the workspace (do this first)

If your `crunch_capital` database from prior weeks is still around, you're adding three new tables to it. If you're starting fresh:

**PostgreSQL:**

```bash
createdb crunch_capital        # skip if it already exists
psql crunch_capital
```

**SQLite (fallback):**

```bash
sqlite3 crunch_capital.db
```

**Python — this week needs nothing new beyond prior weeks:**

```bash
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas numpy numpy-financial sqlalchemy psycopg[binary]
```

Now paste this into your `psql` (or `sqlite3`) shell. It creates the three tables this week is built on: a snapshot of Crunch Machine Co.'s current capital structure, its cap table (who owns what, today), and the terms of the two financing options the board is weighing.

```sql
CREATE TABLE company_snapshot (
    company_name        TEXT    PRIMARY KEY,
    shares_outstanding   INTEGER NOT NULL,
    share_price          NUMERIC NOT NULL,   -- current market price per share
    ebitda               NUMERIC NOT NULL,   -- trailing twelve months
    depreciation_amort   NUMERIC NOT NULL,   -- D&A, trailing twelve months
    existing_debt         NUMERIC NOT NULL,
    existing_debt_rate    NUMERIC NOT NULL,  -- weighted-average rate on existing debt
    tax_rate              NUMERIC NOT NULL,
    cash_on_hand           NUMERIC NOT NULL,
    financing_need          NUMERIC NOT NULL  -- net proceeds required for the Midwest hub
);

INSERT INTO company_snapshot VALUES
('Crunch Machine Co.', 10000000, 20.00, 22000000, 4000000, 30000000, 0.06, 0.25, 3000000, 20000000);

CREATE TABLE shareholders (
    shareholder_id     INTEGER PRIMARY KEY,
    shareholder_name   TEXT    NOT NULL,
    shareholder_type   TEXT    NOT NULL,   -- Founder, Employee-Pool, VC, Institutional, Public-Float
    shares_held        INTEGER NOT NULL    -- pre-raise holding
);

INSERT INTO shareholders VALUES
(1, 'Founder & CEO',              'Founder',          3000000),
(2, 'Founder & CTO',              'Founder',          1500000),
(3, 'Employee Option Pool',       'Employee-Pool',    1000000),
(4, 'Crunch Ventures Fund II',    'VC',               1800000),
(5, 'Meridian Growth Partners',   'VC',               1200000),
(6, 'Public Float (existing)',    'Public-Float',     1500000);

CREATE TABLE financing_terms (
    option_id           INTEGER PRIMARY KEY,
    option_name          TEXT    NOT NULL,   -- 'Equity — Follow-On Offering' / 'Debt — Term Loan'
    instrument_type       TEXT    NOT NULL,   -- 'Equity' / 'Debt'
    net_proceeds_target    NUMERIC NOT NULL,
    price_or_rate           NUMERIC NOT NULL,  -- offer price per share, OR annual coupon rate
    fee_pct                  NUMERIC NOT NULL,  -- underwriting spread, OR loan origination fee
    term_years                INTEGER,          -- NULL for equity (no maturity)
    notes                      TEXT
);

INSERT INTO financing_terms VALUES
(1, 'Equity — Follow-On Offering', 'Equity', 20000000, 19.00, 0.05, NULL,
 '5% new-issue discount to the $20.00 market price; 5% underwriting spread on gross proceeds'),
(2, 'Debt — Term Loan',            'Debt',   20000000, 0.075, 0.01, 5,
 '5-year fully amortizing term loan, annual payments, 1% origination fee, secured, leverage and coverage covenants attached');
```

Sanity check — both should return `1`:

```sql
SELECT COUNT(*) FROM company_snapshot;
SELECT COUNT(*) FROM financing_terms;
```

And this should return `6` (the pre-raise cap table) with `shares_held` summing to `10000000` — matching `company_snapshot.shares_outstanding`:

```sql
SELECT COUNT(*), SUM(shares_held) FROM shareholders;
```

Everything in this week — every lecture, exercise, challenge, and the mini-project — reads from these three tables and, where you compute new results (a dilution table, a loan schedule), writes them back into new tables of your own so they stay queryable, not stranded in a notebook.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Equity financing + dilution mechanics | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Debt instruments, amortization, covenants | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Covenant math; true cost of each path | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Choosing the financing mix; control/signaling | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Covenant breach + mixed-raise challenges | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (model the raise two ways) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-equity-financing-and-dilution.md](./lecture-notes/01-equity-financing-and-dilution.md) | Pricing a follow-on raise, underwriting fees, and exact per-shareholder dilution | 2h |
| 2 | [lecture-notes/02-debt-instruments-and-covenants.md](./lecture-notes/02-debt-instruments-and-covenants.md) | Coupons, amortization, seniority, and the covenants lenders attach to protect themselves | 2h |
| 3 | [lecture-notes/03-choosing-the-financing-mix.md](./lecture-notes/03-choosing-the-financing-mix.md) | True cost of each path, control and signaling tradeoffs, and how real companies mix the two | 2h |
| 4 | [exercises/exercise-01-model-a-share-issuance.md](./exercises/exercise-01-model-a-share-issuance.md) | Price the follow-on offering and compute every shareholder's new ownership stake | 1h |
| 5 | [exercises/exercise-02-build-a-bond-schedule.md](./exercises/exercise-02-build-a-bond-schedule.md) | Build and store the term loan's full amortization schedule | 1h |
| 6 | [exercises/exercise-03-cost-of-each-path.md](./exercises/exercise-03-cost-of-each-path.md) | Compute the true, all-in cost of the equity raise and the term loan, apples to apples | 1h |
| 7 | [challenges/challenge-01-covenant-breach-simulation.md](./challenges/challenge-01-covenant-breach-simulation.md) | Simulate a recession scenario and find exactly where the loan's covenants break | 1.5h |
| 8 | [challenges/challenge-02-mixed-financing-optimization.md](./challenges/challenge-02-mixed-financing-optimization.md) | Optimize a mixed equity/debt raise instead of an all-or-nothing choice | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Model the $20M raise both ways and hand the board a real recommendation | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, filings, and further reading | — |

## By the end of this week you can…

- Price a follow-on equity offering — discount, underwriting fee, whole-share rounding — and compute the exact percentage-point dilution it costs every named shareholder, not just a headline "you'll be diluted" number.
- Build a complete loan or bond amortization schedule from the annuity formula, store it in SQL, and answer real questions against it (how much interest this year, what's still owed after year 3).
- Write the two covenant tests — leverage and interest coverage — that turn a set of loan terms into a pass/fail check against a company's actual financials, and know how much financial cushion the company has under each.
- Put a genuine number (not a vibe) on the true cost of financing through equity versus through debt, for the same dollar raised.
- Explain, with Crunch Machine Co.'s actual numbers, why a CFO doesn't just pick "whichever is cheaper" — and what control, signaling, and flexibility a cheaper option can quietly cost you.

## Up next

[Week 7 — Valuation: DCF and trading comparables](../week-07-valuation-dcf-and-comps/) — now that you can price how a company raises money, Week 7 shows you how the market prices the company itself.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
