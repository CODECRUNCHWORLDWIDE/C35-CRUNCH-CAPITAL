# Week 3 — Capital budgeting — NPV & IRR

> **Goal:** by Sunday you can take any project's stream of cash flows, compute its NPV, IRR, MIRR, payback, and profitability index in Python, rank a slate of mutually exclusive *and* independent projects correctly, and — most importantly — explain in plain language why NPV is the metric that never lies to you, while IRR sometimes does.

Welcome back to **C35 · Crunch Capital**. Week 1 gave you the time-value-of-money engine — discount and compound any cash flow. Week 2 gave you the three financial statements and the ratios that read a company's health from the outside. This week you point that engine at the question every capital allocator actually has to answer, over and over, for the rest of their career: **should we spend this money on this project, or not — and if we can only fund some of our good ideas, which ones?**

That question sounds simple. It isn't, because there are half a dozen popular metrics that claim to answer it — Net Present Value (NPV), Internal Rate of Return (IRR), Modified IRR (MIRR), payback period, and the Profitability Index (PI) — and they don't always agree. Sometimes they *can't* agree, because some of them are built on assumptions that quietly stop making sense once cash flows swing sign more than once, or once you compare projects of different size. This week's job is to make you fluent in all five metrics, and — critically — to make you the person in the room who knows *which one to trust* when they disagree.

We keep working against the same `cash_flows` table you seeded in Week 1: six real-looking capital projects at fictional manufacturer **Crunch Machine Co.**, each with its own stream of outflows and inflows and its own hurdle rate. If you still have that table, you're ready. If not, the setup below rebuilds it from scratch in one paste.

## Learning objectives

By the end of this week, you will be able to:

- **Compute NPV** for a project's full cash-flow stream in Python (and in SQL), and state the accept/reject decision rule in one sentence.
- **Compute IRR, MIRR, payback period (simple and discounted), and profitability index (PI)** for any project, in Python.
- **Rank** a slate of both *mutually exclusive* projects (pick one) and *independent* projects (pick any subset, under a budget) using the correct metric for each situation.
- **Explain, with a constructed example, when IRR misleads** — the scale problem, the timing/crossover problem, and the multiple-IRR problem — and why NPV does not have these failure modes.
- **Set and defend a hurdle rate** for a project, and show how sensitive the accept/reject decision is to that choice.

## Prerequisites

