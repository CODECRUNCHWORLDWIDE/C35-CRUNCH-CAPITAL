# Week 1 — Finance foundations & data tooling

> **Goal:** by Sunday you can explain what corporate finance is actually optimizing, discount and compound any cash flow by hand and in code, and query a cash-flow stream from a real database instead of scrolling a spreadsheet — writing the SQL and Python from memory, on both PostgreSQL and SQLite.

Welcome to **C35 · Crunch Capital**. Corporate finance has a reputation for being either bone-dry theory or Excel wizardry. It's neither. Underneath every acquisition, every loan, every "should we build the warehouse" decision is the same handful of ideas: cash moves through time, time has a price, and every stream of money carries risk. Master those three ideas and a discounted cash flow model, a bond price, and a startup valuation all turn out to be the *same calculation* wearing different clothes.

This week you build the workbench you'll use for the next eleven: PostgreSQL for storage, Python (pandas) for modeling, and the time-value-of-money math that everything else in this course compounds on top of (pun intended). We work against one seed table, `cash_flows` — six real-looking capital projects at a fictional manufacturer, **Crunch Machine Co.**, each with its own stream of outflows and inflows. You set it up once (below), then every lecture, exercise, challenge, and the mini-project queries it.

## Learning objectives

By the end of this week, you will be able to:

- **Explain** what corporate finance optimizes — cash, time, and risk — and use that lens to evaluate any financial decision, not just the ones in a textbook.
- **State and derive** the present value and future value formulas, and explain *why* a dollar today is worth more than a dollar next year (it isn't sentiment — it's opportunity cost and risk).
- **Discount and compound** a single cash flow, and a stream of cash flows, by hand, in raw SQL, and in Python.
- **Set up** PostgreSQL 16 and a Python (pandas) environment as your finance workspace, and connect the two.
- **Store and query** a cash-flow stream as a relational table — filtering, sorting, and aggregating it exactly like any other business dataset.
- **Argue**, with specifics, why a versioned, queryable data pipeline beats a spreadsheet for financial modeling — and say precisely what a spreadsheet is still fine for.

## Prerequisites

- You can run commands in a terminal (open one, type a command, read the output).
- Comfort with basic algebra (rearranging `A = B × C` for `B`) — that's the heaviest math this week.
- Working SQL (`SELECT`, `WHERE`, `ORDER BY`, aggregates like `SUM`/`AVG`) is assumed. If `SELECT * FROM t WHERE x > 5 ORDER BY x;` doesn't feel automatic, do [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) Week 1 first — this course does not re-teach basic `SELECT`.
- Beginner Python (variables, functions, `import`) is assumed. No prior finance, accounting, or pandas knowledge required — this course goes from zero to expert on those.
- **No spreadsheets.** This course's [data tooling rule](../../SYLLABUS.md#data-tooling-rule): every cash flow, statement, price series, and model lives in SQL and Python, never Excel. If a task feels like "this would be a spreadsheet," that's the signal to reach for SQL or pandas instead.

## Set up the workspace (do this first)

You need PostgreSQL 16+ (or SQLite 3.35+ as a fallback) and Python 3.10+ with `pandas`. Full install steps are in [`resources.md`](./resources.md). Once installed:

**PostgreSQL:**

```bash
createdb crunch_capital        # make a database named "crunch_capital"
psql crunch_capital            # open a session against it
```

**SQLite (fallback):**

```bash
sqlite3 crunch_capital.db      # creates the file on first write
```

**Python:**

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas numpy sqlalchemy psycopg[binary]
```

Now create and seed the week's table. Paste this into your `psql` (or `sqlite3`) shell — it works unchanged on both engines:

```sql
CREATE TABLE cash_flows (
    cash_flow_id   INTEGER PRIMARY KEY,
    project_id     INTEGER NOT NULL,
    project_name   TEXT    NOT NULL,
    category       TEXT    NOT NULL,   -- Equipment, Expansion, R&D, Marketing, Sustainability
    sponsor        TEXT    NOT NULL,   -- department that owns the project
    risk_tier      TEXT    NOT NULL,   -- Low, Medium, High
    hurdle_rate    NUMERIC NOT NULL,   -- the project's required annual discount rate
    period         INTEGER NOT NULL,   -- 0 = today, 1..5 = end of each following year
    flow_date      DATE    NOT NULL,
    amount         NUMERIC NOT NULL,   -- negative = outflow (cash out), positive = inflow (cash in)
    flow_type      TEXT    NOT NULL,   -- Capex, Opex, Revenue, Salvage
    note           TEXT                -- nullable on purpose
);

