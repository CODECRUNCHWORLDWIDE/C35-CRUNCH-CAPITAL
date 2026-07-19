# Week 3 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first (if you skipped Week 1)

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+** with a virtual environment: `python3 -m venv .venv && source .venv/bin/activate`.

## This week's Python packages

```bash
pip install pandas numpy numpy-financial scipy sqlalchemy psycopg[binary]
```

- **`numpy-financial`** — `npv()`, `irr()`, `mirr()`, `pv()`, `fv()`, `pmt()`. NumPy itself dropped these financial functions years ago; this small companion package is where they live now: <https://numpy.org/numpy-financial/latest/>
- **`scipy.optimize.brentq`** — the root-finder you'll use for exact crossover rates and hurdle-rate breakevens in the challenges: <https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.brentq.html>

## Required reading (this week's core)

- **Investopedia — "Net Present Value (NPV)":** <https://www.investopedia.com/terms/n/npv.asp>
  *Why: the clearest short definition and worked example of the decision rule this whole week builds on.*
- **Investopedia — "Internal Rate of Return (IRR)":** <https://www.investopedia.com/terms/i/irr.asp>
  *Why: pairs directly with Lecture 2 — read it after, not instead of, the lecture.*
- **Investopedia — "Modified Internal Rate of Return (MIRR)":** <https://www.investopedia.com/terms/m/mirr.asp>
  *Why: the reinvestment-rate assumption explained from a different angle than the lecture.*
- **Investopedia — "Profitability Index":** <https://www.investopedia.com/terms/p/profitability.asp>
  *Why: the standard reference for the PI formula and the capital-rationing use case.*
- **Harvard Business Review — "A Refresher on Internal Rate of Return":** <https://hbr.org/2016/03/a-refresher-on-internal-rate-of-return>
  *Why: a practitioner-voiced version of Lecture 2's core warning — written for managers, not analysts, which is exactly the audience your mini-project memo targets.*

## Reference (keep in tabs)

- **numpy-financial documentation (full function reference):** <https://numpy.org/numpy-financial/latest/index.html>
  *Why: exact signatures for `npv`, `irr`, `mirr`, and the rest — bookmark for every remaining week of this course.*
- **PostgreSQL — Mathematical Functions (`POWER`, `^`, `ROUND`):** <https://www.postgresql.org/docs/current/functions-math.html>
  *Why: the exact operators this week's SQL discounting queries depend on.*
- **PostgreSQL — Aggregate Functions (`SUM`, `FILTER`):** <https://www.postgresql.org/docs/current/functions-aggregate.html>
  *Why: covers the `FILTER (WHERE ...)` clause used in Lecture 3's screen-and-rank query.*
- **Investopedia — "Payback Period":** <https://www.investopedia.com/terms/p/paybackperiod.asp>
  *Why: quick reference for the metric's strengths and its well-known blind spot.*
- **Investopedia — "Capital Rationing":** <https://www.investopedia.com/terms/c/capitalrationing.asp>
  *Why: the formal name for "you have less capital than good ideas," which is Lecture 3 Section 5's entire subject.*
- **Investopedia — "Descartes' Rule of Signs":** <https://www.investopedia.com/terms/d/descartes-rule-of-signs.asp>
  *Why: the mathematical rule behind why some cash-flow streams have multiple IRRs.*

## Practice beyond the seed table

- **CFI (Corporate Finance Institute) — free NPV/IRR primers:** <https://corporatefinanceinstitute.com/resources/valuation/net-present-value-npv/>
  *Why: more worked examples with different cash-flow shapes than this week's six projects.*
- **MIT OpenCourseWare — 15.401 Finance Theory I, capital budgeting lecture notes:** <https://ocw.mit.edu/courses/15-401-finance-theory-i-fall-2008/>
  *Why: a full graduate-level treatment if you want to go deeper than this course requires — free, no signup.*

## Deeper background (optional this week)

- **Lorie, J. H., & Savage, L. J. (1955). "Three Problems in Rationing Capital."** *Journal of Business, 28(4), 229–239.* — the original paper behind the profitability-index ranking rule used in Lecture 3, Section 5.
  *Why: understand where the "rank by PI under a budget" rule actually comes from, not just how to apply it.*
- **Brealey, Myers & Allen, *Principles of Corporate Finance*** — the standard MBA-level corporate finance textbook; its capital-budgeting chapters cover this week's material in far more depth, including the mathematics of multiple-IRR cases.
  *Why: the canonical reference if you want a textbook, not just articles, as you go deeper into this course.*

## Glossary

| Term | Definition |
|------|------------|
| **NPV** | Net Present Value — the sum of a project's cash flows, each discounted to today at the hurdle rate. |
| **Hurdle rate** | The minimum annual return a project must clear to be worth doing; reflects opportunity cost and risk. |
| **IRR** | Internal Rate of Return — the discount rate that makes a project's NPV exactly zero. |
| **MIRR** | Modified IRR — like IRR, but with an explicit, realistic reinvestment rate for interim cash flows. |
| **NPV profile** | A plot of a project's NPV as a function of the discount rate; IRR is where it crosses zero. |
| **Crossover rate** | The discount rate at which two competing projects' NPVs are equal; the NPV-based ranking flips on either side of it. |
| **Payback period** | The number of years until a project's cumulative cash flow turns non-negative; ignores the time value of money unless discounted. |
| **Profitability Index (PI)** | The present value of a project's future cash flows divided by its initial investment; `PI = 1 + NPV / |investment|`. |
| **Capital rationing** | A situation where available capital is less than the total investment needed to fund every positive-NPV project. |
| **Mutually exclusive projects** | Competing projects where choosing one precludes the others (e.g., two designs for the same facility). |
| **Independent projects** | Projects whose acceptance doesn't affect any other project's viability; each is evaluated on its own merits, subject only to a shared budget. |
