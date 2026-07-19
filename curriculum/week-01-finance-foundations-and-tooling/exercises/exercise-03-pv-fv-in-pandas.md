# Exercise 3 — PV and FV in Pandas

**Goal:** Pull the `cash_flows` table into a DataFrame and build a small, reusable set of pandas functions that discount an entire stream at once — the direct forerunner of the mini-project's discounting engine.

**Estimated time:** 60 minutes.

## Setup

Confirm your Python environment is active and can reach the database:

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://localhost/crunch_capital")
# SQLite users: engine = create_engine("sqlite:///crunch_capital.db")

cash_flows = pd.read_sql("SELECT * FROM cash_flows", engine)
print(cash_flows.shape)   # (38, 12)
print(cash_flows.dtypes)
```

Create a file `solutions.py`. Put each task under a `# Task N` comment, and make sure running the whole file top to bottom prints each task's answer clearly labeled.

## Tasks

1. **Load and inspect.** Load the full table into a DataFrame named `cash_flows`. Print the distinct `project_name` values (`.unique()`) and confirm you see all 6 projects.

2. **Filter one project.** Build `cnc = cash_flows[cash_flows["project_id"] == 1].sort_values("period")`. Print it. Confirm it has 7 rows (the New CNC Machine has both a Revenue and a Salvage row at period 5).

3. **A vectorized discount-factor column.** Without a loop, add a new column `discount_factor` computed as `1 / (1 + hurdle_rate) ** period` for **every row of the whole table at once** (pandas applies the formula element-wise across columns). Print the first few rows of `project_name`, `period`, `hurdle_rate`, `discount_factor`.

4. **A vectorized present-value column.** Add a column `pv` = `amount * discount_factor`. Print `project_name`, `period`, `amount`, `pv` for `project_id == 1` only.

5. **NPV per project, the pandas way.** Using `.groupby("project_name")["pv"].sum()`, compute every project's NPV in one line. Print the result sorted from highest to lowest NPV.

6. **Cross-check against Lecture 2.** Confirm the New CNC Machine's NPV from Task 5 is close to the **+71,721.49** computed by hand in Lecture 2, section 3 (small rounding differences are fine; anything off by more than a few dollars means a bug — check your `discount_factor` formula first).

7. **A reusable function: `pv_of_stream`.** Write a function `pv_of_stream(df: pd.DataFrame) -> float` that takes a DataFrame with `amount`, `hurdle_rate`, and `period` columns (any subset of rows) and returns the total present value of that stream — no hardcoded project. Test it against `cash_flows[cash_flows["project_id"] == 2]` (Regional Warehouse Expansion) and print the result.

8. **A reusable function: `fv_of_amount`.** Write a function `fv_of_amount(present: float, rate: float, periods: int) -> float` (same formula as Lecture 2's `future_value`, just renamed to fit this file's convention). Use it to answer: if the company instead invested the Regional Warehouse Expansion's $650,000 capex in a fund earning 10% (matching that project's hurdle rate) for 5 years and did nothing else with it, what would it be worth? Compare that number to the warehouse project's period-5 cash flow alone — which is bigger, and what does that comparison tell you about whether the project is worth doing versus just investing the cash?

9. **Rank all six projects by NPV.** Using your `pv_of_stream` function inside a loop over `cash_flows["project_id"].unique()`, build a small DataFrame or dict of `{project_name: npv}` for all 6 projects, and print it sorted highest to lowest.

## Expected results (spot checks)

- Task 2 → 7 rows for `project_id == 1`.
- Task 5/6 → New CNC Machine's NPV ≈ `71,721` (your exact cents may vary slightly by rounding order — that's fine).
- Task 9 → all 6 projects ranked; note whether the ranking by NPV matches or differs from a naive ranking by raw total cash flow (Exercise 2, Task 3) — a mismatch here is the whole point of discounting.

## Done when…

- [ ] `solutions.py` runs top to bottom with no errors and prints all 9 tasks' results.
- [ ] Task 3 and Task 4's columns are computed **without an explicit Python `for` loop** — pandas vectorizes the arithmetic across the whole column at once. If you wrote a loop, rewrite it as a plain column expression.
- [ ] `pv_of_stream` works on **any** subset of rows you pass it, not just one hardcoded project.
- [ ] You can state, in one sentence, which project ranks highest by NPV and whether that matches your intuition from Lecture 1's worked comparison of the CNC machine vs. the Europe project.

## Stretch

- Add a `Task 10`: write `fv_of_stream(df, target_period)` — the future value, **at a given target period**, of an entire cash-flow stream (hint: you can either compound every flow forward to that period, or compute the stream's NPV first and then compound that single number forward — both give the same answer; try proving that to yourself with a print statement comparing the two approaches).
- These two functions, `pv_of_stream` and `fv_of_stream`, are most of the mini-project's discounting engine. Keep this file — you'll extend it directly.

## Submission

Commit `solutions.py` to your portfolio under `c35-week-01/exercise-03/`.
