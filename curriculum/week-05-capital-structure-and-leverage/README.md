# Week 5 — Capital structure & leverage

> **Goal:** by Sunday you can take any company's EBIT, tax rate, and debt load and answer the question every CFO eventually has to answer with a number, not a hunch: **how much debt should this company actually carry?** You'll value the interest tax shield, price the expected cost of financial distress, find the value-maximizing debt level under a tradeoff model, and quantify exactly how leverage amplifies both the upside and the downside of equity returns.

Welcome back to **C35 · Crunch Capital**. Week 3 taught you to accept or reject a single project. Week 4 taught you the required return to discount it at. This week asks a different question, one level up: forget any single project — **how should the whole company be financed?** More debt is cheaper than equity and shields taxable income, so why doesn't every company borrow to the hilt? And if some companies clearly *are* over-levered, how would you know, and what would "the right amount" even look like as a number?

We keep working the same running case you've followed since Week 2: **Crunch Machine Co.**, the fictional manufacturer, and its `income_statement` and `balance_sheet` tables for fiscal years 2021–2025. You already know this company is quietly sliding toward trouble — Week 2's ratio work showed margins compressing and leverage climbing. This week you get the theory that explains *why* that trajectory is dangerous, and the tools to say, with a number, how far past the value-maximizing point Crunch Machine Co. has actually drifted. Spoiler: by the end of Lecture 2, you'll have hard evidence it's over-levered by roughly 3x the model's optimum — and you'll know exactly how to compute that ratio for any company, not just this one.

## Learning objectives

By the end of this week, you will be able to:

- **Explain** Modigliani-Miller's Proposition I and II, both without taxes (capital structure is irrelevant) and with taxes (debt creates value through the interest tax shield), and state precisely which of the model's assumptions taxes relax.
- **Quantify the interest tax shield** for any company — the annual shield (`Tc × interest`) and its present value under both the perpetual-debt assumption (`Tc × D`) and a finite-life debt schedule.
- **Model financial distress and bankruptcy costs** — direct costs (legal, administrative) versus indirect costs (lost customers, fire-sale asset losses, foregone investment) — and build a stylized probability-of-distress function that rises with leverage.
- **Combine both forces into the tradeoff theory of capital structure**: `V_L = V_U + PV(tax shield) − PV(expected distress costs)`, and explain why this produces an interior optimum instead of "more debt is always better."
- **Measure how leverage amplifies returns and risk** using the Degree of Financial Leverage (DFL), and build a scenario table showing how the same swing in EBIT produces a bigger swing in net income and ROE as leverage rises.
- **Find a value-maximizing debt level** under a tradeoff model — by calculus (take the derivative, set it to zero) and by brute-force numerical search over a grid of debt levels — and defend the answer against a real company's actual capital structure.

## Prerequisites

