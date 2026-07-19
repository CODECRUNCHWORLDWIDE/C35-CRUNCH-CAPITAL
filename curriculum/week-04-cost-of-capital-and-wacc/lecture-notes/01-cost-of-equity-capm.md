# Lecture 1 — Cost of Equity: CAPM

> **Duration:** ~2 hours. **Outcome:** You can state the CAPM formula from memory, explain what each of its three inputs actually measures, source reasonable values for a real company, and compute a cost of equity by hand, in SQL, and in Python.

## 1. The question CAPM answers

Every dollar a company raises has a price. Debt's price is obvious — it's the interest rate on the loan. Equity's price is less obvious, because equity investors don't send you an invoice. When a shareholder buys stock in Crunch Machine Co., they aren't charging an explicit rate — they're accepting risk in exchange for an *expected* return, and if the stock doesn't deliver something close to that expected return over time, they sell it and put their money somewhere else. That walking-away expected return **is** the cost of equity: the minimum return a company must appear likely to deliver to keep equity investors holding the stock instead of selling it.

The **Capital Asset Pricing Model (CAPM)** is the standard tool for estimating that number. It says a stock's required return is built from three pieces:

$$
R_e = R_f + \beta \times (R_m - R_f)
$$

- $R_e$ — the cost of equity (what we're solving for).
- $R_f$ — the **risk-free rate**: the return on an investment with (as close as practically possible to) zero default risk.
- $\beta$ — **beta**: how sensitive this particular stock's returns are to the overall market's returns.
- $(R_m - R_f)$ — the **equity risk premium (ERP)**: how much more return investors demand, on average, for holding risky stocks instead of the risk-free asset. Often written $ERP$ directly rather than as $R_m - R_f$.

So CAPM says: *start with what you'd earn risk-free, then add a risk premium scaled by how much market risk this specific stock carries.* A stock that moves exactly with the market ($\beta = 1$) earns the market's risk premium exactly. A stock that amplifies market moves ($\beta > 1$) earns more; a stock that dampens them ($\beta < 1$) earns less; and — in the theoretical limit — a stock with $\beta = 0$ earns exactly the risk-free rate, no risk premium at all, because CAPM claims the market doesn't reward risk it can diversify away (more on this below).

## 2. Where each input comes from

### 2.1 The risk-free rate ($R_f$)

In practice there's no truly risk-free asset, but government bonds of very safe, currency-issuing sovereigns are the standard stand-in. For US-dollar-denominated analysis, the convention is the **10-year US Treasury yield** — long enough to roughly match the multi-year horizon of most corporate investment decisions, liquid enough to be quoted continuously, and safe enough that default risk is a rounding error.

For Crunch Machine Co., we'll use $R_f = 4.25\%$, the assumed 10-year Treasury yield as of the valuation date (2025-12-31) sitting in `capital_structure.risk_free_rate`. In real work you'd pull this from a live source (the Federal Reserve's FRED database, `DGS10`, is the standard free one — see [`resources.md`](../resources.md)) rather than hard-coding it, precisely *because* it moves: this is the input in the whole WACC calculation that changes fastest, day to day, as monetary policy and inflation expectations shift.

### 2.2 The equity risk premium ($ERP$)

This is the most debated number in all of corporate finance, because it can't be observed directly — you can't look up "the market's required risk premium" on a ticker. Two common approaches:

- **Historical ERP** — average the actual excess return of stocks over bonds across a long historical window (often 50–100+ years). Simple and objective, but backward-looking, and the answer changes noticeably depending on which decades you include.
- **Implied ERP** — back the premium out of current stock prices and analysts' forward earnings/dividend forecasts, on the theory that today's prices already reflect what investors are demanding right now. More forward-looking, but depends on a forecasting model.

Both approaches, done carefully, tend to land in a **4%–6%** range for mature developed markets in most periods. This course uses a supplied, assumed ERP for each scenario rather than re-deriving it from scratch every week — that's a specialized research exercise in its own right (Aswath Damodaran's freely published implied-ERP series, linked in `resources.md`, is the closest thing the industry has to a standard reference). For Crunch Machine Co. we'll use $ERP = 5.00\%$, sitting in `capital_structure.equity_risk_premium`.

**What moves it:** investor risk appetite. In a fearful market (recession, crisis), investors demand a *larger* premium to hold risky assets — ERP widens. In a calm, confident market, it narrows. This is why the ERP, unlike the risk-free rate, isn't quoted anywhere — it has to be estimated, and reasonable analysts can and do disagree on it by a percentage point or more.

### 2.3 Beta ($\beta$)

Beta measures how much of a stock's risk is **systematic** (market-wide, undiversifiable — recessions, rate moves, broad sentiment) versus **idiosyncratic** (specific to that company — a product recall, a lawsuit, a bad quarter). CAPM's central claim is that a rational, diversified investor holds many stocks at once, so idiosyncratic risk washes out across the portfolio and isn't compensated — only the systematic slice, measured by beta, earns a risk premium.

Formally, beta is the slope of a regression of a stock's returns against the market's returns:

$$
\beta = \frac{\text{Cov}(R_{\text{stock}}, R_{\text{market}})}{\text{Var}(R_{\text{market}})}
$$

You don't have to compute this by hand today — Lecture 2 walks through estimating it yourself in pandas from the `stock_prices` table. For this lecture, treat beta as a **given, vendor-supplied number**, the way most working analysts first encounter it: a market-data provider (Bloomberg, a broker's terminal, even a free site like Yahoo Finance) publishes a beta for nearly every public stock, computed from some window of historical returns using some particular methodology they don't always fully disclose. Crunch Machine Co.'s vendor beta is $1.30$, sitting in `capital_structure.vendor_beta`.

