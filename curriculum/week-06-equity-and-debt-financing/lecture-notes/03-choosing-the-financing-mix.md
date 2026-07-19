# Lecture 3 — Choosing the Financing Mix

> **Duration:** ~2 hours. **Outcome:** You can compute the true, all-in cost of an equity raise and a debt raise on the same footing, and argue — with Crunch Machine Co.'s actual numbers — why "whichever is cheaper" is not, by itself, a complete answer to how a company should finance itself.

## 1. Two raises, same $20,000,000, very different shapes

Lecture 1 priced the equity path: sell 1,108,034 new shares, net $20,000,013.70, dilute every existing shareholder by roughly 9.975% of their prior stake, forever. Lecture 2 priced the debt path: borrow $20,000,000 at 7.5%, pay it back over 5 years with $4,943,294.36 annual payments, and stay inside two covenants the whole time. Both raise the same dollar amount for the same distribution-hub project. They are not, in any other sense, the same transaction. This lecture puts a number on exactly how they differ, along three axes: **cost**, **control**, and **flexibility/signaling**.

## 2. The true cost of debt: after-tax, not the sticker coupon

The 7.5% coupon is not what debt actually costs the company, because **interest is tax-deductible**. Every dollar of interest paid reduces taxable income by a dollar, which — at Crunch Machine Co.'s 25% tax rate — hands back 25 cents of that dollar as a tax saving. This is the **interest tax shield** from Week 5, now made concrete:

```
After-tax cost of debt = Coupon rate × (1 − Tax rate)
                        = 7.5% × (1 − 0.25)
                        = 5.625%
```

In dollar terms, summed across the loan's whole 5-year life (using the interest column from Lecture 2's schedule):

```sql
SELECT ROUND(SUM(interest_paid), 2) AS total_interest,
       ROUND(SUM(interest_paid) * (1 - 0.25), 2) AS total_interest_after_tax
FROM term_loan_schedule;
-- total_interest ≈ 4,716,471.78
-- total_interest_after_tax ≈ 3,537,353.84
```