- Week 1 (time value of money), Week 2 (financial statements — you'll reuse `income_statement` and `balance_sheet` directly), and Week 3 (NPV/IRR) of this course, or equivalent.
- Week 4 (cost of capital — CAPM, cost of debt, WACC) or a working knowledge of what an unlevered cost of capital (`r_U`) and a cost of debt (`r_D`) represent. This week takes `r_U = 10%` as a given, stated assumption for Crunch Machine Co. — Week 4 is where you'd derive that number properly from market data; here you just need to know what it means.
- Working SQL (`SELECT`, joins, aggregates, a `WITH` clause) and comfortable Python — functions, loops, and basic `pandas`/`numpy`. This week adds light use of `matplotlib` for one chart in the mini-project, and a basic root-finding or grid-search pattern (no new library required — plain Python is enough).
- **No spreadsheets.** Every capital-structure model, every distress-cost curve, and every debt-level chart lives in SQL and Python this week — see the [data tooling rule](../../SYLLABUS.md#data-tooling-rule). "Should we take on more debt?" is a function you write and call across a range of `D`, not a row of formulas dragged across a workbook.

## Set up the workspace (do this first)

If you completed Week 2 and still have your `crunch_capital` database with `income_statement` and `balance_sheet` loaded, skip straight to the pip install below. Otherwise, rebuild both tables — this is the **exact same data** from Week 2, unchanged, so any ratio work you already did still applies:

**PostgreSQL:**

```bash
createdb crunch_capital        # skip if it already exists
psql crunch_capital
```

**SQLite (fallback):**

```bash
sqlite3 crunch_capital.db
```

**Python — this week's packages:**

```bash
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas numpy matplotlib sqlalchemy psycopg[binary]
```

Then paste this into your `psql` (or `sqlite3`) shell:

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
```

Sanity check — this should print `5`, `5`:

```sql
SELECT COUNT(*) FROM income_statement;
SELECT COUNT(*) FROM balance_sheet;
```

Then run this — your first look at the number this whole week revolves around, **interest coverage** (`EBIT / interest_expense`):

```sql
SELECT i.fiscal_year,
       i.operating_income                        AS ebit,
       i.interest_expense,
       ROUND(i.operating_income / i.interest_expense, 2) AS interest_coverage,
       b.short_term_debt + b.long_term_debt        AS total_debt
FROM income_statement i
JOIN balance_sheet b ON b.fiscal_year = i.fiscal_year
ORDER BY i.fiscal_year;
```

Coverage should fall from **11.52x** in 2021 to **1.61x** in 2025, while total debt climbs from **11,300** to **19,700**. That collapse — safe, boring coverage turning into white-knuckle territory over five years — is the whole week in one query. All figures are in **thousands of dollars**.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target, not a stopwatch.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | MM propositions + the tax shield | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Valuing the shield across finite vs. perpetual debt | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Distress costs + the tradeoff theory | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Leverage, DFL, and return amplification | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Solving for optimal debt; blow-up scenarios | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (find Crunch's D* and chart it) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5.5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-mm-and-the-tax-shield.md](./lecture-notes/01-mm-and-the-tax-shield.md) | MM Prop I & II with and without taxes; the interest tax shield, perpetual and finite-life | 2h |
| 2 | [lecture-notes/02-distress-and-the-tradeoff.md](./lecture-notes/02-distress-and-the-tradeoff.md) | Direct/indirect distress costs; the tradeoff theory; solving for the value-maximizing debt level | 2h |
| 3 | [lecture-notes/03-leverage-and-returns.md](./lecture-notes/03-leverage-and-returns.md) | Degree of Financial Leverage; how debt amplifies ROE and EPS in good and bad years | 2h |
| 4 | [exercises/exercise-01-value-the-tax-shield.md](./exercises/exercise-01-value-the-tax-shield.md) | Compute the annual and PV tax shield for Crunch Machine Co., all 5 years | 1h |
| 5 | [exercises/exercise-02-leverage-return-amplification.md](./exercises/exercise-02-leverage-return-amplification.md) | Build a DFL and ROE-amplification table across EBIT scenarios in pandas | 1.5h |
| 6 | [exercises/exercise-03-distress-cost-curve.md](./exercises/exercise-03-distress-cost-curve.md) | Plot firm value against a grid of debt levels under the tradeoff model | 1.5h |
| 7 | [challenges/challenge-01-optimal-debt-under-tradeoff.md](./challenges/challenge-01-optimal-debt-under-tradeoff.md) | Solve for D* by calculus and by grid search; compare to Crunch's actual debt | 1.5h |
| 8 | [challenges/challenge-02-leverage-blowup-scenario.md](./challenges/challenge-02-leverage-blowup-scenario.md) | Find the EBIT shock that breaches a covenant and the one that wipes out pretax income | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Find Crunch Machine Co.'s value-maximizing debt level and chart value vs. leverage | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, papers, and further reading | — |

## By the end of this week you can…

- State MM Proposition I and II from memory, with and without taxes, and explain in one sentence why taxes are the assumption that breaks capital-structure irrelevance.
- Compute the interest tax shield for any company, both its annual value and its present value under a stated debt-permanence assumption.
- Build a tradeoff model that trades the tax shield off against expected distress costs, and solve it for the value-maximizing debt level — by hand (calculus) and in code (grid search).
- Read a company's interest coverage trend and say, with a number, how close it is to distress — not just "leverage went up."
- Quantify exactly how much a given EBIT shock is amplified into a bigger swing in net income and ROE, using the Degree of Financial Leverage.

## Up next

[Week 6 — Equity vs. debt financing — dilution, coupons, covenants](../week-06-equity-vs-debt-financing/) — now that you know *how much* debt is too much, Week 6 compares debt against the alternative: issuing equity.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