**Reading a beta:**

| Beta | Meaning |
|-----:|---------|
| $\beta = 0$ | No correlation with the market (rare in practice for an operating company) |
| $0 < \beta < 1$ | Moves with the market, but more mildly — think regulated utilities, consumer staples |
| $\beta = 1$ | Moves in lockstep with the market on average |
| $\beta > 1$ | Amplifies market moves — think high-growth tech, cyclical industrials, leveraged companies |
| $\beta < 0$ | Moves *opposite* the market (rare — gold miners sometimes exhibit mild negative beta) |

Crunch Machine Co.'s $\beta = 1.30$ says: when the overall market moves 1%, CRNM stock has historically moved about 1.30% in the same direction, on average. It's a cyclical industrial manufacturer with meaningful operating leverage (Week 1) and now, as we'll see in Lecture 3, real financial leverage too — both of those push beta above 1.

**What moves it:** two things, structurally. **Business risk** — how cyclical the company's revenue is, how much operating leverage (fixed vs. variable costs) it carries — and **financial risk** — how much debt it carries, since debt amplifies the swings that flow through to equity holders. Lecture 2 makes this second relationship precise with the *levered vs. unlevered beta* distinction.

## 3. Computing Crunch Machine Co.'s cost of equity — first pass

With all three inputs in hand, CAPM is one line of arithmetic.

**By hand:**

$$
R_e = 4.25\% + 1.30 \times 5.00\% = 4.25\% + 6.50\% = 10.75\%
$$

**In SQL** — pull the inputs straight from `capital_structure` and compute inline:

```sql
SELECT
    company,
    risk_free_rate,
    vendor_beta,
    equity_risk_premium,
    risk_free_rate + vendor_beta * equity_risk_premium AS cost_of_equity
FROM capital_structure
WHERE company = 'Crunch Machine Co.';
```

```
      company       | risk_free_rate | vendor_beta | equity_risk_premium | cost_of_equity
---------------------+----------------+-------------+----------------------+----------------
 Crunch Machine Co.  |         0.0425 |        1.30 |               0.0500 |         0.1075
```

**In Python:**

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg://localhost/crunch_capital")
cs = pd.read_sql("SELECT * FROM capital_structure WHERE company = 'Crunch Machine Co.'", engine).iloc[0]

