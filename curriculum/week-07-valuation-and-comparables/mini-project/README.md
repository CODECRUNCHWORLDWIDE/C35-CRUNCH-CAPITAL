# Mini-Project — Value Crunch Machine Co. Two Ways, Then Reconcile

> Build a complete valuation of Crunch Machine Co. as of December 31, 2025: a DCF (both terminal-value methods), a trading-comparables analysis, and a precedent-transactions analysis — each bridged to a per-share price — assembled into one football field and reconciled into a single defensible recommendation. This is the week's capstone: every lecture, exercise, and challenge this week built one piece of this pipeline; today you assemble all of them into one coherent deliverable a decision-maker could actually act on.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

An investment analyst is almost never asked "what's your DCF say?" in isolation — they're asked "what's this company worth, and how confident are you?" That question only has a good answer when multiple independent methods have been built, compared, and reconciled honestly. That's what you're producing today: not four disconnected numbers, but one valuation memo with a recommendation you could defend if someone in the room pushed back on every assumption.

---

## Deliverable

A directory in your portfolio `c35-week-07/mini-project/` containing:

1. `valuation_engine.py` — the full valuation pipeline (functions described below).
2. `engine_tests.py` — checks proving each piece of the engine is correct (see Part 4).
3. `valuation_memo.md` — the final write-up: all four methods, the football field, the reconciliation, and your recommended range.
4. `schema.sql` — this week's four table definitions, plus any additional tables your engine writes results to.

Everything runs against this week's seed data (`valuation_assumptions`, `revenue_forecast`, `trading_comps`, `precedent_transactions`), on PostgreSQL or SQLite — note which you used.

---

## Part 1 — The engine's core functions

Build these in `valuation_engine.py`, generalizing Exercises 1–3 and Challenge 1 into a small, clean, importable module:

```python
def project_unlevered_fcf(ltm_revenue: float, drivers: "pd.DataFrame", tax_rate: float) -> "pd.DataFrame":
    """Returns a DataFrame with one row per forecast year: revenue, ebitda, da, ebit,
    nopat, capex, delta_nwc, ufcf. `drivers` has the revenue_forecast columns."""
    ...

def discount_stream(cash_flows: "pd.Series", rate: float) -> "pd.Series":
    """PV of each cash flow, assuming cash_flows are indexed 1..n (years from today)."""
    ...

def terminal_value_perpetuity(fcf_terminal_year: float, wacc: float, terminal_growth: float) -> float:
    ...

def terminal_value_exit_multiple(ebitda_terminal_year: float, exit_multiple: float) -> float:
    ...

def dcf_enterprise_value(forecast: "pd.DataFrame", wacc: float, terminal_growth: float = None,
                          exit_multiple: float = None) -> float:
    """Full DCF EV. Exactly one of terminal_growth / exit_multiple must be provided —
    raise a clear error if both or neither are given."""
    ...

def ev_to_equity_bridge(enterprise_value: float, total_debt: float, cash: float,
                          shares_outstanding: float) -> dict:
    """Returns {'net_debt':..., 'equity_value':..., 'per_share':...}."""
    ...
```

**Design requirement:** every function must work on inputs you pass in, not on hardcoded Crunch Machine Co. constants baked into the function body. Prove this in `engine_tests.py` with at least one call using numbers that are *not* this week's actual data (Challenge 1's Part 1 sanity check is a good template).

## Part 2 — Comps functions

```python
def compute_comp_multiples(comps: "pd.DataFrame") -> "pd.DataFrame":
    """Adds market_cap, enterprise_value, ev_revenue, ev_ebitda columns to a comps
    DataFrame with raw fields (share_price, shares_outstanding, total_debt, cash_and_equivalents,
    revenue_ntm or target_revenue_ltm, ebitda_ntm or target_ebitda_ltm)."""
    ...

def trimmed_range(values: "pd.Series") -> tuple:
    """Drops the single min and single max, returns (low, high) of what's left."""
    ...

def apply_multiple_range(target_metric: float, multiple_range: tuple) -> tuple:
    """Returns (implied_ev_low, implied_ev_high)."""
    ...
```

Run these against both `trading_comps` (NTM figures) and `precedent_transactions` (LTM figures, renaming fields as needed) to get two independent implied-EV ranges.

