# Lecture 3 — Triangulating a Range

> **Duration:** ~2 hours. **Outcome:** You can bridge any enterprise value to a per-share equity price, lay every method's answer for Crunch Machine Co. side by side, explain — in specific, quantified terms — why they disagree, and land on one recommended value or range you can defend.

You now have, from the last two lectures, **four different enterprise-value estimates** for the same company on the same day:

| Method | Enterprise Value |
|---|---:|
| DCF — perpetuity growth terminal value | ≈ $523,900,000 |
| DCF — exit multiple terminal value | ≈ $461,400,000 |
| Trading comps (trimmed EV/EBITDA range) | ≈ $362,000,000 – $447,700,000 |
| Precedent transactions (trimmed EV/EBITDA range) | ≈ $430,500,000 – $487,800,000 |

None of these is wrong. All four are legitimate, defensible answers to slightly different questions, built on different evidence. This lecture does three things: bridges each one to a per-share price (Section 1), explains specifically why they don't agree (Section 2), and gives you a repeatable process for landing on one number anyway (Section 3).

## 1. The EV-to-equity bridge

Enterprise value is the value of the whole business's *operations*, available to all capital providers. To get to what a common shareholder's stake is worth, you have to subtract everything that has a claim on that value *ahead of* common equity, and add back anything the company holds that isn't part of its core operations:

```
Enterprise Value
− Total Debt                (lenders get paid before shareholders)
− Preferred Stock            (if any — preferred holders also rank above common)
− Minority Interest           (if any — the portion of a consolidated subsidiary common shareholders don't own)
+ Cash & Equivalents           (non-operating; belongs to shareholders on top of EV)
+ Non-operating investments     (if any — same logic as cash)
= Equity Value
÷ Diluted Shares Outstanding
= Value Per Share
```

The `Total Debt − Cash` piece is so common it gets its own name — **net debt** — and for a company like Crunch Machine Co. with no preferred stock, minority interest, or non-operating investments, the full bridge collapses to just:

```
Equity Value = Enterprise Value − Net Debt
Net Debt = Total Debt − Cash & Equivalents = 110,000,000 − 35,000,000 = 75,000,000
```

Applying that $75,000,000 net debt and 24,000,000 shares outstanding to every method above:

| Method | Enterprise Value | Equity Value | Per Share |
|---|---:|---:|---:|
| DCF — perpetuity growth | 523,893,110 | 448,893,110 | **$18.70** |
| DCF — exit multiple | 461,366,567 | 386,366,567 | **$16.10** |
| Trading comps — low | 362,000,000 | 287,000,000 | **$11.96** |
| Trading comps — high | 447,700,000 | 372,700,000 | **$15.53** |
| Precedent transactions — low | 430,500,000 | 355,500,000 | **$14.81** |
| Precedent transactions — high | 487,800,000 | 412,800,000 | **$17.20** |

This same bridge runs **backward** just as often in real work: given a *quoted* share price, multiply by shares outstanding to get market cap, add net debt, and you have the market's own implied enterprise value — the starting point for reverse-engineering what growth rate or multiple the market must be assuming (Challenge 2 has you do exactly this).

## 2. What "diluted shares outstanding" is quietly assuming

This week's `valuation_assumptions.shares_outstanding` (24,000,000) is used as-is, but the bridge formula in Section 1 technically asks for **diluted** shares outstanding — the share count *including* the effect of every option, warrant, and convertible security that could turn into common stock. A company with a large employee option pool or convertible notes outstanding has a diluted share count meaningfully higher than its basic count, which means the same equity value gets divided by a bigger number and produces a *lower* per-share price than a careless basic-share calculation would suggest. The standard adjustment (the **treasury stock method** for options: assume every in-the-money option is exercised, and the cash the company receives from that exercise buys back some shares at the current price, partially offsetting the dilution) is beyond this week's scope — Week 11's cap-table work builds it in full — but the rule to carry forward now is simple: **always ask whether the share count you're dividing by already reflects dilution**, because getting this wrong systematically overstates the per-share value of any company with meaningful options or convertibles outstanding.

## 3. Computing the bridge in SQL and Python

Because every method's enterprise value ends up going through the *same* bridge, it's worth writing it once as a reusable piece rather than repeating the subtraction by hand for each method. In SQL, applied directly against `valuation_assumptions`:

