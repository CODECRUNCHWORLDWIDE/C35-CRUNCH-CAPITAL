# Lecture 2 — Estimating Beta from Data

> **Duration:** ~2 hours. **Outcome:** You can pull two price series out of SQL, turn them into returns in pandas, regress one against the other to estimate beta, alpha, and R² yourself, and explain the difference between a levered and an unlevered beta well enough to relever one at a different capital structure.

Lecture 1 used a vendor-supplied beta of 1.30 without asking where it came from. That's fine as a starting point, but a vendor beta is a black box — you don't know its lookback window (2 years of weekly returns? 5 years of monthly?), you don't know if it's raw or "adjusted" toward 1.0 (a common vendor practice), and you can't defend it in a room if someone asks how it was built. This lecture fixes that: you'll compute Crunch Machine Co.'s beta yourself, from the `stock_prices` table already in your database, using nothing but a regression you can explain line by line.

## 1. From prices to returns

A regression needs **returns**, not prices. Prices trend upward over time for reasons that have nothing to do with risk (inflation, growth, compounding); returns — the period-over-period percentage change — are what actually carries the risk signal CAPM cares about.

Pull the price history out of SQL and reshape it in pandas:

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://localhost/crunch_capital")

prices = pd.read_sql(
    "SELECT ticker, price_date, adj_close FROM stock_prices ORDER BY ticker, price_date",
    engine,
    parse_dates=["price_date"],
)

# Pivot so each ticker is its own column, indexed by date
wide = prices.pivot(index="price_date", columns="ticker", values="adj_close")
print(wide.head())
print(wide.shape)   # (61, 2) — 61 month-end prices for CRNM and MKT
```

```
             CRNM     MKT
price_date
2020-12-31  42.00  4000.00
2021-01-31  35.79  3827.13
2021-02-28  36.30  3990.57
2021-03-31  39.77  4229.02
2021-04-30  43.27  4402.82
```

Now compute monthly returns with `pct_change()`, and drop the first row (there's no prior price to compute a return from):

```python
returns = wide.pct_change().dropna()
print(returns.head())
print(returns.shape)   # (60, 2) — 60 monthly returns, the standard "5-year monthly beta" window
```

```
                CRNM       MKT
price_date
2021-01-31 -0.147857 -0.043218
2021-02-28  0.014250  0.042703
2021-03-31  0.095592  0.059759
2021-04-30  0.088006  0.041091
```

Sixty monthly return observations spanning five years is a standard, industry-typical window for a "long-run" beta (Bloomberg's default beta uses roughly this window). Shorter windows react faster to a company's *current* risk profile but are noisier; longer windows are smoother but can blend together a company's old and new risk character if its business has changed.

## 2. Running the regression

Beta is the slope of stock returns regressed on market returns: $R_{\text{stock}} = \alpha + \beta \cdot R_{\text{market}} + \varepsilon$. `numpy.polyfit` fits exactly this line with one call:

```python
import numpy as np

