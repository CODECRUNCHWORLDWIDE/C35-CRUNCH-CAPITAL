# Exercise 3 — Rank Projects Under a Budget

**Time:** ~1 hour. **Builds on:** Lecture 3. **Reuses:** your Exercise 1/2 cash-flow data and NPV numbers.

## Part A — profitability index for all six

1. Write `profitability_index(cash_flows, rate)` per Lecture 3, Section 4.
2. Compute it for all six projects. Add a `pi` column to your running table from Exercise 2.

**Expected:**

| Project | Initial investment | NPV | PI |
|---|---:|---:|---:|
| New CNC Machine | 180,000 | +71,712.84 | 1.398 |
| Regional Warehouse Expansion | 650,000 | −25,296.02 | 0.961 |
| AI Quality-Control R&D | 300,000 | −17,316.03 | 0.942 |
| Brand Relaunch Campaign | 220,000 | +23,570.67 | 1.107 |
| Solar Rooftop Retrofit | 140,000 | −14,581.23 | 0.896 |
| European Market Entry | 900,000 | −259,242.90 | 0.712 |

3. Confirm: **every project with PI > 1 has NPV > 0, and every project with PI < 1 has NPV < 0.** They're the same test in different units — if any row breaks this pattern, you have a bug.

## Part B — screen, then rank

4. Filter your table down to `npv > 0`. You should have exactly **two** rows left: New CNC Machine and Brand Relaunch Campaign.
5. Sort the survivors by PI, descending. New CNC Machine should be first.

Do this in SQL too — one query, `WHERE` then `ORDER BY`, reusing Exercise 1 Part C's query as a CTE:

```sql
WITH project_summary AS (
    SELECT
        project_id,
        project_name,
        -MIN(amount) FILTER (WHERE period = 0) AS initial_investment,
        SUM(amount / POWER(1 + hurdle_rate, period))
            FILTER (WHERE period > 0) AS pv_future_flows,
        SUM(amount / POWER(1 + hurdle_rate, period)) AS npv
    FROM cash_flows
    GROUP BY project_id, project_name
)
SELECT
    project_name,
    initial_investment,
    ROUND(npv, 2) AS npv,
    ROUND(pv_future_flows / initial_investment, 3) AS pi
FROM project_summary
WHERE npv > 0
ORDER BY pi DESC;
```

## Part C — fund under two budgets

6. Write a Python function `fund_under_budget(ranked_projects, budget)` that walks the PI-sorted survivor list top to bottom, adding a project's `initial_investment` to a running total and its `npv` to a running value total, **skipping** any project that would push the running total over budget (don't stop at the first one that doesn't fit — keep checking the rest of the list, in case a smaller project further down still fits).
7. Run it at **budget = $250,000**. Expected: only the New CNC Machine gets funded ($180,000 spent, $70,000 unused, total value captured = $71,712.84).
8. Run it again at **budget = $450,000**. Expected: both projects get funded ($400,000 spent, $50,000 unused, total value captured = $95,283.51).
9. Run it a third time at **budget = $200,000**. Expected: same as step 7 — the New CNC Machine still fits ($180,000 ≤ $200,000), Brand Relaunch still doesn't ($220,000 > $200,000 remaining), so the outcome doesn't change even though the budget did.

## Part D — brute-force sanity check

10. With only two surviving projects, there are exactly **four** possible funding combinations: neither, CNC only, Brand Relaunch only, or both. Write a short loop (or just four `if` branches) that checks the total cost and total NPV of each combination against the $250,000 budget, and confirm the PI-ranked greedy answer from Part C, step 7 (CNC only) is in fact the **best of the four** feasible options — not just *a* feasible option.
11. In one sentence, explain why a brute-force check is realistic here (only two survivors → four combinations) but would stop being realistic if a slate had, say, twenty surviving projects (2²⁰ = 1,048,576 combinations) — and what you'd reach for instead at that scale (a pointer, not a full solution — this comes up again in later weeks).

## Save your work

Finish `solutions.py` (Parts A, C, D) and `solutions.sql` (Part B). This file becomes the backbone of the mini-project — don't discard it.