```sql
SELECT
    company,
    total_debt - cash_and_equivalents                                    AS net_debt,
    523893110 - (total_debt - cash_and_equivalents)                      AS equity_value_dcf_pg,
    ROUND((523893110 - (total_debt - cash_and_equivalents))
          / shares_outstanding::NUMERIC, 2)                              AS per_share_dcf_pg
FROM valuation_assumptions;
```

(The hardcoded `523893110` here is the DCF perpetuity-growth EV from Lecture 1/Exercise 2 — in the mini-project's engine, this becomes a parameter, not a literal, exactly like every other function this week.) In Python, the same bridge as a small reusable function:

```python
def ev_to_equity_bridge(enterprise_value, total_debt, cash, shares_outstanding):
    net_debt = total_debt - cash
    equity_value = enterprise_value - net_debt
    per_share = equity_value / shares_outstanding
    return {"net_debt": net_debt, "equity_value": equity_value, "per_share": per_share}

for label, ev in [("DCF - perpetuity growth", 523_893_110), ("DCF - exit multiple", 461_366_567)]:
    result = ev_to_equity_bridge(ev, 110_000_000, 35_000_000, 24_000_000)
    print(f"{label}: ${result['per_share']:.2f}/share")
```

Writing the bridge once and calling it for every method (rather than re-deriving the subtraction each time) is exactly the kind of small discipline that keeps a valuation model auditable — if the net-debt figure ever needs correcting, it changes in one place and every downstream per-share number updates correctly, the same argument this course has made about SQL and Python over spreadsheets since Week 1.

## 4. Why the methods disagree — quantify it, don't wave at it

"They're all just estimates" is true but useless. A defensible valuation names the *specific* reasons two methods land in different places, in numbers, not vibes.

**DCF vs. trading comps (the biggest gap: ~$18.70 vs. ~$11.96–$15.53).** The DCF assumes Crunch Machine Co.'s own margin-expansion story (21.0% → 23.5% EBITDA margin) plays out exactly as modeled, discounted at a WACC that reflects *this specific company's* risk. Trading comps instead reflect what the market is *currently* willing to pay for a basket of peers — peers whose own growth and margin trajectories may be flatter, and whose trading prices reflect *today's* market sentiment about the whole industrial-equipment sector, which can be temporarily depressed or elevated for reasons that have nothing to do with Crunch Machine Co. specifically. **One useful diagnostic:** back out the growth rate the trading-comps multiple *implicitly* assumes, using the same Gordon-growth relationship from Lecture 1, and compare it to the DCF's explicit 2.5% terminal growth assumption. If the market-implied growth rate is meaningfully lower than what the DCF assumes, that's the quantified reason for the gap, not a mystery — the DCF is pricing in more optimism than the market is currently willing to pay for. Challenge 2 has you compute this exactly.

**DCF perpetuity growth vs. DCF exit multiple ($18.70 vs. $16.10).** These two used the *same* explicit-period forecast — the only difference is the terminal assumption. Recall from Lecture 1, Section 7 that the perpetuity-growth method's $625.2M terminal value implies an exit multiple of about **9.47×** EBITDA (higher than the 8.0× directly assumed in the exit-multiple method), while the exit-multiple method's $528.1M terminal value implies a perpetuity growth rate of only about **1.36%** (lower than the 2.5% directly assumed in the perpetuity method). The two internal assumptions aren't quite consistent with each other — which is precisely why cross-checking one method's *implied* parameter against the other's *explicit* one (Exercise 2) is standard practice, not busywork.

**Trading comps vs. precedent transactions (~$15.53 high vs. ~$17.20 high).** This is the expected, healthy gap from Lecture 2, Section 5 — the control premium. It is *supposed* to exist; a near-zero gap here would be the surprising result worth investigating, not the other way around.

## 5. A process for landing on one number

1. **Rule out anything with a broken assumption first**, before averaging anything. If one comp's business model turns out to be genuinely different from the target's (Exercise 3 and Challenge 2 both probe this), drop it — don't let a bad comp quietly drag a "more data is always better" average in the wrong direction.
2. **Look for convergence.** Lay every method's range on one number line (a **football field** chart — Challenge 1 builds this) and find where multiple *independent* methods' ranges overlap. Overlap between methods that used different evidence (a DCF assumption and a market multiple, say) is much stronger evidence than agreement between two flavors of the same method.
3. **Weight by how much you trust the evidence, not by how many methods you ran.** If the comp set is thin or the peers are only loosely comparable, a well-built DCF might deserve more weight even if it's the "odd one out" numerically. If the DCF rests on aggressive, hard-to-defend margin assumptions, the market-grounded comps deserve more weight instead. This is a judgment call — state it explicitly, don't hide it inside an unweighted average.
4. **Report a range, not a false-precision point number**, unless you're specifically required to name one price (e.g., setting an offer price in a negotiation). "$15.50–$18.00 per share, with a base case around $16.75–$17.00" is more honest, and more useful to a decision-maker, than a single number implying two decimal places of confidence a valuation exercise this uncertain can't actually support.

Applying that process to this week's numbers: the **overlap zone** where DCF (exit-multiple, $16.10), the top of trading comps ($15.53), and precedent transactions ($14.81–$17.20) all sit is roughly **$15.50–$17.20**. The DCF perpetuity-growth result ($18.70) sits above that zone — worth flagging as the optimistic case, not discarding — and the low end of trading comps ($11.96, driven by the lowest-multiple peer) sits below it as the pessimistic case. A recommended range of **$15.50–$18.00, with a base case near $16.75**, is a genuinely defensible answer: it's grounded in where independent methods agree, it doesn't hide the wide DCF-vs-comps disagreement, and it gives a decision-maker something concrete to act on. The mini-project has you build this exact analysis and defend your own version of it.

## 6. What this range is (and isn't) good for

A triangulated range like this is exactly the kind of number that supports a **decision** — is this stock cheap or expensive at its current quoted price; is an acquisition offer at a given price generous or stingy; does a private company's ask in a financing round look reasonable. It is **not** a promise about what the stock will trade at next month, and it is not a substitute for due diligence on the underlying assumptions (comp selection, forecast drivers, the WACC itself) that fed into it. Every number in this week's analysis is only as trustworthy as the weakest assumption feeding it — which is exactly why Challenge 2 makes you interrogate, not just report, the gap between methods.

## 7. Common mistakes

- **Reporting enterprise value as if it were a stock price.** They can differ by tens of dollars a share for a leveraged company — always run the bridge in Section 1 before quoting a per-share number.
- **Averaging every method with equal weight without checking whether that's justified.** A weak comp set and a strong DCF (or vice versa) don't deserve equal votes just because you ran both.
- **Treating disagreement between methods as a bug to be resolved by picking a favorite**, instead of the expected, informative signal it usually is (Section 2).
- **Reporting a single point estimate with implied precision the underlying analysis can't support** — a range, with a stated base case, is more honest and more useful.

## 8. Check yourself

- Write the EV-to-equity bridge from memory. Which balance-sheet items rank *ahead of* common equity, and which get added back?
- Crunch Machine Co.'s DCF (perpetuity growth) and trading-comps-high per-share values differ by more than $3. Name the two *specific, quantified* reasons Section 2 gives for a gap like this.
- What does it mean for two DCF terminal-value methods to have "inconsistent implied parameters," and how do you check for it?
- Why is finding overlap between *independently derived* method ranges stronger evidence than two similar methods agreeing with each other?
- In one sentence, explain why a valuation range is usually more honest than a single point number.
- Why does using *basic* shares outstanding instead of *diluted* shares outstanding systematically overstate a per-share value, for a company with a meaningful option pool?

You now have every piece of this week's mini-project: a DCF (Lecture 1), a comps and precedent-transactions analysis (Lecture 2), and the bridge and reconciliation process to turn all of it into one defensible answer (this lecture). Take the [quiz](../quiz.md), work the exercises and challenges, then build the [mini-project](../mini-project/README.md).

## Further reading

- **Damodaran (NYU Stern) — "The Value of Control":** <https://pages.stern.nyu.edu/~adamodar/New_Home_Page/valcontrol.htm>
- **Investopedia — "Equity Value vs. Enterprise Value":** <https://www.investopedia.com/ask/answers/difference-between-enterprise-value-and-equity-value/>
- **Investopedia — "Football Field Chart":** <https://www.investopedia.com/terms/f/football-field-chart.asp>
- **Damodaran (NYU Stern) — "Valuation: Art, Science, or Craft?" (paper, free PDF):** <https://pages.stern.nyu.edu/~adamodar/pdfiles/papers/artofvaluation.pdf>
