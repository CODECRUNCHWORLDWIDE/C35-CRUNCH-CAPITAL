# Exercise 2 — Regress a Beta from Prices

**Goal:** Reproduce Lecture 2's beta regression yourself, end to end — pull prices out of SQL, compute returns in pandas, and fit the regression — without looking at the lecture code while you write it.

**Estimated time:** 1.5 hours.

## Setup

Confirm the price history is seeded:

```sql
SELECT ticker, COUNT(*), MIN(price_date), MAX(price_date)
FROM stock_prices
GROUP BY ticker;
```

You should see two rows: `CRNM` and `MKT`, each with **61** rows, spanning `2020-12-31` to `2025-12-31`.

Create `solutions.py`. Put each task under a comment with the task number.

## Tasks

1. **Connect and pull.** Connect to `crunch_capital` with SQLAlchemy, pull all of `stock_prices` into a DataFrame, and pivot it so `CRNM` and `MKT` are columns, indexed by `price_date`. Print `.shape` and `.head()`. *(Expected shape: `(61, 2)`.)*

2. **Returns.** Compute monthly returns with `.pct_change()` and drop the first (NaN) row. Print `.shape` and `.head()`. *(Expected shape: `(60, 2)`.)*

3. **Summary stats.** Print the mean and standard deviation of each column's monthly returns (`.mean()`, `.std()`). Which series has the higher average monthly return? Which has the higher standard deviation (more volatile)? Write one sentence answering both.

4. **Regress.** Use `numpy.polyfit` to regress `CRNM` returns on `MKT` returns. Print the beta (slope) and alpha (intercept), the alpha formatted as a percentage. *(Expected: beta ≈ 1.2429, alpha ≈ 0.29% monthly.)*

5. **Fit quality.** Compute the correlation between the two return series and square it to get R². Print both, and write one sentence interpreting what the R² means for how much you should trust this beta. *(Expected: R² ≈ 0.582.)*

6. **Annualize alpha (roughly).** Using the rough approximation "monthly alpha × 12 ≈ annualized alpha" (acknowledging this ignores compounding), compute and print an annualized alpha estimate. *(Expected: ≈ 3.4%.)*

7. **Plug into CAPM.** Using your regression beta from Task 4 and Crunch Machine Co.'s $R_f$ and $ERP$ from `capital_structure`, compute the cost of equity. Compare it, printed side by side, to the cost of equity from the vendor beta (1.30, from Exercise 1). *(Expected: regression-based ≈ 10.46%, vendor-based ≈ 10.75%.)*

## Expected results (spot checks)

- Task 1 shape: `(61, 2)`.
- Task 2 shape: `(60, 2)`.
- Task 4 beta: between 1.20 and 1.30 (should land almost exactly at 1.2429 if your pivot/return steps match the lecture).
- Task 5 R²: between 0.55 and 0.60.
- Task 7: regression-based cost of equity is *lower* than vendor-based, because the regression beta (1.24) is lower than the vendor beta (1.30).

## Done when…

- [ ] `solutions.py` runs top to bottom with no errors, against a freshly seeded `crunch_capital` database.
- [ ] Your Task 4 beta matches the lecture's to at least 2 decimal places.
- [ ] You can explain, without looking it up, why `.dropna()` is necessary after `.pct_change()`.
- [ ] Task 3's two answers (which series has higher return, which has higher volatility) are stated in plain English, not just printed numbers.

## Stretch

- Redo the regression using only the **most recent 24 months** of returns (a shorter, more "current" window) instead of all 60. How much does beta move? Which window would you trust more for a company whose business hasn't changed, versus one that recently took on a lot of new debt?
- If you have `statsmodels` installed, refit with `sm.OLS` and print the full `.summary()`. Find the `std err` on the beta coefficient — that's the uncertainty band around your point estimate. Would you say 1.24 and 1.30 (the vendor's beta) are meaningfully different, given that uncertainty, or within noise of each other?

## Submission

Commit `solutions.py` to your portfolio under `c35-week-04/exercise-02/`.