slope, intercept = np.polyfit(returns["MKT"], returns["CRNM"], 1)
print(f"beta:  {slope:.4f}")
print(f"alpha (monthly): {intercept:.4%}")
```

```
beta:  1.2429
alpha (monthly): 0.2855%
```

**Beta ≈ 1.24.** That's noticeably below the vendor's 1.30 — not wrong, just a *different, self-computed estimate*, built from a specific, known window (60 monthly closes, December 2020–December 2025) using a specific, known method (OLS of returns on returns). That auditability is the entire point of doing this yourself.

**Alpha ≈ 0.29% per month** is the average monthly return CRNM earned *beyond* what its market exposure alone would predict — roughly 3.4% annualized (rough, because compounding twelve months of a small rate isn't exactly ×12, but it's a fine back-of-envelope conversion for spotting the order of magnitude). A positive historical alpha is common in backward-looking regressions and shouldn't be read as "this stock will keep beating its beta-implied return forever" — CAPM's own logic says a stock's alpha should be indistinguishable from noise once the model is correctly specified; a persistently large trailing alpha is much more often measurement noise or omitted risk factors than genuine, repeatable skill.

### 2.1 How good is the fit? R² and correlation

Beta tells you the *slope* of the relationship; R² tells you how much of the stock's variance that slope actually explains — i.e., how *reliable* this beta estimate is.

```python
corr = returns["MKT"].corr(returns["CRNM"])
r_squared = corr ** 2
print(f"correlation: {corr:.4f}")
print(f"R-squared:   {r_squared:.4f}")
```

```
correlation: 0.7630
R-squared:   0.5822
```

**R² ≈ 0.58** means about 58% of CRNM's month-to-month return variance is explained by market-wide moves; the other 42% is idiosyncratic — Crunch Machine Co.-specific news, operational surprises, and plain noise. That's a *reasonable* fit for a single operating company (individual-stock betas typically run R² 0.2–0.6; only diversified portfolios or index funds get much higher), but it's also an honest reminder that beta is an estimate with real uncertainty around it, not a precise physical constant. Two analysts using slightly different windows or frequencies will get slightly different betas, and that's expected, not a sign either one made an error.

### 2.2 Doing it with `statsmodels` (more detail, same answer)

`numpy.polyfit` is fine for a quick slope-and-intercept, but `statsmodels` gives you the full regression report — standard errors, t-statistics, confidence intervals — which matters when you want to say *how* confident you are in the beta estimate, not just what it is:

```python
import statsmodels.api as sm

X = sm.add_constant(returns["MKT"])     # adds the intercept term
model = sm.OLS(returns["CRNM"], X).fit()
print(model.summary())
```

The `coef` on `MKT` in that output is beta; the `coef` on `const` is alpha; `R-squared` matches what you computed above; and the `std err` / `t` / `P>|t|` columns tell you whether the beta estimate is statistically distinguishable from, say, 1.0 or 0 — useful when you want to argue "this stock's beta is meaningfully above the market" rather than just eyeballing the number. (`statsmodels` isn't in this course's core install list — `pip install statsmodels` if you want to follow along; `numpy.polyfit` is sufficient for every exercise this week.)

## 3. Levered vs. unlevered beta

The beta you just estimated — **1.24** — is Crunch Machine Co.'s **levered beta** ($\beta_L$, sometimes called *equity beta*): it reflects the full risk borne by equity holders, which includes both the company's underlying *business* risk and the extra risk added by its *financial* leverage (debt). Debt amplifies equity's risk because interest and principal get paid first, no matter how the business performs — equity absorbs all the remaining variability, on a smaller base of capital. More debt, same business, means a *higher* levered beta, even though nothing about the operations changed.

This matters because it means you can't directly compare betas across two companies with different debt loads and conclude one has a riskier *business* — some of the gap might just be financing choice. The fix is the **Hamada equation**, which strips financial leverage out of a levered beta to get the **unlevered beta** ($\beta_U$, also called *asset beta*):

$$
\beta_U = \frac{\beta_L}{1 + (1-t)\left(\dfrac{D}{E}\right)}
$$

where $t$ is the tax rate and $D/E$ is the company's debt-to-equity ratio (both at market values). $\beta_U$ represents the risk of the company's underlying *assets and operations only*, as if it had no debt at all — the risk two companies in the same industry would share if you stripped their capital structures away.

Once you have an unlevered beta, you can **relever** it at any target capital structure — a different company's, a hypothetical one, or a new division's:

$$
\beta_L^{\text{target}} = \beta_U \times \left(1 + (1-t)\left(\dfrac{D}{E}\right)_{\text{target}}\right)
$$

### 3.1 Worked example: unlever, then relever

Suppose a close peer of Crunch Machine Co. — call it **Steelcrest Automation** — has a levered beta of 1.60, carries $D/E = 0.90$, and faces a 25% tax rate. Its business is comparable to Crunch Machine Co.'s, but it's financed far more aggressively. Unlever it:

```python
beta_L_peer = 1.60
de_peer = 0.90
tax = 0.25

