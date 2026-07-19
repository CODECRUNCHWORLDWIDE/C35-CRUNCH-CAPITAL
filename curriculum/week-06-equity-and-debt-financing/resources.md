# Week 6 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

Nothing new this week beyond your existing setup — the same PostgreSQL 16+ / SQLite 3.35+ and Python 3.10+ with `pandas`, `numpy`, `sqlalchemy`, `psycopg[binary]` from Weeks 1–5. If you're missing any of these, see [Week 1's resources](../week-01-finance-foundations-and-tooling/resources.md) for install steps.

## Required reading (this week's core)

- **Investopedia — "Follow-On Public Offer (FPO)":** <https://www.investopedia.com/terms/f/followonoffering.asp>
  *Why: the exact mechanics Lecture 1 builds on — how a company sells new shares to raise cash after already being public.*
- **Investopedia — "Dilution":** <https://www.investopedia.com/terms/d/dilution.asp>
  *Why: a second, independent explanation of the percentage-ownership mechanic from Lecture 1 — read after the lecture to reinforce, not replace it.*
- **Investopedia — "Amortization":** <https://www.investopedia.com/terms/a/amortization.asp>
  *Why: the loan-schedule mechanic from Lecture 2, explained a second way.*
- **Investopedia — "Debt Covenant":** <https://www.investopedia.com/terms/d/debtcovenant.asp>
  *Why: the covenant concept from Lecture 2, Section 6, with real-world examples beyond this week's two ratios.*
- **Investopedia — "Pecking Order Theory":** <https://www.investopedia.com/terms/p/peckingordertheory.asp>
  *Why: the signaling framework from Lecture 3, Section 6, in a second, independent write-up.*

## Reference (keep in tabs)

- **Investopedia — "Cost of Debt":** <https://www.investopedia.com/terms/c/costofdebt.asp>
  *Why: the after-tax cost formula from Lecture 3, Section 2, with additional worked examples.*
- **Investopedia — "Cost of Equity":** <https://www.investopedia.com/terms/c/costofequity.asp>
  *Why: connects back to Week 4's CAPM work, which this week assumes as given.*
- **Investopedia — "Seniority":** <https://www.investopedia.com/terms/s/seniority.asp>
  *Why: the collateral/capital-structure ranking from Lecture 2, Section 5.*
- **Investopedia — "Interest Coverage Ratio":** <https://www.investopedia.com/terms/i/interestcoverageratio.asp>
  *Why: the exact ratio behind this week's second covenant test.*
- **PostgreSQL — Mathematical Functions (`CEIL`, `ROUND`, `POWER`):** <https://www.postgresql.org/docs/current/functions-math.html>
  *Why: the exact function signatures needed for share-count rounding and amortization math this week.*
- **PostgreSQL — Recursive Queries (`WITH RECURSIVE`):** <https://www.postgresql.org/docs/current/queries-with.html>
  *Why: an alternative, pure-SQL way to build a running-balance loan schedule, if you want to try it beyond the Python approach this week uses.*

## Practice beyond the seed data

- **SEC — "Registered Offerings":** <https://www.sec.gov/education/capitalraising/building-blocks/registered-offerings>
  *Why: how a real follow-on offering gets filed and disclosed — the regulatory context behind Lecture 1's pricing mechanics.*
- **SEC EDGAR — Full-Text Search:** <https://www.sec.gov/cgi-bin/srqsb?text=&first=1&last=40>  ·  or the modern search: <https://www.sec.gov/edgar/search/>
  *Why: search any public company's actual prospectus (Form S-1 or 424B) for a real follow-on offering, and see a real discount/fee structure alongside this week's simplified model.*
- **NumPy Financial functions (`npf.pmt`, `npf.ipmt`, `npf.ppmt`):** <https://numpy.org/numpy-financial/latest/>
  *Why: battle-tested reference implementations of the amortization formulas you're hand-building this week — useful for cross-checking your own `loan_schedule` function.*

## Deeper background (optional this week)

- **Myers, S. and Majluf, N. (1984) — "Corporate Financing and Investment Decisions When Firms Have Information That Investors Do Not Have":** <https://www.nber.org/papers/w1396>
  *Why: the original academic paper behind pecking order theory, referenced in Lecture 3 — the abstract is free even where the full paper is paywalled.*
- **Damodaran (NYU Stern) — "Financing" course materials:** <https://pages.stern.nyu.edu/~adamodar/>
  *Why: one of the most respected free, public sources on corporate financing decisions; look for his capital structure and financing-choice lecture notes.*

## Glossary

| Term | Definition |
|------|------------|
| **Equity financing** | Raising capital by selling new shares of ownership. |
| **Debt financing** | Raising capital by borrowing, with a fixed obligation to repay principal plus interest. |
| **Dilution** | The reduction in existing shareholders' percentage ownership caused by new shares being issued. |
| **Follow-on offering** | A public company selling additional new shares after its initial IPO. |
| **New-issue discount** | The percentage below current market price a new offering is priced at, to incentivize buyers. |
| **Underwriting fee / spread** | The percentage of gross offering proceeds paid to the investment bank(s) structuring and selling the deal. |
| **Coupon rate** | The stated annual interest rate on a bond or loan. |
| **Fully amortizing** | A loan structure with equal periodic payments that pays the balance to exactly zero by maturity. |
| **Seniority** | The ranking of which claims get repaid first if a company is liquidated or restructured. |
| **Secured debt** | Debt backed by specific collateral the lender can seize on default. |
| **Covenant** | A contractual condition a borrower must continuously meet for the life of a loan. |
| **Leverage ratio** | Total Debt ÷ EBITDA — a measure of how much debt a company carries relative to its cash-generating capacity. |
| **Interest coverage ratio** | EBIT ÷ Total Interest Expense — a measure of how comfortably operating earnings cover interest obligations. |
| **Technical default** | A covenant breach that gives the lender contractual remedies, even if every payment has been made on time. |
| **After-tax cost of debt** | Coupon rate × (1 − tax rate) — debt's true cost once the interest tax shield is accounted for. |
| **Cost of equity** | The annual return shareholders require to hold a company's stock, given its risk (from Week 4's CAPM). |
| **Pecking order theory** | The idea that firms prefer internal cash, then debt, then equity, largely because equity issuance carries a negative signaling cost. |
| **Signaling** | What a financing choice reveals to the market about management's private view of the company's prospects. |

---

*Broken link? Open an issue or PR.*
