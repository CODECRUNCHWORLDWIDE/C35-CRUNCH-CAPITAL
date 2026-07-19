# Challenge 2 — Expose a Look-Ahead Bug

**Time:** ~90 minutes. **Difficulty:** Hard. **No single right answer, but a real, specific bug to find.**

## The scenario

A classmate ("Jordan," for the purposes of this challenge — no relation to anyone real) posts their Week 9 backtest results in the course forum: a value-tilted strategy on the same seven-ticker universe, claiming a full-2024–2025 net Sharpe ratio of **2.3** and a max drawdown of only **-4%** — spectacular numbers for a simple long-only equity strategy, and considerably better than anything in this week's exercises. Jordan shares the script below. Before you believe it (or worse, copy the "technique"), your job is to find out why it's too good to be true.

## Jordan's script

```python
import pandas as pd
import numpy as np
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg2://localhost/crunch_capital")

# Pull prices
prices_long = pd.read_sql(
    "SELECT ticker, price_date, close_price FROM daily_prices ORDER BY price_date",
    engine, parse_dates=["price_date"]
)
prices_wide = prices_long.pivot(index="price_date", columns="ticker", values="close_price").sort_index()

# Pull the value factor
value = pd.read_sql(
    "SELECT ticker, ev_ebitda FROM fundamental_snapshot WHERE as_of_date = '2025-12-31'",
    engine
)
value["value_rank"] = value["ev_ebitda"].rank(ascending=True)   # 1 = cheapest

# Monthly rebalance dates
rebal_dates = prices_wide.resample("ME").last().index

# Build positions: every month, go long the 3 cheapest names
positions = []
for rdate in rebal_dates:
    top3 = value.nsmallest(3, "value_rank")["ticker"].tolist()
    for t in top3:
        positions.append({"rebalance_date": rdate, "ticker": t, "weight": 1 / 3})
positions = pd.DataFrame(positions)

# Build weight matrix — weights apply STARTING on the rebalance date itself
weight_matrix = pd.DataFrame(0.0, index=prices_wide.index, columns=prices_wide.columns)
for i, rdate in enumerate(rebal_dates):
    row = positions[positions["rebalance_date"] == rdate].set_index("ticker")["weight"]
    end = rebal_dates[i + 1] if i + 1 < len(rebal_dates) else prices_wide.index[-1]
    mask = (prices_wide.index >= rdate) & (prices_wide.index < end)
    for ticker, w in row.items():
        weight_matrix.loc[mask, ticker] = w

# Returns
daily_rets = prices_wide.pct_change()
gross_return = (weight_matrix * daily_rets).sum(axis=1)
cost = weight_matrix.diff().abs().sum(axis=1) * (10 / 10_000)
net_return = gross_return - cost
equity_curve = (1 + net_return.fillna(0)).cumprod()

sharpe = (net_return.mean() / net_return.std()) * np.sqrt(252)
running_max = equity_curve.cummax()
max_dd = (equity_curve / running_max - 1).min()

print(f"Sharpe: {sharpe:.2f}")
print(f"Max drawdown: {max_dd:.2%}")
```

## Your task

1. **Run it (or trace it by hand) and reproduce Jordan's suspiciously good numbers.** Confirm you get something in the same ballpark before you go hunting — you need to know the bug you're chasing actually produces the symptom described.

2. **Find every bug.** There is **more than one** issue in this script, layered together — this is realistic; in practice, backtest bugs often compound. At minimum, look hard at:
   - How `value` is looked up relative to `daily_prices` dates. (Re-read Lecture 3 Section 1, Example 2, closely — this script is *exactly* that bug, written out in full.)
   - The `mask` condition in the weight-matrix loop, specifically the comparison operators (`>=` vs `>`, `<` vs `<=`) versus what Lecture 2 Section 4 specified and why.
   - The line that computes `gross_return` — compare it, term by term, against Lecture 2 Section 4's version. What's missing?

3. **For each bug found, write (in `challenge-02.md`):**
   - The exact line(s) responsible.
   - In plain English, what information the buggy version lets the strategy "see" that it should not have access to on that date.
   - A **quantified** estimate, if you can produce one, of how much each individual bug inflates the result — fix them one at a time and re-run, so you can attribute the inflation to each specific bug rather than reporting one combined "before/after" number.

4. **Fix all of them** and report the honest, corrected Sharpe ratio and max drawdown. Compare directly to Jordan's original claim.

5. **Write a one-paragraph "code review comment"** — the kind you'd actually leave on a pull request — that Jordan could read and immediately understand what to fix and why, without needing to re-read this entire course.

## Constraints

- Do not simply rewrite the script from scratch using Exercise 2's engine and call it done — the point is to **diagnose the specific lines** in *this* script, the way you'd have to when reviewing someone else's real, already-written code.
- Fix the bugs **one at a time**, in isolation, re-running between each fix, so your write-up can attribute the Sharpe/drawdown inflation to each bug individually rather than reporting a single opaque "it got worse."
- Don't stop at "the value join has no date filter" — say specifically what date range is affected (which months' trades used data that didn't exist yet) and roughly how large that window of leaked information is (in this case: nearly the entire backtest, since the only snapshot is dated at the very end of it).

## Hints

<details>
<summary>On finding the value-join bug specifically</summary>

Ask: on the very first rebalance date in the backtest (early-to-mid 2024), what is the actual, real-world "as of" date of the `ev_ebitda` numbers this script uses to pick the top-3 cheapest names? Compare that date to the rebalance date. If the answer date is *after* the rebalance date, the strategy has time-traveled.

</details>

<details>
<summary>On the weight-matrix mask bug</summary>

Lecture 2 was explicit that a rebalance date is "decision day" — new weights should take effect strictly *after* it, not on it, because you compute the new signal using that day's closing data, and can't have already been holding the new position while that same day's return happened. Compare Jordan's `mask` boundary operators to Lecture 2's `mask` line character by character.

</details>

<details>
<summary>On the gross_return line</summary>

Compare `(weight_matrix * daily_rets).sum(axis=1)` to Lecture 2's `(weights.shift(1) * daily_rets).sum(axis=1)`. One word is missing. That one word is worth a meaningful chunk of Jordan's inflated Sharpe ratio on its own — quantify it.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Bug detection | Finds one bug, misses the others | Finds and separately diagnoses all three |
| Diagnosis quality | "This line looks wrong" | Explains precisely what future information leaks backward, and over what date range |
| Quantification | No before/after numbers | Each bug's individual contribution to the inflated Sharpe is isolated and reported |
| Fix correctness | Fixed version still leaks in a subtler way | Fixed version matches the discipline in Lectures 2–3 exactly |
| Communication | Technical but unreadable | A code-review comment a non-expert teammate could act on |

## Submission

Commit `challenge-02.md` (with your bug-by-bug findings and quantified before/after numbers) and your corrected script to your portfolio under `c35-week-09/challenge-02/`.
