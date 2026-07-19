# Mini-Project — A Discounting Engine Over the Cash-Flow Table

> Build a reusable discounting engine: a SQL table as the store of record, and a pandas layer on top of it that returns the present value, future value, and NPV of **any** cash-flow stream you point it at — not just the six projects in the seed data. This is the week's capstone: prove the SQL and Python you learned this week compose into a real tool.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

A finance team doesn't re-derive the discounting formula from scratch every time someone asks "what's this project worth?" — they build the calculation once, correctly, and reuse it. That's what you're building today: not a one-off answer, but a small, dependable tool that takes any cash-flow stream and any question ("what's it worth today?", "what's it worth in year 5?") and returns the right number, every time, verifiably.

---

## Deliverable

A directory in your portfolio `c35-week-01/mini-project/` containing:

1. `engine.py` — the discounting engine (functions described below).
2. `engine_tests.py` — a set of checks proving the engine is correct (see Part 3).
3. `report.md` — the engine run against all 6 seed projects, plus one new project you add yourself, with results and a short interpretation.
4. `schema.sql` — the `cash_flows` table definition and (if you extend it) any new table your engine writes results to.

Everything runs against the `cash_flows` table from the [week README](../README.md), on PostgreSQL or SQLite — note which you used.

---

## Part 1 — The engine's core functions

Build these four functions in `engine.py`. They should generalize Exercise 3's work into a small, clean, importable module.

```python
def present_value(cash_flow: float, rate: float, period: int) -> float:
    """PV of a single future cash flow."""
    ...

def future_value(present: float, rate: float, period: int) -> float:
    """FV of a present amount, compounded to `period` periods forward."""
    ...

def pv_of_stream(df: "pd.DataFrame", rate_col: str = "hurdle_rate",
                  period_col: str = "period", amount_col: str = "amount") -> float:
    """NPV of an entire cash-flow stream (a DataFrame of rows)."""
    ...

def fv_of_stream(df: "pd.DataFrame", target_period: int,
                  rate_col: str = "hurdle_rate", period_col: str = "period",
                  amount_col: str = "amount") -> float:
    """Value of an entire cash-flow stream, compounded forward to `target_period`."""
    ...
```

Design requirement: **`pv_of_stream` and `fv_of_stream` must work on any DataFrame with the right columns** — not just rows pulled from the `cash_flows` table. Prove this in `engine_tests.py` by building a small synthetic DataFrame by hand (not from SQL) and running both functions on it.

## Part 2 — Connect it to the database

Add a function that pulls a single project's stream straight from Postgres/SQLite and runs it through the engine:

```python
def project_npv(project_id: int, engine) -> float:
    """Pull one project's cash flows from the database and return its NPV."""
    ...

def all_projects_npv(engine) -> "pd.DataFrame":
    """Return a DataFrame of every project's name and NPV, sorted highest to lowest."""
    ...
```

Run `all_projects_npv` against the seed data and put the resulting table in `report.md`.

## Part 3 — Prove it's correct

In `engine_tests.py`, verify your engine against at least these four known cases (use `assert`, or `pytest` if you're comfortable with it):

1. `present_value(60000, 0.07, 4)` should be within a cent of **45,777.31** (Lecture 2's worked example).
2. `pv_of_stream(...)` on the New CNC Machine's full stream should be within a few cents of **71,721.49** (Lecture 2, section 3).
3. `present_value(x, r, 0)` should equal `x` exactly, for any `x` and `r` — a period-0 cash flow is never discounted (Lecture 2's "common mistakes" section).
4. `pv_of_stream` followed by `future_value` on that single NPV number, compounded to period `n`, should equal `fv_of_stream(..., target_period=n)` directly — the two paths to a future value (discount-then-compound vs. compound-each-flow-directly) must agree, because they're mathematically the same operation done in a different order. This is the stretch task from Exercise 3 — bring it in here if you built it.

Print a clear pass/fail for each check when `engine_tests.py` runs.

## Part 4 — Extend it: add a 7th project

Add one new project of your own invention to the `cash_flows` table (reuse your work from Exercise 2, Part B, if you did the stretch there — otherwise invent a new one now). It must have a period-0 outflow and at least 4 more periods of cash flow, with its own `hurdle_rate`.

Run your engine's `project_npv` and `all_projects_npv` again with this project included, and confirm your new project appears correctly ranked among the other 6 in `report.md`.

## Part 5 — Write the report

In `report.md`:

- A table of all 7 projects (6 seed + your addition), sorted by NPV, highest first.
- One sentence per project on whether its NPV ranking is or isn't what you'd have guessed from its raw, undiscounted total cash flow (tie back to Exercise 2, Task 3 and Exercise 3, Task 9).
- A short paragraph: which project would you recommend Crunch Machine Co. fund first, and why, using NPV as your primary argument? (You don't yet have the full capital-budgeting decision framework — that's Week 3 — but you should be able to make a reasoned first pass using what you know now: bigger positive NPV is better, all else equal.)
- One paragraph reflecting on Part 3: which of the four correctness checks, if any, failed on your first attempt, and what was the bug?

---

## Rules

- **The engine functions must be genuinely reusable** — no project-specific constants or hardcoded project IDs inside `present_value`, `future_value`, `pv_of_stream`, or `fv_of_stream`. If you can't run them on a brand-new, never-seen stream without editing the function body, they're not done.
- **No spreadsheets**, per this week's data tooling rule — the store of record is `cash_flows` in SQL, the modeling layer is `engine.py`.
- **Show your verification.** A discounting engine you haven't tested against a known answer is not a trustworthy engine — Part 3 isn't optional polish, it's the proof the rest of the mini-project depends on.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 35% | All four Part 3 checks pass; NPVs match hand/lecture calculations |
| Reusability | 25% | Engine functions work on any valid DataFrame, not just the seed projects |
| Database integration | 15% | `project_npv`/`all_projects_npv` pull live from SQL, not a hardcoded copy of the data |
| Report clarity | 15% | Table + interpretation clearly show which projects rank where and why |
| Extension (7th project) | 10% | New project is realistic, correctly loaded, and correctly ranked |

---

## Why this matters

This is the shape of real financial-engineering work: build the calculation once, prove it's correct against known answers, then point it at new data without touching the internals. Every remaining week of this course reuses exactly this pattern — a SQL table as the store of record, a small tested Python module as the modeling layer. Keep `engine.py`; Week 3's NPV/IRR/MIRR ranking work and later weeks' valuation models build directly on top of the `pv_of_stream` function you just wrote.

When done: take the [quiz](../quiz.md) and start [Week 2 — Financial statements & ratio analysis](../../week-02-financial-statements-and-ratios/).
