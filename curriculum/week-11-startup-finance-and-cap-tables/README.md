# Week 11 — Startup finance & cap tables

> **Goal:** by Sunday you can take a startup from a two-founder incorporation through a seed round of SAFEs, a priced Series A with an investor-mandated option pool, and an acquisition — track every shareholder's ownership at every step in SQL, and run a full liquidation-preference waterfall to compute exactly who gets paid what at the exit.

Welcome back to **C35 · Crunch Capital**. Every week so far has looked at capital from the seat of an already-large company — Crunch Machine Co. raising a follow-on, comping public peers, backtesting a portfolio of listed names. This week we go back to day one. Somewhere, right now, two people are signing incorporation paperwork at a kitchen table, splitting ownership of a company that doesn't have a single dollar of revenue yet. That ownership ledger — who owns what, and what happens to those percentages every time new money comes in — is the **cap table**, and it is simultaneously the simplest spreadsheet-shaped idea in finance and one of the most consistently mismanaged documents in business. Founders lose real money to cap tables they didn't understand. This week you learn to build one that can't lie to you, because it's a database, not a guess.

We follow one company through the whole week: **Solstice Data, Inc.**, a two-founder B2B analytics startup. You'll build its cap table from the incorporation grant, raise a seed round on SAFEs, price a Series A (including the investor-required option pool — the single most common source of "wait, why did my percentage drop *that* much?"), and then, in the final lecture, blow the company up in a simulated acquisition and run the exact liquidation-preference waterfall that decides who gets paid first, second, and last out of the proceeds. Every number in every table this week is queryable — no spreadsheet, ever, per the [data tooling rule](../../SYLLABUS.md#data-tooling-rule). A cap table with a dozen rows and three financing events is exactly the kind of "small but easy to get catastrophically wrong" dataset that belongs in a database, not a `.xlsx` file passed around by email.

## Learning objectives

By the end of this week, you will be able to:

- **Model founder and investor ownership** in a cap table stored as queryable rows — not a spreadsheet — computing fully-diluted ownership percentage at any point in a company's history.
- **Model SAFEs, priced rounds, and conversions** — valuation caps, discounts, the "greater of cap or discount" conversion rule, and how a SAFE becomes real shares the moment a priced round happens.
- **Handle option pools and their dilution** — including the "option pool shuffle," where an investor-mandated pool created *before* a round's price is set dilutes only the existing shareholders, not the incoming investor.
- **Compute post-money ownership across rounds** — pre-money valuation, price per share, new investor shares, and how every existing holder's percentage compresses as new shares are issued.
- **Build an exit waterfall across preferences** — liquidation preference stacks, seniority, the non-participating "convert or take the preference" decision each preferred class makes, and how proceeds actually cascade to real dollars per shareholder.

## Prerequisites