- Week 1 (time value of money — PV, FV, discounting a stream) and Week 2 (financial statements) of this course, or equivalent.
- Working SQL (`SELECT`, `WHERE`, `GROUP BY`, aggregates) and beginner Python (functions, loops, lists).
- Python packages from Week 1: `pandas`, `numpy`. This week adds **`numpy-financial`** (NumPy dropped its old `irr`/`npv` functions years ago; the finance functions now live in this small companion package) and, optionally, `scipy` for root-finding.
- **No spreadsheets.** Every cash-flow stream, every ranking table, and every sensitivity sweep lives in SQL and Python this week — see the [data tooling rule](../../SYLLABUS.md#data-tooling-rule). A ranking of six projects by NPV is a `GROUP BY` and an `ORDER BY`, not a pivot table.

## Set up the workspace (do this first)

If you completed Week 1 and still have your `crunch_capital` database, skip straight to the pip install below — the table is already there. Otherwise, rebuild it:

**PostgreSQL:**

```bash
createdb crunch_capital        # skip if it already exists
psql crunch_capital
```

**SQLite (fallback):**

```bash
sqlite3 crunch_capital.db
```

**Python — add this week's package:**

```bash
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas numpy numpy-financial scipy sqlalchemy psycopg[binary]
```

Then paste this into your `psql` (or `sqlite3`) shell — it is the exact `cash_flows` table from Week 1, unchanged, so your Week 1 answers still apply:

```sql
CREATE TABLE cash_flows (
    cash_flow_id   INTEGER PRIMARY KEY,
    project_id     INTEGER NOT NULL,
    project_name   TEXT    NOT NULL,
    category       TEXT    NOT NULL,
    sponsor        TEXT    NOT NULL,
    risk_tier      TEXT    NOT NULL,
    hurdle_rate    NUMERIC NOT NULL,
    period         INTEGER NOT NULL,
    flow_date      DATE    NOT NULL,
    amount         NUMERIC NOT NULL,   -- negative = outflow, positive = inflow
    flow_type      TEXT    NOT NULL,
    note           TEXT
);

INSERT INTO cash_flows VALUES
(1, 1,'New CNC Machine','Equipment','Operations','Low',0.07,0,'2026-01-01',-180000,'Capex','Includes installation and training'),
(2, 1,'New CNC Machine','Equipment','Operations','Low',0.07,1,'2027-01-01', 55000,'Revenue',NULL),
(3, 1,'New CNC Machine','Equipment','Operations','Low',0.07,2,'2028-01-01', 58000,'Revenue',NULL),
(4, 1,'New CNC Machine','Equipment','Operations','Low',0.07,3,'2029-01-01', 60000,'Revenue',NULL),
(5, 1,'New CNC Machine','Equipment','Operations','Low',0.07,4,'2030-01-01', 60000,'Revenue',NULL),
(6, 1,'New CNC Machine','Equipment','Operations','Low',0.07,5,'2031-01-01', 62000,'Revenue',NULL),
(7, 1,'New CNC Machine','Equipment','Operations','Low',0.07,5,'2031-01-01', 15000,'Salvage','Resale value at end of life'),
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

Six projects, six hurdle rates already baked into the table (7%, 10%, 15%, 10%, 6%, 18% — riskier projects get a higher required return, which you'll unpack properly in Week 4's cost-of-capital lectures). This week you treat `hurdle_rate` as given and correct; you'll learn *where it comes from* in Week 4.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | NPV mechanics + the decision rule | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | IRR, and where it starts to lie | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | MIRR, multiple roots, scale/timing traps | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Payback, PI, ranking under a budget | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Reconciling metrics; sensitivity | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (rank the project slate) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-npv-the-decision-rule.md](./lecture-notes/01-npv-the-decision-rule.md) | Discounting a project's stream to one number; the hurdle rate's role; the accept/reject rule | 2h |
| 2 | [lecture-notes/02-irr-and-its-traps.md](./lecture-notes/02-irr-and-its-traps.md) | IRR, MIRR, multiple roots, scale and timing problems — why IRR can rank projects wrongly | 2h |
| 3 | [lecture-notes/03-payback-pi-and-choosing.md](./lecture-notes/03-payback-pi-and-choosing.md) | Payback, discounted payback, profitability index, ranking under a capital budget | 2h |
| 4 | [exercises/exercise-01-npv-in-python.md](./exercises/exercise-01-npv-in-python.md) | Write an NPV function and run it over the whole `cash_flows` table | 1h |
| 5 | [exercises/exercise-02-irr-and-mirr.md](./exercises/exercise-02-irr-and-mirr.md) | Compute IRR and MIRR side by side for every project | 1h |
| 6 | [exercises/exercise-03-rank-under-a-budget.md](./exercises/exercise-03-rank-under-a-budget.md) | Rank projects by NPV, IRR, and PI under a fixed capital budget | 1h |
| 7 | [challenges/challenge-01-irr-lies-case.md](./challenges/challenge-01-irr-lies-case.md) | Construct a case where IRR picks the wrong project and prove it with NPV | 1.5h |
| 8 | [challenges/challenge-02-sensitivity-on-hurdle-rate.md](./challenges/challenge-02-sensitivity-on-hurdle-rate.md) | Sweep the hurdle rate and find where the accept/reject decision flips | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Rank the full project slate, cross-check every metric, defend a capital-allocation cutoff | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, papers, and further reading | — |

## By the end of this week you can…

- Take any project's cash-flow stream and return a single, defensible NPV — by hand for a sanity check, and in Python for the real work.
- Compute IRR, MIRR, payback, and PI for a project, and know exactly what each one is really measuring.
- Point to a specific, constructed example where IRR ranks two projects in the wrong order, and explain *why* in one sentence: scale, timing, or multiple roots.
- Rank a slate of projects under a limited capital budget using the profitability index, not gut feel.
- Justify a hurdle rate, and show how the accept/reject line moves if that rate is wrong by a percentage point or two.

## Up next

[Week 4 — Cost of capital — CAPM, cost of debt, WACC](../week-04-cost-of-capital-capm-wacc/) — now that you can rank projects at a *given* hurdle rate, Week 4 shows you where that rate actually comes from.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
