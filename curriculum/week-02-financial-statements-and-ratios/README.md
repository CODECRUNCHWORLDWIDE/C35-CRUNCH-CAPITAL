# Week 2 — Financial statements & ratio analysis

> **Goal:** by Sunday you can pull up a company's income statement, balance sheet, and cash-flow statement, load them into normalized SQL tables, prove the three statements tie together, and compute the liquidity, leverage, profitability, and efficiency ratios that tell you — from the numbers alone, before anyone says a word — whether a business is getting healthier or heading for trouble.

Welcome back to **C35 · Crunch Capital**. Last week you learned that a dollar today isn't worth the same as a dollar next year, and you built the discounting engine to prove it. This week you learn to *read* the documents every one of those cash flows eventually shows up in: the three financial statements every company on earth is legally required to produce. They look intimidating stacked in a 10-K filing, but they're really just three views of the same underlying story — what the company owns and owes (balance sheet), what it earned (income statement), and where the cash actually went (cash-flow statement) — and the story only makes sense once you can see how the three connect.

We work all week against one running case: **Crunch Machine Co.**, the fictional manufacturer from Week 1's capital-budgeting projects. This week you get its full company-level filings — income statement, balance sheet, and cash-flow statement — for **fiscal years 2021 through 2025**, loaded as three normalized SQL tables. It's a real five-year story, not a snapshot: watch the numbers as you go, because by Friday you'll be able to tell, from ratios alone, that this company is quietly sliding toward trouble.

## Learning objectives

By the end of this week, you will be able to:

- **Read** the income statement, balance sheet, and cash-flow statement — name what each one measures, what "as of a date" vs. "over a period" means, and where every major line item comes from.
- **Trace** how the three statements link: net income flows into retained earnings on the balance sheet; the cash-flow statement reconciles net income back to actual cash; ending cash on the cash-flow statement must equal cash on the balance sheet.
- **Load** real filings into normalized SQL tables instead of a wide, denormalized dump — and defend why normalization matters for financial data specifically.
- **Compute** the four ratio families — liquidity, leverage, profitability, and efficiency — in SQL, across multiple periods, not just for one snapshot.
- **Spot** financial trouble from ratio *trends* — a single ratio rarely tells you anything; five years of the same ratio moving in one direction tells you almost everything.

## Prerequisites

