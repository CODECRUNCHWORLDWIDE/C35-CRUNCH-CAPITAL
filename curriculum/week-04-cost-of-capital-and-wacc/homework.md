# Week 4 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hand calculation, SQL, Python, and written explanation. Commit each.

All queries and calculations run against `stock_prices`, `debt_tranches`, and `capital_structure` from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Twenty CAPM and cost-of-capital drills (90 min)

Mix of hand calculation, SQL, and Python — your choice per question, but do at least 6 by hand first with the arithmetic shown, and confirm the rest with code. Put everything in `drills.md` with the question number, method used, and answer.

1. Cost of equity: $R_f = 4.0\%$, $\beta = 1.0$, $ERP = 5.0\%$. *(A beta-1 stock should earn exactly $R_f + ERP$ — confirm that's what you get.)*
2. Cost of equity: $R_f = 4.0\%$, $\beta = 0$, $ERP = 5.0\%$. *(What does a beta-0 result confirm about CAPM's claim on undiversifiable risk?)*
3. Cost of equity: $R_f = 3.75\%$, $\beta = 1.65$, $ERP = 4.5\%$.
4. Using Crunch Machine Co.'s own $R_f$ and $ERP$, what beta would produce a cost of equity of exactly 12.00%? *(Solve algebraically, then confirm.)*
5. A bond has a 6% coupon and trades at par (100). What's its approximate YTM? *(No calculation needed — explain why in one sentence.)*
6. A bond has a 6% coupon, trades at 94, and has 4 years to maturity. Approximate its YTM using the formula from Lecture 3.
7. The same bond as #6, but with 8 years to maturity instead of 4, same price and coupon. Is the approximate YTM higher or lower than #6's? Compute it and explain the direction in one sentence.
8. After-tax cost of debt if the pre-tax YTM is 6.5% and the tax rate is 21%.
9. The same 6.5% pre-tax YTM, but at a 35% tax rate instead. Is the after-tax cost of debt higher or lower than #8's, and by how much?
10. A company has $E = \$400\text{M}$, $D = \$100\text{M}$. Compute $E/V$ and $D/V$.
11. Same company as #10, but it issues $\$100M$ of new debt and uses it entirely to buy back stock (so $E$ falls to $\$300M$, $D$ rises to $\$200M$). Recompute $E/V$ and $D/V$. Which direction did the weights move, and does that match intuition?
12. WACC: $E/V = 70\%$, $R_e = 11\%$, $D/V = 30\%$, $R_d = 6\%$ (pre-tax), $t = 25\%$.
13. Recompute #12's WACC if $R_e$ rises to 13% (say, because the market got more fearful and $ERP$ widened) — everything else unchanged. How many basis points did WACC move?
14. Recompute #12's original WACC (11%/6%/25%/70%/30%) if $D/V$ instead rises to 50% (and $E/V$ falls to 50%), holding $R_e$ and $R_d$ fixed. *(This is deliberately the same unrealistic simplification Challenge 2 flags — that's fine for this drill, just note it in one sentence.)*
15. Using `stock_prices`, compute CRNM's single best and single worst monthly return over the whole 60-month window, and the dates they occurred. *(SQL or Python — your choice.)*
16. Using `stock_prices`, compute MKT's single best and single worst monthly return over the same window. Were they the same months as CRNM's? What does that (or its absence) suggest about how correlated the two are in extreme months specifically, versus on average?
17. Using `debt_tranches`, which single tranche has the highest approximate YTM, and by how much does it exceed the lowest?
18. Using `debt_tranches`, compute total face value and total market value across all three tranches, and the dollar difference between them. Is total market value above or below total face value overall, and what does that tell you about the *average* direction rates have moved since these bonds were issued?
19. If Crunch Machine Co.'s regression beta (1.2429) is used instead of the vendor beta (1.30) in the full WACC, by how many basis points does the final WACC change? *(You computed both numbers in Exercise 3 — just report the difference here.)*
20. In one sentence: across today's drills, which single WACC input — beta, leverage ($D/V$), or the pre-tax cost of debt — produced the *largest* swing in WACC per unit of change? Does that match Lecture 3's summary table's claim about which levers a company actually controls?

---

## Problem 2 — Explain WACC to a non-finance colleague (45 min)

In `explain-writeup.md`, write no more than 400 words, in plain language (imagine a colleague from product or engineering who's never seen a discount-rate formula):

1. In one or two sentences, what is WACC actually measuring — what real-world thing does the number represent?
2. Using Crunch Machine Co.'s own numbers, explain why its cost of equity (≈10.46%) is higher than its after-tax cost of debt (≈4.17%) — what's the intuitive, non-technical reason debt is "cheaper," and what's the catch?
3. Explain, without using the word "beta," why a steadier, less-cyclical company generally has a lower cost of equity than a more cyclical one.
4. If your colleague asked "why not just use book value from the balance sheet for the E/V and D/V weights instead of market value?" — answer in two sentences.

---

## Problem 3 — Query and pipeline practice (60 min)

Mix of SQL and Python, all against this week's seed tables. Save SQL in `queries.sql`, Python in `queries.py`.

1. **SQL.** Write a query that returns all three debt tranches sorted by `years_to_mat` ascending, with a computed column `years_to_mat_bucket` that labels each row `'Short (<3y)'`, `'Medium (3-4y)'`, or `'Long (5y+)'` using a `CASE` expression.
2. **SQL.** Write a query against `stock_prices` that returns, for `CRNM` only, each `price_date` alongside the *prior* month's `adj_close`, using a self-join on `price_date` (join the table to itself, matching each row to the row exactly one month earlier — you may need a computed date expression or a window function `LAG()` if your engine supports it).
3. **Python.** Using pandas, confirm your Task 2 SQL self-join produces the same "prior month's price" values as simply calling `.shift(1)` on the pivoted `CRNM` column from Exercise 2. Print both side by side for at least 5 rows and confirm they match.
4. **Python.** Write a function `capm(rf, beta, erp)` and a function `after_tax_cost_of_debt(pretax_ytm, tax_rate)`. Write three `assert` statements testing each function against known values from this week's lectures (e.g., `assert round(capm(0.0425, 1.30, 0.05), 4) == 0.1075`). This is your first taste of testing financial code the way Lecture 3 in Week 1 argued you should — a spreadsheet cell can't do this.

---

## Problem 4 — WACC across a small industry (60 min)

Using the `peer_betas` table you built in Challenge 1 (recreate it if you skipped that challenge — the `CREATE TABLE`/`INSERT` statements are in [`challenges/challenge-01-relever-beta-across-structures.md`](./challenges/challenge-01-relever-beta-across-structures.md)):

1. For each of the four peers, assume they share Crunch Machine Co.'s $R_f$ (4.25%) and $ERP$ (5.00%), but use **their own** `levered_beta` (not unlevered — their beta as actually reported) to compute each peer's own cost of equity.
2. Assume, for simplicity, each peer's pre-tax cost of debt equals Crunch Machine Co.'s (5.56%) and each peer's $D/V$ can be approximated from their `debt_to_equity` column via $D/V = \frac{D/E}{1 + D/E}$. Compute each peer's approximate WACC.
3. Rank all five companies (the four peers plus Crunch Machine Co. itself, using its own regression-beta WACC of ≈9.16%) from lowest to highest WACC. Print the ranked table.
4. In one or two sentences: does the ranking match what you'd expect purely from each company's leverage (higher $D/E$ → you might expect a *lower* WACC, since debt is cheaper — but also a higher beta, which pushes the other way)? Which effect dominates in your ranking?

---

## Problem 5 — Reflection (30 min)

In `reflection.md`, ~250 words:

1. Which of this week's three beta estimates (vendor, regression, comps-relevered) would you trust most if you had to bet real money on Crunch Machine Co.'s WACC being right, and why?
2. Where did market-value vs. book-value weighting almost trip you up, or would have if you hadn't been paying attention?
3. Which formula (CAPM, Hamada unlever/relever, YTM approximation, or the WACC assembly itself) still feels least automatic? What will you drill this coming week?
4. Looking back at Week 3's `cash_flows.hurdle_rate` column, which project's stated hurdle rate now looks, to you, closest to — or furthest from — a WACC-based number you'd actually defend? Why?

---

*Submit all five problems' files together in your portfolio under `c35-week-04/homework/`.*
