# Lecture 2 — SAFEs, Rounds, and Pools

> **Duration:** ~1.5 hours. **Outcome:** You can convert a SAFE using the cap/discount rule, price a priced round from a pre-money valuation, and correctly size an investor-mandated option pool — including seeing exactly whose ownership absorbs the pool's dilution.

Solstice's seed investors don't want to argue about what the company is worth in month three, when there's a product but no revenue. So instead of buying priced shares, they write a check today in exchange for the *right* to shares later, on terms that get worked out the first time somebody else — a "real" investor — sets a price. That instrument is a **SAFE** (Simple Agreement for Future Equity), and understanding exactly how and when it turns into shares is this lecture's first half. The second half is the priced round itself, and the option-pool mechanic almost every priced round bundles into it.

## 1. The SAFE — a check today, shares later

A SAFE is not a loan (no interest, no maturity date, no promise of repayment) and it is not stock (no voting rights, no board seat, nothing until it converts). It's a contract that says: *when this company next raises a priced round, this money converts into shares of that round, at a price no worse than the terms below.* Two terms set that price ceiling and floor:

- **Valuation cap.** The maximum company valuation the SAFE investor's money converts at, no matter how high the priced round actually prices. A $10M cap means: even if Series A prices the company at $50M, this SAFE still converts as if the company were worth $10M — the investor's reward for taking risk early.
- **Discount.** A percentage off the priced round's per-share price, independent of any cap. A 20% discount means the SAFE investor pays 80 cents on every dollar the new Series A investor pays.

A SAFE can have a cap only, a discount only, both, or (rarely) neither. When a SAFE has both, it converts at **whichever price is lower for the investor** — i.e., whichever gives them **more shares** for their money. That "greater of the two deals" rule is the single most important mechanical fact about SAFEs.

**Solstice's seed round** (mid-2023) raises three SAFEs:

```sql
CREATE TABLE safes (
    safe_id         INTEGER PRIMARY KEY,
    investor_name   TEXT    NOT NULL,
    invested_amount NUMERIC NOT NULL,
    valuation_cap   NUMERIC,          -- NULL = uncapped
    discount_pct    NUMERIC,          -- NULL = no discount
    signed_date     DATE    NOT NULL
);

INSERT INTO safes VALUES
(1, 'Basecamp Seed Fund', 500000, 10000000, NULL, '2023-06-01'),
(2, 'Priya Anand',        100000, 10000000, NULL, '2023-06-15'),
(3, 'Tom Fischer',         64000,     NULL, 0.20, '2023-07-01');
```

Basecamp and Priya negotiated a **$10,000,000 cap**, no discount. Tom, writing a smaller angel check slightly later, got a **20% discount, no cap** instead — a different, entirely normal, negotiated shape. None of these SAFEs have converted into shares yet — notice the table has no `shares` column at all. **A SAFE is a dollar amount and a formula, not a share count, until a priced round happens.** That's the whole point of the instrument: it defers the hardest question (what is this company worth?) to a moment when there's more evidence to answer it.

### Locking the cap price

For a capped SAFE, the **cap price per share** is fixed the moment you know the company's fully-diluted share count the cap applies against. We use the simplest, most common teaching convention: the cap price is set against the company's fully diluted share count **at the time the SAFE is signed** — Solstice's founding cap table, 10,000,000 shares, before any pool or later dilution:

```sql
SELECT
    investor_name,
    invested_amount,
    valuation_cap,
    ROUND(valuation_cap / 10000000.0, 6) AS cap_price_per_share
FROM safes
WHERE valuation_cap IS NOT NULL;
```

```
   investor_name    | invested_amount | valuation_cap | cap_price_per_share
---------------------+-----------------+----------------+----------------------
 Basecamp Seed Fund  |          500000 |       10000000 |               1.0000
 Priya Anand         |          100000 |       10000000 |               1.0000
```

$10,000,000 cap ÷ 10,000,000 shares = **$1.00 per share, exactly**. That price is now locked, whatever Series A ends up pricing at.

> ⚠️ **Real-world nuance.** Production cap-table tools compute the cap price against the fully diluted share count *immediately before the priced round* — which usually already includes the new option pool and every other converting SAFE, making the math simultaneous (the pool and the SAFEs each depend on numbers that depend on each other). We use the simpler, non-circular "at signing" convention here because it teaches the core mechanic correctly and is common in beginner-facing cap-table education; `resources.md` links the full NVCA-standard formula for when you're ready for it.

