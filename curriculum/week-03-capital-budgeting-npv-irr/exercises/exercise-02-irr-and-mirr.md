# Exercise 2 — IRR and MIRR Side by Side

**Time:** ~1 hour. **Builds on:** Lecture 2. **Reuses:** your `cash_flows` extraction from Exercise 1.

## Setup

```bash
pip install numpy-financial   # if you haven't already
```

## Part A — sign-change check, before you compute anything

1. For each of the six projects, write out its cash-flow signs as a list (e.g., `[-1, +1, +1, +1, +1, +1]`) and count the number of sign **changes** (consecutive equal signs don't count).
2. Confirm: every one of this week's six projects has **exactly one sign change**. Write one sentence explaining why that means none of them can have a multiple-IRR problem, even the two projects (AI Quality-Control R&D, European Market Entry) that have *two consecutive negative* cash flows at the start.

This step matters — skipping it and blindly trusting `numpy_financial.irr()` is exactly the mistake Lecture 2, Section 5 warns about. Here it turns out to be safe, but you should never assume that; you should *check*.

## Part B — compute IRR for all six

3. Using your Exercise 1 cash-flow lists, compute `npf.irr()` for each project.
4. Build a table with columns: `project_name`, `hurdle_rate`, `irr`, `npv` (from Exercise 1), and a boolean `accept` column that's `True` exactly when `irr > hurdle_rate`.

**Expected:**

| Project | Hurdle | IRR | NPV | Accept? |
|---|---:|---:|---:|:--:|
| New CNC Machine | 7% | 20.17% | +71,712.84 | ✅ |
| Regional Warehouse Expansion | 10% | 8.60% | −25,296.02 | ❌ |
| AI Quality-Control R&D | 15% | 13.27% | −17,316.03 | ❌ |
| Brand Relaunch Campaign | 10% | 15.06% | +23,570.67 | ✅ |
| Solar Rooftop Retrofit | 6% | 2.24% | −14,581.23 | ❌ |
| European Market Entry | 18% | 8.67% | −259,242.90 | ❌ |

5. Confirm every row's `accept` column agrees between the IRR test (`irr > hurdle_rate`) and the NPV test (`npv > 0`). They should agree on **every single row** — that's Lecture 2, Section 2's guarantee for single, independent, normal-shaped projects. If any row disagrees, you have a bug (probably a mismatched hurdle rate or a mis-summed period).

## Part C — compute MIRR for all six, using the hurdle rate as both rates

6. For each project, compute `npf.mirr(cash_flows, finance_rate=hurdle_rate, reinvest_rate=hurdle_rate)`.
7. Add an `mirr` column and an `irr_minus_mirr` column (the gap) to your Part B table.

**Expected `irr_minus_mirr` gaps** (rounded to 2 decimal points, in percentage points):

| Project | IRR | MIRR | Gap |
|---|---:|---:|---:|
| New CNC Machine | 20.17% | 14.42% | 5.75 |
| Regional Warehouse Expansion | 8.60% | 9.13% | −0.53 |
| AI Quality-Control R&D | 13.27% | 13.82% | −0.55 |
| Brand Relaunch Campaign | 15.06% | 12.26% | 2.79 |
| Solar Rooftop Retrofit | 2.24% | 3.69% | −1.45 |
| European Market Entry | 8.67% | 11.01% | −2.34 |

8. Look at the sign of the gap. For the New CNC Machine and Brand Relaunch Campaign — the two positive-NPV projects — IRR **overstates** MIRR (a positive gap). For the four negative-NPV projects, MIRR is actually **higher** than IRR (a negative gap). Write two or three sentences explaining why the direction of this gap flips depending on whether the project's IRR is above or below its hurdle rate. (Hint: think about what MIRR does to interim cash flows that get reinvested at a rate *different* from the project's own IRR, in both directions.)

## Part D — the crossover you'd never see from IRR alone

9. Using the standalone example from Lecture 2 Section 4 (not the seed table), reproduce the crossover table:

   ```python
   front = [-200000, 180000, 60000, 20000]
   back  = [-200000, 20000, 60000, 240000]
   ```

   Compute IRR for both. Then compute NPV for both at `r = 0.10` and again at `r = 0.20`. Confirm the ranking flips between those two rates, and that the project with the *lower* IRR (Back-loaded) wins at the lower rate.

## Save your work

Extend `solutions.py` with a new section for this exercise. Keep the Part B/C table as a DataFrame or list of dicts — you'll append a `pi` column to the same structure in Exercise 3.
