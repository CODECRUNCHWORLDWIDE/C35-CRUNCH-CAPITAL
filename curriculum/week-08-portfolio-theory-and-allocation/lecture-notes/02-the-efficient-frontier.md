# Lecture 2 — The Efficient Frontier

> **Duration:** ~2 hours. **Outcome:** You can set up and solve a mean-variance optimization in closed form, trace the entire efficient frontier for a six-asset universe, and identify the global-minimum-variance and maximum-Sharpe (tangency) portfolios — all in pandas/numpy, no solver library required.

Lecture 1 gave you the covariance matrix and showed that portfolio risk depends on more than each asset's own volatility. This lecture asks the natural next question: **given a covariance matrix and a set of expected returns, which combinations of weights are actually worth holding?** The answer is a curve called the **efficient frontier**, and — for the "no constraints" version of the problem — you can find every point on it with nothing but linear algebra. No iterative solver, no `scipy.optimize`, just matrix inversion. That's the beautiful part of classical mean-variance optimization, and it's why it's still taught first, even though real portfolio construction adds constraints that break the closed form (Challenge 1 gets you there).

## 1. The optimization problem, stated precisely

For a universe of $n$ assets with expected return vector $\mu$ (an $n \times 1$ vector) and covariance matrix $\Sigma$ (an $n \times n$ matrix), a portfolio is a weight vector $w$ with $\sum_i w_i = 1$ (fully invested — no cash left over, unless cash is itself one of the $n$ assets). The portfolio's expected return and variance are:

$$\mu_p = w^T \mu \qquad \qquad \sigma_p^2 = w^T \Sigma \, w$$

**Mean-variance optimization** asks: for a *target* expected return $\mu_p = r_{\text{target}}$, what weights $w$ minimize $\sigma_p^2$, subject to $w^T \mu = r_{\text{target}}$ and $w^T \mathbf{1} = 1$ (weights sum to 1)? Sweep $r_{\text{target}}$ across a range and you trace a curve of "best possible" portfolios — the **efficient frontier**: for every level of risk, the frontier gives you the highest achievable expected return (equivalently, for every target return, the lowest achievable risk).

**Important, and easy to miss:** this basic version of the problem allows **negative weights** — short positions — because we haven't added a $w_i \geq 0$ constraint. That's mathematically convenient (it's what keeps the problem solvable in closed form) but not always realistic; Challenge 1 adds the no-short constraint back and shows you how much the answer changes.

## 2. Two-fund separation — the key structural fact

Before the formulas, the one idea worth internalizing: **every portfolio on the efficient frontier is a linear combination of just two "special" portfolios.** This is called the **two-fund separation theorem**. Practically, it means once you've found *any two* distinct portfolios on the frontier, every other frontier portfolio is some weighted blend of those two. The two special portfolios this lecture solves for explicitly are the **global minimum-variance (GMV) portfolio** (the single lowest-risk point on the entire frontier) and the **maximum-Sharpe (tangency) portfolio** (the portfolio with the best risk-adjusted return, which you'll define precisely in Section 5). Once you have those two, you effectively have the whole frontier.

## 3. Setting up the inputs

Load the covariance matrix and expected returns exactly as in Lecture 1:

```python
import pandas as pd
import numpy as np
from sqlalchemy import create_engine

engine = create_engine("postgresql://localhost/crunch_capital")
prices_long = pd.read_sql("SELECT * FROM asset_prices ORDER BY price_date, ticker", engine)
prices = prices_long.pivot(index="price_date", columns="ticker", values="adj_close")
prices.index = pd.to_datetime(prices.index)

port_tickers = ["CRNQ", "CRNI", "CRNB", "CRNH", "CRNG", "CRNR"]
returns = prices[port_tickers].pct_change().dropna()

mu = returns.mean().values * 12          # annualized expected return, shape (6,)
Sigma = returns.cov().values * 12        # annualized covariance matrix, shape (6,6)
n = len(port_tickers)
ones = np.ones(n)
```

Run this against the seed and `mu` should come out close to `[0.1380, 0.1378, 0.0468, 0.0999, 0.0806, 0.0872]` (in ticker order `CRNQ, CRNI, CRNB, CRNH, CRNG, CRNR`) — the same numbers Lecture 1 walked through.

## 4. The global minimum-variance portfolio, in closed form

The GMV portfolio solves the simplest version of the problem — minimize $w^T \Sigma w$ subject *only* to $w^T \mathbf{1} = 1$ (no return target, just "give me the lowest possible risk, full stop"). Using Lagrange multipliers (the derivation is standard optimization-theory and worth doing once on paper, but every finance textbook has it — the *result* is what you need to know cold):