## 2. Pricing the round — pre-money, post-money, price per share

Six months later, Solstice raises a **priced Series A** — the first round where somebody actually names a valuation and everyone gets real shares, not a promise. **Meridian Ventures** offers to lead:

```sql
CREATE TABLE financing_rounds (
    round_id           INTEGER PRIMARY KEY,
    round_name         TEXT    NOT NULL,
    lead_investor      TEXT    NOT NULL,
    pre_money_valuation NUMERIC NOT NULL,
    new_money           NUMERIC NOT NULL,
    option_pool_target_pct NUMERIC,     -- target % of pre-money FD shares, post-topup
    close_date          DATE    NOT NULL
);

INSERT INTO financing_rounds VALUES
(1, 'Series A', 'Meridian Ventures', 20000000, 5000000, 0.20, '2024-02-01');
```

Two definitions do all the work in a priced round:

- **Pre-money valuation** — what the company is worth *before* the new money arrives. Negotiated, not computed.
- **Post-money valuation** = pre-money + new money. $20,000,000 + $5,000,000 = **$25,000,000**.

**Price per share** is pre-money valuation divided by the pre-money **fully diluted** share count — but "fully diluted" at this exact moment includes every converting SAFE *and* the new option pool the investor is about to require. That's the twist the next section untangles.

## 3. The option pool shuffle

Meridian's term sheet includes a standard clause: Solstice must create (or top up) an **option pool** — unallocated shares reserved for future hires — equal to **20% of the fully diluted cap table, measured *after* the pool is created but *before* the new investor's shares are issued.** This sounds like a footnote. It is not. It's the single most consequential clause in most term sheets, and here's why: the pool is created **pre-money**, meaning **only existing shareholders (founders, advisors, converted SAFEs) absorb its dilution — Meridian's percentage is unaffected by it.**

Work the algebra once, carefully, because you'll run it every time a term sheet mentions a pool.

**Step 1 — existing fully diluted shares before any pool top-up**, including the SAFEs that convert at their locked cap price:

```sql
-- Basecamp: $500,000 / $1.00 = 500,000 shares
-- Priya:    $100,000 / $1.00 = 100,000 shares
SELECT SUM(shares) AS existing_fd_shares FROM (
    SELECT shares FROM cap_table_events WHERE round_name = 'Founding'
    UNION ALL
    SELECT invested_amount / (valuation_cap / 10000000.0) AS shares
    FROM safes WHERE valuation_cap IS NOT NULL
) x;
```

Founders (10,000,000) + capped SAFEs (500,000 + 100,000) = **10,600,000 existing fully diluted shares**. (Tom's discount-only SAFE isn't included yet — its price depends on the round price we haven't set, so it's handled *after* pricing, in the next section.)

**Step 2 — solve for the pool top-up.** Let `X` be the new pool shares needed so the pool equals 20% of (existing shares + pool), *before* new investor shares are added:

```
Pool = 0.20 × (10,600,000 + Pool)
Pool = 2,120,000 + 0.20 × Pool
0.80 × Pool = 2,120,000
Pool = 2,650,000
```

```sql
-- Do this in Python or a recursive CTE rather than by hand once real deals get messier:
WITH RECURSIVE solve(pool, iter) AS (
    SELECT 0.0, 0
    UNION ALL
    SELECT 0.20 * (10600000 + pool), iter + 1
    FROM solve WHERE iter < 25   -- converges in a handful of iterations
)
SELECT pool FROM solve ORDER BY iter DESC LIMIT 1;
```

Both the closed-form algebra and the iterative CTE land on **2,650,000 pool shares**. Check it: `2,650,000 / (10,600,000 + 2,650,000) = 2,650,000 / 13,250,000 = 20.00%` exactly.

**Step 3 — price the round** using pre-money valuation over the *now-topped-up* pre-money fully diluted count:

```sql
SELECT ROUND(20000000.0 / 13250000, 6) AS price_per_share;
-- 1.509434
```

**Step 4 — issue Meridian's shares** at that price:

```sql
SELECT ROUND(5000000 / (20000000.0 / 13250000)) AS new_investor_shares;
-- 3,312,500
```

**Step 5 — convert Tom's discount-only SAFE** now that a round price exists. His 20% discount applies to the price Meridian just paid:

```sql
SELECT ROUND(64000 / ((20000000.0 / 13250000) * 0.80)) AS tom_shares;
-- 53,000
```

### The full post-Series-A cap table

