# Lecture 1 — Risk, Return, and Covariance

> **Duration:** ~2 hours. **Outcome:** You can compute expected return, variance, and covariance from raw prices in SQL and pandas, state precisely why a portfolio's variance is *not* the weighted average of its parts' variances, and explain — with a real number, not a slogan — why combining imperfectly correlated assets reduces risk.

Every other week of this course asked "is this thing good." This week asks a different question: "is this thing good *in combination with everything else I own*." That shift turns out to change the math completely, and the piece of math that changes it is **covariance** — how two assets' returns move together. Get comfortable with covariance this lecture and the rest of the week (frontier, tangency portfolio, CAPM) is just building on top of a matrix you already understand.

## 1. Setting up the universe

Confirm the seed loaded:

```sql
SELECT ticker, COUNT(*), MIN(price_date), MAX(price_date), MIN(adj_close), MAX(adj_close)
FROM asset_prices
GROUP BY ticker
ORDER BY ticker;
```

You should see seven tickers, each with 61 monthly prices from `2020-12-31` to `2025-12-31`. Six are **investable funds** — `CRNQ`, `CRNI`, `CRNB`, `CRNH`, `CRNG`, `CRNR` — and one, `MKT`, is a **non-investable benchmark index** used only for the CAPM work in Lecture 3. Don't put `MKT` in a portfolio; you can't buy "the market" directly, only proxies for it.

Pull everything into pandas and pivot it into a wide table — one column per ticker, one row per month, exactly the shape you'll use for the rest of the week:

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql://localhost/crunch_capital")
# or: engine = create_engine("sqlite:///crunch_capital.db")

