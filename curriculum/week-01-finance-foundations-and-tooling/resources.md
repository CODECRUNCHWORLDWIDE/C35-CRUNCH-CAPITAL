# Week 1 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** — the course's primary data store:
  <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+** — <https://www.python.org/downloads/>. Check with `python3 --version`.
- **pandas, NumPy, SQLAlchemy, psycopg** — installed via `pip` once you have a virtual environment (see Lecture 3, section 3). No signup, all free and open-source.
- **A GUI (optional, later)** — [DBeaver](https://dbeaver.io/) (free, both engines) or [pgAdmin](https://www.pgadmin.org/) (Postgres). This week, stay in the terminal and a Python REPL — it teaches you more about what's actually happening.

## Required reading (this week's core)

- **Investopedia — "Time Value of Money":** <https://www.investopedia.com/terms/t/timevalueofmoney.asp>
  *Why: the clearest short, free explanation of the exact idea Lecture 2 builds on.*
- **Investopedia — "Present Value" and "Future Value":** <https://www.investopedia.com/terms/p/presentvalue.asp> · <https://www.investopedia.com/terms/f/futurevalue.asp>
  *Why: a second, independent explanation of the same two formulas — read after Lecture 2 to reinforce, not replace it.*
- **Investopedia — "Effective Annual Interest Rate":** <https://www.investopedia.com/terms/e/effectiveinterest.asp>
  *Why: the nominal-vs-effective-rate distinction from Lecture 2, section 4, explained a second way.*
- **pandas — "10 minutes to pandas":** <https://pandas.pydata.org/docs/user_guide/10min.html>
  *Why: the fastest legitimate on-ramp to DataFrame basics if pandas is new to you — needed for Exercise 3 and the mini-project.*

## Reference (keep in tabs)

- **PostgreSQL — Mathematical Functions (`POWER`, `^`, `ROUND`):** <https://www.postgresql.org/docs/current/functions-math.html>
  *Why: the exact function signatures you need for every discounting query this week.*
- **SQLite — Core Functions:** <https://www.sqlite.org/lang_corefunc.html>
  *Why: confirms which math functions (including `POWER`) your SQLite build supports.*
- **pandas — `read_sql` / `to_sql`:** <https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html>
  *Why: the exact reference for moving data between SQL and DataFrames, used throughout Lecture 3 and every exercise after it.*
- **SQLAlchemy — Engine Configuration:** <https://docs.sqlalchemy.org/en/20/core/engines.html>
  *Why: connection-string syntax for both PostgreSQL and SQLite, if Lecture 3's examples need adapting to your setup.*
- **psycopg 3 — Basic usage:** <https://www.psycopg.org/psycopg3/docs/basic/index.html>
  *Why: the PostgreSQL driver reference, for when a connection error needs debugging.*

## Practice beyond the seed table

- **Investopedia — "Net Present Value (NPV)":** <https://www.investopedia.com/terms/n/npv.asp>
  *Why: previews Week 3's capital budgeting decision rule, built directly on this week's discounting mechanics.*
- **NumPy Financial functions (`npf.pv`, `npf.fv`, `npf.pmt`):** <https://numpy.org/numpy-financial/latest/>
  *Why: a battle-tested reference implementation of the exact formulas you're hand-building this week — useful for cross-checking your own functions, not a replacement for understanding the math yourself. (Requires the separate `numpy-financial` package; core NumPy no longer bundles these.)*
- **pgexercises.com** — free, browser-based, graded `SELECT` drills: <https://pgexercises.com/>
  *Why: keeps your general SQL fluency sharp alongside this course's finance-specific queries.*

## Deeper background (optional this week)

- **Damodaran (NYU Stern) — free corporate finance course materials:** <https://pages.stern.nyu.edu/~adamodar/>
  *Why: one of the most respected free, public sources on corporate finance and valuation; you'll return to Damodaran's valuation datasets in Week 7.*
- **"What's Wrong with Spreadsheets" — European Spreadsheet Risks Interest Group:** <http://www.eusprig.org/horror-stories.htm>
  *Why: real, documented cases of spreadsheet errors causing material financial harm — the evidence behind Lecture 3's argument and useful ammunition for Challenge 2.*

## Glossary

| Term | Definition |
|------|------------|
| **Cash flow** | Money actually moving in or out, as distinct from accounting profit. |
| **Present value (PV)** | What a future cash flow is worth today, after discounting. |
| **Future value (FV)** | What a present amount grows to after compounding forward. |
| **Discounting** | Shrinking a future cash flow to express it in today's dollars. |
| **Compounding** | Growing a present amount forward through time, with interest earning interest. |
| **Discount rate / hurdle rate** | The required annual return used to discount a stream, reflecting time and risk. |
| **Discount factor** | `1 / (1 + r)^n` — the multiplier that converts a future cash flow to its present value. |
| **Net present value (NPV)** | The sum of every cash flow in a stream's present value, signed, including the upfront investment. |
| **Nominal rate** | A quoted annual rate that ignores compounding frequency. |
| **Effective annual rate (EAR)** | The true annual rate after accounting for how often interest compounds. |
| **Amortization** | Splitting a level loan payment into interest and principal over the loan's life. |
| **Vectorized operation** | A pandas/NumPy computation applied to a whole column at once, without an explicit loop. |
| **Store of record** | The single authoritative place data lives — in this course, always SQL, never a spreadsheet. |

---

*Broken link? Open an issue or PR.*
