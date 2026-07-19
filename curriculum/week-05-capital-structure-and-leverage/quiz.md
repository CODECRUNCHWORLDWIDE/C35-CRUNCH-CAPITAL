# Week 5 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 6. A mix of multiple-choice, short calculation, and "what does this formula say" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** Under MM Proposition I **without taxes**, the value of a levered firm compared to an otherwise identical unlevered firm is:

- A) Always higher, because debt is cheaper than equity
- B) Always lower, because debt adds risk
- C) Exactly the same — capital structure doesn't change total firm value in a perfect market
- D) Undefined without knowing the interest rate

---

**Q2.** Under MM Proposition II **without taxes**, as a firm's `D/E` ratio rises, its cost of equity `rE`:

- A) Falls, because debt is cheaper
- B) Rises, exactly offsetting the effect of adding cheaper debt so WACC stays constant
- C) Stays flat regardless of leverage
- D) Becomes negative at high leverage

---

**Q3.** What single real-world feature of most tax codes is the reason MM Proposition I's "capital structure is irrelevant" result breaks down once taxes are introduced?

- A) Dividends are taxed at a lower rate than capital gains
- B) Interest expense is tax-deductible, but dividend payments are not
- C) Corporate tax rates change every year
- D) Debt must be repaid, but equity doesn't

---

**Q4.** The annual interest tax shield is computed as:

- A) `interest expense × (1 − Tc)`
- B) `Tc × total debt`
- C) `Tc × interest expense`
- D) `total debt / Tc`

---

**Q5.** Under the perpetual-debt assumption, the present value of the interest tax shield simplifies to `Tc × D`. Why does the cost of debt `rD` cancel out of that formula?

- A) `rD` is assumed to be zero for all companies
- B) The annual shield (`Tc × rD × D`) is divided by `rD` (the perpetuity discount rate), and the `rD` terms cancel algebraically
- C) Tax shields are never discounted
- D) `rD` only matters for finite-life debt, never for perpetual debt

---

**Q6.** A company with `D = 40,000,000` in permanent debt and `Tc = 21%` has a tax-shield present value of:

- A) `8,400,000`
- B) `40,000,000`
- C) `21,000,000`
- D) Cannot be determined without knowing `rD`

---

**Q7.** Which of the following is an **indirect** cost of financial distress, as distinguished from a direct cost?

- A) Court filing fees for a bankruptcy proceeding
- B) Legal fees paid to restructuring attorneys
- C) Customers avoiding a supplier they fear won't exist in two years
- D) Accounting fees for the bankruptcy process

---

**Q8.** In this week's stylized tradeoff model, the probability of distress is modeled as `p(D) = (D/VU)²`. Why is squaring the ratio, rather than using the ratio directly, important to the model's behavior?

- A) It has no effect; a linear function would produce an identical result
- B) It creates convexity — distress risk rises slowly at low leverage and much faster at high leverage, which is what produces an interior value-maximizing debt level
- C) Squaring is required to keep the units in dollars instead of a percentage
- D) It ensures `p(D)` is always exactly equal to 1

---

**Q9.** In the tradeoff model `V_L(D) = V_U + Tc·D − (α/VU)·D²`, which term is responsible for the fact that firm value eventually **falls** as debt keeps increasing?

- A) `VU`, because unlevered value is always shrinking
- B) `Tc·D`, because the tax rate eventually turns negative
- C) `(α/VU)·D²`, the expected distress-cost term, because it grows faster (quadratically) than the linear tax-shield term as `D` increases
- D) None of the terms — firm value never falls as debt rises under this model

---

**Q10.** Solving `dV_L/dD = 0` for the tradeoff model gives the value-maximizing debt level:

- A) `D* = Tc × VU`
- B) `D* = Tc × VU / (2α)`
- C) `D* = α / (2 × Tc × VU)`
- D) `D* = VU / Tc`

---

**Q11.** Using this week's Crunch Machine Co. FY2025 parameters (`VU = 16,687.5`, `Tc = 0.25`, `α = 0.30`), the model's `D*` is approximately `6,953`. Crunch Machine Co.'s **actual** FY2025 total debt is `19,700`. This means, according to the model, Crunch Machine Co. is:

- A) Under-levered by about half
- B) Almost exactly at the optimum
- C) Over-levered by roughly 2.8 times the model's optimal debt level
- D) The model cannot compare actual debt to `D*` directly

---

**Q12.** The Degree of Financial Leverage (DFL) is computed as:

- A) `interest expense / EBIT`
- B) `EBIT / pretax income` (equivalently, `EBIT / (EBIT − interest)`)
- C) `net income / total equity`
- D) `total debt / total equity`

---

**Q13.** If a company's DFL is `2.5`, and its EBIT falls by `8%`, its net income (holding share count fixed, roughly) falls by approximately:

- A) `8%`
- B) `3.2%`
- C) `20%`
- D) `2.5%`

---

**Q14.** Crunch Machine Co.'s DFL rose from about `1.10` in FY2021 to about `2.63` in FY2025. What does that trend, by itself, tell you?

- A) The company's revenue fell every year
- B) The company's earnings became substantially more sensitive to swings in EBIT — the same percentage change in operating performance now produces a much larger percentage swing in net income than it did in 2021
- C) The company's tax rate increased significantly
- D) DFL is unrelated to how risky the company's earnings are

---

**Q15.** In this week's stylized distress model, `p(D) = (D/VU)²` can exceed `1` once `D` grows larger than `VU`. What is the correct way to handle this when it happens?

- A) Let the calculation proceed unchanged — a probability greater than 1 is a valid edge case in this model
- B) Cap `p(D)` at `1`, since it represents a probability and cannot legitimately exceed certainty; recognize this as a sign the debt level has pushed the model past its valid range
- C) Set `p(D)` to `0` whenever it would exceed `1`
- D) Switch to a completely different formula for `VU` instead

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **C** — MM Proposition I (no taxes): `VL = VU`. Slicing the same cash-flow pie into different debt/equity claims doesn't change the size of the pie in a perfect market.
2. **B** — `rE` rises linearly with `D/E`, exactly offsetting the lower weight given to expensive equity as cheap debt's share grows, holding WACC (and thus firm value) constant.
3. **B** — interest is deductible, dividends are not; that asymmetry is the entire reason debt financing creates a tax advantage equity financing doesn't have.
4. **C** — `Tc × interest expense`. The annual tax shield is the tax rate applied to the deductible interest payment.
5. **B** — the annual shield is `Tc × rD × D`; dividing by the perpetuity discount rate `rD` cancels the `rD` in the numerator, leaving `Tc × D` with no rate term at all.
6. **A** — `0.21 × 40,000,000 = 8,400,000`.
7. **C** — lost customers is an indirect cost (a behavioral/reputational consequence of perceived distress); A, B, and D are all direct, out-of-pocket costs of an actual bankruptcy proceeding.
8. **B** — the square creates convexity: distress risk barely moves at low leverage but accelerates sharply at high leverage, which is exactly the shape needed to eventually outpace the linear tax shield and produce an interior maximum.
9. **C** — the quadratic distress-cost term grows faster than the linear `Tc·D` tax-shield term as `D` rises, eventually dominating it and pulling `V_L(D)` back down.
10. **B** — `D* = Tc·VU / (2α)`, derived by setting the derivative `Tc − (2α/VU)D` to zero and solving for `D`.
11. **C** — `19,700 / 6,953 ≈ 2.83`, so the company is carrying roughly 2.8 times the model's value-maximizing debt level — meaningfully over-levered under this model's assumptions.
12. **B** — `DFL = EBIT / pretax income`, equivalently `EBIT / (EBIT − interest)`.
13. **C** — `8% × 2.5 = 20%`. DFL is the multiplier applied to a percentage change in EBIT to get the resulting percentage change in net income.
14. **B** — a rising DFL means the same-sized EBIT swing produces an increasingly larger swing in net income — the company's earnings became mechanically more volatile as its fixed interest burden grew relative to a shrinking EBIT base.
15. **B** — cap `p(D)` at 1; a probability can never legitimately exceed certainty, and hitting this case is itself a signal the debt level has been pushed past the region the stylized model was built to describe.

</details>

**Scoring:** 13+ → start Week 6. 10–12 → re-read the lecture sections behind your misses (especially Lecture 2's derivation if you missed Q8–Q11). <10 → re-read all three lectures from the top; this week's formulas compound directly into Week 6's financing-choice comparisons.
