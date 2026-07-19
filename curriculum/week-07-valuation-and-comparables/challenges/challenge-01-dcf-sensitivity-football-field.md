# Challenge 1 — DCF Sensitivity & Football Field

**Time:** ~90 minutes. **Difficulty:** Medium-high. **No single right implementation.**

## The scenario

The base-case DCF from Lecture 1 and Exercise 2 lands on an enterprise value of **≈$523.9M** using a 9.2% WACC and a 2.5% terminal growth rate — but both of those numbers are estimates, not facts handed down from anywhere certain. Before anyone signs off on a valuation built on them, they want to see **how much the answer moves** if either input is off by even half a point. This is standard practice in real valuation work: a DCF that hasn't been stress-tested against its own assumptions is a single point estimate dressed up as a rigorous analysis.

Your job: build a full **sensitivity grid** across WACC and terminal growth rate, then combine every valuation method this week produced (DCF, both terminal-value flavors; trading comps; precedent transactions) into a single **football field** — a chart showing every method's low–high range on one axis, so a decision-maker can see where they converge and where they don't at a glance.

## Part 1 — Build the sensitivity engine

Write a function `dcf_enterprise_value(sum_pv_explicit, fcf_terminal_year, wacc, terminal_growth, t_terminal=5)` that computes the perpetuity-growth enterprise value for **any** combination of inputs — not hardcoded to this week's specific numbers:

```python
def dcf_enterprise_value(sum_pv_explicit, fcf_terminal_year, wacc, terminal_growth, t_terminal=5):
    fcf_next = fcf_terminal_year * (1 + terminal_growth)
    tv = fcf_next / (wacc - terminal_growth)
    pv_tv = tv / (1 + wacc) ** t_terminal
    return sum_pv_explicit + pv_tv
```

Confirm it reproduces the base case: `dcf_enterprise_value(121_271_769, 40_865_886, 0.092, 0.025)` should be within a few thousand dollars of **$523,893,110**.

## Part 2 — The grid

Build a 5×5 grid: WACC from **8.2% to 10.2%** in 0.5-point steps, terminal growth from **1.5% to 3.5%** in 0.5-point steps (25 combinations total). For each cell, compute the enterprise value with Part 1's function, holding `sum_pv_explicit` and `fcf_terminal_year` fixed at this week's actual values.

Present the result as a DataFrame with WACC as the row index and terminal growth as the column headers — a real, queryable sensitivity table, not a printed wall of numbers.

**Verify these four corners** before trusting the rest of the grid:

| WACC | g | Enterprise Value |
|---:|---:|---:|
| 8.2% | 1.5% | ≈ $538.7M |
| 8.2% | 3.5% | ≈ $728.1M |
| 10.2% | 1.5% | ≈ $414.6M |
| 10.2% | 3.5% | ≈ $509.7M |

If any corner is off by more than roughly 1%, check whether you're using the *periodic* rates correctly (both `wacc` and `terminal_growth` are annual decimals, e.g. `0.082` not `8.2`) before debugging anything else.

## Part 3 — What the grid tells you

Answer these directly from your grid, in `sensitivity-writeup.md`:

1. Holding WACC fixed at the base case (9.2%), how much does enterprise value change moving from `g=1.5%` to `g=3.5%` — in dollars, and as a percentage of the base-case EV?
2. Holding `g` fixed at the base case (2.5%), how much does enterprise value change moving from `WACC=8.2%` to `WACC=10.2%`?
3. Which input — WACC or terminal growth — does the DCF's answer respond to more strongly, over these ranges? Is that consistent with what you'd expect from the `TV = FCF / (WACC − g)` formula as `WACC` and `g` get closer together?
4. At what combination of `WACC` and `g` in your grid does the model produce its *most* extreme result, and why does the formula behave that way as `WACC` approaches `g`?

## Part 4 — Build the football field

Assemble every method's per-share range from this week's lectures and exercises into one table:

| Method | Low | High |
|---|---:|---:|
| DCF — exit multiple | (Exercise 2) | (Exercise 2) |
| DCF — perpetuity growth (your grid's full WACC/g range) | (Part 2's grid min) | (Part 2's grid max) |
| Trading comps (trimmed) | (Exercise 3) | (Exercise 3) |
| Precedent transactions (trimmed) | (Lecture 2/3) | (Lecture 2/3) |

Convert every enterprise-value range to a per-share range using the EV-to-equity bridge from Lecture 3, Section 1 (net debt $75,000,000, 24,000,000 shares). Then render it as a **horizontal bar chart** — one horizontal bar per method, `Low` to `High`, all sharing one dollar-per-share x-axis, so the visual overlap (or lack of it) between methods is immediately obvious. `matplotlib.pyplot.barh` with a start offset (`plt.barh(y, high-low, left=low)`) is the simplest way to draw this; a plain formatted-text table with `█` characters scaled to a fixed width is an acceptable substitute if you don't have `matplotlib` available.

## Constraints

- `dcf_enterprise_value` must work for **any** valid inputs, not just this week's numbers — prove it by also running one sanity-check case with round numbers you pick yourself (e.g., `sum_pv_explicit=100_000_000`, `fcf_terminal_year=20_000_000`, `wacc=0.10`, `g=0.03`) and hand-verifying the result matches your function's output.
- No spreadsheets, per this week's data tooling rule — the grid lives in a DataFrame, generated by code.
- The football field must clearly label each method and its dollar range — a chart nobody can read the numbers off of isn't useful to the decision-maker it's for.

## Hints

<details>
<summary>On the grid producing wildly implausible numbers in one corner</summary>

Watch what happens as `WACC` and `g` get close together — the denominator `(WACC − g)` shrinks toward zero, and the terminal value (and therefore the whole EV) blows up. This is a real, well-known property of the Gordon growth model, not a bug in your code — but it's exactly why terminal growth rates are conventionally kept well below WACC (a common rule of thumb: `g` should never exceed the long-run growth rate of the overall economy, roughly 2–3%). If your grid's most extreme corner isn't the one where `WACC` is lowest and `g` is highest, you likely have a sign error.

</details>

<details>
<summary>On the football field bars not lining up visually</summary>

Make sure every method's range is converted to the *same* unit (per-share dollars) before plotting — plotting a mix of enterprise-value ranges and per-share ranges on one axis produces a chart that looks fine but is comparing incompatible numbers.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Correctness | Grid corners don't match the four verification values | All four corners match within ~1%, and the base case reproduces Exercise 2 exactly |
| Generality | `dcf_enterprise_value` hardcodes this week's numbers inside the function body | Function works correctly on your own hand-picked sanity-check case |
| Interpretation | Part 3 just reports numbers | Explains *why* the grid is more sensitive to `g` as `g` approaches `WACC` — ties back to the formula's denominator |
| Football field | Missing methods, unlabeled bars, or mixed units | All four methods present, correctly bridged to per-share, clearly labeled |

## Submission

Commit `sensitivity_engine.py`, `sensitivity-writeup.md`, and your football-field chart (as a saved image, or the code that renders it) to your portfolio under `c35-week-07/challenge-01/`.
