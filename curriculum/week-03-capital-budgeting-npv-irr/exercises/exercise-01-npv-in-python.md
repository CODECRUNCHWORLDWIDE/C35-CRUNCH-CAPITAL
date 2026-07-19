# Exercise 1 — NPV Over a Project Stream

**Time:** ~1 hour. **Builds on:** Lecture 1.

## Setup

You need the `cash_flows` table from this week's [README](../README.md) seeded and reachable, and a Python environment with `pandas`, `numpy`, `numpy-financial`, and `sqlalchemy`/`psycopg[binary]` (or `sqlite3`) installed.

## Part A — write and verify your own NPV function (by hand + Python)

1. Write `npv(rate, cash_flows)` from scratch — no `numpy_financial` yet. `cash_flows[0]` is period 0.
2. Run it on the New CNC Machine's stream at its 7% hurdle rate:

   ```python
   cnc_flows = [-180000, 55000, 58000, 60000, 60000, 77000]
   ```

   **Expected:** `71712.84` (matches Lecture 1, Section 3's worked table).
3. Cross-check with `numpy_financial.npv()`. The two numbers must match to the cent. If they don't, print each individual discounted term (`cf / (1+rate)**t` for every `t`) and find where your version diverges — the usual culprits are discounting period 0, or using `t+1` instead of `t` somewhere.

## Part B — pull every project out of SQL and compute NPV in Python

4. Write a SQL query that returns `project_id`, `period`, `amount`, and `hurdle_rate` for **all six projects**, ordered by `project_id, period`.
5. In Python (pandas), group the result by `project_id`, build each project's `cash_flows` list (remember: some projects have two rows in the same period — sum them into one number for that period before calling `npv()`), and compute NPV for all six using your Part A function.
6. Print a table: `project_name`, `hurdle_rate`, `npv`, sorted by `npv` descending.

**Expected output** (rounded to the cent):

| Project | Hurdle | NPV |
|---|---:|---:|
| New CNC Machine | 7% | +71,712.84 |
| Brand Relaunch Campaign | 10% | +23,570.67 |
| AI Quality-Control R&D | 15% | −17,316.03 |
| Solar Rooftop Retrofit | 6% | −14,581.23 |
| Regional Warehouse Expansion | 10% | −25,296.02 |
| European Market Entry | 18% | −259,242.90 |

Stop and look at this table before moving on. **Only two of six projects clear their hurdle rate.** That's not a bug in your code — verify it against the CNC number above, which you already hand-checked in Part A, and if that one's right, trust the rest.

## Part C — the same computation, natively in SQL

7. Now write the equivalent as a single SQL query — no Python — using `POWER()` and `SUM()` with `GROUP BY`. It should return the same six rows, same order, same numbers.

```sql
SELECT
    project_id,
    project_name,
    hurdle_rate,
    ROUND(SUM(amount / POWER(1 + hurdle_rate, period)), 2) AS npv
FROM cash_flows
GROUP BY project_id, project_name, hurdle_rate
ORDER BY npv DESC;
```

Run it. Confirm every value matches your Part B table exactly. This is the query you'll reuse, unmodified, in Exercise 3 and the mini-project.

## Part D — undiscounted vs. discounted totals

8. For each project, also compute the plain **sum of undiscounted cash flows** (no discounting at all) — one extra column, `raw_total`.
9. Compare `raw_total` to `npv` for the Regional Warehouse Expansion specifically. Its `raw_total` should look reasonably positive (it *is* — check it), while its NPV is negative. Write one sentence explaining, in your own words, why those two numbers disagree so much for this project (hint: look at how far out its biggest cash flows sit, and how high its hurdle rate is relative to the CNC Machine's).

## Save your work

`solutions.py` (Parts A, B, D) and `solutions.sql` (Part C), each answer under a numbered comment — you'll build directly on both files in Exercise 2 and Exercise 3.