```sql
INSERT INTO cap_table_events (event_id, holder_name, holder_type, security_type, shares, invested_amount, price_per_share, event_date, round_name, notes) VALUES
(4, 'Basecamp Seed Fund', 'SAFE-Investor',       'Preferred-Seed',    500000,    500000, 1.000000, '2024-02-01', 'Series-A', 'Converted at $10M cap'),
(5, 'Priya Anand',        'SAFE-Investor',       'Preferred-Seed',    100000,    100000, 1.000000, '2024-02-01', 'Series-A', 'Converted at $10M cap'),
(6, 'Tom Fischer',        'SAFE-Investor',       'Preferred-Seed',     53000,     64000, 1.207547, '2024-02-01', 'Series-A', 'Converted at 20% discount'),
(7, 'Option Pool',        'Pool',                'Option-Pool',      2650000,       NULL,     NULL, '2024-02-01', 'Series-A', 'New pool, 20% of pre-money FD post-topup'),
(8, 'Meridian Ventures',  'Preferred-Investor',  'Preferred-SeriesA', 3312500,   5000000, 1.509434, '2024-02-01', 'Series-A', 'Lead investor, new money');
```

```sql
SELECT holder_name, shares,
       ROUND(100.0 * shares / SUM(shares) OVER (), 2) AS ownership_pct
FROM cap_table_events
ORDER BY shares DESC;
```

```
     holder_name     |  shares  | ownership_pct
----------------------+----------+---------------
 Maya Chen            |  5500000 |         33.10
 Diego Ruiz           |  4500000 |         27.08
 Option Pool          |  2650000 |         15.95
 Meridian Ventures    |  3312500 |         19.94
 Basecamp Seed Fund   |   500000 |          3.01
 Priya Anand          |   100000 |          0.60
 Tom Fischer          |    53000 |          0.32
```

Total fully diluted shares: **16,615,500.**

## 4. Whose ownership actually absorbed the pool?

This is the punchline of the whole lecture. Before the pool, founders held 100% of 10,600,000 pre-SAFE-adjusted shares among themselves and the SAFE holders (founders alone were 94.3% of that 10,600,000). After the 2,650,000-share pool and Meridian's 3,312,500 new shares, founders are down to 60.19% combined. **Meridian's post-money ownership is exactly $5,000,000 / $25,000,000 = 20.00%** — precisely the check size over the post-money valuation, undiluted by the pool at all, because the pool was carved out of the *pre-money* side before Meridian's shares were even calculated. Every share of that pool's dilution landed on the founders and the converting SAFE holders. This is why experienced founders negotiate the *size* of the pool as hard as they negotiate the valuation — a bigger pool at the same pre-money valuation is a smaller effective valuation for the founders, even though the headline number in the term sheet never changes.

One more subtlety worth internalizing: **post-money valuation ($25,000,000) is not the same as price-per-share × total fully diluted shares** (`1.509434 × 16,615,500 ≈ $25,079,000`). The gap is Tom's SAFE — he bought in at a 20% discount, so his shares are worth less, per share, than what Meridian paid for the same class of economic claim. Post-money valuation is a defined negotiated number; it is *not* simply "current share price times share count" the instant any instrument other than the priced round's own shares exists in the table.

## 5. Check yourself

- A SAFE has a $12M cap and a 15% discount. At a priced round with an implied $9/share, does the cap or the discount give the investor more shares, and how do you know without computing both?
- Why does an option pool created *pre-money* dilute only existing holders and not the new investor?
- If Meridian's term sheet asked for a 20% pool measured on a *post-money* basis instead of pre-money, would the pool be bigger or smaller than 2,650,000 shares? (You don't need to solve it — just reason about the direction.)
- Why can't you compute post-money valuation as "price per share × total fully diluted shares" once a discounted SAFE is in the cap table?
- What's the difference between the round's "new money" and Meridian's post-money ownership percentage, and why do they turn out to be numerically equal in Solstice's case specifically (no pool absorbed by Meridian)?

Lecture 3 takes this exact cap table to an exit and asks the harder question: when the company sells, who actually gets paid, in what order, and how much.

## Further reading

- **Y Combinator — SAFE primer & standard documents:** <https://www.ycombinator.com/documents>
- **Carta — "How option pool shuffles work":** <https://carta.com/learn/startups/equity-management/option-pool/>
- **AngelList — "Priced rounds vs. SAFEs":** <https://www.angellist.com/learn>