- Week 4 (cost of capital) and Week 6 (equity vs. debt financing) of this course, or equivalent — you should already be comfortable with the idea of pricing a share issuance and computing dilution for a single round. This week extends that into a multi-round, multi-instrument history.
- Working SQL (`SELECT`, `WHERE`, joins, `GROUP BY`, `CASE`, window functions like `SUM(...) OVER (...)`) and Python with `pandas`/`numpy` — the same stack from every prior week.
- **No spreadsheets.** A cap table is the textbook example of a dataset people wrongly reach for Excel to manage. Every grant, every SAFE, every round, and every waterfall payout this week lives in SQL and Python, per the [data tooling rule](../../SYLLABUS.md#data-tooling-rule). The moment your cap table has more than one financing event, "which tab is current?" becomes a real and expensive question — a database never has that problem.

## Set up this week's workspace (do this first)

This week is a hard context-switch: a brand-new, pre-revenue startup, not Crunch Machine Co. We use a **new database** so nothing collides with prior weeks.

**PostgreSQL:**

```bash
createdb solstice_captable
psql solstice_captable
```

**SQLite (fallback):**

```bash
sqlite3 solstice_captable.db
```

**Python:**

```bash
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install pandas numpy sqlalchemy psycopg[binary]
```

Now paste this into your `psql` (or `sqlite3`) shell. It creates the one table every lecture builds on this week — a single ledger of every security anyone has ever held in Solstice Data, Inc. We start it with just the two founders; every later lecture and exercise **adds rows**, never rewrites history.

```sql
CREATE TABLE cap_table_events (
    event_id        INTEGER PRIMARY KEY,
    holder_name     TEXT    NOT NULL,
    holder_type     TEXT    NOT NULL,   -- Founder, Advisor, Employee, Pool, SAFE-Investor, Preferred-Investor
    security_type   TEXT    NOT NULL,   -- Common, Option-Pool, SAFE, Preferred-Seed, Preferred-SeriesA, Preferred-SeriesB
    shares          NUMERIC,            -- NULL until a SAFE converts
    invested_amount NUMERIC,            -- cash actually paid in (SAFEs, priced rounds); NULL for founder/option grants
    price_per_share NUMERIC,            -- NULL for unconverted SAFEs and the unallocated pool
    event_date      DATE    NOT NULL,
    round_name      TEXT    NOT NULL,   -- Founding, Seed-SAFE, Series-A, Series-B, ...
    notes           TEXT
);

INSERT INTO cap_table_events
(event_id, holder_name, holder_type, security_type, shares, invested_amount, price_per_share, event_date, round_name, notes) VALUES
(1, 'Maya Chen',  'Founder', 'Common', 5500000, NULL, 0.0001, '2023-02-01', 'Founding', 'CEO — 55% at incorporation, 4yr vest / 1yr cliff'),
(2, 'Diego Ruiz', 'Founder', 'Common', 4500000, NULL, 0.0001, '2023-02-01', 'Founding', 'CTO — 45% at incorporation, 4yr vest / 1yr cliff');
```

Sanity check — this should print `10000000`:

```sql
SELECT SUM(shares) FROM cap_table_events WHERE round_name = 'Founding';
```

Solstice's authorized common stock is 10,000,000 shares and every one of them is issued to the two founders on day one — the cleanest possible starting cap table. That won't last: by the end of Lecture 2 there will be five more rows and a very different picture of who owns what.

## Weekly schedule

The schedule below adds up to approximately **28 hours** (the course's full-time pace). Treat it as a target.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Cap table basics, founding grant, vesting | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | SAFEs — caps, discounts, conversion | 1.5h | 1.5h | 0h | 0.5h | 1h | 0h | 4.5h |
| Wednesday | Priced rounds + the option pool shuffle | 1.5h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | The exit waterfall | 1h | 1h | 1h | 0.5h | 1h | 1h | 5.5h |
| Friday | Challenges — multi-round + preferences | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project (3 rounds to an exit) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5h** | **3h** | **3.5h** | **5h** | **5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it, and the cap table you build carries forward from lecture to lecture to mini-project.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-cap-table-basics.md](./lecture-notes/01-cap-table-basics.md) | Founders, authorized/issued shares, vesting, fully-diluted %, the cap table as an event ledger | 2h |
| 2 | [lecture-notes/02-safes-rounds-and-pools.md](./lecture-notes/02-safes-rounds-and-pools.md) | SAFE mechanics, cap/discount conversion, pre/post-money, the option pool shuffle | 1.5h |
| 3 | [lecture-notes/03-the-exit-waterfall.md](./lecture-notes/03-the-exit-waterfall.md) | Liquidation preferences, participating vs. non-participating, seniority, the convert-or-take-the-preference decision | 1h |
| 4 | [exercises/exercise-01-build-a-founding-cap-table.md](./exercises/exercise-01-build-a-founding-cap-table.md) | Schema, founder grants, vesting schedule, ownership % query | 1h |
| 5 | [exercises/exercise-02-convert-a-safe.md](./exercises/exercise-02-convert-a-safe.md) | Convert three SAFEs into Series A shares using the cap/discount rule | 1h |
| 6 | [exercises/exercise-03-add-an-option-pool.md](./exercises/exercise-03-add-an-option-pool.md) | Solve the pool-shuffle algebra, insert the pool, recompute the full post-round table | 1h |
| 7 | [challenges/challenge-01-multi-round-dilution.md](./challenges/challenge-01-multi-round-dilution.md) | Add a Series B and track every holder's % across three rounds | 1h |
| 8 | [challenges/challenge-02-waterfall-under-preferences.md](./challenges/challenge-02-waterfall-under-preferences.md) | Run the waterfall at three exit prices; compare participating vs. non-participating | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Founding → Seed SAFEs → Series A → Series B → exit waterfall, end to end | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced across the week | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, calculators, and the few links worth your time | — |

## By the end of this week you can…

- Read any startup's cap table and know exactly what question to ask about vesting, the option pool, and the preference stack before trusting a percentage.
- Convert a SAFE correctly, including the case where the discount — not the cap — is the better deal for the investor.
- Explain, with real numbers, why an investor-mandated option pool dilutes founders and not the incoming money.
- Build a liquidation waterfall that tells you, for any exit price, exactly who gets paid first and how much common stock is worth at the end of the line.

## Up next

[Week 12 — Capstone: full deal, risk, and governance](../week-12-capstone-deal-risk-and-governance/) — everything this course taught you, applied to one end-to-end deal.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
