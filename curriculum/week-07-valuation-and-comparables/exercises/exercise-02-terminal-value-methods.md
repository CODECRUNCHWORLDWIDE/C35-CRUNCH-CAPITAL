# Exercise 2 — Terminal Value, Two Ways

**Goal:** Compute Crunch Machine Co.'s terminal value with the perpetuity-growth method and the exit-multiple method, discount both back to today, assemble two full enterprise values, and cross-check each method's assumption against what the *other* method implicitly assumes.

**Estimated time:** 75 minutes.

## Setup

You need Exercise 1's five-year unlevered FCF forecast (reuse your `solutions.py`/`solutions.sql` output, or hardcode the five FY2026–2030 values from Lecture 1's table if you're confident they're correct), plus `valuation_assumptions` for `wacc` (9.2%), `terminal_growth_rate` (2.5%), and `exit_multiple_ev_ebitda` (8.0×). Have Lecture 1, Sections 6–7 open.

Create `solutions.py` (this exercise is cleaner in Python than SQL, given the algebra in Part C — a SQL version is a stretch task).

## Part A — Discount the explicit period

1. **Task 1.** Write a function `discount_factor(rate, t)` returning `1 / (1 + rate) ** t`. Using it, build a small table (a list of dicts, or a DataFrame) with one row per forecast year: `fiscal_year`, `t` (1 through 5), `ufcf`, `discount_factor`, `pv`.
2. **Task 2.** Sum the `pv` column. This is the present value of the explicit period alone — before any terminal value.

## Part B — Terminal value, perpetuity growth

3. **Task 3.** Compute `fcf_2031 = fcf_2030 * (1 + terminal_growth_rate)`.
4. **Task 4.** Compute the terminal value: `TV = fcf_2031 / (wacc - terminal_growth_rate)`.
5. **Task 5.** Discount `TV` back to today using **the same discount factor as FY2030** (`t=5`) — not `t=6`. Explain in a comment why `t=5` is correct even though `TV` represents *years 2031 and beyond*.
6. **Task 6.** Enterprise value (perpetuity growth) = Task 2's sum + Task 5's discounted terminal value.

## Part C — Terminal value, exit multiple

7. **Task 7.** Compute `TV = ebitda_2030 * exit_multiple_ev_ebitda`.
8. **Task 8.** Discount it back using the `t=5` discount factor, same as Task 5.
9. **Task 9.** Enterprise value (exit multiple) = Task 2's sum + Task 8's discounted terminal value.

## Part D — Cross-check each method against the other

This is the part that turns "I computed two numbers" into "I understand what each number is claiming."

10. **Task 10 — implied exit multiple from the perpetuity-growth method.** The perpetuity-growth method's *undiscounted* terminal value (Task 4) is implicitly claiming the business would sell for some multiple of FY2030 EBITDA. Compute that implied multiple: `implied_multiple = TV_perpetuity / ebitda_2030`. Compare it to the `exit_multiple_ev_ebitda` assumption (8.0×) actually given in `valuation_assumptions` — is the perpetuity-growth method more or less optimistic than the assumption you were handed for the exit-multiple method?
11. **Task 11 — implied growth rate from the exit-multiple method.** Symmetrically, the exit-multiple method's terminal value (Task 7) is implicitly claiming some constant perpetual growth rate. Solve the Gordon-growth formula for `g`:

```
TV = FCF_2030 × (1 + g) / (WACC − g)
```

Rearranged for `g` (show this algebra in a comment in your code):

```
g = (TV × WACC − FCF_2030) / (TV + FCF_2030)
```

Compute it, and compare to the `terminal_growth_rate` assumption (2.5%) actually given for the perpetuity-growth method. Is the exit-multiple method more or less optimistic?

12. **Task 12.** In 3–4 sentences in a comment or a `reflection.md`, answer: do the two methods' *explicit* assumptions (8.0× multiple, 2.5% growth) roughly agree with each other once translated into the same units (Task 10 and 11)? If not, which method do you think is more optimistic overall, and does that match which one produced the higher enterprise value?

## Expected results (spot checks)

- Task 2 → sum of PVs (explicit period) ≈ **$121,271,769**.
- Task 4 → perpetuity-growth TV ≈ **$625,187,064**.
- Task 5 → discounted TV (perpetuity growth) ≈ **$402,621,341**.
- Task 6 → EV (perpetuity growth) ≈ **$523,893,110**.
- Task 7 → exit-multiple TV ≈ **$528,096,368**.
- Task 8 → discounted TV (exit multiple) ≈ **$340,094,798**.
- Task 9 → EV (exit multiple) ≈ **$461,366,567**.
- Task 10 → implied exit multiple from the perpetuity-growth method ≈ **9.47×** — *higher* than the 8.0× directly assumed, meaning the perpetuity-growth method is the more optimistic of the two.
- Task 11 → implied growth rate from the exit-multiple method ≈ **1.36%** — *lower* than the 2.5% directly assumed, confirming the same conclusion from the opposite direction: the exit-multiple method is the more conservative one.

## Done when…

- [ ] `solutions.py` has all 12 tasks, each printing a clearly labeled result.
- [ ] Both enterprise values match the "Expected results" within $500 (rounding).
- [ ] Task 5/Task 8's comment correctly explains why the terminal value discounts at `t=5`, not `t=6`.
- [ ] Task 12's reflection states, in your own words, which method is more optimistic and why — referencing the *implied* parameters from Tasks 10–11, not just the two final EV numbers.

## Stretch

- Add a Task 13: recompute the perpetuity-growth enterprise value if `terminal_growth_rate` were 3.0% instead of 2.5% (everything else held constant). How much did EV change for that single 0.5-point move? This is a preview of Challenge 1's full sensitivity grid.
- Add a Task 14: rewrite Tasks 1–9 as a single SQL query (or a short sequence of CTEs) instead of Python. This is genuinely harder than the Python version because of the `POWER()` calls and the two-branch terminal-value logic — a good test of whether you actually understand the calculation or just the pandas syntax.

## Submission

Commit `solutions.py` and `reflection.md` to your portfolio under `c35-week-07/exercise-02/`.