$$w_{\text{GMV}} = \frac{\Sigma^{-1}\mathbf{1}}{\mathbf{1}^T \Sigma^{-1}\mathbf{1}}$$

That's it — one matrix inversion, one matrix-vector product, and a scalar normalization so the weights sum to 1.

```python
Sigma_inv = np.linalg.inv(Sigma)

w_gmv = Sigma_inv @ ones / (ones @ Sigma_inv @ ones)

gmv_return = w_gmv @ mu
gmv_vol = np.sqrt(w_gmv @ Sigma @ w_gmv)

for t, w in zip(port_tickers, w_gmv):
    print(f"{t}: {w: .4f}")
print(f"GMV expected return: {gmv_return:.4%}   GMV volatility: {gmv_vol:.4%}")
```

Run it against the seed and you'll land close to:

| Ticker | GMV weight |
|--------|-----------:|
| CRNQ | 20.9% |
| CRNI | −0.5% |
| CRNB | 94.2% |
| CRNH | −26.8% |
| CRNG | 9.0% |
| CRNR | 3.2% |

**Expected return ≈ 5.55%, volatility ≈ 3.98%.** Two things worth staring at: first, the GMV portfolio is overwhelmingly weighted toward the *lowest-volatility* asset (`CRNB`, core bonds, at 94.2%) — that's expected, minimizing variance naturally gravitates toward low-variance building blocks. Second, notice `CRNH` (high-yield bonds) gets a **negative** weight (−26.8%) — the unconstrained optimizer wants to *short* it, using the proceeds to buy more of the low-vol/negatively-correlated assets, because `CRNH` is positively correlated with several other holdings and contributes more risk than return-per-risk to the mix. A real fund can't always take that short position cheaply (or at all); this is exactly the tension Challenge 1 makes you confront.

## 5. Sharpe ratio and the maximum-Sharpe (tangency) portfolio

The **Sharpe ratio** of a portfolio measures excess return per unit of risk:

$$\text{Sharpe} = \frac{\mu_p - r_f}{\sigma_p}$$

where $r_f$ is the risk-free rate. It answers "how much extra return am I getting for each unit of volatility I'm taking on, above what a risk-free asset would pay?" Higher is better, and it's the standard way to compare portfolios (or funds, or strategies) that differ in *both* return and risk — a portfolio returning 15% at 25% volatility isn't automatically better than one returning 9% at 8% volatility; their Sharpe ratios tell you which is actually more efficient.

