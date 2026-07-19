# Lecture 1 — What Corporate Finance Optimizes

> **Duration:** ~2 hours. **Outcome:** You can look at any business decision — a purchase, a hire, a loan, an acquisition — and name the three things it's actually trading off: cash, time, and risk. You can use that lens to explain *why* a decision is good or bad, not just recite a rule.

## 1. The one-sentence definition

Corporate finance is the discipline of deciding **which cash flows to create, when to take them, and how much risk to accept for them** — for the purpose of making the company (and the people who own it) better off.

That's it. Strip away the jargon — WACC, NPV, EBITDA, beta — and every technique in this 12-week course exists to answer one of three questions about a stream of money:

1. **How much cash does this generate or consume, and when?**
2. **What is that cash worth today, given that time has a price?**
3. **How risky is actually getting it — and does the reward justify that risk?**

Everything else (capital budgeting, valuation, capital structure, M&A, portfolio theory) is this same question asked about a bigger or more complicated thing.

## 2. The three levers: cash, time, risk

### Cash

Cash is the only thing a business can actually spend. Not "profit," not "revenue," not "assets on the balance sheet" — cash. A company can report a profit and still go bankrupt if the cash to pay its bills isn't in the account when the bills are due. This is why corporate finance is obsessive about **cash flow**, not accounting profit, as the unit of analysis. You'll see this distinction sharpen in Week 2 when we tie the income statement to the cash flow statement and show exactly where they diverge (hint: non-cash items like depreciation, and timing items like receivables).

Every decision a company makes is, underneath, a decision about a **stream of cash flows** — some negative (you pay for the machine, the marketing campaign, the acquisition), some positive (the machine makes product you sell, the campaign brings customers, the acquired company generates revenue). This week's seed table, `cash_flows`, is literally that: six projects, each one just a signed list of numbers on a timeline. Learn to see *any* business decision this way and the rest of this course becomes mechanical.

### Time

A dollar you receive today is worth more than a dollar you're promised a year from now — even with zero inflation and zero risk of not getting paid. Two independent reasons:

- **Opportunity cost.** A dollar in hand today can be invested (in a bond, a bank account, another project) and start earning a return immediately. A dollar promised in a year has been doing nothing for you all year.
- **Time preference.** People and companies generally prefer consumption or use of resources sooner rather than later, all else equal — this is a weaker, more behavioral reason, but it's real and shows up in how markets price things.

This is the **time value of money**, and it's the single idea Lecture 2 is entirely about. For now, internalize the consequence: **you cannot compare cash flows that happen at different times without adjusting for time.** Comparing $100 today to $110 next year, without discounting, is comparing apples wearing different clothes than oranges — the comparison is meaningless until both numbers are expressed at the *same point in time*.

### Risk

Not all promised cash flows are equally likely to actually show up. $100,000 of revenue a stable, profitable customer has already contracted to pay you next quarter is a very different $100,000 than $100,000 of revenue a brand-new market you've never sold into "should" produce in year three of a five-year plan. Both are, on paper, "$100,000 in the future" — but one is far more certain than the other.

Corporate finance's answer to this isn't "avoid risk." It's: **price risk into the required return.** The riskier the cash flow, the higher the return you demand for accepting it, and the more heavily you discount it when comparing it to certain money today. That's why, in this week's seed data, the European Market Entry project (High risk) carries an 18% hurdle rate while the Solar Rooftop Retrofit (Low risk, a fixed utility-savings stream) carries just 6%. Same company, same currency, wildly different required return — because the *certainty* of the cash flows is wildly different. You'll build the machinery to set that number precisely (CAPM, cost of capital) in Week 4; for now, understand *why* the number differs at all.

## 3. The operator's mental model

Here's the mental model this course keeps coming back to. When you face a financial decision — as a founder, an operator, an investor, or an analyst — ask, in order:

