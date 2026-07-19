# Lecture 3 — SQL and Python, Not Spreadsheets

> **Duration:** ~2 hours. **Outcome:** You have a working PostgreSQL 16 + Python (pandas) environment, you can move data between them in both directions, and you can argue — with specifics, not vibes — why a queryable, versioned pipeline is the right home for financial models, while naming exactly what spreadsheets remain good for.

## 1. Why this lecture exists

Most people's first exposure to financial modeling is a spreadsheet: a grid of cells, some formulas, a chart. It's approachable, and for a five-minute back-of-envelope calculation, it's fine. The problem is that spreadsheets are also where a huge amount of *production* financial modeling still happens — models that drive real capital allocation, real loan approvals, real M&A decisions — and that's where things go wrong. This course teaches finance the way a modern, technically competent finance team actually should build it: in a relational database and a reproducible modeling language. This lecture sets up that workspace and makes the case for it explicitly, once, so you understand *why* every remaining week of this course reaches for SQL and Python instead of a workbook.

## 2. Install and verify PostgreSQL

Full platform-specific install steps are in [`resources.md`](../resources.md); here's the fast path.

**macOS:** [Postgres.app](https://postgresapp.com/) is the easiest — download, drag to Applications, open it, and it starts a local server. **Linux:** `sudo apt install postgresql` (Debian/Ubuntu) or `sudo dnf install postgresql-server` (Fedora/RHEL). **Windows:** the EDB installer at <https://www.postgresql.org/download/windows/>.

Verify it's running:

```bash
psql --version
# psql (PostgreSQL) 16.x

createdb crunch_capital
psql crunch_capital -c "SELECT version();"
```

If you'd rather not install a server at all this week, **SQLite** is the zero-setup fallback used throughout this course — it ships with macOS and most Linux distributions:

```bash
sqlite3 --version
sqlite3 crunch_capital.db
```

Everything in this course runs on either engine; we call out the handful of places syntax differs (mostly date functions and string case-sensitivity).

## 3. Install and verify Python + pandas

```bash
python3 --version          # 3.10 or newer
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install pandas numpy sqlalchemy psycopg[binary]
```

Verify:

```bash
python3 -c "import pandas as pd; print(pd.__version__)"
```

- **pandas** — the DataFrame library; think "a spreadsheet's grid, but as a first-class programming object with a name, that you can save, diff, test, and version."
- **NumPy** — the numerical engine pandas is built on; you'll use it directly for a few finance functions (`numpy.irr`-style operations show up starting Week 3).
- **SQLAlchemy** + **psycopg** — how Python talks to PostgreSQL. SQLAlchemy is the general database toolkit; `psycopg` is the specific PostgreSQL driver it uses underneath.

## 4. Connecting Python to Postgres

This is the pipeline shape you'll use every week of this course: **store in SQL, pull into pandas, model, and (often) write results back.**

```python
import pandas as pd
from sqlalchemy import create_engine

# Adjust user/host if your local Postgres needs them; defaults usually work on a
# freshly installed local server with your OS username.
engine = create_engine("postgresql+psycopg://localhost/crunch_capital")

# Pull an entire table straight into a DataFrame
cash_flows = pd.read_sql("SELECT * FROM cash_flows", engine)
print(cash_flows.head())
print(cash_flows.shape)   # (38, 12)
```

For SQLite, the connection string changes but the pattern is identical:

```python
engine = create_engine("sqlite:///crunch_capital.db")
cash_flows = pd.read_sql("SELECT * FROM cash_flows", engine)
```

You can also push a DataFrame **back** into the database — useful once you start computing derived tables (say, an amortization schedule) that you want to persist and query further:

```python
cash_flows.to_sql("cash_flows_backup", engine, if_exists="replace", index=False)
```

This round-trip — SQL as the store of record, pandas as the modeling and analysis layer, and the ability to write results back into SQL — is the backbone of every remaining week in this course.

## 5. Why not just use a spreadsheet? The specific case

"Use the right tool for the job" is true but unhelpfully vague. Here is the specific, concrete case for SQL + Python over Excel/Sheets for financial modeling, with an example of each failure mode:

### 5.1 Silent formula drift

In a spreadsheet, a formula lives in a cell, and "the model" is really thousands of individually-editable cells that are supposed to all agree. It is trivially easy to change `=B2*(1+$C$2)^A2` in row 47 but forget row 48, and now half your discount calculation uses one formula and half uses another — with **no error, no warning, nothing visually different** unless you go cell-by-cell. In SQL or Python, the discount calculation is written **once**, in one place (a query or a function), and every row uses that same logic every time, guaranteed. Change it once, it's changed everywhere.

### 5.2 No real version control

`git diff` on a spreadsheet file shows you an unreadable binary blob. You cannot see *what changed* between last quarter's model and this quarter's — who changed which assumption, when, or why — without manually clicking through and comparing by eye (or paying for specialized diffing tools that half-solve it). A `.sql` file or a `.py` script is plain text: `git diff` shows you the exact line that changed, `git blame` shows you who changed it and when, and a pull request lets a colleague review the *actual change*, not the whole file again.

### 5.3 No enforced types or constraints

Nothing stops you from typing "N/A" into a cell that's supposed to hold a number, and now a `SUM()` a few rows down either errors or — worse — silently ignores that row without telling you. A SQL table with `amount NUMERIC NOT NULL` **cannot** contain "N/A" — the database rejects the insert at write time, not three formulas downstream when a total looks wrong.

### 5.4 Doesn't scale past "fits on one screen"

A cash-flow table with 6 projects and 38 rows, like this week's, is comfortable in a spreadsheet. A real corporate finance function tracking hundreds of projects, years of price history, or a portfolio of thousands of positions is not — spreadsheets get slow, formulas get impossible to audit at scale, and "select all projects with a hurdle rate above 10%, sorted by NPV" becomes a manual filter-and-sort exercise instead of one line of SQL:

```sql
SELECT DISTINCT project_name, hurdle_rate
FROM cash_flows
WHERE hurdle_rate > 0.10
ORDER BY hurdle_rate DESC;
```

### 5.5 Hard to test

You can write an automated test — `assert present_value(60000, 0.07, 4) == pytest.approx(45777.31)` — for a Python function. There is no equivalent for "prove this spreadsheet cell always computes correctly," short of manually re-checking it, which people stop doing after the first few times it's correct.

## 6. What spreadsheets are still fine for

This isn't a claim that spreadsheets are useless — it's a claim about *fit for purpose*. Spreadsheets remain a reasonable tool for: a true one-off, throwaway back-of-envelope estimate nobody will revisit; a presentation-layer summary table pasted from a model that lives elsewhere; or ad-hoc exploration by someone who genuinely will never need to reproduce or audit the calculation. What spreadsheets are *not* a good fit for — and what this entire course refuses to use them for — is: anything that will be **reused**, anything that will be **reviewed by someone else**, anything that needs to be **provably correct**, and anything that needs to be **queried** rather than eyeballed. Corporate finance work is almost always all four. (This course does teach spreadsheets properly, on their own terms, in [C41 Crunch Excel](../../../C41-CRUNCH-EXCEL/) — that's the right place for that skill; it's simply not this course.)

## 7. The pipeline shape you'll use all course

From this week forward, the pattern repeats:

1. **Data lives in PostgreSQL** — statements, cash flows, price series, cap tables, deal terms. Typed, constrained, queryable.
2. **Modeling logic lives in Python (pandas/NumPy)** — functions with names, that you can test, reuse, and import into next week's model without retyping.
3. **SQL does the heavy filtering/aggregation**; pandas does the heavier numerical modeling (optimization, simulation, statistics) that's awkward or impossible in pure SQL.
4. **Results, where useful, get written back to SQL** — so they're queryable too, not stranded in a notebook only you can open.

You already have both halves of this running: the `cash_flows` table from the week README, and the Python environment from section 3. Exercise 3 has you build the first real pandas layer on top of it.

## 8. Check yourself

- Name one specific failure mode of spreadsheet-based modeling that SQL/Python structurally prevents, and explain *how* it prevents it.
- What's the difference between where "the store of record" lives and where "the modeling logic" lives, in this course's pipeline?
- Give one legitimate, honest use case where a spreadsheet is still the right tool — and explain why it doesn't conflict with this week's rule.
- Write the one line of Python that connects to a local Postgres database named `crunch_capital` and pulls the entire `cash_flows` table into a DataFrame.

You now have the full workbench: PostgreSQL running, `cash_flows` loaded, Python + pandas installed and connected. Every exercise, challenge, and the mini-project this week runs on exactly this setup.

## Further reading

- **SQLAlchemy — Engine Configuration:** <https://docs.sqlalchemy.org/en/20/core/engines.html>
- **pandas — `read_sql` / `to_sql`:** <https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html>
- **psycopg 3 — Basic usage:** <https://www.psycopg.org/psycopg3/docs/basic/index.html>
- **"What's Wrong with Spreadsheets" — European Spreadsheet Risks Interest Group (real, documented spreadsheet-error incidents):** <http://www.eusprig.org/horror-stories.htm>
