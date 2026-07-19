# Lecture 3 — Payback, PI, and Choosing

> **Duration:** ~2 hours. **Outcome:** You can compute simple and discounted payback period and the profitability index for any project, rank a slate of independent projects under a fixed capital budget using PI, and reconcile all five metrics from this week into one defensible recommendation when they don't agree.

The last two lectures gave you NPV (the metric that's always right) and IRR/MIRR (the metrics that are intuitive but can mislead). This lecture adds the two remaining tools in a capital budgeter's kit — payback and the profitability index — and then does the part that actually matters on the job: **what do you do when a limited budget means you can't fund every project with positive NPV?**

## 1. Payback period — how fast do you get your money back?

**Payback period** answers a narrower, more practical question than NPV or IRR: not "does this create value," but "how many years until the cumulative cash flow turns positive?" It's a liquidity and risk-exposure measure, not a value measure.

```python
def payback_period(cash_flows: list[float]) -> float:
    """Years until cumulative cash flow first turns non-negative.
    Returns a fractional year via linear interpolation within the crossing period."""
    cumulative = 0.0
    for t, cf in enumerate(cash_flows):
        prev_cumulative = cumulative
        cumulative += cf
        if cumulative >= 0 and t > 0:
            fraction = -prev_cumulative / cf
            return (t - 1) + fraction
    return float("inf")   # never pays back within the given horizon

cnc_flows = [-180000, 55000, 58000, 60000, 60000, 77000]
print(round(payback_period(cnc_flows), 2))   # 3.12
```

Walk the cumulative total by hand to see where 3.12 comes from: after period 1, cumulative is −125,000; after period 2, −67,000; after period 3, −7,000; after period 4, +53,000. The crossing happens *during* period 4 — you recovered the last $7,000 of the investment 7,000/60,000 ≈ 0.12 of the way through year 4, so payback ≈ 3 + 0.12 = 3.12 years.

**Simple payback ignores the time value of money entirely** — a dollar in year 1 and a dollar in year 4 count exactly the same toward "recovering the investment." That's a real flaw, and it's why payback is a *secondary* screen, never the primary decision rule.

## 2. Discounted payback — the fix

**Discounted payback** applies the exact same walk, but on the discounted cash flows instead of the raw ones — reusing the discount factors from Lecture 1:

```python
def discounted_payback_period(cash_flows: list[float], rate: float) -> float:
    discounted = [cf / (1 + rate) ** t for t, cf in enumerate(cash_flows)]
    return payback_period(discounted)

print(round(discounted_payback_period(cnc_flows, 0.07), 2))   # 3.63
```

Discounted payback (3.63 years) is always **longer than or equal to** simple payback (3.12 years) for a normal project, because discounting shrinks every future inflow, so it takes longer for the shrunk inflows to add up to the full initial outflow. If a project never achieves positive NPV, its discounted payback never resolves at all — it's `inf` no matter how long the horizon, which is itself a useful diagnostic: **a project whose discounted payback never arrives within its own lifespan has negative NPV.**

## 3. Why companies still use payback despite the flaw

Payback survives in real capital-budgeting policy for reasons that have nothing to do with rigor: it's dead simple to explain to a non-finance board member, it works as a **rough proxy for risk** (cash locked up for 8 years is exposed to 8 years of things going wrong that cash back in 2 years isn't), and many companies impose a **maximum payback threshold** (e.g., "we don't fund anything that doesn't pay back within 4 years") as a liquidity constraint layered *on top of* NPV, not instead of it. Treat payback as a screen and a risk signal, never as the primary accept/reject rule — a project can have a payback that eats most of its useful life and still be the best investment on the table. The New CNC Machine, this week's strongest project by both NPV and PI, doesn't recoup its investment on a discounted basis until 3.63 years into its 5-year life — nearly three-quarters of the project's entire lifespan — and it's still comfortably the best use of capital in the slate (verify this yourself in Exercise 1).

## 4. The profitability index (PI) — value per dollar invested

NPV tells you *how much* value a project creates in absolute dollars. It says nothing about **efficiency** — how much value per dollar of scarce capital. That's what the **Profitability Index** measures:

```
PI = PV(future cash flows, periods 1..n) / |initial investment|
```

Equivalently, since NPV = PV(future flows) − |investment|:

```
PI = (NPV + |initial investment|) / |initial investment| = 1 + NPV / |initial investment|
```

**Decision rule: accept if PI > 1 (equivalent to NPV > 0 — same information, different units).** The number that matters for ranking is what PI adds on top of NPV: **PI expresses value creation per dollar spent**, which is exactly the number you need when dollars, not projects, are the scarce resource.

