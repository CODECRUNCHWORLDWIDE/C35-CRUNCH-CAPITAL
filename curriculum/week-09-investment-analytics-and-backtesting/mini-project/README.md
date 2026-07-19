# Mini-Project — Backtest a Rules-Based Strategy, Honestly

> Design your own rules-based signal, run it through a cost-aware backtest engine you built, report its full performance profile, and prove — out-of-sample — whether it's real. One strategy, one honest verdict, one clean deliverable.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's deliberately structured like a real research memo, not a homework set: a portfolio manager doesn't want to see your code, they want to see a strategy, a cost-adjusted result, and an honest answer to "does this survive out-of-sample?" You'll build all three.

---

## Deliverable

A directory in your portfolio `c35-week-09/mini-project/` containing:

1. `strategy.sql` — every SQL query that builds your signal (self-contained, runnable against `crunch_capital` as set up in the week README).
2. `backtest.py` — your backtest engine and the code that runs it (may import/reuse your Exercise 2 engine, cited as such).
3. `report.md` — the full write-up (structure below).
4. `results/` — your `equity_curve.csv` (daily net_return, cost, gross_return, equity_curve) and any supporting CSVs (walk-forward window results, etc.).

Everything runs against the seed data from the [week README](../README.md). Note whether you used PostgreSQL or SQLite.

---

## The brief

**Design your own strategy.** You may extend this week's momentum + value composite (change the weighting, the lookback windows, top-N vs. threshold, or add the 5-day reversal factor from Lecture 1 Section 4), or design a genuinely different rule from the same three thesis families (pure momentum, pure value, pure mean-reversion, or a different combination) — as long as it is precisely, unambiguously computable from `daily_prices` and `fundamental_snapshot` with no human judgment at trade time. State your rule in one paragraph of plain English **before** you show a single line of code — this is the thesis statement Lecture 1 opened with, and a reader should be able to understand your strategy from that paragraph alone.

**Requirements — your strategy must:**

1. Trade only the seven-ticker universe (`CRNM`, `TFI`, `MMS`, `AFC`, `VDT`, `FPG`, `IMW`).
2. Rebalance on a fixed schedule (monthly, as in the exercises, or a different frequency you justify — e.g., quarterly, to reduce turnover and cost).
3. Size positions with a stated, consistent rule (equal weight is fine; anything more sophisticated must be explained).
4. Be backtested with **realistic transaction costs and slippage** — you choose the bps assumption, but you must state it and, ideally, show a sensitivity check (Exercise 2's stretch goal) at two or three different cost levels.
5. Avoid every look-ahead trap from Lecture 3 — explicitly. If your strategy uses the value factor, you must address the single-snapshot-date problem directly (either restrict your test window to dates on/after `2025-12-31`, which will leave you almost no history — acceptable and worth discussing — or build a defensible workaround and explain exactly what assumption it rests on).

**Requirements — your validation must:**

6. Report full-sample performance metrics: CAGR, annualized volatility, Sharpe ratio, max drawdown (with dates), and hit rate.
7. Include **at least one genuine out-of-sample test** — either a single frozen train/test split (train on 2024, test on 2025, or an equivalent split for your strategy's data requirements) or a walk-forward validation (Challenge 1's method). State which you used and why.
8. Report the out-of-sample result **even if it's worse than the in-sample result** — that comparison, reported honestly, is the single most important number in this whole project.

---

## `report.md` structure

```markdown
# Mini-Project — [Your Strategy's Name]

## 1. Thesis
One paragraph, plain English. What do you believe, and why would it be true?

## 2. Signal specification
Exact rule: which factor(s), what lookback windows, how combined, top-N or threshold, rebalance frequency.

## 3. Backtest assumptions
Universe, position sizing, cost/slippage bps (and why you chose that number), start/end dates,
and how you handled the fundamentals-snapshot look-ahead issue specifically.

## 4. Full-sample results
CAGR, annualized vol, Sharpe, max drawdown (+ dates), hit rate. Show the equity curve
(a simple line description or ASCII/markdown table of key checkpoints is fine — no
charting library required, though one is welcome if you have it).

## 5. Out-of-sample validation
Method used (train/test split or walk-forward), the out-of-sample result(s),
and an honest comparison to the full-sample number from Section 4.

## 6. Cost sensitivity
Results at 2-3 different cost assumptions (e.g., 0bps, 10bps, 25bps one-way).
How much of the strategy's edge is cost-fragile?

## 7. Verdict
Would you trade this strategy with real capital? State your reasoning plainly,
including what would make you more or less confident.

## 8. Known limitations
What this backtest does NOT account for (see Lecture 2 Section 6 for the starting list)
and, specific to your strategy, anything else a critical reader should be skeptical of.
```

---

## Milestones

- **Milestone 1 (45 min):** Finalize your thesis and signal spec. Write Section 1–2 of `report.md` *before* writing any code — committing to the rule in writing first is what keeps you from quietly tuning it after seeing results.
- **Milestone 2 (60 min):** Build and run the full-sample backtest. Get Sections 3–4 done and the numbers sanity-checked (signs correct, ranges plausible per the exercises' spot checks).
- **Milestone 3 (45 min):** Run your out-of-sample validation. Write Sections 5–6, including the honest comparison.
- **Milestone 4 (30 min):** Write Sections 7–8 and do a final read-through — does the report read as one coherent argument, or a pile of disconnected numbers?

---

## Rules

- **No spreadsheets, anywhere.** Every table, every result, lives in SQL/Python/CSV per the [course data tooling rule](../../SYLLABUS.md#data-tooling-rule).
- **State every assumption you make.** Cost bps, risk-free rate for Sharpe, train/test split date, universe — all of it, in `report.md`, not buried in code comments only.
- **The out-of-sample number is not optional and cannot be silently improved after the fact.** If you look at it and don't like it, you may not go back and re-tune the strategy against that same window — see Lecture 3 Section 3's rule. If you want to iterate, say so explicitly and use a *different*, still-untouched slice of time, or be upfront that you're now reporting an in-sample-only result and lost the ability to validate this particular version.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Signal precision | 20% | Rule is unambiguous, computable, stated before results were seen |
| Backtest correctness | 20% | No look-ahead bias, costs modeled realistically, weights lag correctly |
| Metrics | 15% | All five performance metrics computed correctly with correct signs/ranges |
| Out-of-sample honesty | 25% | A genuine held-out test, reported as-is, explicitly compared to in-sample |
| Cost sensitivity | 10% | At least 2-3 cost scenarios shown and interpreted |
| Written verdict | 10% | Clear, specific, defensible recommendation with stated caveats |

---

## Why this matters

This mini-project is the shape of real quantitative research: someone has a hunch, you turn it into a rule, you cost it honestly, and — the step almost everyone skips under deadline pressure — you try, in good faith, to prove yourself wrong before you'd ever risk real money on it. Do this once, rigorously, and you'll never again trust a backtest that skips the out-of-sample step, whether it's someone else's or your own.

When done: push, then take the [quiz](../quiz.md) and start [Week 10 — M&A & deal analysis](../../week-10-mergers-acquisitions-and-deals/).
