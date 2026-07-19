# Week 2 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 3. A mix of multiple-choice and short "compute this" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** Which of the three financial statements is a "snapshot," reporting figures as of one exact date rather than over a period?

- A) Income statement
- B) Balance sheet
- C) Cash-flow statement
- D) All three are snapshots

---

**Q2.** The balance sheet identity `Assets = Liabilities + Equity` holds:

- A) Only for healthy companies
- B) Only after the accountants "plug" a balancing entry
- C) For every company, on every date, by construction — always
- D) Only in the United States; other countries use a different identity

---

**Q3.** Gross profit is calculated as:

- A) Revenue − SG&A
- B) Revenue − COGS
- C) Revenue − COGS − SG&A − R&D
- D) Net income + interest expense + tax expense

---

**Q4.** Depreciation & amortization is added back in the operating section of the cash-flow statement because:

- A) It was never actually a real expense
- B) It's an accounting expense that reduced net income, but no cash left the company for it *this* period — the cash went out when the asset was originally purchased
- C) The IRS requires it
- D) It offsets an increase in accounts payable

---

**Q5.** If accounts receivable *increases* by $900K during the year, the cash-flow statement's operating section shows:

- A) A $900K positive adjustment (a source of cash)
- B) A $900K negative adjustment (a use of cash)
- C) No adjustment — AR doesn't affect cash flow
- D) A $900K adjustment to investing activities, not operating

---

**Q6.** A company's retained earnings were $23,850K at the start of the year. Net income for the year was $4,882K, and dividends paid were $5,582K. What is retained earnings at the end of the year?

- A) $34,314K
- B) $23,150K
- C) $28,732K
- D) $13,386K

---

**Q7.** In financial-statement terms, "articulation" means:

- A) The statements are written in clear, jargon-free language
- B) The three statements connect and reconcile with each other — the same figures appear consistently across them
- C) The auditor has signed off on the filing
- D) The company uses GAAP instead of IFRS

---

**Q8.** A company's current ratio has fallen from 2.68 to 1.73 over five years, but is still above 1.0 every year. The correct read is:

- A) No concern at all — it's still above 1.0, full stop
- B) Automatic distress — any decline in the current ratio means bankruptcy is near
- C) A meaningful warning sign — the *direction and pace* of the decline matters, even though the level hasn't crossed 1.0 yet
- D) Meaningless — the current ratio only matters for banks

---

**Q9.** The quick ratio differs from the current ratio by:

- A) Adding long-term debt to the numerator
- B) Excluding inventory from current assets in the numerator
- C) Using net income instead of current assets
- D) They are the same ratio with different names

---

**Q10.** Interest coverage = operating income ÷ interest expense. A ratio of 1.6× means:

- A) The company earns 1.6 times its total revenue in operating income
- B) Operating income covers this year's interest expense only 1.6 times over — a thin cushion if operating income drops further
- C) The company pays 1.6% interest on its debt
- D) The company's debt-to-equity ratio is 1.6

---

**Q11.** Debt-to-equity crossing above 1.0 specifically means:

- A) The company is bankrupt
- B) Total debt now exceeds total equity — lenders have a larger claim on the assets than the owners do
- C) The company can no longer pay dividends
- D) The interest rate on the company's debt exceeds 1%

---

**Q12.** A company has accounts receivable of $7,600K and annual revenue of $44,500K. Its days sales outstanding (DSO) is approximately:

- A) 5.9 days
- B) 17.1 days
- C) 62.3 days
- D) 170.5 days

---

**Q13.** The cash conversion cycle is calculated as:

- A) DSO + DIO + DPO
- B) DSO + DIO − DPO
- C) DPO − DSO − DIO
- D) (DSO + DIO + DPO) ÷ 3

---

**Q14.** Why does this course insist on reading a ratio's five-year *trend* rather than a single period's value?

- A) A single ratio value is always calculated incorrectly
- B) Regulators require multi-year reporting
- C) A ratio's absolute level can look fine in isolation while its direction reveals a company steadily getting healthier or steadily deteriorating — the trend carries information a snapshot can't
- D) Trends are easier to compute in SQL than single values

---

**Q15.** A company's ROE *rises* even as its net income falls, because its total equity is shrinking faster than its net income. What should you check next before concluding the company is doing better?

- A) Nothing — a rising ROE always means things are improving
- B) The dividend payout ratio only
- C) Leverage (debt-to-equity) and the reason equity is shrinking — rising ROE can be an artifact of increasing debt rather than genuine improvement
- D) The company's stock price

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — the balance sheet reports figures "as of" one date. The income statement and cash-flow statement both report "over" a period — they're flows, not snapshots.
2. **C** — `Assets = Liabilities + Equity` is a structural identity of double-entry accounting. It's not something that happens to be true for healthy companies; it's true by construction for every company, always, on every date.
3. **B** — gross profit = revenue − COGS, full stop. SG&A and R&D are subtracted later, to get operating income.
4. **B** — D&A is a real accounting expense (it reduces net income), but the cash left the building years ago when the asset was purchased. Adding it back converts net income toward actual cash generated this period.
5. **B** — an AR increase means revenue was booked but not yet collected in cash — a use of cash, shown as a negative adjustment.
6. **C** — $23,850 + $4,882 − $5,582 = $23,150. (Option A adds instead of subtracting dividends; option D subtracts net income instead of adding it.)
7. **B** — articulation means the three statements' figures reconcile with each other: net income flows into retained earnings, ending cash on the cash-flow statement matches the balance sheet's cash line, etc.
8. **C** — this is the core lesson of Lecture 3. A ratio's *direction and pace* of movement is often more informative than whether it's crossed some absolute threshold yet. Waiting for the threshold to be crossed before acting means noticing the trouble years after it started.
9. **B** — the quick ratio strips inventory out of the numerator because inventory is the least liquid current asset — it must be sold before it becomes cash.
10. **B** — interest coverage measures how many times operating income could pay this year's interest bill. 1.6× is a thin, concerning cushion; most lenders get nervous below roughly 2×–3×.
11. **B** — debt-to-equity above 1.0 means total debt (short-term + long-term) exceeds total equity — lenders' claim on the company's assets is now larger than the owners'.
12. **C** — DSO = AR ÷ revenue × 365 = 7,600 ÷ 44,500 × 365 ≈ 62.3 days.
13. **B** — cash conversion cycle = DSO + DIO − DPO. Days to collect, plus days inventory sits, minus days you delay paying your own suppliers.
14. **C** — a ratio's level can look acceptable in isolation while its multi-year direction shows a company steadily improving or steadily deteriorating; the trend carries information a single snapshot structurally cannot.
15. **C** — ROE = net income ÷ equity. If equity shrinks (often because losses or heavy dividends erode retained earnings, or leverage rises), ROE can rise even as the underlying business weakens. Always read ROE next to leverage, not alone.

</details>

**Scoring:** 12+ → start Week 3. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; ratio analysis compounds directly into valuation next week.