beta_U_peer = beta_L_peer / (1 + (1 - tax) * de_peer)
print(f"unlevered beta: {beta_U_peer:.4f}")   # 0.9552
```

That **0.9552** is Steelcrest's *asset* beta — its business risk alone, financing stripped out. Now relever it at Crunch Machine Co.'s own capital structure ($D/E \approx 0.2623$, from Lecture 3's balance-sheet numbers) to see what Steelcrest's business risk would imply for a company financed the way Crunch Machine Co. actually is:

```python
de_crnm = 0.2623
beta_relevered = beta_U_peer * (1 + (1 - tax) * de_crnm)
print(f"relevered beta: {beta_relevered:.4f}")   # 1.1442
```

**1.1442** — noticeably below Crunch Machine Co.'s own regression beta of 1.24. That gap is informative: it suggests either Crunch Machine Co.'s own stock carries extra idiosyncratic volatility that a single peer doesn't capture (remember, R² was only 0.58 — 42% of the variance is unexplained by the market alone), or Steelcrest isn't quite as close a comparable as assumed. Neither conclusion is wrong to draw — this is exactly the kind of judgment call a working analyst makes by triangulating *multiple* beta estimates rather than trusting any single one blindly.

### 3.2 Why bother relevering at all?

Three real situations where this matters:

1. **Valuing a private company or a new division** with no traded stock of its own — you have no beta to regress. You gather several *public, comparable* companies' betas, unlever each (removing their idiosyncratic financing choices), average the unlevered betas, then relever the average at the *target's* actual or planned capital structure. Challenge 1 this week is exactly this workflow, done properly across four peers.
2. **Comparing business risk across companies with different leverage** — you can't fairly say Company A's operations are "riskier" than Company B's just because A has a higher levered beta, if A also happens to carry twice the debt.
3. **Modeling how a company's own beta (and therefore its own cost of equity, and therefore its own WACC) would change if it changed its capital structure** — issue more debt, buy back equity, or delever. Unlever at the current structure, relever at the proposed one, and you get the *implied* new equity beta before the change ever happens. (This is where Week 5 — capital structure & leverage — picks up.)

## 4. Which beta should you actually use?

By the end of this week you'll have computed Crunch Machine Co.'s beta **three different, individually defensible ways**:

| Method | Beta | Where it comes from |
|--------|-----:|---------------------|
| Vendor-supplied | 1.30 | A data provider's black-box calculation (Lecture 1) |
| Self-computed regression | 1.24 | Your own OLS of CRNM's returns on MKT's returns, this lecture |
| Comparables (unlever/relever) | ~1.09 | Averaging peers' unlevered betas, relevered at Crunch Machine Co.'s structure (Challenge 1) |

There is no single "correct" answer among these — that's not a bug in the exercise, it's the honest state of the practice. A careful analyst computes more than one, understands *why* they differ (different data windows, different underlying assumptions, different peer sets), and either picks the one best suited to the decision at hand or reports a range instead of false precision. What you should never do is present a single beta — vendor, regression, or comps-based — as though it were an unambiguous fact. It's an estimate, built on a specific method, and the method is part of the answer.

## 5. Check yourself

- Why does a beta regression use *returns* rather than raw *prices*?
- What do the slope and intercept of the regression represent, in plain English?
- If R² is 0.58, what fraction of the stock's return variance is *not* explained by the market, and what does that unexplained portion represent?
- Write the Hamada unlevering formula from memory, and explain in one sentence why more debt raises a levered beta even if the underlying business is unchanged.
- A peer has $\beta_L = 1.10$, $D/E = 0.40$, $t = 0.24$. Compute its unlevered beta.
- Name two real situations where you'd need to relever a beta rather than just using a company's own regression beta.

Lecture 3 uses the regression-derived beta (1.24) as the "house" beta for Crunch Machine Co.'s WACC, prices its actual outstanding debt, and assembles the full weighted average cost of capital.

## Further reading

- **pandas — `DataFrame.pct_change`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pct_change.html>
- **NumPy — `polyfit`:** <https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html>
- **statsmodels — Ordinary Least Squares (OLS):** <https://www.statsmodels.org/stable/regression.html>
- **Aswath Damodaran — "Estimating Risk Parameters and Betas" (levered/unlevered beta, comparables method):** <https://pages.stern.nyu.edu/~adamodar/pdfiles/papers/beta.pdf>
