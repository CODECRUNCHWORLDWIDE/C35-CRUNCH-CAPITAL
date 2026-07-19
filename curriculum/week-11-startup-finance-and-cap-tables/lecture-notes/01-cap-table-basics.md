# Lecture 1 — Cap Table Basics: Founders, Shares, and Ownership

> **Duration:** ~2 hours. **Outcome:** You can explain authorized vs. issued shares, model a founding grant with vesting, compute fully-diluted ownership percentage, and see exactly why a cap table belongs in a database instead of a spreadsheet.

A **capitalization table** — "cap table" — is nothing more than a ledger: every security anyone has ever been granted or purchased in a company, and what it converts into. That's it. The reason cap tables have a reputation for being terrifying is that real ones accumulate years of grants, SAFEs, priced rounds, and option pools, and the moment two people are computing "current ownership" from two different spreadsheet tabs, someone is wrong. This week you build one the way a cap table actually *should* be built: as an append-only event log, queried, never eyeballed.

## 1. What a cap table actually tracks

Every row in a cap table answers the same four questions: **who** holds a security, **what kind** of security it is, **how many** shares (or dollars, for a not-yet-converted instrument), and **when**. Four columns, endless complexity comes from how the rows interact.

Solstice Data, Inc. incorporates with two founders. In the real world this starts with a **certificate of incorporation** authorizing a number of shares the company is *allowed* to ever issue — not the number it has issued yet. That distinction matters:

| Term | Meaning |
|------|---------|
| **Authorized shares** | The ceiling set in the corporate charter — the most the company could ever issue without amending its charter. |
| **Issued shares** | Shares that have actually been granted or sold to someone. |
| **Outstanding shares** | Issued shares still held by someone (issued minus any repurchased/cancelled). |
| **Fully diluted shares** | Outstanding shares **plus** every security that could convert into shares — unexercised options, the unallocated option pool, and unconverted SAFEs/warrants, all counted *as if* they were already common stock. |

Solstice authorizes **10,000,000** shares and issues every one of them on day one:

```sql
CREATE TABLE cap_table_events (
    event_id        INTEGER PRIMARY KEY,
    holder_name     TEXT    NOT NULL,
    holder_type     TEXT    NOT NULL,
    security_type   TEXT    NOT NULL,
    shares          NUMERIC,
    invested_amount NUMERIC,
    price_per_share NUMERIC,
    event_date      DATE    NOT NULL,
    round_name      TEXT    NOT NULL,
    notes           TEXT
);

INSERT INTO cap_table_events VALUES
(1, 'Maya Chen',  'Founder', 'Common', 5500000, NULL, 0.0001, '2023-02-01', 'Founding', 'CEO — 55% at incorporation'),
(2, 'Diego Ruiz', 'Founder', 'Common', 4500000, NULL, 0.0001, '2023-02-01', 'Founding', 'CTO — 45% at incorporation');
```

Founder splits are a negotiation, not a formula — Maya and Diego agreed 55/45 to reflect that Maya originated the idea and raised the pre-incorporation angel commitment that funded the first few months. There's no "correct" split; there's only a split both founders can live with two hard years later.

**Fully diluted ownership %** is always computed the same way, no matter how many rows exist:

```sql
SELECT
    holder_name,
    shares,
    ROUND(100.0 * shares / SUM(shares) OVER (), 2) AS ownership_pct
FROM cap_table_events
WHERE round_name = 'Founding'
ORDER BY shares DESC;
```

```
 holder_name | shares  | ownership_pct
-------------+---------+---------------
 Maya Chen   | 5500000 |         55.00
 Diego Ruiz  | 4500000 |         45.00
```

The window function `SUM(shares) OVER ()` sums the *entire result set* without collapsing it into one row — exactly what you want for a percentage-of-total column. This one query pattern is the workhorse of the whole week: as rows get added (a SAFE converts, a pool gets created, a new round prices), the same `SUM(...) OVER ()` denominator picks up the new shares automatically. You never hardcode a total.

## 2. Vesting — why "5,500,000 shares" isn't the same as "5,500,000 shares today"

Founder and employee stock is almost always **vested**, not handed over free and clear at grant. The standard schedule — so standard it barely gets negotiated — is **four years, with a one-year cliff**: nothing vests for the first 12 months, then 25% vests all at once at the 1-year mark, then the remaining 75% vests monthly (1/48th of the total) over the next 36 months. Vesting exists to solve a real problem: if a co-founder leaves after two months, unvested stock protects the company (and the *other* founder) from having handed away a quarter of the company for two months of work.

Add the vesting terms as columns and compute vested shares as of any date:

```sql
ALTER TABLE cap_table_events ADD COLUMN vest_start DATE;
ALTER TABLE cap_table_events ADD COLUMN cliff_months INTEGER;
ALTER TABLE cap_table_events ADD COLUMN vest_months INTEGER;

UPDATE cap_table_events
SET vest_start = '2023-02-01', cliff_months = 12, vest_months = 48
WHERE round_name = 'Founding';
```

