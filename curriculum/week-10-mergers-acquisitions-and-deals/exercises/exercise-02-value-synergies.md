# Exercise 2 — Value deal synergies

**Time:** ~90 minutes. **Builds on:** Exercise 1 and Lecture 2, Sections 6–8 (synergies, and the NPV method).

## Goal

Value the two synergy streams in `synergies` — one cost, one revenue — using the phase-in and risk-discount rules from Lecture 2, then re-run Exercise 1's accretion/dilution model with the full run-rate synergies added to combined EBIT.

## Part A — Cost synergy NPV (SQL or Python)

Using the `synergies` row where `synergy_type = 'Cost'`:

1. Compute the after-tax run-rate: `annual_run_rate × (1 − tax_rate)`. (Use CRNM's 25% tax rate from `acquirer_financials`.)
2. Compute the Year 1 after-tax cash flow using `year1_phase_in_pct`.
3. Compute the Year 2+ (full run-rate) perpetuity value, valued as of the *end of Year 1*.
4. Discount both pieces back to today at CRNM's WACC — **9.2%**, carried forward unchanged from Weeks 4 and 7.

Reproduce Lecture 2, Section 8's worked example exactly before moving on.

**Expected result:** cost synergy NPV ≈ **$23,429,965**.

## Part B — Revenue synergy NPV (SQL or Python)

Using the `synergies` row where `synergy_type = 'Revenue'`, this one has three extra wrinkles versus Part A:

1. Apply the `risk_discount_pct` haircut to `annual_run_rate` **first** — before anything else. (Lecture 2, Section 7 explains why this order matters.)
2. This synergy phases in over **three** years (`year1_phase_in_pct`, `year2_phase_in_pct`, `year3_phase_in_pct` — the schema has all three; the cost synergy just happens to reach 100% by year 2).
3. Tax the risk-adjusted, phased cash flows the same way as Part A, and value the Year 3+ perpetuity discounted back the appropriate number of years.

**Expected result:** revenue synergy NPV ≈ **$3,742,615**.

## Part C — Combined synergy value

```
total_synergy_npv = cost_synergy_npv + revenue_synergy_npv
```

**Expected result:** ≈ **$27,172,580**.

Compare this to the $195,000,000 equity purchase price and the ~$45,000,000 control premium CRNM is paying over SPSY's unaffected value (`19.50 - 15.00) × 10,000,000`). Write one sentence: does the synergy value CRNM is underwriting cover the premium it's paying, and by how much?

## Part D — Re-run accretion/dilution with synergies

Take Exercise 1, Part C's pro forma statement and add the **full run-rate** synergy amount (not the NPV — the annual dollar figure, `$3,000,000` cost + risk-adjusted revenue run-rate) to combined EBIT, exactly as Lecture 2, Section 6 does. Recompute combined EBT, tax, net income, and pro forma EPS.

Hint: the "full run-rate" figure for the income-statement add-back is different from the NPV — it's simply the steady-state annual dollar amount from Section 6, not a discounted value. Don't confuse the two; NPV values a synergy in *today's dollars for the whole future stream*, while the accretion/dilution model adds one *year's* run-rate to one year's income statement.

**Expected result:** pro forma EPS with synergies ≈ **$0.9906**; accretion ≈ **+16.5%**.

## Verify your work

| Metric | Expected value |
|---|---:|
| Cost synergy NPV | ≈ $23,429,965 |
| Revenue synergy NPV | ≈ $3,742,615 |
| Total synergy NPV | ≈ $27,172,580 |
| Pro forma EPS with synergies | ≈ $0.9906 |
| Accretion/(dilution) with synergies | ≈ +16.5% |

If your revenue synergy NPV comes out roughly double the expected value, you likely forgot the 50% risk discount — check that you applied it to `annual_run_rate` before phasing in, not after.