```python
def profitability_index(cash_flows: list[float], rate: float) -> float:
    initial_investment = -cash_flows[0]                      # make it positive
    future_flows = cash_flows[1:]
    pv_future = sum(cf / (1 + rate) ** (t + 1) for t, cf in enumerate(future_flows))
    return pv_future / initial_investment

pi = profitability_index(cnc_flows, 0.07)
print(round(pi, 3))   # 1.398
```

PI ≈ 1.398 means: for every dollar of capital put into the CNC Machine, the project returns about $1.40 of present value — a 40-cent surplus per dollar invested, above and beyond getting your capital back. Compare that to the Regional Warehouse Expansion: it looks like the biggest, most impressive project in the table by dollars requested ($650,000), but its PI is **0.961 — below 1**, meaning every dollar you put in comes back worth only 96 cents. NPV and PI agree here (NPV is negative too, at −$25,296.02); size alone told you nothing about quality. Exercise 1 has you compute NPV, and Exercise 3 has you compute PI, for the whole table — go run both queries now if you haven't, because the rest of this section assumes you know which projects actually clear zero.

## 5. Screening first, then ranking under a capital budget

Run NPV across all six projects in the `cash_flows` table (Exercise 1) and a hard truth shows up immediately: **only two of the six clear their hurdle rate** — the New CNC Machine (+$71,712.84) and the Brand Relaunch Campaign (+$23,570.67). The other four — Regional Warehouse Expansion, AI Quality-Control R&D, Solar Rooftop Retrofit, and European Market Entry — are all **negative NPV at their own stated hurdle rates**, despite three of them having eye-catching raw dollar totals in the pitch (the Warehouse project alone asks for $650,000). This is not a contrived example — it is exactly what a first honest NPV pass over a real project slate usually looks like, and it's the single best argument in this course for running the numbers before a budget conversation happens, not during one.

**Step 1 — screen: reject every negative-NPV project, unconditionally, regardless of budget size.** No amount of available capital makes a value-destroying project worth funding. A $10 million budget does not rescue the European Market Entry project's −$259,242.90 NPV; it just means you have $10 million to spend on things that *do* clear zero. This step needs no ranking metric at all — it's a plain `WHERE npv > 0` filter.

**Step 2 — only among the survivors, if the budget is tight enough to force a choice, rank by PI.** Here that means comparing CNC Machine (PI 1.398, needs $180,000) against Brand Relaunch (PI 1.107, needs $220,000). If Crunch Machine Co.'s capital budget this cycle is $250,000, you cannot fund both ($400,000 combined) — CNC wins the choice on every metric (higher NPV, higher PI, higher IRR), so it gets funded and Brand Relaunch waits for next cycle despite being a genuinely good, positive-NPV project. If the budget is $450,000 or more, fund both — they're independent, and both clear zero.

### Why PI, not NPV, decides the ranking when the budget is genuinely tight and lumpy

The CNC-vs-Brand-Relaunch choice above didn't need PI to break a tie — NPV and PI agreed. But they don't always. Here's the textbook case where a naive "rank by NPV, fund top-down" approach leaves value on the table, using three small illustrative projects (not this week's seed data — just numbers to prove the point):

| Project | Initial cost | NPV | PI |
|---|---:|---:|---:|
| **X** | $500,000 | $150,000 | 1.30 |
| **Y** | $300,000 | $120,000 | 1.40 |
| **Z** | $250,000 | $110,000 | 1.44 |

With a **$550,000 budget**: ranking by NPV top-down picks X alone ($150,000 of value, $50,000 of budget left over — not enough for anything else). Ranking by PI top-down picks Z then Y ($250,000 + $300,000 = $550,000 spent exactly, $110,000 + $120,000 = **$230,000 of value** — over 50% more than the NPV-first approach, from the identical budget). **The right approach (the Lorie-Savage rule): rank by PI, descending, and fund down the list until the budget is exhausted.** Ranking by "value created per dollar spent" and greedily taking the best-per-dollar projects first is the approach that tends to maximize total value created subject to a fixed capital constraint — because it explicitly optimizes the scarce resource (dollars), not the count of projects or their absolute size.

### Doing the screen-then-rank step in SQL

```sql
WITH project_summary AS (
    SELECT
        project_id,
        project_name,
        hurdle_rate,
        -MIN(amount) FILTER (WHERE period = 0) AS initial_investment,
        SUM(amount / POWER(1 + hurdle_rate, period))
            FILTER (WHERE period > 0) AS pv_future_flows,
        SUM(amount / POWER(1 + hurdle_rate, period)) AS npv
    FROM cash_flows
    GROUP BY project_id, project_name, hurdle_rate
)
SELECT
    project_name,
    initial_investment,
    ROUND(npv, 2)                                   AS npv,
    ROUND(pv_future_flows / initial_investment, 3)  AS pi
FROM project_summary
WHERE npv > 0                                        -- Step 1: screen first
ORDER BY pi DESC;                                     -- Step 2: rank survivors by PI
```

