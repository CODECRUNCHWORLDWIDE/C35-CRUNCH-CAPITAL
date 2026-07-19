# Lecture 3 — CAPM and Factor Tilts

> **Duration:** ~2 hours. **Outcome:** You can regress each fund's beta against the market benchmark, price its CAPM-required return, explain the capital market line and how it differs from the efficient frontier, and reason carefully about adding a factor tilt to a baseline allocation.

Lecture 2 built the efficient frontier assuming the *only* assets available were the six risky funds. This lecture adds one more building block — a genuinely risk-free asset — and shows how that single addition simplifies the entire allocation decision into "how much risk-free asset, and how much of one specific risky portfolio." That result is the **Capital Asset Pricing Model (CAPM)**, and you already used half of its machinery back in Week 4 to price one company's cost of equity. This week applies the exact same regression to six funds at once, and then asks the harder, more honest question: once you know each asset's beta, should you *deliberately* tilt an allocation toward or away from it — and what are you actually betting on if you do?

## 1. Adding a risk-free asset changes everything

Lecture 2's frontier assumed every dollar had to go into one of the six risky funds. Real portfolios also have access to something closer to genuinely risk-free — cash, a money-market fund, short-term Treasury bills — earning the risk-free rate $r_f$ with (for practical purposes) zero volatility and zero correlation with everything else.

Once a risk-free asset is available, the efficient "frontier" is no longer a curve — it becomes a **straight line**, called the **capital market line (CML)**, running from $(0, r_f)$ on the volatility/return plane, through the tangency portfolio you solved for in Lecture 2, and extending onward:

$$\mu_p = r_f + \left(\frac{\mu_{\text{tan}} - r_f}{\sigma_{\text{tan}}}\right) \sigma_p$$

The slope of that line is exactly the tangency portfolio's Sharpe ratio — the best risk-adjusted deal available in the whole universe. **Every point on this line dominates every point on the risky-only frontier at the same volatility** (except the single point where they touch, the tangency portfolio itself) — which is precisely why the line is tangent to the curve, and precisely why it's a strictly better set of choices once cash is allowed in the mix.

This produces one of the cleanest results in all of finance, often called the **two-fund theorem with a risk-free asset**, or **fund separation**: *every* rational investor, regardless of their personal risk tolerance, should hold some mix of just two things — the risk-free asset, and the *one* tangency portfolio — varying only the split between them.

- **More conservative investor:** mostly risk-free asset, a small slice of the tangency portfolio — a point on the CML *below* the tangency portfolio, at lower volatility.
- **More aggressive investor:** more than 100% in the tangency portfolio, financed by borrowing at (something close to) the risk-free rate — a point on the CML *above* the tangency portfolio, at higher volatility than the tangency portfolio itself.

```python
import numpy as np

rf = 0.03
tan_return, tan_vol = 0.0804, 0.0559   # from Lecture 2's tangency solution
cml_slope = (tan_return - rf) / tan_vol   # this IS the tangency Sharpe ratio, 0.90

for target_vol in [0.02, 0.04, 0.0559, 0.08, 0.10]:
    cml_return = rf + cml_slope * target_vol
    print(f"vol {target_vol:.2%} -> CML return {cml_return:.4%}")
```