Add the 1% origination fee (`$200,000`, paid once at closing — this course treats it as a simple non-deductible closing cost, since the exact tax treatment of financing fees varies by jurisdiction and instrument, and that detail isn't the point here) and the **all-in cost of the debt raise, over its 5-year life**, is:

```
Total after-tax cost ≈ 3,537,353.84 + 200,000 = 3,737,353.84
```

Spread evenly across the 5 years the loan is outstanding, that's roughly **$747,470.77 per year** — and then it stops. Once the final payment lands in year 5, the debt's cost disappears entirely; the company owes nothing further.

## 3. The true cost of equity: no tax shield, and no expiration date

Equity has no coupon and no maturity — but it is not free. Shareholders who buy into the offering are demanding a **return**, exactly the cost of equity Week 4 taught you to compute with CAPM. Assume Crunch Machine Co.'s cost of equity, from Week 4's CAPM work, is **11%** — the annual return shareholders require to hold this stock given its risk. That required return applies to the **entire $20,000,013.70** raised, and unlike a dividend, it is not a discretionary payment the company can skip in a bad year without consequence — it's the return the market is *pricing the stock to deliver*, whether or not it's paid out as cash:

```
Annual cost of equity capital = Cost of equity × Net proceeds
                               = 11% × 20,000,013.70
                               ≈ 2,200,001.51 per year
```

Critically, **this cost never turns off**. New shareholders own their 9.975% stake forever (until they sell it to someone else — but the company itself never buys it back or repays it, the way a loan gets repaid). There is no year 5 where equity's cost disappears the way debt's does. And unlike interest, **there is no tax shield** — dividends paid to shareholders are not tax-deductible, so every dollar of equity's required return is a full, un-discounted dollar of cost to the company, not a tax-reduced 5.625%-equivalent one.

## 4. Side by side

| | Equity — Follow-On Offering | Debt — Term Loan |
|---|---|---|
| Net proceeds | $20,000,013.70 | $20,000,000.00 |
| Upfront cost | $1,052,632.30 underwriting fee | $200,000.00 origination fee |
| Annual cost (%) | 11.00% (cost of equity, no tax shield) | 5.625% (after-tax cost of debt) |
| Annual cost ($, approx.) | ≈ $2,200,000/year | ≈ $747,471/year (5 years only) |
| Duration of the cost | **Forever** — no maturity, no repayment | **5 years** — then the obligation ends entirely |
| Payment mandatory? | No — dividends are discretionary | **Yes** — missed payment can trigger default |
| Dilutes ownership/control? | **Yes** — new shareholders get votes | No — lenders get no equity vote |
| Constrains operations? | Generally no | **Yes** — leverage and coverage covenants |

The gap between 11.00% and 5.625% — **5.375 percentage points, annually, on $20 million** — is the tax-advantaged, fixed-claim discount from Week 5's capital structure lecture, now sitting in front of you as real dollars: roughly **$1.45 million a year** cheaper to service as debt than as equity, for as long as the debt is outstanding. On cost alone, debt wins, decisively. If cost were the *only* axis, this would be a short lecture. It isn't the only axis.

## 5. Control: who you answer to versus what you're allowed to do

Debt and equity restrict a company in structurally different ways, and conflating them is the single most common mistake in a rushed financing decision.

- **Equity dilutes control.** New shareholders get voting rights, proportional to their stake. From Lecture 1: the two founders together held 45.000% of the company before the raise (30.000% + 15.000%); after, they hold a combined 40.511% (27.007% + 13.504%). Still comfortably the largest bloc, and nowhere near losing control on this single raise — but every subsequent equity round chips further into that number, and enough rounds can eventually put a founder below a blocking-minority threshold in a company they started. This is a **permanent, cumulative** cost that compounds across every future equity raise.
- **Debt constrains action, not voting.** The term loan's lenders get no board seat and no vote on strategy. But the covenants from Lecture 2 — leverage ≤ 3.00×, interest coverage ≥ 4.00× — actively restrict what the company can do for the life of the loan: no casual additional secured borrowing, and (typically) no dividend increases if leverage climbs. This is a **real** loss of freedom, but it is **contractual and temporary** — it ends when the loan is repaid, unlike equity's dilution, which is permanent from the moment the shares are issued.

Put simply: **equity costs you a permanent share of who decides. Debt costs you a temporary set of rules about what you're allowed to decide.** Which one is worse depends entirely on how much a company's founders and existing shareholders value control versus flexibility — a judgment call finance formulas alone cannot make for you.

## 6. Signaling: what the market infers when you announce either one

Markets don't just price the transaction — they price **what the transaction reveals about what management believes**. This is the core insight of the finance concept called **pecking order theory** (Myers and Majluf, 1984): managers know more about their company's true prospects than outside investors do, and outside investors know this. That asymmetry shapes how each financing announcement gets read:

- **Announcing an equity offering** often triggers a **negative** stock-price reaction. The market's logic: if management genuinely believed the stock was undervalued at $20.00, they would rather borrow, or wait, than sell new ownership on the cheap — selling equity *now* is a signal (fair or not) that management thinks the stock is fully or over-valued, and this is a good time to sell shares while the price is favorable to the company. This is exactly why the follow-on is priced at a discount in the first place (Lecture 1, Section 2) — the discount partly compensates new buyers for stepping into a trade the market suspects is stacked against them.
- **Announcing a debt raise** is typically read more neutrally, or even positively — taking on a fixed, mandatory obligation signals confidence that future cash flows will be strong enough to cover it. It also avoids diluting existing shareholders, which the market generally reads as management protecting, rather than cashing out of, its own conviction in the stock.

This is exactly why the **pecking order** most companies actually follow, in practice, is: fund projects with **internal cash** first if available, reach for **debt** second, and treat **equity** as a last resort reserved for when debt capacity is exhausted or a specific strategic reason (an acquisition, delevering, a genuine belief that the stock is overvalued) makes it the right call anyway. This isn't a rule this course asks you to accept on faith — Homework and the Challenges have you stress-test it against cases where the pecking order gets overridden for good reason.

## 7. Flexibility: fixed obligations versus discretionary ones

The last axis is what happens in a **bad year**. If Crunch Machine Co.'s EBITDA falls sharply (Challenge 1 makes this concrete), the term loan's $4,943,294.36 payment is still due, in full, on schedule — that obligation does not flex with how the business is actually doing, and missing it can trigger covenant breach, technical default, and a scramble to renegotiate or refinance under pressure. A dividend, by contrast, can simply be **cut or skipped** in a bad year with no legal consequence beyond an unhappy shareholder base and a likely stock-price hit — painful, but survivable, and entirely within management's control. Debt's fixed obligation is precisely what makes it cheaper (lenders bear less uncertainty about getting paid, so they charge less for the privilege) and precisely what makes too much of it dangerous (Week 5's distress-cost curve). Equity's flexibility is precisely what makes it more expensive (shareholders bear the residual risk of everything else being paid first, so they demand more) and precisely what makes it safer to hold more of when the future is genuinely uncertain.

## 8. A framework, not a formula

There is no single number that resolves "equity or debt" the way NPV resolves "accept or reject" in Week 3. What you can do — and what the mini-project asks you to do for Crunch Machine Co. specifically — is lay out all three axes side by side, with real numbers on each, and make an explicit, defensible recommendation instead of an implicit, unexamined one:

1. **Cost** — compute the true after-tax annual cost of each path, as done in Sections 2–4. Cheaper is not automatically better, but it's never irrelevant.
2. **Capacity** — does the company have room for more debt without breaching covenants or drifting into the distress-cost territory Week 5 warned about? Check the leverage and coverage math from Lecture 2 before assuming debt is available at all.
3. **Control** — how much does existing ownership (especially founders, if control matters strategically) stand to lose, and does the company have room to give some up without consequence?
4. **Flexibility and signaling** — how certain is the company that it can service a fixed obligation through a bad year, and what does the market likely infer from either choice?

A CFO who can answer all four, with real numbers, is doing the job properly. A CFO who stops at "debt is cheaper" has only answered one of the four questions that actually matter.

## 9. Common mistakes

- **Comparing debt's finite-life cost to equity's forever cost without normalizing.** $747,471/year for 5 years is not directly comparable to $2,200,000/year forever without deciding on a comparison horizon or discounting both to a present value — Exercise 3 makes you do this properly.
- **Treating cost as the only axis.** As Sections 5–7 show, control and flexibility are real, quantifiable-in-consequence costs even when they don't show up as a percentage rate.
- **Assuming debt is always available at the quoted rate.** A company already near its leverage covenant on existing debt may not be able to add $20M more without breaching it immediately — capacity has to be checked, not assumed (this is exactly Challenge 1's setup).
- **Ignoring signaling because it isn't in the spreadsheet — or rather, the SQL table.** The market's reaction to a financing announcement is a real cost (or benefit) even though it doesn't appear as a line item in any of this week's tables.

## 10. Check yourself

- Compute, in your own words, why debt's after-tax cost (5.625%) is lower than its stated coupon (7.5%), and why equity has no equivalent discount.
- Name one cost of equity financing that is permanent and one cost of debt financing that is temporary. Why does that distinction matter for a company that expects its financing needs to change significantly in 5–10 years?
- Explain pecking order theory in two sentences, without using the phrase "asymmetric information" — say what it actually means in plain terms.
- If Crunch Machine Co.'s EBITDA fell 15% next year, which of the two financing paths would put the company under more immediate pressure, and specifically why?

You now have the full picture: dilution mechanics (Lecture 1), debt mechanics and covenants (Lecture 2), and the cost/control/flexibility framework to weigh them against each other (this lecture). The exercises turn this into working code against Crunch Machine Co.'s real numbers; the mini-project asks you to make the actual recommendation.

## Further reading

- **Investopedia — "Pecking Order Theory":** <https://www.investopedia.com/terms/p/peckingordertheory.asp>
- **Investopedia — "Cost of Equity":** <https://www.investopedia.com/terms/c/costofequity.asp>
- **Investopedia — "Cost of Debt":** <https://www.investopedia.com/terms/c/costofdebt.asp>
- **Investopedia — "Signaling":** <https://www.investopedia.com/terms/s/signaling.asp>
- **Myers, S. and Majluf, N. (1984) — "Corporate Financing and Investment Decisions When Firms Have Information That Investors Do Not Have" (the original pecking-order paper; abstract freely available):** <https://www.nber.org/papers/w1396>
