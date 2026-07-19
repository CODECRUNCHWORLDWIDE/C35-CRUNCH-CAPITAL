# Week 2 — Resources

Curated, free, official-first. You do not need to read everything here — this is a reference to come back to, not a reading assignment.

## Install / environment (reused from Week 1)

If you haven't already set up your workspace, do this first — everything else in this week assumes it's running.

- **PostgreSQL 16+** — [postgresql.org/download](https://www.postgresql.org/download/). macOS: [Postgres.app](https://postgresapp.com/) is the fastest path. Linux: `sudo apt install postgresql` (Debian/Ubuntu) or `sudo dnf install postgresql-server` (Fedora/RHEL). Windows: the EDB installer.
- **SQLite 3.35+** (zero-setup fallback; window functions like `LAG()` need 3.25+, well below what ships on any current OS) — ships with macOS and most Linux distributions. Verify with `sqlite3 --version`.
- **Python 3.10+ with pandas** —
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate     # Windows: .venv\Scripts\activate
  pip install pandas numpy sqlalchemy psycopg[binary]
  ```

## Reading real financial statements

- **SEC EDGAR full-text search** — <https://www.sec.gov/edgar/search/> — search any U.S. public company and pull its actual 10-K (annual) or 10-Q (quarterly) filing, free, no account needed. This is the primary source for the mini-project.
- **SEC — Beginners' Guide to Financial Statements** — <https://www.investor.gov/introduction-investing/investing-basics/how-stock-markets-work/beginners-guide-financial> — a short, plain-English regulator's-eye overview of what the three statements are for.
- **FASB Accounting Standards Codification, Topic 230 (Statement of Cash Flows)** — <https://asc.fasb.org/230/> — the actual U.S. GAAP rule governing how the cash-flow statement is built.
- **stockanalysis.com** — <https://stockanalysis.com/> — free, pre-extracted multi-year statement tables for most public companies; a good way to cross-check your own manual extraction from a 10-K during the mini-project.
- **macrotrends.net** — <https://www.macrotrends.net/> — another free source of pre-extracted historical statements and pre-computed ratios, useful for sanity-checking your own SQL output.

## Ratio analysis, deeper

- **CFA Institute — Financial Analysis Techniques (Level I curriculum overview)** — <https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/financial-analysis-techniques> — the industry-standard treatment of exactly the four ratio families this week covers, at the next level of depth.
- **Aswath Damodaran (NYU Stern) — free corporate finance teaching materials** — <https://pages.stern.nyu.edu/~adamodar/> — one of the most-cited free sources in finance education; his ratio-analysis and valuation notes go well beyond this week if you want to go deeper early.
- **European Spreadsheet Risks Interest Group — real, documented spreadsheet-error incidents** — <http://www.eusprig.org/horror-stories.htm> — concrete cases of real financial losses caused by uncaught spreadsheet errors; useful context for why this course insists on SQL/Python instead.

## SQL references used this week

- **PostgreSQL — Window Functions tutorial** (`LAG`, `LEAD`, running comparisons) — <https://www.postgresql.org/docs/current/tutorial-window.html>
- **PostgreSQL — Common Table Expressions (`WITH`)** — <https://www.postgresql.org/docs/current/queries-with.html>
- **SQLite — Window Functions** — <https://www.sqlite.org/windowfunctions.html>
- **pandas — `read_sql` / `to_sql`** — <https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html>

## Where this week leads

Week 3 of this course turns "is this company healthy" (this week) into "what is this company worth" (valuation and the cost of capital) — the ratios you computed this week, especially profitability and leverage, become direct inputs into that model. Keep your `income_statement`, `balance_sheet`, and `cash_flow_statement` tables — you'll query them again.