```sql
-- Vested shares for each founder as of 2024-08-01 (18 months in)
SELECT
    holder_name,
    shares AS total_shares,
    CASE
        WHEN 12 * (EXTRACT(YEAR FROM AGE(DATE '2024-08-01', vest_start))) +
             EXTRACT(MONTH FROM AGE(DATE '2024-08-01', vest_start)) < cliff_months
            THEN 0
        ELSE ROUND(shares *
             LEAST(1.0,
                 (12 * EXTRACT(YEAR FROM AGE(DATE '2024-08-01', vest_start)) +
                       EXTRACT(MONTH FROM AGE(DATE '2024-08-01', vest_start)))
                 / vest_months::NUMERIC
             ))
    END AS vested_shares
FROM cap_table_events
WHERE round_name = 'Founding';
```

At the 18-month mark, both founders are past their 1-year cliff and have vested `18/48` of their grant — about 37.5%. Notice the query *itself* is where "am I past the cliff" and "how much has vested since" live — no separate spreadsheet formula copied down a column that someone forgets to update when the reference date changes. On SQLite, swap `AGE()`/`EXTRACT()` for `(julianday('2024-08-01') - julianday(vest_start)) / 30.44` to approximate months — a good moment to appreciate that Postgres's date arithmetic is genuinely nicer for this kind of calendar math.

**Unvested shares are usually subject to repurchase.** If Diego leaves after 10 months, Solstice (or the board) typically has the right to buy back his unvested shares at the original (near-zero) price he paid. The vesting schedule is what makes founder equity *earned* over time rather than a single irrevocable gift.

## 3. The advisor and early-employee pattern

Founders aren't the only holders on day one for most real startups. Say Solstice brings on a design advisor a few months in:

```sql
INSERT INTO cap_table_events VALUES
(3, 'Sam Okafor', 'Advisor', 'Common', 100000, NULL, 0.05, '2023-05-15', 'Founding',
 'Design advisor grant, 2yr vest, no cliff');
```

Two things changed the moment this row lands: total issued shares grew from 10,000,000 to 10,100,000, and every existing holder's *percentage* — not their share count — shrank slightly. This is **dilution**, and it is the single most important idea in this entire week: **your share count almost never goes down. Your percentage goes down because the denominator grows.** Maya still holds exactly 5,500,000 shares; her ownership just went from 55.00% to `5500000/10100000` = 54.46%.

```sql
SELECT holder_name, shares,
       ROUND(100.0 * shares / SUM(shares) OVER (), 2) AS ownership_pct
FROM cap_table_events
ORDER BY shares DESC;
```

Wait — where does that extra advisor share come from if authorized shares were fixed at 10,000,000? In reality, the board would either amend the charter to authorize more shares, or (more commonly) the company reserves an unallocated **option pool** carved out of authorized shares *before* anyone needs it — the subject of the next lecture, where investors will insist Solstice do exactly that before Series A prices.

## 4. Why this lives in SQL, not a spreadsheet

A spreadsheet cap table works fine for exactly one event: the founding grant. The moment a second event happens — an advisor grant, a SAFE, a priced round — a spreadsheet forces you to choose between two bad options: overwrite the old numbers (and lose the history of *when* things changed) or add a new tab (and now "current ownership" depends on remembering which tab is current). Neither scales past a handful of financing events, and startups routinely have a dozen+ by the time they exit.

An event-sourced table like `cap_table_events` has neither problem. Every historical fact is a row that never changes. "What did the cap table look like the day before Series A closed?" is a `WHERE event_date < '...'` clause, not an archaeology project through old files. "What's current ownership?" is the same `SUM(...) OVER ()` query you'll run all week, over however many rows exist. This is the whole thesis of the data tooling rule: a cap table's *complexity* comes from correctly relating rows to each other over time, which is precisely what a relational database is built to do — and precisely what a grid of cells is not.

## 5. Check yourself

- What's the difference between authorized, issued, outstanding, and fully diluted shares?
- Why does a founder's *share count* almost never decrease, even as their ownership percentage does?
- A founder is 14 months past their vest start date, with a standard 4yr/1yr-cliff schedule. Roughly what fraction of their grant has vested?
- If Solstice grants an advisor 100,000 shares out of a previously-unissued reserve, whose percentage ownership goes down, and by how does their *share count* change?
- Why is `SUM(shares) OVER ()` a better pattern for "percent of total" than computing the total in application code and dividing there?

Lecture 2 introduces the SAFE — the instrument that lets a seed investor write a check today for a number of shares that isn't determined until later — and the priced Series A round where that number finally gets pinned down.

## Further reading

- **Y Combinator — "Cap Table 101":** <https://www.ycombinator.com/library/6a-cap-tables-101>
- **Carta — "What is a cap table?":** <https://carta.com/learn/startups/equity-management/cap-table/>
- **NVCA — Model Legal Documents (incorporation, vesting):** <https://nvca.org/model-legal-documents/>
