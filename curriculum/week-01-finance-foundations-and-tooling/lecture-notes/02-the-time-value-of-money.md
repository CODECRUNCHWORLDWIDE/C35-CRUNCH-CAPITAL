# Lecture 2 — The Time Value of Money

> **Duration:** ~2 hours. **Outcome:** You can state and derive the present value and future value formulas, compute either one for a single cash flow or a stream, reason correctly about compounding frequency, and explain precisely why a dollar today beats a dollar tomorrow — not as a slogan, but as arithmetic you can defend.

Lecture 1 established that a dollar today is worth more than a dollar in the future. This lecture turns that into a formula you can compute, by hand, in SQL, and in Python — and shows you exactly what "worth more" means in dollars and cents.

## 1. Future value — growing money forward through time

If you invest $1,000 today at an annual rate of 8%, in one year you have:

```
FV = 1000 × (1 + 0.08) = 1080
```

You earned $80 of interest. If you leave the full $1,080 invested for a second year at the same 8%, you don't earn another flat $80 — you earn 8% of $1,080:

```
FV = 1080 × (1 + 0.08) = 1166.40
```

That extra $6.40 over "two flat $80 payments" ($1,160) is **compounding** — interest earning interest. Generalize to `n` years:

```
FV = PV × (1 + r)^n
```

Where:

- `PV` = present value — the amount you have today.
- `r` = the periodic interest (or growth, or discount) rate, as a decimal (8% = 0.08).
- `n` = the number of periods.
- `FV` = future value — what that amount grows to after `n` periods at rate `r`.

Check it: `1000 × (1.08)^2 = 1000 × 1.1664 = 1166.40`. Matches the year-by-year calculation above.

### Why this matters for a business

Future value answers: *"If I have this money now and I don't need it, and I can earn `r`, what will it become?"* Companies use this to project the future value of retained cash, to compare "take the cash now" vs. "reinvest it," and — critically — to reason about the *flip side*, which is what most of corporate finance actually needs: given a future amount, what is it worth **today**?

## 2. Present value — discounting money backward through time

Flip the future-value formula around. If you know a cash flow arrives in the future and you want to know what it's worth *today*, solve for `PV`:

```
FV = PV × (1 + r)^n
⟹  PV = FV / (1 + r)^n
```

This is **discounting**, and it is the single most-used formula in this entire course. Read it in words: *the present value of a future cash flow is that cash flow, shrunk by a factor that grows with both time and the discount rate.*