prices_long = pd.read_sql("SELECT * FROM asset_prices ORDER BY price_date, ticker", engine)
prices = prices_long.pivot(index="price_date", columns="ticker", values="adj_close")
prices.index = pd.to_datetime(prices.index)
print(prices.shape)   # (61, 7)
print(prices.head())
```

## 2. From prices to returns

You never compute statistics on price *levels* — a stock at $50 and one at $5,000 aren't comparable, and prices trend, which breaks the "roughly stationary" assumption most of this week's math leans on. You compute them on **returns**: the percentage change from one period to the next.

```python
returns = prices.pct_change().dropna()
print(returns.shape)   # (60, 7) -- 61 prices give 60 returns
```

`.pct_change()` computes `(P_t - P_{t-1}) / P_{t-1}` row by row; the first row is `NaN` because there's no prior price to compare against, hence `.dropna()`. This week works exclusively in **simple monthly returns** (not log returns) because portfolio weights combine linearly under simple returns — a property Lecture 2's whole optimization leans on. (Log returns are more convenient for *single-asset* time-series work, which is why C33/other analytics contexts sometimes prefer them — but they don't add across assets the way a portfolio return needs to, so simple returns are the right tool here.)

## 3. Expected return — the easy half

The **expected return** of an asset, over some future period, is what you predict its return will be. In practice, without a crystal ball, the standard (if imperfect) starting estimate is the **sample mean of historical returns**:

```python
mean_monthly = returns.mean()
mean_annual = mean_monthly * 12
print(mean_annual.round(4))
```

Run this against the seed and you'll see the six funds' annualized mean returns cluster in a believable range — the two equity funds (`CRNQ`, `CRNI`) highest at roughly **13.8%**, the core bond fund (`CRNB`) lowest at roughly **4.7%**, and the others in between. (`× 12`, not compounding, is the standard *arithmetic* annualization used for this kind of quick estimate — good enough for comparing assets at a glance; a precise compounded annual growth rate would use `(1+mean_monthly)**12 - 1`, which gives a very slightly lower number because of Jensen's inequality. This week annualizes arithmetically throughout for consistency; note the distinction in your own notes.)

**The honest caveat**, up front, because it matters for everything downstream: five years of monthly data is 60 observations — nowhere near enough to pin down a *true* expected return with any real precision. The standard error of a mean estimated from 60 monthly draws of an asset with 19% annual volatility is on the order of several percentage points. Using trailing sample means as a stand-in for "expected" returns is standard practice and exactly what you'll do all week — but it's a known weak point of naive mean-variance optimization, and Challenge 2 makes you feel exactly how much that weakness matters.

## 4. Variance and volatility — risk for one asset

**Variance** measures how spread out an asset's returns are around their mean — literally, the average squared deviation from the mean:

$$\text{Var}(r) = \sigma^2 = \frac{1}{n-1}\sum_{t=1}^{n}(r_t - \bar r)^2$$

**Volatility** (standard deviation, $\sigma$) is just the square root of variance, back in the same units as returns (not squared-returns) — it's the number people actually quote ("this fund runs about 17% annual volatility").

```python
var_monthly = returns.var(ddof=1)      # ddof=1 = sample variance (n-1 denominator)
vol_monthly = returns.std(ddof=1)
vol_annual = vol_monthly * (12 ** 0.5) # volatility scales with SQUARE ROOT of time, not time itself
print(vol_annual.round(4))
```

**Why `√12` and not `12`?** Variances of independent periods *add*: `Var(12 months) = 12 × Var(1 month)`, because variance is additive across independent draws. Volatility is the square root of variance, so `Vol(12 months) = √12 × Vol(1 month)`. This "square-root-of-time" scaling is one of the most load-bearing facts in all of quantitative finance — it's why risk grows slower than the time horizon, and why long-horizon investors can typically tolerate more volatile assets than a naive linear scaling would suggest.

Run this and you'll see `CRNB` (bonds) sit around **5.7%** annual volatility while `CRNI` (international equity) sits around **19.2%** — over three times as risky, by this measure, as the bond fund.

## 5. Covariance — the piece that makes portfolios interesting

Here's the idea this whole week is built on. **Covariance** measures how two assets' returns move *together*:

$$\text{Cov}(r_i, r_j) = \frac{1}{n-1}\sum_{t=1}^n (r_{i,t} - \bar r_i)(r_{j,t} - \bar r_j)$$

- **Positive covariance**: when asset $i$ is above its average, asset $j$ tends to be above its average too (they move together).
- **Negative covariance**: when asset $i$ is above its average, asset $j$ tends to be below its average (they move oppositely).
- **Zero (or near-zero) covariance**: no consistent relationship.

Notice the formula: it's the *same* deviation-product idea as variance, except now for two *different* assets — and in fact, `Cov(r_i, r_i) = Var(r_i)`. Variance is just a special case of covariance where an asset is compared with itself. This is why the full picture of risk for a multi-asset universe is one single object: the **covariance matrix**.

```python
cov_monthly = returns.cov()          # a 7x7 DataFrame; diagonal = variances
cov_annual = cov_monthly * 12        # covariance scales linearly with time (not sqrt!)
print(cov_annual.round(4))
```

Note the scaling rule is different from volatility: **covariance (like variance) scales linearly with time**, `× 12`, not `× √12`. Only *volatility* (a square root of variance) gets the square-root scaling. This distinction trips people up constantly — keep it straight: variance and covariance annualize by `× 12`; volatility and correlation-derived standard deviations annualize by `× √12`.

## 6. Correlation — covariance, standardized

Covariance's units are hard to interpret on their own (it's in "return-squared" units, scaled by two different assets' volatility levels, so a covariance of `0.003` between two low-vol bond funds might represent a *stronger* relationship than a covariance of `0.02` between two high-vol equity funds). **Correlation** fixes this by standardizing covariance to always sit between −1 and +1:

$$\rho_{i,j} = \frac{\text{Cov}(r_i, r_j)}{\sigma_i \, \sigma_j}$$

```python
corr = returns.corr()   # standardized version of .cov(); doesn't need annualizing (it's dimensionless)
print(corr.round(3))
```

Read a few real numbers out of this week's seed data:

- `CRNQ` and `CRNI` (US and international equity) correlate around **+0.77** — high, unsurprising, both are equity risk reacting to a lot of the same global macro news.
- `CRNQ` and `CRNB` (US equity and core bonds) correlate around **−0.28** — *negative*. When equities sell off, bonds in this dataset tend to do relatively better, the classic "flight to safety" pattern.
- `CRNG` (gold) correlates with almost everything at **close to zero** (roughly +0.02 to +0.20 across the other five funds) — gold in this dataset behaves like it's driven by a mostly different set of forces than either stocks or bonds.

Those three numbers alone are the entire intuition behind diversification. Now let's make it precise.

## 7. Why covariance — not just individual risk — determines portfolio risk

Here is the single most important formula of the week. For a portfolio with weights $w_1, w_2, \dots, w_n$ in $n$ assets (weights sum to 1), the portfolio's **variance** is:

$$\sigma_p^2 = \sum_{i=1}^n \sum_{j=1}^n w_i \, w_j \, \text{Cov}(r_i, r_j)$$

In matrix form — the notation you'll use for the rest of the week — with $w$ the column vector of weights and $\Sigma$ the covariance matrix:

$$\sigma_p^2 = w^T \Sigma \, w$$

This is a **quadratic form**: every pair of assets $(i, j)$ — including each asset paired with itself — contributes a term $w_i w_j \text{Cov}(r_i, r_j)$ to the total. Critically, this is **not** the weighted average of the individual variances. Watch what happens with just two assets:

$$\sigma_p^2 = w_1^2 \sigma_1^2 + w_2^2 \sigma_2^2 + 2 w_1 w_2 \, \text{Cov}(r_1, r_2)$$

If the two assets were **perfectly positively correlated** ($\rho = 1$, so $\text{Cov} = \sigma_1 \sigma_2$), this formula collapses to $\sigma_p = w_1 \sigma_1 + w_2 \sigma_2$ — a straight-line weighted average, no benefit from combining them. But the moment correlation drops below 1, that cross term shrinks, and $\sigma_p$ ends up **strictly less** than the weighted average of the two individual volatilities. If correlation is *negative*, the effect is dramatic — the cross term actually subtracts risk.

### A concrete two-asset example, by hand

Take `CRNQ` (annual vol ≈ 17.08%) and `CRNB` (annual vol ≈ 5.67%), correlation ≈ −0.28, at a 60/40 weight split ($w_1=0.6$ equity, $w_2=0.4$ bonds):

```python
import numpy as np

