# Quantifying risk — Value at Risk, scenario analysis, and stress testing an investment thesis

> **Duration:** ~2 hours. **Outcome:** You can compute historical and parametric Value at Risk for a single position and for a portfolio that includes it, run a probability-weighted scenario analysis on your valuation, and design a stress test that answers "what specifically would have to be true for this thesis to fail badly" — with a number attached to every claim.

Lecture 1 gave you an upside: buy CRNM at $12.50, target $15.29, size it at 8% of the book. A number like "22% upside" is exactly half of an investment case. The other half — the half beginners skip and the half that actually separates a professional risk process from a hunch with a spreadsheet attached — is: **how much could this cost me, and under what conditions?** This lecture builds that half with three tools: Value at Risk (VaR), scenario analysis, and stress testing. They answer three different questions, and conflating them is one of the most common — and most expensive — mistakes in risk management.

## 1. Value at Risk — what it actually claims

**Value at Risk (VaR)** at a confidence level `c` over a horizon `h` is a single number: the loss you would **not** expect to exceed, `c`% of the time, over that horizon. A 1-day 95% VaR of $23,200 means: on a typical trading day, there is a 95% chance your loss is smaller than $23,200. It does **not** mean the worst you could ever lose is $23,200 — the other 5% of days are, by construction, excluded, and some of them are far worse than the 95th percentile. VaR describes the *typical* bad day, not the *worst* day. Confusing the two is how firms get blown up by "one-in-a-thousand" events that VaR was never designed to bound. Keep this distinction sharp through the rest of the lecture.

There are two standard ways to compute it, and they can disagree — which is itself useful information.

## 2. Parametric VaR — assume a distribution, use the formula

Parametric (also called "variance-covariance") VaR assumes returns are normally distributed and uses the position's volatility directly:

```
VaR(c, h) = z(c) × σ_daily × √h × position_value
```

where `z(c)` is the standard-normal quantile for your confidence level (1.645 for 95%, 2.326 for 99%) and `σ_daily` is the position's daily volatility.

```python
import numpy as np

annual_vol = 0.28          # capstone_portfolio_context.annual_volatility for CRNM
daily_vol = annual_vol / np.sqrt(252)
position_value = 800_000    # 8% of a $10M book

z_95, z_99 = 1.645, 2.326
var_95_1day = z_95 * daily_vol * position_value
var_99_1day = z_99 * daily_vol * position_value

print(f"daily vol: {daily_vol:.4%}")          # ≈ 1.76%
print(f"95% 1-day VaR: ${var_95_1day:,.0f}")   # ≈ $23,200
print(f"99% 1-day VaR: ${var_99_1day:,.0f}")   # ≈ $32,800
```

Parametric VaR is fast, defensible, and cheap to compute — and it inherits every flaw of the normal-distribution assumption. Real return distributions have **fat tails**: extreme moves happen more often than a normal curve predicts. Parametric VaR systematically *understates* tail risk for exactly the events you most need to be warned about. That's why you never rely on it alone.

## 3. Historical VaR — let the data speak, no distribution assumed

Historical VaR skips the normality assumption entirely: take the position's actual historical daily returns, sort them, and read off the empirical percentile.

```sql
-- Daily returns for CRNM from the seeded price history
WITH daily_returns AS (
    SELECT
        price_date,
        close_price,
        close_price / LAG(close_price) OVER (ORDER BY price_date) - 1 AS daily_return
    FROM capstone_price_history
    WHERE ticker = 'CRNM'
)
SELECT
    PERCENTILE_CONT(0.05) WITHIN GROUP (ORDER BY daily_return) AS pct_5th,
    PERCENTILE_CONT(0.01) WITHIN GROUP (ORDER BY daily_return) AS pct_1st
FROM daily_returns
WHERE daily_return IS NOT NULL;
```

The 5th-percentile daily return is your **95% historical VaR** as a return; multiply by `-1 × position_value` to express it in dollars (the 5th-percentile return is negative — a loss — so flip the sign to report VaR as a positive number). The 1st percentile gives you 99% VaR the same way.