1. **What are the cash flows?** List them out: what goes out, what comes in, and *when*, as precisely as you can. If you can't name the cash flows, you don't understand the decision yet.
2. **What's the appropriate discount rate for these cash flows?** This depends on how risky they are and what else you could do with the money (the opportunity cost). A dollar of guaranteed government-bond interest and a dollar of speculative startup equity are not discounted at the same rate.
3. **What is the present value of the stream, net of what you have to pay?** This is the number that actually tells you whether the decision creates value. A project that "sounds exciting" but has negative net present value destroys value, full stop — no amount of enthusiasm changes the arithmetic.
4. **What could go wrong, and can you survive it?** Even a positive-NPV decision can bankrupt you if it requires more cash than you have when you need it, or if the downside scenario is worse than your base case assumed. This is where risk becomes not just "a number in the discount rate" but a question of solvency and survival.

You'll formalize steps 2–3 starting next lecture and build the full toolkit for step 1 and step 4 across the rest of this course (statements in Week 2, capital budgeting in Week 3, cost of capital in Week 4, capital structure and distress in Week 5).

## 4. A worked example, in plain English

Look at Project 1 in this week's seed data, **New CNC Machine**:

- Today (period 0): pay **$180,000** to buy and install the machine.
- Years 1–5: the machine saves the company money (fewer defects, faster cycle time), worth **$55,000–$62,000 a year**.
- End of year 5: sell the old machine it replaces for **$15,000** salvage.
- Risk tier: **Low** — this is a known machine, a known process, a known factory. The company assigns it a 7% hurdle rate.

Compare that to Project 6, **European Market Entry**:

- Today: **$900,000** to set up operations.
- Year 1: another **$100,000** just on regulatory and localization — before a single euro of revenue.
- Years 2–5: revenue that *might* ramp from $150,000 to $500,000 — or might not, because the company has never sold in this market before.
- Risk tier: **High** — new geography, new regulation, new customers, new competitors. The company assigns it an 18% hurdle rate — more than double the CNC machine's.

Notice what hasn't been decided yet: **which project is better.** That's not obvious from eyeballing the numbers — the warehouse project has bigger absolute cash flows than the CNC machine, and the Europe project has the biggest of all. "Bigger total cash" is not the same as "better use of capital," because the money arrives at different times and carries different risk. Untangling exactly that — which project actually creates the most value once you account for time and risk — is precisely what Lecture 2 (time value of money) and Week 3 (capital budgeting) equip you to do.

## 5. Why this course avoids spreadsheets for this

You could build all of the above in a spreadsheet — plenty of companies do. But a spreadsheet has no memory of *why* a cell has the value it has, no way to enforce that `amount` is always signed correctly, no way to ask "show me every project with a hurdle rate above 10%" without manually re-filtering, and no way to prove to your future self (or an auditor, or a teammate) that last quarter's model and this quarter's model used the same formula. A table in a real database, queried with SQL, and modeled with a versioned Python script gives you all of that for free: types are enforced, every query is reproducible, and the model is text you can diff, review, and put in source control. Lecture 3 makes this case in full; for now, just notice that everything in this lecture — cash flows, projects, risk tiers — is already sitting in the `cash_flows` table you seeded, not a workbook.

## 6. Check yourself

- In your own words, what are the three things every corporate finance decision is a trade-off between?
- Why is cash, not accounting profit, the unit corporate finance actually cares about?
- Give two independent reasons a dollar today is worth more than a dollar in a year.
- Why does the European Market Entry project carry a higher hurdle rate than the Solar Rooftop Retrofit, even though both are run by the same company?
- Why can't you compare $100,000 today to $500,000 in year 5 just by looking at the two numbers?

If those are automatic, Lecture 2 gives you the actual formula — present value and future value — that turns "time has a price" into a number you can compute.

## Further reading

- **Damodaran (NYU Stern) — "What is Corporate Finance?"** (free lecture notes): <https://pages.stern.nyu.edu/~adamodar/>
- **Investopedia — "Corporate Finance" overview:** <https://www.investopedia.com/terms/c/corporatefinance.asp>
- **Investopedia — "Time Value of Money":** <https://www.investopedia.com/terms/t/timevalueofmoney.asp>