cost_of_equity = cs["risk_free_rate"] + cs["vendor_beta"] * cs["equity_risk_premium"]
print(f"Cost of equity: {cost_of_equity:.2%}")   # Cost of equity: 10.75%
```

Read that number the way an investor would: **Crunch Machine Co.'s shareholders require roughly a 10.75% annual return to keep holding the stock, given how risky it is relative to the market.** If the company can't plausibly deliver that — through some combination of dividends and share-price appreciation — rational shareholders eventually sell, the stock price falls until the *expected* return rises back to something near 10.75%, and the company finds it harder and more expensive to raise new equity.

This is a **first pass**, and it's worth being upfront about why: it leans on a vendor's beta, which is a number you can't audit — you don't know its window, its frequency, or its methodology. Lecture 2 fixes that by having you estimate beta yourself, directly from the `stock_prices` data already sitting in your database, so every input in this formula is one you can defend from first principles.

## 4. A second scenario, to build intuition

Numbers land better in contrast. Suppose a much steadier company — a regulated water utility — has $\beta = 0.55$, and everything else about the market environment is unchanged:

$$
R_e = 4.25\% + 0.55 \times 5.00\% = 4.25\% + 2.75\% = 7.00\%
$$

The utility's shareholders require a materially lower return — 7.00% versus Crunch Machine Co.'s 10.75% — purely because the utility's cash flows are steadier and less correlated with the broader economy. Same risk-free rate, same market risk premium, different beta, nearly a 4-point gap in what investors demand. This is the entire point of CAPM: it turns "how risky is this stock, exactly?" into a single number you can plug into a formula, instead of a vague adjective.

## 5. What CAPM does *not* capture

Be honest about the model's limits, because every number this week inherits them:

- **It assumes investors are diversified.** A founder with 90% of their net worth in one stock cares about total risk, not just systematic risk — CAPM's "idiosyncratic risk isn't compensated" claim doesn't apply to them.
- **Beta is estimated from the past.** It's a statistical summary of history, not a guarantee about the future — a company can change its risk profile (a new product line, a leveraged buyout, a merger) faster than its trailing beta updates.
- **The ERP is an estimate, not a fact.** Reasonable, informed analysts land on different numbers, and CAPM's output is only as good as that input.
- **Real markets have frictions CAPM ignores** — taxes, transaction costs, borrowing constraints, and investor behavior that isn't perfectly rational.

None of this makes CAPM useless — it's still the industry-standard starting point precisely because it's simple, transparent, and auditable: every input is a number you can look up or estimate and defend, unlike more exotic multi-factor models that trade transparency for (debatable) precision. It makes CAPM a *first, defensible estimate* that a good analyst then sanity-checks, not a black box you trust blindly.

## 6. Check yourself

- Write the CAPM formula from memory, and name what each symbol stands for.
- Why is the 10-year Treasury yield the conventional choice for $R_f$, rather than, say, a 3-month Treasury bill?
- What does a beta of 1.30 tell you about how a stock has historically moved relative to the market?
- Two companies have identical betas but different ERPs are used to value them (one analyst uses 4%, another uses 6%). Will their cost-of-equity estimates agree? Why might two reasonable analysts choose different ERPs?
- Compute the cost of equity for a stock with $R_f = 3.5\%$, $\beta = 0.80$, $ERP = 5.5\%$.
- Name one concrete limitation of CAPM and explain, in one sentence, why it matters for how much you trust the output.

If those are automatic, Lecture 2 shows you how to stop trusting a vendor's beta and compute your own from the price data already sitting in your database.

## Further reading

- **Federal Reserve Economic Data (FRED) — 10-Year Treasury Constant Maturity Rate:** <https://fred.stlouisfed.org/series/DGS10>
- **Aswath Damodaran — Equity Risk Premiums (updated data and methodology):** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/ctryprem.html>
- **Investopedia — Capital Asset Pricing Model (CAPM):** <https://www.investopedia.com/terms/c/capm.asp>
- **CFA Institute — Cost of Capital overview (candidate-level reading, freely accessible):** <https://www.cfainstitute.org/en/membership/professional-development/refresher-readings/cost-capital>
