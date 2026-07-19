# Challenge 1 — Find the Case Where IRR Lies

**Time:** ~90 minutes. **Builds on:** Lecture 2 (Traps #1 and #2), Exercise 2 Part D.

## The brief

You're advising Crunch Machine Co.'s capital committee on a choice between two **mutually exclusive** projects — they can build one facility, not both, on the same plot of land. One committee member is pushing hard for "the one with the higher IRR." Your job: construct a real pair of cash-flow streams where that instinct is provably wrong at the company's actual hurdle rate, and make the case for the other project using nothing but numbers.

You may build the scale trap, the timing trap, or both — but the case has to be **yours**: different cash-flow numbers from anything in the lectures or exercises, and different enough that reusing the lecture's own front/back example (or renaming its variables) doesn't count.

## Requirements

1. **Construct two mutually exclusive projects**, each 3–6 periods long, each with a single initial outflow followed only by inflows (a "normal" cash-flow shape — no multiple-IRR complications; that's a different failure mode, already covered in Lecture 2 Section 5). Give them plausible-looking dollar amounts (not $100 / $200 — pick numbers in the tens or hundreds of thousands, like the rest of this course).
2. **Compute IRR for both.** Confirm the one you're "arguing against" has the higher IRR.
3. **Find the exact crossover rate** — the discount rate where the two projects' NPVs are equal — using `scipy.optimize.brentq` (or an equivalent root-finder) on the function `npv(r, project_a) - npv(r, project_b)`. Report it to two decimal places.
4. **Plot or tabulate the full NPV profile** for both projects across a range of rates that spans well below and well above the crossover (at least 8 rate points). Show the ranking flip explicitly — a row where Project A wins, a row where Project B wins.
5. **State a specific hurdle rate** for this hypothetical decision (pick one on the "wrong-instinct" side of your crossover — i.e., the side where the higher-IRR project is NOT the higher-NPV project) and compute the exact dollar value destroyed if the committee follows IRR instead of NPV at that rate.
6. **Write the case up in 200–400 words**, addressed to the (fictional) committee member who wants to go with IRR. No jargon dump — explain the crossover concept in one paragraph a non-finance executive could follow, then show the numbers.

## What "strong" looks like

- The crossover rate is solved numerically, not eyeballed off a table.
- The NPV profile table/plot makes the flip visually obvious, not just asserted in prose.
- The dollar figure in step 5 is stated plainly: "at a hurdle rate of X%, following the IRR ranking costs the company $Y compared to the NPV-optimal choice."
- The writeup in step 6 would actually change a reasonable executive's mind — it doesn't lean on "trust me, NPV is better," it shows the specific number they'd be leaving on the table.

## Stretch goal

Do the same exercise for the **scale trap** instead of (or in addition to) the timing trap: construct a small project with a very high IRR and a large project with a lower IRR but a much higher NPV at the company's actual hurdle rate, and quantify — in dollars, not percentage points — how much value the "go with the higher IRR" instinct would cost.

## Deliverable

A short writeup (`challenge-01/README.md` or similar) with your two cash-flow streams, your code (Python file or notebook), the NPV profile table, the solved crossover rate, and the 200–400 word case from step 6.
