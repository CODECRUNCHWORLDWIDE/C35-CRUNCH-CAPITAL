# Lecture 1 — Equity Financing and Dilution

> **Duration:** ~2 hours. **Outcome:** You can price a follow-on equity offering end to end — discount, underwriting fee, whole-share rounding — and compute the exact ownership dilution it creates for every shareholder in a cap table, in SQL, from memory.

## 1. What "raising equity" actually means

When a company raises equity, it isn't borrowing money — it's **selling ownership**. It creates new shares that didn't exist before and sells them to investors for cash. Nobody has to be paid back, there's no maturity date, and there's no interest. That sounds like free money, and new founders sometimes talk about it that way. It isn't free — it costs **permanent ownership** in every dollar the company will ever make, forever, split among however many shares now exist. Every existing shareholder — founders, employees, earlier investors — ends up owning a smaller slice of a company that now has more slices in it. That's **dilution**, and it is the central mechanic of this lecture.

Crunch Machine Co. needs **$20,000,000** to build the Midwest distribution hub. Before the raise, its cap table (from this week's seed data) is six shareholders holding **10,000,000 shares** at a **$20.00** market price — an **implied market cap of $200,000,000**:

```sql
SELECT
    shareholder_name,
    shareholder_type,
    shares_held,
    ROUND(100.0 * shares_held / (SELECT shares_outstanding FROM company_snapshot), 3) AS ownership_pct
FROM shareholders
ORDER BY shares_held DESC;
```

| shareholder_name | shareholder_type | shares_held | ownership_pct |
|---|---|---:|---:|
| Founder & CEO | Founder | 3,000,000 | 30.000% |
| Crunch Ventures Fund II | VC | 1,800,000 | 18.000% |
| Founder & CTO | Founder | 1,500,000 | 15.000% |
| Public Float (existing) | Public-Float | 1,500,000 | 15.000% |
| Meridian Growth Partners | VC | 1,200,000 | 12.000% |
| Employee Option Pool | Employee-Pool | 1,000,000 | 10.000% |

That's the "before" picture. This lecture builds the "after."

## 2. Pricing the offering: discount and underwriting fee

A public follow-on offering almost never prices at the exact current market price. Two costs shrink what the company actually nets from every share it sells:

1. **The new-issue discount.** Investors buying a large new block of stock want a reason to buy *now* instead of just buying on the open market — so the offering is priced below the current trading price, typically 3–7%. Crunch Machine Co.'s deal is priced at a **5% discount** to the $20.00 market price: an **offer price of $19.00**.
2. **The underwriting spread.** Investment banks that structure and sell the offering take a fee — typically 3–7% of gross proceeds for a follow-on — for finding buyers, taking on placement risk, and putting their name behind the deal. This week's terms carry a **5% underwriting spread**.

The company needs **$20,000,000 net** — the underwriting fee comes *out of* the gross raise, so the company has to raise more than $20,000,000 gross to net that much:

```
Gross proceeds = Net proceeds needed / (1 − fee_pct)
               = 20,000,000 / (1 − 0.05)
               = 20,000,000 / 0.95
               = 21,052,631.58
```

That gross amount, divided by the $19.00 offer price, gives the number of shares needed:

```
Shares needed = Gross proceeds / Offer price
              = 21,052,631.58 / 19.00
              = 1,108,033.24
```

## 3. You can't sell a fraction of a share

`1,108,033.24` shares is not a real number of shares — offerings sell whole shares only. The underwriter **rounds up** to the next whole share, so the company is guaranteed to net *at least* its target, never less:

```
Shares issued = CEIL(1,108,033.24) = 1,108,034
```

That one extra fraction of a share rounds the deal to slightly more than the $20,000,000 target:

```sql
SELECT
    ROUND(20000000.0 / (1 - 0.05) / 19.00, 6)      AS shares_exact,
    CEIL(20000000.0 / (1 - 0.05) / 19.00)           AS shares_issued,
    CEIL(20000000.0 / (1 - 0.05) / 19.00) * 19.00   AS actual_gross,
    CEIL(20000000.0 / (1 - 0.05) / 19.00) * 19.00 * 0.05 AS actual_fee,
    CEIL(20000000.0 / (1 - 0.05) / 19.00) * 19.00 * 0.95 AS actual_net;
```

Run that and you should land on **1,108,034 shares**, **\$21,052,646.00** gross, **\$1,052,632.30** in fees, and **\$20,000,013.70** net — thirteen dollars and change over target, purely from rounding up to a whole share. On a $20 million deal that's immaterial; it's worth seeing once so "shares must be whole numbers" stops being an afterthought and becomes something you check for, in every issuance you ever model.

`CEIL()` (ceiling — round up to the next integer) exists in both PostgreSQL and SQLite; SQLite's version requires a build with the math functions extension enabled, which most modern builds include. If yours doesn't, compute it in Python instead: `math.ceil(shares_exact)`.

## 4. The new share count, and where the dilution comes from

Before the raise there were 10,000,000 shares. After, there are:

```
Shares outstanding (post) = Shares outstanding (pre) + Shares issued
                           = 10,000,000 + 1,108,034
                           = 11,108,034
```

Every existing shareholder's **share count doesn't change** — the Founder & CEO still holds exactly 3,000,000 shares after the raise, same as before. What changes is the **denominator**: those same 3,000,000 shares are now a slice of 11,108,034 total shares instead of 10,000,000. That's the entire mechanism of dilution in one sentence: **your numerator is fixed, the denominator grows, so your percentage shrinks** — even though you didn't sell or lose a single share.

```sql
SELECT
    s.shareholder_name,
    s.shares_held,
    ROUND(100.0 * s.shares_held / 10000000, 3)                                 AS pct_before,
    ROUND(100.0 * s.shares_held / (10000000 + 1108034), 3)                     AS pct_after,
    ROUND(100.0 * s.shares_held / 10000000
        - 100.0 * s.shares_held / (10000000 + 1108034), 3)                     AS pct_point_dilution
FROM shareholders s
ORDER BY s.shares_held DESC;
```

| shareholder_name | shares_held | pct_before | pct_after | pct_point_dilution |
|---|---:|---:|---:|---:|
| Founder & CEO | 3,000,000 | 30.000% | 27.007% | 2.993 |
| Crunch Ventures Fund II | 1,800,000 | 18.000% | 16.204% | 1.796 |
| Founder & CTO | 1,500,000 | 15.000% | 13.504% | 1.496 |
| Public Float (existing) | 1,500,000 | 15.000% | 13.504% | 1.496 |
| Meridian Growth Partners | 1,200,000 | 12.000% | 10.803% | 1.197 |
| Employee Option Pool | 1,000,000 | 10.000% | 9.002% | 0.998 |
| **New Public Investors** | 1,108,034 | 0.000% | **9.975%** | — |

Two things worth sitting with:

1. **Dilution is proportional, not equal in dollar terms.** Every existing holder loses roughly the same *fraction* of their stake (about 9.975% of what they had, since `1 − 10,000,000/11,108,034 ≈ 9.975%`) — but the founder loses nearly 3 full percentage points while the employee pool loses about 1. Bigger stakes feel bigger dilution in absolute points; everyone loses the same *relative* share.
2. **The new investors' stake (9.975%) isn't arbitrary** — it equals exactly `shares_issued / shares_outstanding_after`, which is also, not coincidentally, very close to the discount-and-fee-adjusted share of the company's value the company just sold. Section 6 makes that connection explicit.

## 5. Does dilution mean existing shareholders lose money?

Not automatically — and this is the detail that separates "dilution is scary" from "dilution is a tradeoff." Before the raise, each existing shareholder owned a slice of a $200,000,000 company. After the raise, that same absolute share count owns a smaller *percentage* of a company that is now worth **more** — because it just received $20,000,000 in cash it didn't have before, cash it's presumably deploying into a distribution hub that (if the project has positive NPV, per Week 3) creates value beyond the cash raised.

**Dollar value before, for the Founder & CEO:**

```
3,000,000 shares × $20.00/share = $60,000,000
```

**Dollar value after** (assuming, simply, that the market values the new post-raise company at its old market cap plus the cash raised — $200,000,000 + $20,000,000 = $220,000,000 — a simplification that ignores any value the hub project itself creates or destroys, and ignores any market reaction to the announcement):

```
Post-raise share price ≈ $220,000,000 / 11,108,034 shares ≈ $19.805/share
3,000,000 shares × $19.805/share ≈ $59,414,760
```

The Founder & CEO's *percentage* ownership dropped from 30.000% to 27.007% — real dilution — but the *dollar* value dropped only slightly (from $60.0M to about $59.4M), because the company is now worth more in absolute terms. **If the hub project itself creates value beyond its $20M cost (a positive NPV, in Week 3's language), the founder's dollar value can end up *higher* after dilution than before** — proportionally smaller slice of a sufficiently bigger pie. This is why dilution alone is never the whole story; dilution *combined with what the money buys* is the story. A raise that dilutes you 10% to fund a project with negative NPV is a bad trade. The same 10% dilution to fund a strongly positive-NPV project can leave you richer, not poorer, in dollar terms.