INSERT INTO cash_flows VALUES
(1, 1,'New CNC Machine','Equipment','Operations','Low',0.07,0,'2026-01-01',-180000,'Capex','Includes installation and training'),
(2, 1,'New CNC Machine','Equipment','Operations','Low',0.07,1,'2027-01-01', 55000,'Revenue',NULL),
(3, 1,'New CNC Machine','Equipment','Operations','Low',0.07,2,'2028-01-01', 58000,'Revenue',NULL),
(4, 1,'New CNC Machine','Equipment','Operations','Low',0.07,3,'2029-01-01', 60000,'Revenue',NULL),
(5, 1,'New CNC Machine','Equipment','Operations','Low',0.07,4,'2030-01-01', 60000,'Revenue',NULL),
(6, 1,'New CNC Machine','Equipment','Operations','Low',0.07,5,'2031-01-01', 62000,'Revenue',NULL),
(7, 1,'New CNC Machine','Equipment','Operations','Low',0.07,5,'2031-01-01', 15000,'Salvage','Resale value of the old machine''s replacement at end of life'),
(8, 2,'Regional Warehouse Expansion','Expansion','Operations','Medium',0.10,0,'2026-01-01',-650000,'Capex',NULL),
(9, 2,'Regional Warehouse Expansion','Expansion','Operations','Medium',0.10,1,'2027-01-01', 120000,'Revenue',NULL),
(10,2,'Regional Warehouse Expansion','Expansion','Operations','Medium',0.10,2,'2028-01-01', 150000,'Revenue',NULL),
(11,2,'Regional Warehouse Expansion','Expansion','Operations','Medium',0.10,3,'2029-01-01', 175000,'Revenue',NULL),
(12,2,'Regional Warehouse Expansion','Expansion','Operations','Medium',0.10,4,'2030-01-01', 190000,'Revenue',NULL),
(13,2,'Regional Warehouse Expansion','Expansion','Operations','Medium',0.10,5,'2031-01-01', 210000,'Revenue','Assumes lease renewal at current rate'),
(14,3,'AI Quality-Control R&D','R&D','Engineering','High',0.15,0,'2026-01-01',-300000,'Capex',NULL),
(15,3,'AI Quality-Control R&D','R&D','Engineering','High',0.15,1,'2027-01-01', -50000,'Opex','Extended model tuning before rollout'),
(16,3,'AI Quality-Control R&D','R&D','Engineering','High',0.15,2,'2028-01-01', 90000,'Revenue',NULL),
(17,3,'AI Quality-Control R&D','R&D','Engineering','High',0.15,3,'2029-01-01', 140000,'Revenue',NULL),
(18,3,'AI Quality-Control R&D','R&D','Engineering','High',0.15,4,'2030-01-01', 160000,'Revenue',NULL),
(19,3,'AI Quality-Control R&D','R&D','Engineering','High',0.15,5,'2031-01-01', 150000,'Revenue','Competing in-house tool expected to erode gains'),
(20,4,'Brand Relaunch Campaign','Marketing','Marketing','Medium',0.10,0,'2026-01-01',-220000,'Capex',NULL),
(21,4,'Brand Relaunch Campaign','Marketing','Marketing','Medium',0.10,1,'2027-01-01', 80000,'Revenue',NULL),
(22,4,'Brand Relaunch Campaign','Marketing','Marketing','Medium',0.10,2,'2028-01-01', 95000,'Revenue',NULL),
(23,4,'Brand Relaunch Campaign','Marketing','Marketing','Medium',0.10,3,'2029-01-01', 70000,'Revenue',NULL),
(24,4,'Brand Relaunch Campaign','Marketing','Marketing','Medium',0.10,4,'2030-01-01', 40000,'Revenue','Campaign lift fades without reinvestment'),
(25,4,'Brand Relaunch Campaign','Marketing','Marketing','Medium',0.10,5,'2031-01-01', 20000,'Revenue',NULL),
(26,5,'Solar Rooftop Retrofit','Sustainability','Facilities','Low',0.06,0,'2026-01-01',-140000,'Capex','Net of the state clean-energy rebate'),
(27,5,'Solar Rooftop Retrofit','Sustainability','Facilities','Low',0.06,1,'2027-01-01', 28000,'Revenue',NULL),
(28,5,'Solar Rooftop Retrofit','Sustainability','Facilities','Low',0.06,2,'2028-01-01', 28000,'Revenue',NULL),
(29,5,'Solar Rooftop Retrofit','Sustainability','Facilities','Low',0.06,3,'2029-01-01', 28000,'Revenue',NULL),
(30,5,'Solar Rooftop Retrofit','Sustainability','Facilities','Low',0.06,4,'2030-01-01', 28000,'Revenue',NULL),
(31,5,'Solar Rooftop Retrofit','Sustainability','Facilities','Low',0.06,5,'2031-01-01', 28000,'Revenue',NULL),
(32,5,'Solar Rooftop Retrofit','Sustainability','Facilities','Low',0.06,5,'2031-01-01', 10000,'Salvage','Panels retain resale value at year 5'),
(33,6,'European Market Entry','Expansion','Strategy','High',0.18,0,'2026-01-01',-900000,'Capex',NULL),
(34,6,'European Market Entry','Expansion','Strategy','High',0.18,1,'2027-01-01', -100000,'Opex','Regulatory and localization spend before revenue starts'),
(35,6,'European Market Entry','Expansion','Strategy','High',0.18,2,'2028-01-01', 150000,'Revenue',NULL),
(36,6,'European Market Entry','Expansion','Strategy','High',0.18,3,'2029-01-01', 300000,'Revenue',NULL),
(37,6,'European Market Entry','Expansion','Strategy','High',0.18,4,'2030-01-01', 420000,'Revenue',NULL),
(38,6,'European Market Entry','Expansion','Strategy','High',0.18,5,'2031-01-01', 500000,'Revenue','Assumes no new tariff action');
```

Sanity check — this should print `38`:

```sql
SELECT COUNT(*) FROM cash_flows;
```

The table is **denormalized on purpose** — `project_name`, `category`, `hurdle_rate`, and friends repeat on every row of the same project instead of living in a separate `projects` table. That's a deliberate simplification for Week 1 so you can do everything with a single `SELECT`, no joins (joins arrive in later courses' multi-table weeks; this course stays single-table by design, since capital budgeting rarely needs more than one flat cash-flow ledger). `amount` is signed: **negative = cash out** (capex, opex), **positive = cash in** (revenue, salvage). `note` is `NULL` on most rows and filled in only where there's real context — you'll use that for `IS NULL` practice.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Install stack; what finance optimizes | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Time value of money — PV, FV, discounting | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Compounding frequency; SQL over cash flows | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Postgres + pandas pipeline; querying the table | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Amortization + spreadsheet-vs-pipeline | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (discounting engine) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-what-corporate-finance-optimizes.md](./lecture-notes/01-what-corporate-finance-optimizes.md) | Cash, time, and risk — the operator's mental model | 2h |
| 2 | [lecture-notes/02-the-time-value-of-money.md](./lecture-notes/02-the-time-value-of-money.md) | PV, FV, discounting, compounding — the core formula, derived | 2h |
| 3 | [lecture-notes/03-sql-python-not-spreadsheets.md](./lecture-notes/03-sql-python-not-spreadsheets.md) | Setting up Postgres + pandas; why a pipeline beats a workbook | 2h |
| 4 | [exercises/exercise-01-discount-a-single-cash-flow.md](./exercises/exercise-01-discount-a-single-cash-flow.md) | Discount one cash flow by hand, in SQL, and in Python | 1h |
| 5 | [exercises/exercise-02-build-a-cash-flow-table.md](./exercises/exercise-02-build-a-cash-flow-table.md) | Query and extend the `cash_flows` table in SQL | 1h |
| 6 | [exercises/exercise-03-pv-fv-in-pandas.md](./exercises/exercise-03-pv-fv-in-pandas.md) | Write reusable PV/FV functions in pandas, over data pulled from SQL | 1h |
| 7 | [challenges/challenge-01-amortization-schedule-engine.md](./challenges/challenge-01-amortization-schedule-engine.md) | Build a full loan amortization schedule engine | 1.5h |
| 8 | [challenges/challenge-02-spreadsheet-vs-pipeline-writeup.md](./challenges/challenge-02-spreadsheet-vs-pipeline-writeup.md) | Defend the SQL+Python pipeline over a workbook, with specifics | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Build a discounting engine over the `cash_flows` table | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, install guides, and further reading | — |

## By the end of this week you can…

- Look at any business decision and name what it's really trading off: cash now vs. cash later vs. how sure you are of getting it.
- Discount or compound any single cash flow, or a whole stream, correctly — by hand, in SQL, and in pandas — and get the same answer three ways.
- Stand up a Postgres + Python workspace from nothing and load a real dataset into it.
- Query a cash-flow table the way an analyst does: filter by project, sum by category, sort by size, and spot a `NULL` before it corrupts a total.
- Explain, with a concrete example, why the model you build this week could never safely live in a spreadsheet.

## Up next

[Week 2 — Financial statements & ratio analysis](../week-02-financial-statements-and-ratios/) — once you can move a dollar through time, we teach you to read the three statements that report where the dollars actually went.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
