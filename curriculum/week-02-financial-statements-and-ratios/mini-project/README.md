# Mini-Project — A Ratio Dashboard for a Real Company

> Every query this week ran against a fictional company whose numbers were built to teach you a specific lesson. This project is the reality check: pick a **real, publicly traded company**, load its actual filings into the same schema you've used all week, run your dashboard query against real numbers, and write a one-page verdict on its financial health — backed by ratios, not vibes.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's deliberately the first time this course asks you to leave the fictional Crunch Machine Co. dataset. Real filings are messier than a seed script — line items are labeled differently company to company, some companies don't break out R&D separately, some report in millions instead of thousands, some have fiscal years that don't end in December. Wrestling a real filing into your clean schema, the way you practiced in Exercise 1, is exactly the skill this project is testing.

---

## Deliverable

A directory in your portfolio `c35-week-02/mini-project/` containing:

1. `schema.sql` — your `income_statement`, `balance_sheet`, and `cash_flow_statement` tables (same structure as this week's, plus a `company` column if you want to keep it in the same database as Crunch Machine Co. — your call), loaded with **at least 3 fiscal years** of real data for your chosen company.
2. `sources.md` — for every number you loaded, where it came from: the filing, the fiscal year, and a link. This is non-negotiable — a ratio with no traceable source is not usable in real analysis, and this file is the difference between "I looked this up" and "I made this up."
3. `dashboard.sql` — your Challenge 1 dashboard query, adapted to run against your real company's data.
4. `health-read.md` — a one-page (400–600 word) written verdict on the company's financial health. See the structure below.

---

## Step 1 — Pick a company

Pick any real, publicly traded company you can find at least 3 consecutive fiscal years of filings for. Good starting points:

- A company you already know something about (makes the write-up easier to sanity-check against reality).
- A **mid-size, single-segment business** is easier than a sprawling conglomerate — fewer footnotes, cleaner statements, less judgment needed about what to include.
- Avoid banks and insurance companies this week — their balance sheets don't follow the current/long-term asset-liability structure this week's ratios assume (that's a real, standard exception in finance, not a limitation of your SQL).

Free sources for real filings (see [resources.md](../resources.md) for the full list):

- **SEC EDGAR** (<https://www.sec.gov/edgar/search/>) — the primary source, free, for any U.S. public company's actual 10-K.
- **stockanalysis.com** or **macrotrends.net** — pre-extracted, free, multi-year statement tables for most public companies, useful for cross-checking your own EDGAR extraction.

## Step 2 — Load the data

Adapt this week's `CREATE TABLE` statements (add a `company` column if useful) and hand-enter at least 3 years of `income_statement`, `balance_sheet`, and `cash_flow_statement` rows for your company. This is manual data entry from a real filing — expect it to take real time and to require judgment calls about which filing line maps to which of your columns, the same skill Exercise 1 drilled. Document every judgment call in `sources.md`.

**Verify articulation before you trust anything.** Run the same checks from Exercise 2 (retained-earnings tie-out, cash tie-out, balance-sheet identity) against your real data. If they don't tie out, first suspect a transcription error on your end — but if you're confident your entry is correct and it still doesn't tie, note that in `sources.md` too; real filings occasionally have minor rounding differences or one-off reconciling items your simplified schema doesn't capture, and saying so honestly is part of the deliverable.

## Step 3 — Run the dashboard

Adapt your Challenge 1 query to your real company's data and run it. Save the query as `dashboard.sql` and its output (as a Markdown table, same as Challenge 1) inline in `health-read.md` or as a companion file.

## Step 4 — Write the health read

`health-read.md`, 400–600 words, structured as:

1. **The company, in one paragraph.** What it does, what industry it's in, why you picked it.
2. **The headline number.** Pick the single ratio (or trend) that tells the most important story about this company's health, state it plainly, and say why you picked that one over the other thirteen.
3. **Liquidity.** What the current/quick/cash ratios say, and the trend direction.
4. **Leverage.** What debt-to-equity and interest coverage say, and the trend direction.
5. **Profitability.** What the margins and returns say, and the trend direction.
6. **Efficiency.** What DSO/DIO/DPO and the cash conversion cycle say, and the trend direction.
7. **Verdict.** One paragraph: is this company financially healthy, stable, or showing signs of stress? What would you want to see in next year's filing to change your mind?

Write it for a reader who trusts your numbers but hasn't run the queries themselves — every claim should be backed by a specific figure from your dashboard, not a vague impression.

---

## Rules

- **Real data only.** No fictional or synthetic company for this project — the whole point is practicing on numbers you didn't get to design yourself.
- **Minimum 3 fiscal years.** A single year of ratios has almost nothing to say about trend — the entire lesson of this week is that trend is the signal.
- **Every number sourced.** `sources.md` must let a reviewer trace every figure back to a real filing.
- **All SQL, no spreadsheets** — same [data tooling rule](../../SYLLABUS.md#data-tooling-rule) as every other deliverable this week.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Data quality | 25% | Real, correctly mapped filing data, ≥3 years, sourced |
| Articulation | 20% | Statements verifiably tie out (or honestly explained if not) |
| Dashboard correctness | 20% | All ratios computed correctly against the real numbers |
| Health read quality | 25% | Every claim backed by a specific number; verdict is specific, not generic |
| Sourcing discipline | 10% | `sources.md` lets a reviewer independently verify every figure |

---

## Why this matters

Everything this week taught you — reading the three statements, proving they articulate, computing the ratio families, reading trend over snapshot — was practiced on a company designed to teach you a lesson. Real companies don't cooperate that neatly. This project is where you find out whether the skill transfers: can you take a genuinely messy real 10-K, load it correctly, and produce a verdict you'd actually stand behind if someone questioned it? That's the job.

When done: push, then take the [quiz](../quiz.md) and get ready for Week 3 — valuation and the cost of capital, where you start turning "is this company healthy" into "what is this company worth."