## Part 3 — Run the full pipeline

Using the functions above, in `valuation_memo.md`, produce:

1. **DCF forecast table** — the full five-year `project_unlevered_fcf` output.
2. **DCF enterprise value, both terminal methods** — perpetuity growth and exit multiple, each with the explicit-period PV, the terminal value, and the final EV shown separately (not just the final number).
3. **Cross-check** — the implied exit multiple from the perpetuity-growth method, and the implied growth rate from the exit-multiple method (Exercise 2, Part D).
4. **Trading comps table** — every comp's computed EV and multiples, the trimmed range, and the resulting implied EV for Crunch Machine Co.
5. **Precedent transactions table** — the same, for `precedent_transactions`.
6. **EV-to-equity bridge** — every method's EV converted to a per-share price, in one summary table.
7. **Football field** — a chart or a clearly formatted table showing every method's per-share range on one axis.

## Part 4 — Prove it's correct

In `engine_tests.py`, verify against at least these known values (`assert`, or `pytest`):

1. `project_unlevered_fcf(...)`'s FY2026 row should match Lecture 1's worked example: revenue ≈ $226,800,000, unlevered FCF ≈ $22,995,000.
2. `dcf_enterprise_value(..., terminal_growth=0.025)` at `wacc=0.092` should be within a few thousand dollars of **$523,893,110**.
3. `dcf_enterprise_value(..., exit_multiple=8.0)` at `wacc=0.092` should be within a few thousand dollars of **$461,366,567**.
4. `compute_comp_multiples` on `trading_comps` should produce a median `ev_ebitda` within 0.05 of **8.50**.
5. `ev_to_equity_bridge` called on the perpetuity-growth EV should return a `per_share` within a few cents of **$18.70**.

Print a clear pass/fail for each check.

## Part 5 — Write the recommendation

Close `valuation_memo.md` with:

- Your own version of Lecture 3's reconciliation: where do the four ranges overlap, and where does the DCF diverge most from the comps-based methods?
- A stated set of weights (à la Challenge 2, Part 3) and the resulting weighted per-share value.
- **One final recommended range**, with a stated base-case point estimate, that you could defend if someone in the room asked "why not higher? why not lower?"
- One paragraph naming the single assumption in your whole analysis you're **least** confident about, and how much the final range would move if that assumption were meaningfully wrong (tie back to Challenge 1's sensitivity work if it's a DCF assumption, or Exercise 3's comp-quality discussion if it's a comps assumption).

---

## Rules

- **The engine functions must be genuinely reusable** — no Crunch-Machine-Co.-specific constants hardcoded inside `valuation_engine.py`'s functions. If you can't run `dcf_enterprise_value` on a different company's forecast without editing the function body, it's not done.
- **No spreadsheets**, per this week's data tooling rule — every table lives in SQL, every calculation lives in Python.
- **Show your verification.** A valuation engine you haven't tested against known answers is not a trustworthy engine — Part 4 isn't optional polish, it's the proof the rest of the mini-project depends on.
- **The recommendation must be defensible, not just computed.** A weighted average with no stated reasoning for the weights doesn't meet this bar (same standard as Challenge 2).

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 30% | All five Part 4 checks pass; every method's numbers match this week's lectures/exercises |
| Reusability | 20% | Engine functions work on non-Crunch inputs without modification |
| Comps rigor | 15% | Trimmed ranges, median vs. mean reasoning, and comp-quality judgment are all present and correct |
| Reconciliation | 20% | Specific, quantified reasons for method disagreement; justified weights; defensible final range |
| Memo clarity | 15% | Every method's numbers are shown, not just final answers; the football field is readable and correctly bridged to per-share |

---

## Why this matters

This is what a real valuation deliverable looks like: not one number confidently asserted, but multiple independent methods, each shown with its own work, reconciled into a range with the disagreement explained rather than hidden. Every remaining week of this course that touches a company's value — Week 10's M&A modeling, Week 12's capstone — reuses exactly this shape of analysis. Keep `valuation_engine.py`; you'll extend it, not rebuild it, when the stakes get higher later in the course.

When done: take the [quiz](../quiz.md) and start Week 8 — Portfolio theory.
