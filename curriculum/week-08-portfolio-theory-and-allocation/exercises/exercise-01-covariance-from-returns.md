# Exercise 1 — Build a Covariance Matrix

**Goal:** Reproduce Lecture 1's return, variance, and covariance calculations yourself, end to end, for the full six-fund universe — without looking at the lecture code while you write it.

**Estimated time:** 1.5 hours.

## Setup

Confirm the price history is seeded:

```sql
SELECT ticker, COUNT(*), MIN(price_date), MAX(price_date)
FROM asset_prices
GROUP BY ticker
ORDER BY ticker;
```

You should see seven tickers (`CRNQ`, `CRNI`, `CRNB`, `CRNH`, `CRNG`, `CRNR`, `MKT`), each with **61** rows, spanning `2020-12-31` to `2025-12-31`.

Create `solutions.py`. Put each task under a comment with the task number.

## Tasks

1. **Connect and pivot.** Connect to `crunch_capital` with SQLAlchemy, pull all of `asset_prices` into a DataFrame, and pivot it so each ticker is a column, indexed by `price_date`. Print `.shape` and `.head()`. *(Expected shape: `(61, 7)`.)*

2. **Isolate the investable universe.** Build a second DataFrame containing only the six investable tickers (`CRNQ`, `CRNI`, `CRNB`, `CRNH`, `CRNG`, `CRNR`) — leave `MKT` out of this one; you'll use it separately in Exercise 3's stretch goal and in Lecture 3. Print `.shape`. *(Expected: `(61, 6)`.)*

3. **Returns.** Compute monthly returns for the six-fund DataFrame with `.pct_change()` and drop the first (`NaN`) row. Print `.shape`. *(Expected: `(60, 6)`.)*

4. **Annualized mean and volatility.** For each of the six funds, compute the annualized mean return (`monthly mean × 12`) and annualized volatility (`monthly std × √12`). Print both as a table, sorted by volatility ascending. Which fund has the lowest volatility? Which has the highest annualized mean return?

5. **The full covariance matrix.** Compute the $6\times6$ **annualized** covariance matrix (`.cov() * 12`). Print it rounded to 4 decimal places. Confirm the diagonal entries equal each fund's own annualized variance (compare against your Task 4 volatilities, squared) — print both side by side to show they match.

6. **The correlation matrix.** Compute the $6\times6$ correlation matrix. Print it rounded to 3 decimal places. Find the single highest off-diagonal correlation (two *different* funds) and the single lowest. Name both pairs and their correlation values.

7. **A hand-check on one pair.** Pick any two funds. Using only their individual volatilities (from Task 4) and their correlation (from Task 6), reconstruct their covariance with `Cov = ρ × σ_1 × σ_2` and confirm it matches the corresponding cell in your Task 5 covariance matrix (within rounding).

## Expected results (spot checks)

- Task 1 shape: `(61, 7)`. Task 2 shape: `(61, 6)`. Task 3 shape: `(60, 6)`.
- Task 4: `CRNB` should have the lowest annualized volatility (≈ 5.7%); `CRNQ` and `CRNI` should be roughly tied for the highest annualized mean return, both near 13.8%.
- Task 6: the highest correlation should be between `CRNQ` and `CRNI` (≈ +0.77); the lowest should be between `CRNQ` (or `CRNI`) and `CRNB` (a negative number, roughly −0.22 to −0.28).
- Task 7: your hand-reconstructed covariance should match the matrix cell to at least 3 decimal places.

## Done when…

- [ ] `solutions.py` runs top to bottom with no errors, against a freshly seeded `crunch_capital` database.
- [ ] Your $6\times6$ covariance matrix is symmetric (`Cov[i][j] == Cov[j][i]`) — print a check confirming this, not just an assertion.
- [ ] You can state, without looking it up, why the diagonal of a covariance matrix always equals each asset's own variance.
- [ ] Task 6's answer names the actual ticker pairs, not just the numeric extremes.

## Stretch

- Compute the covariance matrix a second way: manually, using `np.cov(returns.values, rowvar=False)` on the raw numpy array instead of pandas' `.cov()`. Confirm the two approaches agree (watch for the `ddof` default difference between `numpy.cov` — which defaults to `ddof=1` inside `np.cov`, same as pandas — and don't assume without checking).
- Add `MKT` back into a 7-asset correlation matrix and find which of the six investable funds has the *lowest* correlation with the market benchmark. Does that match what you'd expect from a low-beta asset (you'll formalize this in Lecture 3)?

## Submission

Commit `solutions.py` to your portfolio under `c35-week-08/exercise-01/`.
