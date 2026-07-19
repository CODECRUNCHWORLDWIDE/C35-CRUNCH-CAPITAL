# Exercise 2 — Trace the Efficient Frontier

**Goal:** Reproduce Lecture 2's closed-form mean-variance optimization yourself — build the $A, B, C, D$ scalars, solve for a portfolio at any target return, and trace the full efficient frontier across a grid of targets — without looking at the lecture code while you write it.

**Estimated time:** 1.5 hours.

## Setup

Reuse (or rebuild) your six-fund returns DataFrame from Exercise 1. You'll need `mu` (the annualized mean-return vector) and `Sigma` (the annualized covariance matrix) as numpy arrays.

```python
port_tickers = ["CRNQ", "CRNI", "CRNB", "CRNH", "CRNG", "CRNR"]
mu = returns[port_tickers].mean().values * 12
Sigma = returns[port_tickers].cov().values * 12
```

Create `solutions.py`. Put each task under a comment with the task number.

## Tasks

1. **The four scalars.** Compute `Sigma_inv = np.linalg.inv(Sigma)`, then `A = ones @ Sigma_inv @ ones`, `B = ones @ Sigma_inv @ mu`, `C = mu @ Sigma_inv @ mu`, `D = A*C - B**2`. Print all four. *(Sanity check: `D` should be a positive number — if it's zero or negative, something upstream is wrong; a real 6-asset covariance matrix built from 60 months of genuinely different assets will not produce `D <= 0`.)*

2. **Write `frontier_portfolio(target_return)`.** A function that takes a target annual return and returns `(weights, volatility)` using the formula from Lecture 2 (`λ`, `γ`, then `w = λ·Σ⁻¹1 + γ·Σ⁻¹μ`). Test it once at `target_return = 0.08` and print the weights (they should sum to 1 — print the sum as a check) and the resulting volatility.

3. **Trace the frontier.** Build a DataFrame with one row per target return, for `target_return` ranging from `0.03` to `0.14` in steps of `0.005` (23 points). Each row should have the target return and the corresponding frontier volatility from your Task 2 function. Print the full table.

4. **Find the vertex.** From your Task 3 table, find the row with the *minimum* volatility. Report its target return and volatility. How close is this to the GMV portfolio's volatility from Lecture 2 (≈ 3.98%)? *(It won't match exactly unless your grid happens to land precisely on the GMV return — that's expected; a finer grid gets you closer.)*

5. **Plot it (text description is fine if you don't have a plotting library, but try `matplotlib` if you have it).** Volatility on the x-axis, target return on the y-axis. You should see the hyperbola shape Lecture 2 described — a vertex, with the curve opening upward and to the right, and (if your grid goes low enough) a mirror-image left branch below the vertex. If you have `matplotlib`:

   ```python
   import matplotlib.pyplot as plt
   plt.plot(frontier_df["volatility"], frontier_df["target_return"], marker="o")
   plt.xlabel("Volatility (annualized)")
   plt.ylabel("Expected return (annualized)")
   plt.title("Crunch Capital Advisors -- Efficient Frontier")
   plt.savefig("frontier.png")
   ```

6. **Identify the inefficient branch.** From your Task 3 table, which rows (if any) sit on the *left* branch — i.e., have a *lower* target return than the vertex row but a *higher* volatility than the vertex? List them, or state that your grid didn't extend low enough to capture any (targets below roughly 5.5% start entering that territory for this dataset — check whether your grid's lowest target, 3%, actually produced a volatility *higher* than the vertex's).

## Expected results (spot checks)

- Task 1: `D` should be a large positive number (order of magnitude in the tens to low hundreds, depending on units — the exact value isn't the point, its *sign* is).
- Task 2 at `target_return = 0.08`: volatility should land close to **5.55%**, and weights should sum to `1.0000` (allow for floating-point noise at the 10th decimal place).
- Task 4: vertex volatility should land close to **3.98–4.05%**, at a target return somewhere between 5% and 6%.
- Task 6: with a grid starting at 3%, you should find at least one or two rows on the inefficient left branch (higher volatility than the vertex, at a lower return than the vertex return).

## Done when…

- [ ] `solutions.py` runs top to bottom with no errors, against a freshly seeded `crunch_capital` database.
- [ ] Your Task 3 table has exactly 23 rows and is sorted by target return ascending.
- [ ] You can explain, without looking it up, why the weights from `frontier_portfolio()` always sum to 1 regardless of the target return you pass in.
- [ ] Task 6's answer correctly identifies (or correctly rules out) any left-branch, inefficient points in your grid.

## Stretch

- Add a **no-short-selling check**: for each row in your Task 3 table, compute `min(weights)` and flag any row where a weight is negative. At roughly what target return does the frontier first require a short position in this dataset? (This previews Challenge 1.)
- Recompute the entire frontier using a *different* risk-free rate assumption of your choice (try 1% and 5%) and see how the **tangency point** (where the capital market line touches the frontier) shifts — you'll need the tangency formula from Lecture 2, Section 5, not just this exercise's frontier formula, to actually locate that point precisely.

## Submission

Commit `solutions.py` (and `frontier.png` if you plotted it) to your portfolio under `c35-week-08/exercise-02/`.