sigma_1, sigma_2 = 0.1708, 0.0567
rho = -0.28
cov_12 = rho * sigma_1 * sigma_2

w1, w2 = 0.6, 0.4
naive_weighted_avg_vol = w1*sigma_1 + w2*sigma_2
portfolio_var = (w1**2)*sigma_1**2 + (w2**2)*sigma_2**2 + 2*w1*w2*cov_12
portfolio_vol = np.sqrt(portfolio_var)

print(f"Naive weighted-average vol (WRONG for a portfolio): {naive_weighted_avg_vol:.4%}")
print(f"Actual portfolio vol (accounts for correlation):    {portfolio_vol:.4%}")
```

Run it and you'll see the actual portfolio volatility (roughly **9.9%**) sits meaningfully *below* the naive weighted-average (roughly **12.5%**) — nearly 2.6 percentage points of "free" risk reduction, earned purely from the fact that these two assets don't move in lockstep. Nobody gave up any expected return to get that reduction; the combination is simply less risky than either building block would suggest on its own. **This is diversification, and now you can compute exactly how much of it you're getting, instead of just believing the slogan.**

## 8. Building the full covariance matrix in SQL + pandas

For a six-asset universe, you need the full $6\times6$ covariance matrix, not just one pair. This is where pandas' `.cov()` earns its keep — but it's worth seeing the SQL path too, because sometimes your returns live in a database and you need summary statistics before you ever touch Python.

**In pandas (the primary tool this week):**

```python
port_tickers = ["CRNQ", "CRNI", "CRNB", "CRNH", "CRNG", "CRNR"]
returns_port = returns[port_tickers]

cov_matrix = returns_port.cov() * 12       # annualized covariance matrix
corr_matrix = returns_port.corr()          # correlation is already "annualization-free"

print(cov_matrix.round(4))
print(corr_matrix.round(3))
```

**In SQL**, computing a single pairwise covariance requires `AVG()` of a product of deviations — SQL has no `COVAR_SAMP` shortcut you can rely on in SQLite (Postgres does have `COVAR_SAMP`, but SQLite doesn't, so writing it by hand keeps both engines working):

```sql
-- Sample covariance between CRNQ and CRNB monthly returns, computed in pure SQL,
-- using window functions to get lagged prices -> returns, then the covariance formula directly.
WITH px AS (
    SELECT ticker, price_date, adj_close,
           LAG(adj_close) OVER (PARTITION BY ticker ORDER BY price_date) AS prev_close
    FROM asset_prices
    WHERE ticker IN ('CRNQ', 'CRNB')
),
rets AS (
    SELECT ticker, price_date, (adj_close - prev_close) / prev_close AS ret
    FROM px
    WHERE prev_close IS NOT NULL
),
paired AS (
    SELECT a.price_date, a.ret AS ret_q, b.ret AS ret_b
    FROM rets a
    JOIN rets b ON a.price_date = b.price_date
    WHERE a.ticker = 'CRNQ' AND b.ticker = 'CRNB'
),
means AS (
    SELECT AVG(ret_q) AS mean_q, AVG(ret_b) AS mean_b FROM paired
)
SELECT
    SUM((p.ret_q - m.mean_q) * (p.ret_b - m.mean_b)) / (COUNT(*) - 1) AS sample_covariance
FROM paired p CROSS JOIN means m;
```

You'll get a small negative number (monthly covariance) — multiply by 12 to annualize and it should land close to the `CRNQ`/`CRNB` cell of your pandas covariance matrix. **In practice, you'll do this kind of statistic in pandas** — the SQL version exists so you understand what `.cov()` is actually computing under the hood, not because you should build every covariance matrix by hand in SQL every time.

## 9. Check yourself

- Why does portfolio variance require the *full* covariance matrix, not just each asset's individual variance?
- If two assets have correlation exactly +1, what happens to the diversification benefit of combining them?
- Why does volatility scale by `√12` when going from monthly to annual, while covariance scales by `12`?
- In the 60/40 `CRNQ`/`CRNB` example, would the portfolio volatility have been *higher* or *lower* than it was if the correlation between the two assets had been +0.8 instead of −0.28? Why?
- What's the practical weakness of using trailing sample means as your estimate of "expected return" going forward?

Lecture 2 takes this covariance matrix and asks: across every possible combination of weights, which ones are actually *worth holding* — and that's the efficient frontier.

## Further reading

- **Markowitz (1952), "Portfolio Selection"** — the original paper that started modern portfolio theory: <https://www.jstor.org/stable/2975974>
- **pandas — `DataFrame.cov()` / `DataFrame.corr()`:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cov.html>
- **PostgreSQL — window functions (`LAG`)** used above: <https://www.postgresql.org/docs/current/tutorial-window.html>
