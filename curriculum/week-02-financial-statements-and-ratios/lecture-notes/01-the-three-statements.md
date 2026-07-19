# Lecture 1 — The Three Statements

> **Duration:** ~2 hours. **Outcome:** You can open any company's income statement, balance sheet, and cash-flow statement and correctly explain what every major line means, what period or point in time it covers, and how it differs from the other two statements — using Crunch Machine Co.'s real five-year filings as your working example.

## 1. Three statements, three different questions

Every public company on earth publishes the same three core financial statements, and every one of them answers a different question:

| Statement | Answers |
|---|---|
| **Income statement** (a.k.a. P&L, statement of operations) | "How much did the company *earn* over this period?" |
| **Balance sheet** (a.k.a. statement of financial position) | "What does the company *own and owe*, as of this exact date?" |
| **Cash-flow statement** | "Where did the company's *actual cash* come from and go, over this period?" |

That distinction — **period vs. point in time** — is the single most important idea in this lecture. The income statement and cash-flow statement both cover a *span* (a quarter, a fiscal year). The balance sheet is a *snapshot*, frozen at one instant: the close of business on the last day of that period. When Crunch Machine Co.'s FY2025 balance sheet says `cash = 1600`, that's not "1600 was the average during 2025" — it's "on December 31, 2025, there was exactly $1,600,000 in the bank." Confuse period and point-in-time and every ratio you build this week will be subtly wrong.

## 2. The income statement — how much did we earn?

The income statement walks from **revenue** down to **net income** in one direction: subtract costs, in order, until nothing is left but profit (or loss). Here's Crunch Machine Co.'s FY2021 income statement, in thousands of dollars:

```
Revenue                    42,000
  Cost of goods sold      (26,040)
Gross profit                15,960
  SG&A expense             (7,560)
  R&D expense               (1,260)
Operating income              7,140
  Interest expense            (620)
Pretax income                6,520
  Tax expense                (1,630)
Net income                    4,890
```

Walk it top to bottom:

- **Revenue** — money earned from selling the product, recognized when the sale happens (not necessarily when cash is collected — more on that in Lecture 2). This is the top line, and everything below it is a subtraction from it.
- **Cost of goods sold (COGS)** — the direct cost of making what was sold: materials, direct labor, factory overhead. Revenue minus COGS is **gross profit**, and gross profit ÷ revenue is **gross margin** — the cleanest read on pricing power and production efficiency, because it's untouched by how the company spends on marketing, R&D, or debt.
- **SG&A (selling, general & administrative)** — sales salaries, marketing, rent, admin overhead. The cost of *running* the business, as opposed to *making* the product.
- **R&D (research & development)** — money spent building the next product. Some companies fold this into SG&A; Crunch Machine Co. breaks it out, which is common for manufacturers.
- **Operating income** (a.k.a. EBIT — earnings before interest and taxes) — gross profit minus SG&A and R&D. This is the profit from the *core business*, before you touch financing or tax decisions. It's the number analysts trust most for comparing two companies, because it strips out how each one happens to be financed or taxed.
- **Interest expense** — the cost of the company's debt. This is a financing decision, not an operating one, which is exactly why it sits below operating income rather than above it.
- **Pretax income** — operating income minus interest expense (plus any other non-operating items, which Crunch Machine Co. doesn't have this week).
- **Tax expense** — pretax income × the effective tax rate (25% throughout Crunch Machine Co.'s history).
- **Net income** — the bottom line. What's actually left for shareholders, either paid out as dividends or retained in the business.

Query it directly:

```sql
SELECT fiscal_year, revenue, gross_profit,
       ROUND(gross_profit / revenue * 100, 1) AS gross_margin_pct,
       operating_income,
       ROUND(operating_income / revenue * 100, 1) AS operating_margin_pct,
       net_income,
       ROUND(net_income / revenue * 100, 1) AS net_margin_pct
FROM income_statement
ORDER BY fiscal_year;
```

Run that now. You'll see gross margin fall from 38.0% in 2021 to 29.0% in 2025, and net margin collapse from 11.6% to 1.4% — the first hard evidence of the story this week's data is telling. Hold that thought; Lecture 3 gives you the vocabulary to talk about it precisely.

## 3. The balance sheet — what do we own and owe?

The balance sheet is built on one identity that can never be violated, for any company, on any date:

```
Assets = Liabilities + Equity
```

Read it as: *everything the company owns (assets) was paid for either by borrowing (liabilities) or by the owners' own money (equity).* Every dollar of assets has a source, and the balance sheet just shows both sides of that same dollar. This is why it's called a "balance" sheet — the two sides are mathematically guaranteed to balance, always, by construction (you'll prove this in SQL in Exercise 2).

Crunch Machine Co.'s FY2021 balance sheet, in thousands:

```
ASSETS                              LIABILITIES
Cash                       7,200    Accounts payable            3,400
Accounts receivable        4,600    Short-term debt              1,800
Inventory                   6,000    Other current liabilities   1,750
Other current assets          800    Total current liabilities    6,950
Total current assets       18,600
                                     Long-term debt               9,500
PP&E, net                  24,600    Other LT liabilities            900
Other LT assets              3,000    Total liabilities            17,350
Total assets                46,200
                                     EQUITY
                                     Common stock                 5,000
                                     Retained earnings            23,850
                                     Total equity                 28,850

                                     Total liabilities + equity   46,200
```

Assets are grouped **current** (expected to turn into cash, or be used up, within a year) and **long-term**:

- **Cash** — literally cash and cash equivalents on hand.
- **Accounts receivable (AR)** — money customers owe for goods already delivered but not yet paid for. This is revenue that's been *earned* but not yet *collected* — the single biggest source of the income-statement-vs-cash-flow gap you'll dig into in Lecture 2.
- **Inventory** — unsold product sitting in a warehouse. Cash that's been spent but hasn't turned back into cash yet.
- **PP&E (property, plant & equipment), net** — factories, machines, buildings, minus accumulated depreciation. "Net" means after subtracting the wear-and-tear the asset has already absorbed.

Liabilities are grouped the same way — **current** (due within a year) and **long-term**:

- **Accounts payable (AP)** — money the company owes its own suppliers, not yet paid. The mirror image of AR, but on the liability side.
- **Short-term debt** — borrowing due within a year (a revolving credit line, the current portion of a term loan).
- **Long-term debt** — borrowing due beyond a year (bonds, term loans).

Equity is what's left after subtracting liabilities from assets — the owners' stake:

- **Common stock** — capital the company raised by selling shares, at the price shares were sold at (not today's market price).
- **Retained earnings (RE)** — the cumulative sum of every dollar of net income the company has ever earned, minus every dollar it has ever paid out as dividends, since the day it was founded. This is the line that connects the balance sheet to the income statement — the whole subject of Lecture 2.

Query the balance sheet's shape across all five years:

```sql
SELECT fiscal_year,
       total_current_assets, total_assets,
       total_current_liabilities, total_liabilities,
       total_equity
FROM balance_sheet
ORDER BY fiscal_year;
```

Notice total assets barely move (46,200 → 46,350) while total liabilities climb steadily (17,350 → 27,340) and total equity falls just as steadily (28,850 → 19,010). Same total pie, but the slice financed by debt is growing and the slice financed by the owners is shrinking. That's leverage rising — you'll put a number on it in Lecture 3.

## 4. The cash-flow statement — where did the cash actually go?

The income statement can say a company earned $4.9 million and still be lying about how much cash is in the bank — not through fraud, just through **accrual accounting**: revenue and expenses are recorded when they're *earned* or *incurred*, not when cash actually changes hands. A sale on credit shows up as revenue today even if the customer pays in 60 days. The cash-flow statement exists specifically to answer the question the income statement can't: **did the cash actually move?**

It's split into three sections, each a different *kind* of cash movement:

```
OPERATING ACTIVITIES
Net income                              4,890
  + Depreciation & amortization          2,400
  + Change in accounts receivable         (400)
  + Change in inventory                   (500)
  + Change in accounts payable               200
  + Change in other working capital           50
Cash from operations (CFO)                6,640

INVESTING ACTIVITIES
  Capital expenditures (capex)          (3,000)
Cash from investing (CFI)               (3,000)

FINANCING ACTIVITIES
  Net debt issued (repaid)                  800
  Dividends paid                        (5,240)
Cash from financing (CFF)               (4,440)

Net change in cash                        (800)
Cash, beginning of year                  8,000
Cash, end of year                        7,200
```

**Operating activities (CFO)** start from net income and adjust it back to cash:

- **+ Depreciation & amortization (D&A)** — D&A is an *accounting* expense on the income statement (it reduced net income), but no cash actually left the building this year for it — the cash went out years ago when the asset was bought. So it gets added back.
- **+ Change in AR** — if AR *increased*, the company booked revenue it hasn't collected yet, so that increase is subtracted from cash (shown here as a negative adjustment). If AR *decreased*, customers paid down old balances — that's a cash inflow.
- **+ Change in inventory** — if inventory *increased*, cash was spent building stock that hasn't sold — a use of cash. If it *decreased*, the company sold down existing stock without needing to spend cash to replace it — a source.
- **+ Change in AP** — the mirror image: if AP *increased*, the company is holding onto cash longer before paying its own suppliers — a source of cash.

**Investing activities (CFI)** cover money spent on or received from long-term assets — mainly **capex** (buying equipment, buildings), and occasionally selling old assets or making acquisitions. This week Crunch Machine Co. only has capex.

**Financing activities (CFF)** cover money moving between the company and its capital providers — lenders and shareholders: debt issued or repaid, dividends paid, shares issued or bought back.

Sum the three sections and you get the **net change in cash** for the period — and beginning cash plus that change must equal ending cash, which (as you'll prove in Lecture 2 and Exercise 2) must also equal the cash figure reported on the balance sheet.

Query it:

```sql
SELECT fiscal_year, cfo, cfi, cff, net_change_in_cash, cash_beginning, cash_ending
FROM cash_flow_statement
ORDER BY fiscal_year;
```

Look closely at 2024 and 2025: `cff` turns *positive* even while `dividends_paid` stays deeply negative. That only happens if `debt_issued_repaid_net` is large enough to outweigh the dividend — meaning the company is borrowing more than it's paying out. Keep that observation; it's the seed of Challenge 2.

## 5. How the three statements are laid out differently — and why

A subtlety worth naming explicitly: the income statement and cash-flow statement are **flow** statements — every number is "over the year." The balance sheet is a **stock** statement — every number is "as of the date." Finance people sometimes say "flows connect stocks": the income statement (a flow) explains *why* retained earnings (a stock on the balance sheet) changed from one year to the next; the cash-flow statement (a flow) explains *why* cash (a stock on the balance sheet) changed from one year to the next. That's not a coincidence — it's the whole reason the three statements have to agree with each other, which is exactly what Lecture 2 proves.

## 6. Reading order in practice

When you open a real company's filing, most analysts read in this order, not the order the filing presents them:

1. **Income statement first** — get the headline: is revenue growing, and are margins expanding or shrinking?
2. **Cash-flow statement second** — sanity-check the income statement against real cash. A company can report a profit and still be bleeding cash (rising AR, ballooning inventory) — that gap is often where trouble first shows up, well before it hits the income statement.
3. **Balance sheet last** — see the accumulated state: how much debt has piled up, how much cash cushion is left, whether equity is growing or eroding.

That's also the order this week is built in: Lecture 1 (read all three), Lecture 2 (see how they connect), Lecture 3 (turn them into ratios that tell you, precisely, whether the trend is good or bad).

## 7. Check yourself

- In one sentence each, what question does the income statement answer, what question does the balance sheet answer, and what question does the cash-flow statement answer?
- Why is the balance sheet called a "snapshot" while the other two are called "flows"?
- A company's D&A is $2,950K this year. Does that number represent cash leaving the company this year? Why does it get added back in the cash-flow statement?
- Look at Crunch Machine Co.'s FY2025 income statement and FY2025 cash-flow statement. Net income is $634K, but CFO is $1,894K — nearly three times higher. Name two line items on the cash-flow statement that explain most of that gap.
- Why must `Assets = Liabilities + Equity` hold on *every* balance sheet, for *every* company, with no exceptions?

## Further reading

- **SEC — Beginners' Guide to Financial Statements:** <https://www.investor.gov/introduction-investing/investing-basics/how-stock-markets-work/beginners-guide-financial>
- **SEC EDGAR — full-text search for real 10-K filings:** <https://www.sec.gov/edgar/search/>
- **FASB — Statement of Cash Flows (ASC 230) overview:** <https://asc.fasb.org/230/>
