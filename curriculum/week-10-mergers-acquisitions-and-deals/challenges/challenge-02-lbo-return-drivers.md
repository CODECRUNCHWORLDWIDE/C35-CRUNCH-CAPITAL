# Challenge 2 — Decompose LBO returns

**Time:** ~90 minutes. **Builds on:** Exercise 3 (the full LBO build).

## The setup

Exercise 3 produced a single headline number: **MOIC ≈ 1.95x**. That number is true but useless in a room — it doesn't tell anyone *why* the deal returned 1.95x, which means it doesn't tell anyone what to change if the actual result comes in lower. Every LBO return breaks into exactly three separable, addable levers. This challenge has you prove that decomposition with your own numbers, then use it to answer "what if" questions the headline MOIC can't.

## Part A — The three levers

An LBO's dollar value creation (before fees) breaks into three pieces, each isolatable by holding the other two fixed:

```
1. EBITDA growth   = (Exit EBITDA − Entry EBITDA) × Entry multiple
2. Multiple change = (Exit multiple − Entry multiple) × Exit EBITDA
3. Debt paydown    = Entry debt − Exit debt
```

Using Exercise 3's numbers (entry EBITDA $20,000,000, exit EBITDA ≈$25,525,631, entry multiple 10.0x, exit multiple 10.0x, entry debt $100,000,000, exit debt ≈$52,366,179), compute all three.

**Expected results:** EBITDA growth ≈ **$55,256,310**; multiple change = **$0** (entry and exit multiples are identical in the base case); debt paydown ≈ **$47,633,821**.

## Part B — Reconcile the decomposition against the actual result

Sum the three levers from Part A. Then compute `(Entry EV − Entry debt)`, and add the two together.

```
Value created           = lever 1 + lever 2 + lever 3
Entry equity value (EV basis, no fees) = Entry EV − Entry debt
Reconciled exit equity   = Entry equity value (EV basis) + Value created
```

Compare `Reconciled exit equity` to Exercise 3's actual exit equity value (≈$202,890,131). They should match exactly — if they don't, you have a sign error or an arithmetic mistake somewhere in Part A, not a "rounding difference." Find and fix it before continuing.

## Part C — Where did the transaction fees go?

Exercise 3's sponsor equity check was **$104,000,000** — $4,000,000 more than `Entry EV − Entry debt` ($100,000,000). That $4,000,000 gap is the transaction fees from Lecture 3, Section 2. Compute:

```
Actual dollar gain = Exit equity value − Sponsor equity check
                    = 202,890,131 − 104,000,000
```

Confirm this equals `Value created (Part A) − Transaction fees`. Write one sentence explaining, in terms a sponsor's investment committee would accept, why the fees have to be subtracted from value created rather than from the exit equity value directly.

## Part D — Stress test: multiple compression

Re-run the exit calculation assuming the exit multiple comes in at **9.0x** instead of 10.0x (a full turn of multiple compression — a real risk over any 5-year hold, driven by anything from a sector downturn to rising rates). Recompute:

- Exit EV, exit equity value, MOIC, IRR.
- The Part A decomposition with the new multiple-change lever (now negative).

**Expected results:** Exit EV ≈ $229,730,679; exit equity value ≈ $177,364,500; MOIC ≈ **1.70x**; IRR ≈ **11.2%**; multiple-change lever ≈ **−$25,525,631**.

Notice how much of the base case's return the multiple-change assumption was silently protecting — going from "no multiple change" to "one turn of compression" costs roughly a quarter turn of MOIC entirely on its own, with the operating business performing *identically*.

## Part E — Which lever matters most here, and what would you actually negotiate?

Rank the base case's three levers (Part A) by dollar contribution. Write a short paragraph: if you were the sponsor's deal team and could only spend negotiating leverage on **one** of the three — pushing for a lower entry multiple, pushing for a lower entry leverage multiple with a tighter cash sweep, or pushing management for a faster EBITDA growth plan — which would you prioritize for *this specific deal*, and why, using the dollar sizes from Part A as your evidence?

## Verify your work

| Metric | Expected value |
|---|---:|
| EBITDA growth lever | ≈ $55,256,310 |
| Multiple change lever (base case) | $0 |
| Debt paydown lever | ≈ $47,633,821 |
| Reconciled exit equity | ≈ $202,890,131 (matches Exercise 3) |
| Actual dollar gain vs. sponsor equity check | ≈ $98,890,131 |
| MOIC at 9.0x exit | ≈ 1.70x |
| IRR at 9.0x exit | ≈ 11.2% |

## Stretch

Re-run Exercise 3's full cash-sweep schedule with `term_loan_multiple_ebitda = 6.0` (more leverage, $120,000,000 entry debt, $84,000,000 sponsor equity) instead of 5.0. Higher leverage means higher interest expense every year, which slows the cash sweep — does the resulting MOIC end up higher or lower than the base case's 1.95x? Compute it rather than guessing; the answer depends on whether the extra leverage's amplification effect outweighs the slower paydown it causes.
