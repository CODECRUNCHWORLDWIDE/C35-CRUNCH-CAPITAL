# Week 2 — Homework

Extra practice, spread across the week (~5 hours total). Four parts: conceptual short answers, extended SQL against Crunch Machine Co., a second company to contrast against it, and a short writing task. Do these after the exercises — they lean on the schema and ratios you've already built.

---

## Part A — Conceptual short answers (~1 hour)

Answer each in 2–4 sentences. No SQL required.

1. A friend tells you: "This company had a great year — net income was up 20%." Name two reasons that statement alone could still be misleading without also looking at the cash-flow statement.
2. Explain, in plain language, why accounts receivable *increasing* is a **use** of cash, even though AR itself is an asset the company now has more of.
3. A company's current ratio is exactly 1.0. Is that automatically a red flag? Under what circumstances would a current ratio of 1.0 be perfectly normal for a healthy business?
4. Two companies have identical net income and identical revenue. Company A has ROE of 25%; Company B has ROE of 10%. Name one non-profitability explanation (something about the balance sheet, not the income statement) for why their ROE could differ this much.
5. Why do analysts generally exclude banks and insurance companies from the standard liquidity-ratio framework (current ratio, quick ratio) taught this week?

## Part B — Extended queries against Crunch Machine Co. (~2 hours)

Work against the six-year dataset (FY2020–FY2025) from this week's exercises. Write each as a query in `homework.sql`.

6. **Equity multiplier** = total assets ÷ total equity. Compute it for all six years. This is the third leg of the DuPont identity (ROE = net margin × asset turnover × equity multiplier) — you're not asked to compute the full identity, just this one piece.
7. **Asset turnover** = revenue ÷ total assets. Compute it for all six years, rounded to 2 decimals. Does Crunch Machine Co.'s declining profitability show up in this ratio too, or does asset turnover stay roughly flat while margins collapse? State which, with the numbers.
8. **Fixed-charge coverage**, a stricter cousin of interest coverage: (operating income) ÷ (interest expense). You already computed this as `interest_coverage` in Exercise 3 — this time, add a second column showing what interest coverage *would have been* in FY2025 if interest expense had stayed at its FY2021 level ($620K) instead of rising to $1,380K. How much of FY2025's coverage collapse is explained by falling operating income alone, versus rising interest expense alone?
9. **Free cash flow (FCF)** = CFO − capex (use `capex` as reported, which is already negative, so `FCF = cfo + capex`). Compute FCF for all six years. Then compute **FCF ÷ dividends paid** (as a positive ratio) for all six years — in which years could the company's own free cash flow have covered its dividend without new borrowing, and in which years could it not?
10. Using `LAG()`, compute the **year-over-year percentage change in revenue** for all six years. In which year does revenue growth first turn negative, and how does that line up with the point where net margin drops below 5%?

## Part C — A second company for contrast (~1.5 hours)

Here is a two-year excerpt from a different, much healthier fictional company, **Bolt Robotics Inc.** (dollars in thousands):

| Line item | FY2024 | FY2025 |
|---|---:|---:|
| Revenue | 18,000 | 21,000 |
| COGS | 9,000 | 10,290 |
| SG&A | 4,000 | 4,410 |
| R&D | 2,000 | 2,100 |
| Interest expense | 120 | 110 |
| Net income | 2,160 | 3,067 |
| Cash | 5,000 | 7,200 |
| Accounts receivable | 2,200 | 2,500 |
| Inventory | 1,800 | 1,950 |
| Total current assets | 9,300 | 11,950 |
| Total assets | 15,800 | 18,750 |
| Accounts payable | 1,400 | 1,500 |
| Short-term debt | 300 | 200 |
| Total current liabilities | 2,100 | 2,120 |
| Long-term debt | 1,200 | 900 |
| Total liabilities | 3,500 | 3,220 |
| Total equity | 12,300 | 15,530 |

11. Load these two years into your own small `income_statement` / `balance_sheet` tables for Bolt Robotics (a fresh pair of tables or a `company` column — your choice), and compute: gross margin, operating margin, net margin, current ratio, quick ratio, and debt-to-equity for both years.
12. Write 3–5 sentences comparing Bolt Robotics' FY2025 ratios directly against Crunch Machine Co.'s FY2025 ratios (from Exercise 3). Which company would you rather lend money to, and which single ratio makes that decision easiest to defend?

## Part D — Short writing (~30 minutes)

13. In 150–250 words, explain to someone with no finance background why a company can be *profitable* (positive net income, every year) and still be in real financial danger. Use Crunch Machine Co.'s FY2025 numbers as your concrete example — it earned a $634K profit, and yet several ratios say otherwise. Don't just list ratio values; explain *why* profit alone isn't the full picture.

---

## Submission

Commit `homework.sql` (Parts B and C's SQL) and `writeup.md` (Parts A, C.12, and D's written answers) to your portfolio under `c35-week-02/homework/`.