**Worked example.** The New CNC Machine project (from this week's seed data) pays $60,000 at the end of year 4 (period 4), and the company's hurdle rate for this project is 7%. What is that $60,000 worth *today*?

```
PV = 60000 / (1 + 0.07)^4
   = 60000 / 1.31079601
   = 45,777.31   (rounded)
```

That $60,000 four years from now is worth about **$45,777 today**. The gap — nearly $14,223 — is the price of waiting four years, at a 7% required return.

### The discount factor

`1 / (1 + r)^n` is called the **discount factor** — a number between 0 and 1 that you multiply any future cash flow by to bring it to today's value. It's worth building intuition for how it moves:

| Rate `r` | Period `n` | Discount factor `1/(1+r)^n` | $60,000 today's worth |
|---------:|-----------:|-----------------------------:|-----------------------:|
| 7% | 1 | 0.9346 | $56,075 |
| 7% | 4 | 0.7629 | $45,777 |
| 7% | 10 | 0.5083 | $30,498 |
| 18% | 4 | 0.5158 | $30,948 |
| 18% | 10 | 0.1911 | $11,467 |

Two things to notice, and both are load-bearing for the rest of this course:

1. **Discounting compounds.** Doubling `n` from 4 to 8 years does *not* halve the discount factor a second time linearly — it multiplies the shrinkage. Time and rate interact exponentially, not additively.
2. **Rate matters more than most people's intuition expects.** At period 4, moving the rate from 7% to 18% cuts the present value nearly in half ($45,777 → $30,948) for the *exact same future cash flow*. This is why getting the discount rate right (Week 4, cost of capital) is not a rounding-error detail — it can flip a decision from "do it" to "don't."

## 3. Discounting a stream of cash flows

Real projects aren't one cash flow — they're a stream, one per period. The present value of a stream is just the **sum of each cash flow's individual present value**:

```
PV(stream) = Σ CFₜ / (1 + r)^t     for t = 0, 1, 2, ..., n
```

Note `t = 0` is *today* — and `(1 + r)^0 = 1`, so a period-0 cash flow is never discounted; it's already in today's dollars. This matters because most capital projects start with a negative cash flow at period 0 (the initial investment), and that number needs **no discounting at all**.

**Worked example** — the full New CNC Machine stream at its 7% hurdle rate:

| Period | Cash flow | Discount factor | PV of this flow |
|-------:|----------:|-----------------:|-----------------:|
| 0 | −180,000 | 1.0000 | −180,000.00 |
| 1 | 55,000 | 0.9346 | 51,401.87 |
| 2 | 58,000 | 0.8734 | 50,660.05 |
| 3 | 60,000 | 0.8163 | 48,977.85 |
| 4 | 60,000 | 0.7629 | 45,777.44 |
| 5 | 62,000 + 15,000 = 77,000 | 0.7130 | 54,904.28 |
| **Sum** | | | **≈ 71,721.49** |

That sum — roughly **+$71,721** — is the project's **net present value (NPV)**: the value it creates today, in today's dollars, once every future cash flow has been discounted back and the upfront cost has been netted out. A positive NPV means the project is worth more than it costs, at the given discount rate. You'll formalize NPV as a decision rule in Week 3 — this week, focus on the mechanics of getting each term right; NPV is nothing more than "discount every cash flow, then add them up."

## 4. Compounding frequency — annual isn't the only option

So far every example compounded **once a year**. Many real instruments compound more often — monthly loan payments, daily interest on a savings account, continuously in some theoretical models. The formula generalizes:

```
FV = PV × (1 + r/m)^(m×n)
```

Where `m` is the number of compounding periods per year (12 for monthly, 4 for quarterly, 365 for daily) and `n` is still the number of *years*. The rate `r` here is the **nominal annual rate**; each period you actually earn `r/m`.

**Why this matters:** a nominal 12% annual rate compounded **monthly** is *not* the same as 12% compounded **annually**. Monthly compounding means you earn 1% every month, and that 1% starts earning its own interest sooner:

```
Annual:   FV = 1000 × (1 + 0.12)^1        = 1120.00
Monthly:  FV = 1000 × (1 + 0.12/12)^12    = 1126.83
```

The extra $6.83 is the effect of compounding more frequently. This gap between the **nominal rate** (12%) and the **effective annual rate** (here, 12.68%) is exactly what banks are required to disclose as APR vs. APY — and it's exactly the mechanic behind Challenge 1's amortization schedule this week, and the bond and loan pricing you'll do in later weeks.

**Effective annual rate (EAR)** formula, so you can always convert a nominal rate at any compounding frequency to a single comparable annual number:

```
EAR = (1 + r/m)^m − 1
```

## 5. Doing this in SQL

This week's whole point is that this math belongs in a query, not a cell reference. PostgreSQL and SQLite both support the `POWER()` function (and the `^` operator in Postgres), so the discount-factor formula translates directly:

```sql
-- Present value of a single future cash flow
SELECT 60000 / POWER(1 + 0.07, 4) AS present_value;
-- ≈ 45777.31

-- Present value of the entire New CNC Machine stream, using the seed table
SELECT
    period,
    amount,
    ROUND(amount / POWER(1 + hurdle_rate, period), 2) AS pv_of_flow
FROM cash_flows
WHERE project_id = 1
ORDER BY period;

-- The project's NPV: sum every row's discounted value
SELECT
    project_name,
    ROUND(SUM(amount / POWER(1 + hurdle_rate, period)), 2) AS npv
FROM cash_flows
WHERE project_id = 1
GROUP BY project_name;
```

Run that last query and you should land close to the **+71,721.49** we computed by hand above (small differences are just rounding at each step — the query carries full precision throughout). Notice what this buys you that a spreadsheet formula copied down a column doesn't: change `hurdle_rate` once, in one row, and every downstream number recalculates correctly, with the calculation logic sitting in one auditable place instead of copied across dozens of cells that could each silently diverge.

## 6. Doing this in Python

The same formula, as a reusable function — the shape you'll build out fully in Exercise 3 and the mini-project:

```python
def present_value(cash_flow: float, rate: float, period: int) -> float:
    """PV of a single future cash flow, discounted at `rate` for `period` periods."""
    return cash_flow / (1 + rate) ** period

def future_value(present: float, rate: float, period: int) -> float:
    """FV of a present amount, compounded at `rate` for `period` periods."""
    return present * (1 + rate) ** period

# The same $60,000-in-4-years example from section 2:
pv = present_value(60000, 0.07, 4)
print(round(pv, 2))   # 45777.31
```

A function, unlike a spreadsheet cell, has a **name**, can be **tested**, and can be **imported** into every later model in this course without retyping the formula. This is the seed of the discounting engine you build in the mini-project.

## 7. Common mistakes

- **Discounting the period-0 cash flow.** It's already today's money — `(1+r)^0 = 1` — dividing it by anything other than 1 is a bug.
- **Mixing nominal and effective rates.** If a rate is quoted as "12% compounded monthly," don't plug 0.12 straight into the annual formula — either convert to EAR first, or use the `r/m`, `m×n` version.
- **Forgetting that `amount` is signed.** In this week's table, capex and opex are negative, revenue and salvage are positive. Sum them *with* their sign to get NPV; summing absolute values gives you a meaningless number.
- **Confusing "biggest total cash flow" with "best project."** As Lecture 1's worked example showed, the project with the largest raw dollar total is not automatically the one with the highest NPV — discounting and timing can reorder the ranking entirely. You'll prove this to yourself in the exercises.

## 8. Check yourself

- Write the FV formula and the PV formula from memory. Which one is the other, rearranged?
- Why is a period-0 cash flow never discounted?
- Two identical $60,000 cash flows arrive in year 4 — one discounted at 7%, one at 18%. Which present value is bigger, and by roughly how much (order of magnitude, not exact cents)?
- What's the difference between a nominal annual rate and an effective annual rate, and when do they diverge most?
- In your own words, what does "NPV" mean, mechanically — not as a rule, just as an arithmetic operation?

Lecture 3 sets up the actual PostgreSQL + pandas workspace you'll run every formula above in for the rest of the course, and makes the case for why that workspace — not a spreadsheet — is where this modeling belongs.

## Further reading

- **Investopedia — "Present Value":** <https://www.investopedia.com/terms/p/presentvalue.asp>
- **Investopedia — "Future Value":** <https://www.investopedia.com/terms/f/futurevalue.asp>
- **Investopedia — "Effective Annual Interest Rate":** <https://www.investopedia.com/terms/e/effectiveinterest.asp>
- **PostgreSQL — Mathematical Functions (`POWER`, `^`):** <https://www.postgresql.org/docs/current/functions-math.html>
