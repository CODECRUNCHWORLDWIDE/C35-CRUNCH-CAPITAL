# Mini-Project — Compute a Real Company's WACC from Market Data

> Pick one real, publicly traded company. Gather its real market data — price history, a market benchmark, its actual outstanding debt or interest expense, its shares outstanding — and compute its weighted average cost of capital exactly the way you did for Crunch Machine Co. this week, end to end, in SQL and pandas. No fictional numbers this time: everything traces back to a real, citable source.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's the closest thing this course has done so far to actual sell-side or corporate-finance analyst work: someone hands you a ticker, and a few hours later you hand back a defensible discount rate with your assumptions laid out. Every number this week's lectures used for Crunch Machine Co. was fictional but realistic-looking; this project asks you to do the identical calculation on a company that actually exists, using data anyone can freely pull.

---

## Choosing a company

Pick a real, publicly traded company you can get data for easily. Good choices:

- A large, liquid, well-known company (more reliable price history, easier-to-find debt information) — e.g., a large industrial, consumer, or technology company you already know something about.
- **Avoid** companies with unusual capital structures (banks and insurers, whose "debt" doesn't work like industrial debt — their whole balance sheet is different) for your first attempt at this. Stick to a regular operating company.
- Write down your choice and *why* you picked it (one sentence) in `report.md` before you start pulling data.

## Gathering the data (all free, no API key required for the sources below)

You need four things. Get each from a real source and **record the source and pull date** for every one — a WACC without traceable sources isn't usable in real work.

1. **Monthly price history for your company**, at least 3 years (5 is better), and **a market benchmark** over the identical dates (a broad index like the S&P 500 works as the benchmark; if you can only find a proxy ETF's price history, that's fine too — say which you used).
   - **Stooq** (<https://stooq.com/>) offers free historical daily/monthly CSV downloads with no signup, e.g. `https://stooq.com/q/d/l/?s=<TICKER>.us&i=m` for monthly US-listed data. This is the easiest scriptable source.
   - Yahoo Finance's historical data page also works (manual CSV export from a browser, if Stooq doesn't have your ticker).

2. **The risk-free rate.** FRED's `DGS10` series (10-Year Treasury Constant Maturity Rate) — pull the most recent value: <https://fred.stlouisfed.org/series/DGS10>. FRED also offers a free CSV download with no signup.

3. **Shares outstanding and current share price** — the most recent share count is in the company's latest 10-Q or 10-K (SEC EDGAR, <https://www.sec.gov/edgar/search/>, free, no signup — search the company name, open the most recent quarterly or annual filing, the cover page states shares outstanding as of a specific date). Current share price is the most recent close in your own price-history pull from step 1.

4. **Debt information.** This is the hardest to get precisely without a paid terminal — two acceptable approaches, pick one and say which:
   - **Approach A (preferred if available):** Find the company's most recent 10-K balance sheet and debt footnote (SEC EDGAR again) — it lists outstanding bonds/notes with coupon rates and maturities. You may not find current market prices for each bond without a paid source; if so, **approximate each bond's price as par (100)** and say explicitly that you did this — your YTM then just equals the coupon, which is a known simplification, not an error, as long as you disclose it.
   - **Approach B (acceptable fallback):** Use the company's total interest expense (from the income statement) divided by total debt (from the balance sheet) as an approximate blended pre-tax cost of debt, instead of bond-by-bond YTM. State this is what you did — it's a legitimate, commonly used shortcut when bond-level data isn't accessible.

## Deliverable

A directory in your portfolio `c35-week-04/mini-project/` containing:

1. **`load_data.py`** — a script that loads your gathered data (prices, debt info, capital structure inputs) into new tables in your `crunch_capital` database (e.g., `real_stock_prices`, `real_debt`, `real_capital_structure` — name them clearly, distinct from this week's Crunch Machine Co. tables so you don't overwrite the seed data).
2. **`compute_wacc.py`** — the full pipeline: pulls your loaded data back out with pandas, computes returns and regresses beta, computes cost of debt, computes market-value weights, and assembles the final WACC. Should run start to finish and print every intermediate number.
3. **`report.md`** — see the required sections below.

## Required sections in `report.md`

1. **Company and why you picked it** (1–2 sentences).
2. **Data sources**, one line each: where you got price history (URL + date range), the risk-free rate (URL + value + date), shares outstanding (filing + date), and debt information (filing/approach + date). Be specific enough that someone else could reproduce your pull.
3. **Your beta**, both self-regressed (report beta, alpha, R², and your window length) and, if you can find one, a vendor-published beta for comparison (many finance sites publish a beta on a stock's summary page for free) — note any gap and one sentence on why they might differ.
4. **Your cost of debt**, and which approach (A or B above) you used, and why.
5. **Your capital weights** ($E/V$, $D/V$) with the underlying market values shown.
6. **Your final WACC**, computed using your own regression beta.
7. **A gut check**: does this WACC feel reasonable for this company, given what you know about its size, industry, and risk? (A mature utility should land meaningfully lower than a small-cap growth company — if your number looks wildly off from that intuition, say so and take a guess at what might be wrong with your inputs, rather than silently reporting a number you don't believe.)

## Rules

- **Real data only.** Every number in `report.md` must trace to a real, cited source — no invented figures, even as placeholders.
- **Same pipeline shape as this week.** SQL for storage, pandas for the modeling — no spreadsheets, per the course's data tooling rule, even though you're pulling from external CSV/filing sources.
- **Disclose every simplification.** If you approximated a bond's price at par, or used the interest-expense shortcut for cost of debt, or used a shorter price-history window than 5 years because that's all you could find — say so plainly. A stated assumption is a feature of good analyst work, not a weakness.
- **Sanity-check your beta.** If your regression comes back with a beta below 0 or above 3 for an ordinary operating company, something is almost certainly wrong with your data alignment (dates not matching between the stock and the benchmark is the single most common cause) — debug it before moving on, don't just report an implausible number.

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Data sourcing | 20% | Every input traces to a specific, real, cited source with a pull date |
| Pipeline correctness | 30% | Beta regression, cost of debt, weights, and WACC assembly all mechanically correct |
| SQL + pandas discipline | 15% | Data genuinely lives in SQL tables and is queried, not just read from a CSV straight into a script and never touching the database |
| Disclosed assumptions | 15% | Every simplification (par-priced bonds, interest-expense shortcut, shorter window, etc.) is explicitly stated |
| Interpretation | 20% | `report.md`'s gut-check section shows real judgment about whether the number is plausible, not just a number reported and left alone |

## Stretch goals

- Compute your chosen company's WACC **two ways** — once with your self-regressed beta, once with a vendor-published beta you look up — and report both, the way this week's lectures did for Crunch Machine Co.
- If your company has more than one class of debt or more than one class of shares (some companies do), handle the blending explicitly rather than picking just one instrument.
- Pull a **second** company in the same industry and compute its WACC too. Compare the two — do they land in a similar range? If not, can you explain the gap from what you know about the two businesses (leverage, size, cyclicality)?
- Store your final WACC, with its full set of inputs, in a `wacc_results` table (`company`, `beta`, `cost_of_equity`, `cost_of_debt_after_tax`, `e_over_v`, `d_over_v`, `wacc`, `computed_date`) so it's queryable alongside any future company you run through the same pipeline.

## Why this matters

Every other calculation this week ran against fictional-but-realistic Crunch Machine Co. data, seeded for you, clean and complete. Real companies are messier — data sources disagree, some numbers are hard to find, and you have to make judgment calls about what a "close enough" approximation looks like. That messiness *is* the job. An analyst who can only compute WACC when every input is handed to them in a tidy table isn't actually useful yet; one who can go find the data, flag what they had to approximate, and still produce a number they can defend, is. That's exactly the skill this project is built to force.

When done: push, then take the [quiz](../quiz.md).