**Expected result:** because `capstone_price_history` was generated from a geometric Brownian motion (log-normal by construction, per the README's generation script), historical VaR should land close to the parametric estimate — typically within 15–25% of it. That closeness is a *property of the simulated data*, not a general truth: real markets have fatter tails than GBM, and on real data historical VaR at the 1% level is usually noticeably worse than parametric VaR predicts. Don't let this week's tidy agreement make you complacent about that gap on live data.

```python
import pandas as pd

returns = prices["close_price"].pct_change().dropna()
hist_var_95 = -returns.quantile(0.05) * position_value
hist_var_99 = -returns.quantile(0.01) * position_value
```

## 4. Portfolio VaR — why 8% weight in a 28%-vol asset doesn't mean 8% of the pain

The naive instinct is: CRNM is 8% of the book and twice as volatile as the core, so it must be responsible for a lot more than 8% of the portfolio's risk. **Correlation changes that answer**, and by how much depends entirely on how CRNM moves relative to everything else you already hold.

Treat the six-fund core as one block (its own aggregate volatility is already known from Week 8's optimizer output — call it 8.5%) and combine it with CRNM using the two-asset portfolio variance formula:

```
σ_p² = w_core² σ_core² + w_crnm² σ_crnm² + 2 w_core w_crnm σ_core σ_crnm ρ(core, crnm)
```

```python
w_core, w_crnm = 0.92, 0.08
sigma_core, sigma_crnm = 0.085, 0.28

# Weighted-average correlation of the core book to CRNM,
# from capstone_portfolio_context.corr_with_crnm, weighted by each fund's core weight
rho = 0.27

port_var = (w_core**2 * sigma_core**2
            + w_crnm**2 * sigma_crnm**2
            + 2 * w_core * w_crnm * sigma_core * sigma_crnm * rho)
port_vol = port_var ** 0.5
print(f"portfolio volatility: {port_vol:.4%}")   # ≈ 8.70%
```

Adding an 8% position in a 28%-vol asset moved the **whole portfolio's** volatility from 8.50% to only 8.70% — twenty basis points. That's the diversification math from Week 8 doing real work: CRNM's correlation to the core book (0.27, moderate, not extreme) means most of its idiosyncratic risk gets diversified away at the portfolio level, even though it's a genuinely risky position held on its own. **This is exactly why position sizing has to happen at the portfolio level, not the position level** — sizing CRNM by "how risky is CRNM alone" would badly overstate the true cost of adding it.

Run the same VaR formula from Section 2 at the portfolio level using `port_vol` instead of `annual_vol`, scaled to the full book size, to get a 95%/99% VaR for the whole $10M portfolio *with* the CRNM position — the number you'd actually put in front of the IC.

## 5. Scenario analysis — probability-weighted, not worst-case

VaR tells you about the *typical* bad day. **Scenario analysis** asks a different question: given a handful of *named, specific* things that could happen, what happens to the valuation itself? `capstone_scenarios` gives you six scenarios with probability weights that sum to 1.00 and shocks to growth, margin, exit multiple, and WACC.

```sql
-- Recompute each scenario's shocked EBITDA and terminal value inputs
-- against the Base Case cash-flow stream (Exercise 2 builds the full
-- shocked five-year rebuild; this shows the shape of the join)
SELECT
    s.scenario_name,
    s.probability,
    s.revenue_growth_shock,
    s.ebitda_margin_shock,
    s.exit_multiple_shock,
    s.wacc_shock,
    d.exit_multiple_ev_ebitda + s.exit_multiple_shock AS shocked_exit_multiple,
    d.wacc + s.wacc_shock AS shocked_wacc
FROM capstone_scenarios s
CROSS JOIN capstone_deal_snapshot d
ORDER BY s.probability DESC;
```

The full mechanics — rebuilding the five-year FCF stream under each scenario's shocked growth/margin, then re-discounting at the shocked WACC — are Exercise 2. The output you want is a table of **six per-share values, each with a probability**, from which you compute a probability-weighted expected value:

```
E[per_share] = Σ (probability_i × per_share_i)
```

That expected value is a genuinely different, more honest number than the single-point $15.29 blended target from Lecture 1 — it explicitly bakes in the 20% chance of a recession scenario and the 3% chance of a credit crunch, rather than pretending the base case is the only case.

## 6. Stress testing — the scenario nobody assigned a probability to

Scenario analysis covers cases you thought to name. **Stress testing** covers the case you didn't — deliberately extreme, often historically-inspired, with no probability attached because the point isn't "how likely," it's "how bad, and do we survive it." A standard stress test combines the worst elements of your named scenarios simultaneously, even if that combination feels implausible:

```python
# A stress case: Recession's growth/margin shock, Credit Crunch's WACC shock,
# AND Stagflation's exit-multiple shock — three "unlikely on their own"
# scenarios' worst individual components, combined
stress_growth_shock = -0.040      # from Recession
stress_margin_shock = -0.020      # from Recession
stress_wacc_shock = 0.015         # from Credit Crunch
stress_multiple_shock = -2.0      # from Stagflation
```

Run this combined shock through the same DCF machinery from Lecture 1 and see what per-share value comes out. If it's still comfortably above the market price, your thesis has real margin of safety. If it's below the market price, you've just learned something no single named scenario told you: **there is a plausible combination of bad news under which this position loses money even against today's already-low price** — and now you have to decide whether your position size accounts for that, which is exactly what Challenge 1 asks you to argue.

## 7. Putting the three tools in one sentence each

- **VaR** answers: "on a normal bad day, how much do I lose?" — a statement about *frequency*.
- **Scenario analysis** answers: "if this specific, named thing happens, what's my valuation?" — a statement about *named, probability-weighted futures*.
- **Stress testing** answers: "what's the worst *plausible* combination, and do I survive it?" — a statement about *tail severity*, deliberately without a probability attached.

A complete risk section uses all three. Using only one — usually VaR, because it's the easiest to compute — is the single most common gap between a memo that looks rigorous and one that actually is.

## 8. Check yourself

- In one sentence, what does a 95% 1-day VaR of $23,200 actually promise you — and what does it explicitly NOT promise?
- Why does historical VaR on this week's simulated data land close to parametric VaR, and why shouldn't you expect that on real market data?
- Walk through why an 8% position in a 28%-vol asset raised portfolio volatility by only 20bps instead of something much larger.
- What's the structural difference between a "scenario" and a "stress test" in this lecture's usage?
- Why does a stress test deliberately combine components from scenarios that were each individually assigned a low probability?

If those are automatic, Lecture 3 turns to the part of risk that no VaR number captures at all: whether the people running the target company, and you yourself, have incentives pointed the same direction as the numbers you just computed.

## Further reading

- **J.P. Morgan — RiskMetrics Technical Document (the parametric-VaR origin paper, free PDF):** <https://www.msci.com/documents/10199/5915b101-4206-4ba0-aee2-3449d5c7e95a>
- **Basel Committee on Banking Supervision — "Messages from the academic literature on risk measurement for the trading book":** <https://www.bis.org/publ/bcbs_wp19.htm>
- **PostgreSQL — `PERCENTILE_CONT` and ordered-set aggregates:** <https://www.postgresql.org/docs/current/functions-aggregate.html#FUNCTIONS-ORDERED-SET-TABLE>