Run that and exactly two rows come back: the New CNC Machine (PI 1.398) and the Brand Relaunch Campaign (PI 1.107) — in that order. Fund projects top-down by PI until the running total of `initial_investment` would exceed the budget. At a $250,000 budget, that's CNC alone ($180,000 spent, $70,000 unused — not enough left for Brand Relaunch's $220,000). At a $450,000+ budget, fund both ($400,000 spent, $95,283.51 of NPV captured). This is precisely the mechanic Exercise 3 and the mini-project have you build in full, generalized to any budget you're handed.

**Caveat — PI ranking is a heuristic, not a guarantee, when projects are "lumpy."** As Section 5's X/Y/Z illustration showed, if the budget doesn't divide evenly among the top-ranked projects, the greedy PI ranking can leave money on the table versus a combinatorial search over every possible subset. For a small number of surviving projects, checking all feasible combinations by hand (or in a short Python loop) is entirely realistic, and the mini-project asks you to verify the PI ranking against a brute-force check for exactly this reason.

## 6. Reconciling the five metrics — a decision framework

You now have five numbers per project: NPV, IRR, MIRR, payback (simple + discounted), and PI. Here's the order of authority when they disagree:

1. **NPV is always the tiebreaker of last resort.** If every other metric is ambiguous or contradictory, NPV wins — it's the only metric with no failure modes among the five, for a company with unconstrained access to capital at the hurdle rate.
2. **Under a hard capital budget, PI outranks NPV for selecting a subset** (Section 5) — but always sanity-check the PI ranking's total value created against a brute-force NPV-maximizing combination, especially with a small number of "lumpy" projects.
3. **IRR/MIRR are sanity checks and communication tools, never the primary rule**, per Lecture 2 — use them to explain a decision to a non-technical stakeholder, or as a fast first screen, but never as the final ranking for mutually exclusive projects of different size or timing.
4. **Payback is a risk/liquidity screen layered on top of the above**, not a substitute — a company might reject an otherwise-fundable positive-NPV project because its payback exceeds a hard policy threshold (long capital lockup, exposure to more years of things going wrong), but that's a *risk* decision made explicitly, not payback silently overriding NPV.

## 7. Common mistakes

- **Using simple (undiscounted) payback as if it accounted for the time value of money.** It doesn't — that's what discounted payback is for.
- **Ranking a capital-constrained project slate by NPV instead of PI.** This is the single most common capital-budgeting error in practice — it systematically favors large projects over a more efficient combination of smaller ones.
- **Applying PI ranking blindly without checking for a better combination when projects are lumpy.** Section 5's caveat — always spot-check with a brute-force comparison when the numbers are small enough to do so.
- **Letting payback override NPV rather than sit alongside it.** A long-payback, high-NPV project (like this week's New CNC Machine, which doesn't recoup its discounted investment until 72% of the way through its life) can still be the best investment on the table; a strict payback cutoff should be a deliberate risk policy, stated explicitly, not an accidental substitute for NPV.

## 8. Check yourself

- What does payback period measure that NPV does not, and why does that make it useful *despite* ignoring the time value of money?
- Write the PI formula two ways: as a ratio of present values, and in terms of NPV and the initial investment.
- Why does ranking by PI, rather than NPV, maximize total value created under a fixed capital budget?
- Give one scenario where a company would reasonably reject a positive-NPV project on payback grounds, and explain why that's a defensible risk decision rather than a modeling error.
- You have six independent candidate projects and a fixed capital budget. Describe, step by step, how you'd decide which subset to fund — including what you do *before* you ever look at the budget number.

That's the full metric toolkit for this week. The exercises turn every formula above into working Python and SQL over the whole `cash_flows` table; the challenges make you break IRR on purpose and then stress-test a hurdle-rate assumption; the mini-project has you run the entire process — rank, cross-check, fund a subset under budget, and write up the defense — exactly as a real capital-allocation committee would.

## Further reading

- **Investopedia — "Payback Period":** <https://www.investopedia.com/terms/p/paybackperiod.asp>
- **Investopedia — "Profitability Index":** <https://www.investopedia.com/terms/p/profitability.asp>
- **Investopedia — "Capital Rationing":** <https://www.investopedia.com/terms/c/capitalrationing.asp>
- **Harvard Business Review — "A Refresher on Internal Rate of Return":** <https://hbr.org/2016/03/a-refresher-on-internal-rate-of-return>