The **maximum-Sharpe portfolio**, also called the **tangency portfolio**, is the single portfolio on the efficient frontier with the highest possible Sharpe ratio. Geometrically (you'll build this exact picture in Exercise 3), if you plot every portfolio's volatility on the x-axis and expected return on the y-axis, and draw a straight line from the risk-free rate on the y-axis out to *every* frontier portfolio, the tangency portfolio is the one where that line is **steepest** — literally the line that's tangent to the frontier curve, hence the name.

There's a closed-form solution here too, using **excess returns** ($\mu - r_f \mathbf{1}$) in place of raw returns:

$$w_{\text{tan}} = \frac{\Sigma^{-1}(\mu - r_f \mathbf{1})}{\mathbf{1}^T \Sigma^{-1}(\mu - r_f \mathbf{1})}$$

```python
rf = 0.03   # from capital_market_assumptions

excess = mu - rf
w_tan_raw = Sigma_inv @ excess
w_tan = w_tan_raw / (ones @ w_tan_raw)     # normalize so weights sum to 1

tan_return = w_tan @ mu
tan_vol = np.sqrt(w_tan @ Sigma @ w_tan)
tan_sharpe = (tan_return - rf) / tan_vol

for t, w in zip(port_tickers, w_tan):
    print(f"{t}: {w: .4f}")
print(f"Tangency return: {tan_return:.4%}   vol: {tan_vol:.4%}   Sharpe: {tan_sharpe:.4f}")
```

Against the seed, you should land close to:

| Ticker | Tangency weight |
|--------|-----------------:|
| CRNQ | 23.5% |
| CRNI | −0.6% |
| CRNB | 45.1% |
| CRNH | 8.2% |
| CRNG | 20.1% |
| CRNR | 3.7% |

**Expected return ≈ 8.04%, volatility ≈ 5.59%, Sharpe ≈ 0.90.** Compare this to a naive **equal-weight** portfolio (1/6 in each fund): expected return ≈ 9.84%, volatility ≈ 8.58%, Sharpe ≈ **0.80**. The tangency portfolio has a *lower* expected return than equal-weight, but a meaningfully *better* Sharpe ratio — it's giving up some raw return in exchange for a disproportionately larger reduction in risk. Whether that trade is "better" depends entirely on what the investor actually wants (Section 7 comes back to this).

## 6. Tracing the whole frontier

With the GMV and tangency portfolios as your two reference points (two-fund separation, Section 2), you can trace the *entire* efficient frontier by solving the constrained problem for a grid of target returns. The closed-form solution (from the same Lagrange-multiplier derivation, now with two constraints — sum-to-one *and* hit a target return) uses four scalar building blocks:

$$A = \mathbf{1}^T \Sigma^{-1}\mathbf{1} \qquad B = \mathbf{1}^T \Sigma^{-1}\mu \qquad C = \mu^T \Sigma^{-1}\mu \qquad D = AC - B^2$$

and then, for a target return $r$:

$$w(r) = \lambda(r)\, \Sigma^{-1}\mathbf{1} + \gamma(r)\, \Sigma^{-1}\mu \qquad \text{where} \quad \lambda(r) = \frac{C - Br}{D}, \quad \gamma(r) = \frac{Ar - B}{D}$$

```python
A = ones @ Sigma_inv @ ones
B = ones @ Sigma_inv @ mu
C = mu @ Sigma_inv @ mu
D = A * C - B**2

def frontier_portfolio(target_return):
    lam = (C - B * target_return) / D
    gam = (A * target_return - B) / D
    w = lam * (Sigma_inv @ ones) + gam * (Sigma_inv @ mu)
    vol = np.sqrt(w @ Sigma @ w)
    return w, vol

targets = np.arange(0.03, 0.145, 0.01)
frontier = [(t, frontier_portfolio(t)[1]) for t in targets]
for t, v in frontier:
    print(f"target return {t:.2%} -> volatility {v:.4%}")
```

Run it and you'll see the frontier trace out a **hyperbola** shape: volatility starts around 5.7% near the low-return end, dips to a minimum around **4.0%** near a 5–6% target return (that's the GMV portfolio you already found — the frontier's vertex), and then rises again as the target return climbs past that point. That vertex is *exactly* the GMV portfolio: it's the single point on the frontier where volatility is minimized, full stop — every other point trades additional risk for additional expected return.

**One critical fact about this shape**: the frontier is really the *right branch* of that hyperbola. The left branch (below the GMV point) is mathematically valid but never worth holding — for any given volatility, there's always a higher-return portfolio available on the right branch at that *same* volatility level. Portfolios on the left branch are called "inefficient" for exactly this reason, and it's why the curve is called the **efficient** frontier — only the upper-left-to-upper-right sweep past the GMV vertex actually belongs on it.

## 7. Where risk tolerance comes back in

Notice what this whole lecture has *not* done: it hasn't told you which single portfolio to hold. The frontier gives you every efficient option; the tangency portfolio gives you the single best risk-adjusted option *if you're allowed to also hold cash or borrow at the risk-free rate* (more on that combination — the **capital market line** — in Lecture 3). But an investor's actual choice among frontier points depends on their own risk tolerance: a young investor with a long horizon might rationally choose a *higher*-volatility point further out on the frontier than the tangency portfolio, borrowing conceptually against future income; a retiree drawing down savings might choose something closer to the GMV end. **Mean-variance optimization doesn't answer "how much risk should I take" — it only answers "given a chosen level of risk (or return), what's the best mix to hold."** Keep those two questions separate; conflating them is a common and consequential mistake.

## 8. Check yourself

- What are the two constraints in the basic mean-variance optimization problem, and what happens to the closed-form solution if you also require $w_i \geq 0$ for every asset?
- In your own words, what does "two-fund separation" mean, and why does it make tracing the whole frontier easier?
- Why does the GMV portfolio sit at the *vertex* of the frontier hyperbola rather than somewhere along one of its branches?
- The tangency portfolio in this week's seed has a lower expected return but a higher Sharpe ratio than equal-weight. Explain, in one or two sentences, why "higher Sharpe" doesn't automatically mean "the portfolio everyone should hold."
- Why is the left branch of the mean-variance hyperbola never actually on the *efficient* frontier?

Lecture 3 takes the market-benchmark ticker (`MKT`) you've been ignoring all week, uses it to estimate each fund's beta, and connects this frontier machinery to CAPM — including what happens once you're allowed to mix a risky portfolio with a genuinely risk-free asset.

## Further reading

- **Investopedia — Efficient Frontier:** <https://www.investopedia.com/terms/e/efficientfrontier.asp>
- **numpy — `linalg.inv`, matrix/vector products:** <https://numpy.org/doc/stable/reference/generated/numpy.linalg.inv.html>
- **CFA Institute — Portfolio Risk and Return (Level I curriculum overview):** <https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/portfolio-risk-and-return-part-i>