## 6. Why the new investors' stake matches the discount

Look again at the new investors' 9.975% stake. They put in $20,000,013.70 of *net* cash into a company that — by the simplifying assumption above — is worth $220,000,000 after the raise. `20,000,013.70 / 220,000,000 ≈ 9.09%`, noticeably *less* than the 9.975% ownership stake they actually received. The gap is the price of the **discount**: because the offer price ($19.00) sits below the pre-raise market price ($20.00), new investors get *more shares per dollar* than the pre-raise price would have given them — which is exactly the incentive the discount exists to create, and exactly the cost existing shareholders bear for it. A shallower discount means less dilution for existing holders but a harder sell to new investors (who have less incentive to buy now versus waiting); a deeper discount sells more easily but costs existing holders more ownership per dollar raised. This tension — how much discount to offer — is a real negotiation between the company and its underwriters on every follow-on deal.

## 7. Doing this in Python

The same pipeline, as functions you'll extend in Exercise 1 and the mini-project:

```python
import math

def price_offering(net_target: float, offer_price: float, fee_pct: float) -> dict:
    """Given a net-proceeds target, offer price, and fee %, return the whole-share deal terms."""
    gross_needed = net_target / (1 - fee_pct)
    shares_exact = gross_needed / offer_price
    shares_issued = math.ceil(shares_exact)
    actual_gross = shares_issued * offer_price
    actual_fee = actual_gross * fee_pct
    actual_net = actual_gross - actual_fee
    return {
        "shares_issued": shares_issued,
        "gross_proceeds": actual_gross,
        "fee_paid": actual_fee,
        "net_proceeds": actual_net,
    }

deal = price_offering(net_target=20_000_000, offer_price=19.00, fee_pct=0.05)
print(deal)
# {'shares_issued': 1108034, 'gross_proceeds': 21052646.0,
#  'fee_paid': 1052632.3, 'net_proceeds': 20000013.7}

def dilution_table(shares_held: dict[str, int], shares_issued: int) -> dict[str, dict]:
    """Return pre/post ownership % for every existing holder, plus the new investors' stake."""
    shares_before = sum(shares_held.values())
    shares_after = shares_before + shares_issued
    result = {
        name: {
            "pct_before": 100 * held / shares_before,
            "pct_after": 100 * held / shares_after,
        }
        for name, held in shares_held.items()
    }
    result["New Public Investors"] = {
        "pct_before": 0.0,
        "pct_after": 100 * shares_issued / shares_after,
    }
    return result
```