- Comfortable with `SELECT`, `WHERE`, `ORDER BY`, aggregates (`SUM`, `AVG`), and basic joins. If joins across two or three small tables aren't automatic yet, skim [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) Weeks 1–2 before continuing — this week joins `income_statement`, `balance_sheet`, and `cash_flow_statement` on `fiscal_year` constantly.
- Comfortable with window functions is a plus but not required — `LAG()` is introduced from scratch in Lecture 3 and Exercise 3, exactly where you need it for trend ratios.
- [Week 1 of this course](../week-01-finance-foundations-and-tooling/) — you should already have PostgreSQL (or SQLite) and Python + pandas set up, and be comfortable with present value / future value.
- **No spreadsheets.** Same [data tooling rule](../../SYLLABUS.md#data-tooling-rule) as every week: every statement, every ratio, every trend lives in SQL and Python. A ratio dashboard is a `SELECT`, not a workbook tab.

## Set up this week's filings (do this first)

You need the same PostgreSQL 16+ (or SQLite 3.35+) and Python 3.10+ / pandas environment from Week 1. If it's already running, skip straight to the schema.

**PostgreSQL:**

```bash
createdb crunch_capital        # reuse the same database from Week 1, or create it fresh
psql crunch_capital
```

**SQLite (fallback):**

```bash
sqlite3 crunch_capital.db
```

Now create the three normalized tables and load five years of Crunch Machine Co.'s filings. Paste this into your shell — it works unchanged on both engines:

```sql
CREATE TABLE income_statement (
    fiscal_year       INTEGER PRIMARY KEY,
    revenue           NUMERIC NOT NULL,
    cogs              NUMERIC NOT NULL,
    gross_profit      NUMERIC NOT NULL,
    sga_expense       NUMERIC NOT NULL,
    rd_expense        NUMERIC NOT NULL,
    operating_income  NUMERIC NOT NULL,
    interest_expense  NUMERIC NOT NULL,
    pretax_income     NUMERIC NOT NULL,
    tax_expense       NUMERIC NOT NULL,
    net_income        NUMERIC NOT NULL
);

CREATE TABLE balance_sheet (
    fiscal_year                  INTEGER PRIMARY KEY,
    cash                         NUMERIC NOT NULL,
    accounts_receivable          NUMERIC NOT NULL,
    inventory                    NUMERIC NOT NULL,
    other_current_assets         NUMERIC NOT NULL,
    total_current_assets         NUMERIC NOT NULL,
    ppe_net                      NUMERIC NOT NULL,
    other_lt_assets              NUMERIC NOT NULL,
    total_assets                 NUMERIC NOT NULL,
    accounts_payable             NUMERIC NOT NULL,
    short_term_debt               NUMERIC NOT NULL,
    other_current_liabilities    NUMERIC NOT NULL,
    total_current_liabilities    NUMERIC NOT NULL,
    long_term_debt                NUMERIC NOT NULL,
    other_lt_liabilities         NUMERIC NOT NULL,
    total_liabilities            NUMERIC NOT NULL,
    common_stock                 NUMERIC NOT NULL,
    retained_earnings            NUMERIC NOT NULL,
    total_equity                 NUMERIC NOT NULL
);

CREATE TABLE cash_flow_statement (
    fiscal_year                 INTEGER PRIMARY KEY,
    net_income                  NUMERIC NOT NULL,
    depreciation_amortization   NUMERIC NOT NULL,
    change_ar                   NUMERIC NOT NULL,   -- negative = AR grew (cash use)
    change_inventory            NUMERIC NOT NULL,   -- negative = inventory grew (cash use)
    change_ap                   NUMERIC NOT NULL,   -- positive = AP grew (cash source)
    change_other_wc             NUMERIC NOT NULL,
    cfo                         NUMERIC NOT NULL,   -- cash from operations
    capex                       NUMERIC NOT NULL,   -- negative = cash spent
    cfi                         NUMERIC NOT NULL,   -- cash from investing
    debt_issued_repaid_net      NUMERIC NOT NULL,   -- positive = net borrowing
    dividends_paid              NUMERIC NOT NULL,   -- negative = cash out
    cff                         NUMERIC NOT NULL,   -- cash from financing
    net_change_in_cash          NUMERIC NOT NULL,
    cash_beginning               NUMERIC NOT NULL,
    cash_ending                  NUMERIC NOT NULL
);

INSERT INTO income_statement VALUES
(2021, 42000, 26040, 15960, 7560, 1260, 7140,  620, 6520, 1630, 4890),
(2022, 45000, 28350, 16650, 8100, 1350, 7200,  690, 6510, 1628, 4882),
(2023, 46500, 30225, 16275, 8835, 1395, 6045,  810, 5235, 1309, 3926),
(2024, 46000, 31280, 14720, 9200, 1380, 4140, 1050, 3090,  773, 2317),
(2025, 44500, 31595, 12905, 9345, 1335, 2225, 1380,  845,  211,  634);

INSERT INTO balance_sheet VALUES
(2021, 7200, 4600, 6000, 800, 18600, 24600, 3000, 46200, 3400, 1800, 1750,  6950,  9500, 900, 17350, 5000, 23850, 28850),
(2022, 6100, 5100, 6800, 800, 18800, 25250, 3000, 47050, 3700, 2300, 1800,  7800, 10200, 900, 18900, 5000, 23150, 28150),
(2023, 4500, 5800, 7900, 800, 19000, 25350, 3000, 47350, 3900, 3000, 1820,  8720, 11300, 900, 20920, 5000, 21430, 26430),
(2024, 2800, 6700, 9200, 800, 19500, 24500, 3000, 47000, 4300, 4000, 1830, 10130, 12800, 900, 23830, 5000, 18170, 23170),
(2025, 1600, 7600,10600, 800, 20600, 22750, 3000, 46350, 4900, 5200, 1840, 11940, 14500, 900, 27340, 5000, 14010, 19010);

INSERT INTO cash_flow_statement VALUES
(2021, 4890, 2400, -400, -500,  200,   50, 6640, -3000, -3000,  800, -5240, -4440, -800, 8000, 7200),
(2022, 4882, 2550, -500, -800,  300,   50, 6482, -3200, -3200, 1200, -5582, -4382, -1100, 7200, 6100),
(2023, 3926, 2700, -700,-1100,  200,   20, 5046, -2800, -2800, 1800, -5646, -3846, -1600, 6100, 4500),
(2024, 2317, 2850, -900,-1300,  400,   10, 3377, -2000, -2000, 2500, -5577, -3077, -1700, 4500, 2800),
(2025,  634, 2950, -900,-1400,  600,   10, 1894, -1200, -1200, 2900, -4794, -1894, -1200, 2800, 1600);
```

Sanity check — this should print `5`, `5`, `5`:

```sql
SELECT COUNT(*) FROM income_statement;
SELECT COUNT(*) FROM balance_sheet;
SELECT COUNT(*) FROM cash_flow_statement;
```

And this should print `t` / `1` on every row — assets balance every single year, on both engines:

```sql
SELECT fiscal_year,
       total_assets = (total_liabilities + total_equity) AS balances
FROM balance_sheet
ORDER BY fiscal_year;
```

All figures are in **thousands of dollars**. Every table is keyed on `fiscal_year` alone because this week tracks one company — joins across the three tables are just `... JOIN ... ON a.fiscal_year = b.fiscal_year`, no messier than that. Don't skim past what the numbers say yet — Lecture 3 and the challenges spend real time making you read this exact five-year run and name what's going wrong.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|-----------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | The three statements, line by line | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | How the statements link; articulation | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Loading + reconciling filings in SQL | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Ratios that predict trouble | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Ratio dashboard + distress detection | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (real-company health read) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **4h** | **2h** | **3.5h** | **5h** | **5h** | — |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-the-three-statements.md](./lecture-notes/01-the-three-statements.md) | The income statement, balance sheet, and cash-flow statement — what each measures and how to read every major line | 2h |
| 2 | [lecture-notes/02-how-the-statements-link.md](./lecture-notes/02-how-the-statements-link.md) | Net income → retained earnings, the cash-flow reconciliation, why the model must tie out | 2h |
| 3 | [lecture-notes/03-ratios-that-predict-trouble.md](./lecture-notes/03-ratios-that-predict-trouble.md) | Liquidity, leverage, profitability, and efficiency ratios; reading trends over single points | 2h |
| 4 | [exercises/exercise-01-load-filings-into-sql.md](./exercises/exercise-01-load-filings-into-sql.md) | Parse a raw FY2020 filing and load it into the normalized tables yourself | 1h |
| 5 | [exercises/exercise-02-reconcile-the-statements.md](./exercises/exercise-02-reconcile-the-statements.md) | Prove, in SQL, that cash and net income tie across all three statements | 1h |
| 6 | [exercises/exercise-03-compute-core-ratios.md](./exercises/exercise-03-compute-core-ratios.md) | Compute the full liquidity/leverage/profitability/efficiency ratio set, year over year | 1.5h |
| 7 | [challenges/challenge-01-ratio-dashboard-query.md](./challenges/challenge-01-ratio-dashboard-query.md) | Ship one query that returns a full ratio dashboard, one row per year | 1h |
| 8 | [challenges/challenge-02-detect-distress-signals.md](./challenges/challenge-02-detect-distress-signals.md) | Write SQL that automatically flags the years a company is heading for trouble | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Load a real public company's filings and write a one-page ratio-driven health read | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, filing sources, and further reading | — |

## By the end of this week you can…

- Open any company's 10-K or annual report and know exactly what each statement is telling you and what it can't tell you alone.
- Load a set of filings into clean, normalized SQL tables instead of a flat spreadsheet dump.
- Prove, with a query, that a financial model actually ties out — instead of trusting that it does.
- Compute every major liquidity, leverage, profitability, and efficiency ratio from raw statement data, and read the *trend*, not just the number.
- Look at five years of one company's ratios and say, specifically, where the trouble started and what's driving it.

## Up next

Week 3 — Valuation & the cost of capital — once you can read what a company *is*, we teach you to price what it's *worth*.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
