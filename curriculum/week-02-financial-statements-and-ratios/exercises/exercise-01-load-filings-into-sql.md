# Exercise 1 — Load Filings Into Normalized Tables

**Goal:** Take a raw filing excerpt — written the way a real annual report actually reads, with real-world line-item labels instead of your tidy column names — and turn it into correct `INSERT` statements against the three normalized tables you built in the week README. This is the unglamorous skill underneath every analyst's SQL dashboard: someone hands you numbers in prose, tables, or a PDF, and you're the one who has to map them onto a clean schema without dropping or double-counting anything.

**Estimated time:** 60 minutes.

## Setup

Confirm your three tables from the week README exist and hold FY2021–FY2025:

```sql
SELECT MIN(fiscal_year), MAX(fiscal_year), COUNT(*) FROM income_statement;
-- expect: 2021, 2025, 5
```

This exercise adds **FY2020** — the year immediately before the data you already have. Once it's loaded, every year-over-year query in this week's lectures and remaining exercises (which use `LAG()`) has a real prior year for FY2021 to compare against, instead of a `NULL`.

## The raw filing

Below is Crunch Machine Co.'s FY2020 annual report excerpt, condensed to the line items you need. It is **not** in your schema's column order, and it uses the labels a real filing would use, not the column names you defined. All figures are in thousands of dollars. Parenthesized figures are negative (a use of cash).

> **Crunch Machine Co. — Fiscal Year 2020 (year ended December 31, 2020)**
>
> **From the income statement:**
> Net sales, $40,000. Cost of revenue, $24,400. Selling, general and administrative expenses, $6,800. Research and development expenses, $1,200. Interest expense, net, $560. Provision for income taxes, $1,760 (effective tax rate: 25%, consistent with every other year on file).
>
> **From the balance sheet, as of December 31, 2020:**
> Cash and cash equivalents, $8,000. Trade receivables, net, $4,200. Inventories, $5,500. Prepaid expenses and other current assets, $800. Property, plant and equipment, net, $24,000. Goodwill and other long-term assets, $3,000. Accounts payable, $3,200. Current portion of long-term debt, $1,500. Accrued and other current liabilities, $1,700. Long-term debt, $9,000. Deferred tax and other long-term liabilities, $900. Common stock and additional paid-in capital, $5,000. Retained earnings, $24,200.
>
> **From the statement of cash flows, year ended December 31, 2020:**
> Depreciation and amortization, $2,300. Increase in trade receivables, $(350). Increase in inventories, $(450). Increase in accounts payable, $200. Other working capital changes, $50. Purchases of property and equipment, $(3,200). Net borrowings under revolving credit facility, $2,700. Dividends paid to shareholders, $(4,530). Cash and cash equivalents, beginning of year, $6,000.

## Tasks

Create a working file `solutions.sql`. Work through these in order — later tasks depend on subtotals from earlier ones.

1. **Map the income statement.** From the raw figures above, identify which filing label maps to which of your `income_statement` columns (`revenue`, `cogs`, `sga_expense`, `rd_expense`, `interest_expense`, `tax_expense`). Note: "net sales" → `revenue`, "cost of revenue" → `cogs`.

2. **Compute the missing subtotals.** The filing doesn't hand you `gross_profit`, `operating_income`, `pretax_income`, or `net_income` directly — compute each from the line items you mapped in Task 1, the same way Lecture 1 walked the income statement top to bottom.

3. **Map and total the balance sheet.** Map each raw label to your `balance_sheet` columns, then compute `total_current_assets`, `total_assets`, `total_current_liabilities`, `total_liabilities`, and `total_equity` (`common_stock + retained_earnings`).

4. **Verify the balance sheet balances** — before you insert anything, confirm by hand (or a scratch calculation) that `total_assets = total_liabilities + total_equity`. If it doesn't, you mis-mapped or mis-summed something in Task 3 — find it now, not after loading bad data.

5. **Map and total the cash-flow statement.** Map each raw label to your `cash_flow_statement` columns, compute `cfo` (sum of `net_income`, `depreciation_amortization`, and the three working-capital change lines), `cfi` (just `capex` this year), `cff` (`debt_issued_repaid_net + dividends_paid`), `net_change_in_cash` (`cfo + cfi + cff`), and `cash_ending` (`cash_beginning + net_change_in_cash`).

6. **The articulation check.** Compare the `cash_ending` you just computed in Task 5 to the `cash` figure you mapped onto the balance sheet in Task 3. They must match — this is the same check Lecture 2 taught you to run in SQL, except this time you're doing it by hand, before the row ever enters the database, exactly the way a careful analyst double-checks a filing before trusting it.

7. **Write and run the three `INSERT` statements** — one row into each of `income_statement`, `balance_sheet`, and `cash_flow_statement`, for `fiscal_year = 2020`, using the values you computed above.

8. **Confirm the load.** Run:

   ```sql
   SELECT MIN(fiscal_year), MAX(fiscal_year), COUNT(*) FROM income_statement;
   -- expect: 2020, 2025, 6
   ```

   and repeat for `balance_sheet` and `cash_flow_statement`.

## Expected result (spot checks)

- Task 2 → `gross_profit = 15600`, `operating_income = 7600`, `pretax_income = 7040`, `net_income = 5280`.
- Task 3 → `total_current_assets = 18500`, `total_assets = 45500`, `total_current_liabilities = 6400`, `total_liabilities = 16300`, `total_equity = 29200`.
- Task 4 → `45500 = 16300 + 29200` ✓.
- Task 5 → `cfo = 7030`, `cfi = -3200`, `cff = -1830`, `net_change_in_cash = 2000`, `cash_ending = 8000`.
- Task 6 → `8000 = 8000` ✓ — the filing articulates.
- Task 8 → all three tables report `MIN = 2020`, `MAX = 2025`, `COUNT = 6`.

## Done when…

- [ ] `solutions.sql` contains your worked subtotal calculations (as comments) above each `INSERT`, not just the final `INSERT` statements — show the arithmetic, not just the answer.
- [ ] All three `INSERT` statements ran without error.
- [ ] Every count in Task 8 reads `6`.
- [ ] You can explain, in one sentence, why Task 6's check matters before you trust any ratio computed from this row.

## Stretch

- Re-run the two articulation queries from [Lecture 2](../lecture-notes/02-how-the-statements-link.md) (the `re_check` CTE and the cash-tie query) now that FY2020 is loaded — FY2021's `LAG()` should now resolve to a real prior year instead of `NULL`. Confirm FY2021 now shows `ties_out = true` on the retained-earnings check too.
- The filing text didn't give you a `change_other_wc` value that traces to a specific balance-sheet line the way `change_ar`, `change_inventory`, and `change_ap` do. In one or two sentences, explain what kinds of real balance-sheet items typically hide inside an "other working capital changes" line on a real 10-K.

## Submission

Commit `solutions.sql` to your portfolio under `c35-week-02/exercise-01/`.