Notice: at `target_vol = 0.0559` (the tangency portfolio's own volatility), the CML gives back exactly the tangency return, 8.04% — that's the tangent point. Below it, you're blending cash with the tangency portfolio; above it, you'd need to borrow to get there (a real portfolio implements that with leverage, which this course flags but does not build out this week — leverage introduces margin, financing-rate, and liquidation risk well beyond the scope of a mean-variance model).

## 2. Estimating beta — the same regression as Week 4, six times over

CAPM says a single asset's **required return** depends only on how much of the market's risk it carries, measured by **beta**:

$$E[r_i] = r_f + \beta_i \left(E[r_m] - r_f\right)$$

$\beta_i$ measures how sensitive asset $i$'s returns are to the market's returns — the slope of a regression of the asset's returns on the market's returns:

$$\beta_i = \frac{\text{Cov}(r_i, r_m)}{\text{Var}(r_m)}$$

This is exactly the beta regression from Week 4, applied to all six funds against the `MKT` benchmark:

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql://localhost/crunch_capital")
prices_long = pd.read_sql("SELECT * FROM asset_prices ORDER BY price_date, ticker", engine)
prices = prices_long.pivot(index="price_date", columns="ticker", values="adj_close")
prices.index = pd.to_datetime(prices.index)

returns = prices.pct_change().dropna()
port_tickers = ["CRNQ", "CRNI", "CRNB", "CRNH", "CRNG", "CRNR"]

mkt_var = returns["MKT"].var(ddof=1)
betas = {}
for t in port_tickers:
    cov_with_mkt = returns[t].cov(returns["MKT"])
    betas[t] = cov_with_mkt / mkt_var

for t, b in betas.items():
    print(f"{t}: beta = {b:.4f}")
```

Against the seed, you should land close to:

| Ticker | Beta | Interpretation |
|--------|-----:|----------------|
| CRNQ | 1.06 | Slightly more volatile than the market, moves closely with it |
| CRNI | 1.06 | Similar market sensitivity to the US equity fund |
| CRNB | −0.09 | Essentially uncorrelated with the market, very slightly *opposite* |
| CRNH | 0.29 | Some market sensitivity, well below 1 |
| CRNG | 0.05 | Almost no relationship with the market — a genuine diversifier |
| CRNR | 0.42 | Moderate market sensitivity |

A beta above 1 means the asset amplifies market moves (bigger swings both up and down); a beta below 1 (but positive) means it dampens them; a beta near zero means the asset's returns are, on this evidence, largely disconnected from broad market moves — `CRNG` (gold) is exactly this kind of asset in the seed data, which is *why* it earned a meaningful slice of the tangency portfolio in Lecture 2 despite a modest expected return: low correlation with everything else is valuable in its own right.

You can also get beta (and the full regression) with `numpy.polyfit`, exactly as Week 4 did:

```python
beta, alpha = np.polyfit(returns["MKT"], returns["CRNQ"], deg=1)
print(f"CRNQ: beta={beta:.4f}, monthly alpha={alpha:.4%}")
```

`polyfit` with `deg=1` fits `y = beta*x + alpha`, so it returns slope (beta) first, intercept (alpha) second — the same answer as the covariance-ratio formula, just via ordinary least squares instead of the closed-form ratio (for a simple one-variable regression, they're mathematically identical).

## 3. Pricing each fund's CAPM-required return

With betas in hand, plug them into CAPM using the risk-free rate and equity risk premium from `capital_market_assumptions`:

```python
rf = 0.03
erp = 0.055   # from capital_market_assumptions -- a SUPPLIED assumption, not derived from the data

for t in port_tickers:
    capm_required = rf + betas[t] * erp
    actual_realized = returns[t].mean() * 12
    print(f"{t}: CAPM-required {capm_required:.4%}   realized (5yr) {actual_realized:.4%}")
```

Run this and something should jump out: **every fund's realized 5-year return sat above its CAPM-required return** — sometimes by a lot (`CRNG`, gold, had a CAPM-required return around 3.3% but realized roughly 8.1%). Is this evidence that gold has genuine, repeatable "alpha" — skill or a structural edge that beats what its risk alone would predict? Or is it simply what a strong five-year bull run for *most* assets looks like in the rearview mirror, with plenty of noise in a 60-month sample? **CAPM makes no claim about any single historical window** — it's a statement about the *expected* return investors should require for a given level of *systematic* (market) risk, not a promise about what any five years will actually deliver. A responsible analyst treats a five-year gap between realized and CAPM-required return as a question to investigate, never as proof of skill on its own.

## 4. What CAPM does and doesn't explain

CAPM's central claim is that **only systematic risk (beta) is priced** — risk that can't be diversified away, because it's driven by broad market movements every asset is at least somewhat exposed to. **Idiosyncratic risk** — the part of an asset's variance that's specific to it and uncorrelated with the market — is, in CAPM's world, risk an investor can eliminate for free simply by diversifying, so the market doesn't compensate anyone for bearing it.

You can decompose any asset's total variance into these two pieces:

$$\sigma_i^2 = \underbrace{\beta_i^2 \, \sigma_m^2}_{\text{systematic}} + \underbrace{\sigma_{\epsilon,i}^2}_{\text{idiosyncratic}}$$

```python
mkt_var_annual = returns["MKT"].var(ddof=1) * 12
for t in port_tickers:
    total_var = returns[t].var(ddof=1) * 12
    systematic_var = (betas[t] ** 2) * mkt_var_annual
    idiosyncratic_var = total_var - systematic_var
    pct_systematic = systematic_var / total_var
    print(f"{t}: {pct_systematic:.1%} of variance is systematic (market-driven)")
```

`CRNQ` and `CRNI`, with betas near 1.0 and high correlation to `MKT`, will show a large fraction of their variance as systematic. `CRNG` (gold), with a beta near zero, will show almost none of its variance as systematic — nearly all of gold's risk in this dataset is idiosyncratic, asset-specific movement that CAPM says the market doesn't compensate you for bearing (in real markets, gold's actual return drivers — inflation expectations, real interest rates, currency dynamics, geopolitical demand — are real and often studied as their own priced factors; CAPM's single-factor lens simply doesn't capture them, which is exactly the gap that motivated the multi-factor models below).

## 5. Factor tilts — going beyond a single market beta

CAPM uses exactly one factor: sensitivity to the overall market. Decades of empirical research since (Fama-French, and much subsequent factor-investing work) found that other systematic characteristics — company size, valuation ("value" vs. "growth"), momentum, and others — also show persistent relationships with average returns beyond what market beta alone explains. A **factor tilt** means deliberately over- or under-weighting an allocation along one of these characteristics, relative to a market-cap-weighted or naive baseline.

This week's universe is deliberately simple (six broad asset-class funds, not individual stocks with size/value/momentum characteristics), so we can't run a real Fama-French-style regression here — that's a natural extension for later coursework. But the *logic* of a factor tilt is fully teachable with what you already have, because **beta itself is a factor**, and you can tilt toward or away from it directly:

- **A low-beta tilt**: overweight `CRNB` and `CRNG` (betas near zero) relative to their tangency-portfolio weights, accepting lower expected return in exchange for a portfolio less sensitive to broad market drawdowns — a defensive posture appropriate for an investor who specifically wants to reduce market-timing risk.
- **A high-beta tilt**: overweight `CRNQ`/`CRNI` (betas near 1.06) relative to the tangency weights, deliberately taking on more market sensitivity than the "optimal" Sharpe-maximizing mix would suggest — appropriate only for an investor with a specific view that the market itself is about to outperform, or a long horizon and high risk tolerance that makes maximizing *raw* expected return more relevant than maximizing the Sharpe ratio.

```python
# Illustrative low-beta tilt: shift 15 percentage points of weight from the two
# highest-beta funds into the two lowest-beta funds, relative to the tangency portfolio
w_tan = np.array([0.235, -0.006, 0.451, 0.082, 0.201, 0.037])  # CRNQ,CRNI,CRNB,CRNH,CRNG,CRNR order
tilt = np.array([-0.10, -0.05, 0.08, 0.00, 0.07, 0.00])         # sums to 0 -- a REALLOCATION, not new money
w_tilted = w_tan + tilt

mu = np.array([0.1380, 0.1378, 0.0468, 0.0999, 0.0806, 0.0872])
betas_arr = np.array([1.0637, 1.0581, -0.0945, 0.2935, 0.0503, 0.4241])

print("Tangency portfolio beta:", w_tan @ betas_arr)
print("Tilted portfolio beta:  ", w_tilted @ betas_arr)
print("Tangency expected return:", w_tan @ mu)
print("Tilted expected return:  ", w_tilted @ mu)
```

The honest way to describe any tilt like this: **you are deliberately moving away from the mathematically Sharpe-optimal portfolio, on purpose, because you hold a view (about future beta pricing, about a specific risk you want to avoid, about your own horizon) that the historical mean-variance optimization can't see.** A tilt is a *judgment call layered on top of the math*, not itself an output of the optimizer — say that explicitly whenever you make one, rather than presenting a tilted portfolio as if the optimizer produced it.

## 6. Check yourself

- Why does adding a genuinely risk-free asset turn the efficient frontier from a curve into a straight line?
- What does the slope of the capital market line represent, in terms you've already computed?
- If an asset has a beta of exactly 0, what does CAPM say its required return should be, and does that match your intuition for why?
- In the seed data, every fund's 5-year realized return beat its CAPM-required return. Give two different explanations for this — one that would mean the funds have genuine skill/edge, one that's just statistical noise in a bull-market sample — and explain how you'd start to distinguish between them with more data.
- What's the difference between systematic and idiosyncratic risk, and why does CAPM say only one of them is compensated?
- Describe, in your own words, what a "low-beta tilt" is trying to accomplish, and what an investor is implicitly giving up to get it.

This closes out the lecture content for the week. Head to the exercises to build the covariance matrix, trace the frontier, and solve the tangency portfolio yourself from scratch — then the challenges push you into constrained, more realistic territory, and the mini-project asks you to run this entire pipeline on a portfolio you construct.

## Further reading

- **Sharpe (1964), "Capital Asset Prices: A Theory of Market Equilibrium under Conditions of Risk"** — the original CAPM paper: <https://www.jstor.org/stable/2977928>
- **Fama & French (1993), "Common Risk Factors in the Returns on Stocks and Bonds"** — the paper that launched multi-factor investing: <https://doi.org/10.1016/0304-405X(93)90023-5>
- **Investopedia — Capital Market Line:** <https://www.investopedia.com/terms/c/cml.asp>
- **CFA Institute — Capital Market Theory (Level I curriculum overview):** <https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/portfolio-risk-and-return-part-ii>