## 8. Common mistakes

- **Discounting the fee off the offer price instead of gross proceeds.** The underwriting spread applies to the *gross dollars raised*, not to each share's price individually — get the order of operations (discount the price, *then* apply the fee to the resulting gross total) backwards and every downstream number is wrong.
- **Forgetting whole-share rounding.** A share count with a decimal point is a bug, not a rare edge case — it happens on essentially every real offering, because the exact math almost never lands on an integer.
- **Confusing percentage dilution with dollar-value loss.** As Section 5 showed, they move independently. A shareholder can be diluted in percentage terms and still be *richer* in dollar terms if the raise funds something valuable.
- **Assuming dilution is split evenly across shareholders.** It isn't — it's proportional to each holder's existing stake, which is exactly why founders with large stakes feel dilution in bigger percentage-point terms than a small individual holder, even though everyone loses the same relative fraction.

## 9. Check yourself

- Walk through, in order, the three numbers you need before you can compute a share count for an offering: net target, discount, fee. What does each one do to the final share count if it goes up?
- Why must a share count always be rounded up, never down or to the nearest whole number?
- A shareholder's percentage ownership drops after a raise. Under what condition does their *dollar* value still go up?
- In your own words, why do new investors in a discounted offering end up with a slightly larger ownership percentage than their net dollars invested, as a fraction of the post-raise company, would naively suggest?

Lecture 2 does the same full treatment for the other side of the board's decision: the $20,000,000 term loan, its coupon and amortization schedule, and the covenants a lender attaches to protect its money.

## Further reading

- **Investopedia — "Follow-On Public Offer (FPO)":** <https://www.investopedia.com/terms/f/followonoffering.asp>
- **Investopedia — "Dilution":** <https://www.investopedia.com/terms/d/dilution.asp>
- **SEC — "Registered Offerings" (how a follow-on offering is actually filed):** <https://www.sec.gov/education/capitalraising/building-blocks/registered-offerings>
- **PostgreSQL — Mathematical Functions (`CEIL`, `ROUND`):** <https://www.postgresql.org/docs/current/functions-math.html>
