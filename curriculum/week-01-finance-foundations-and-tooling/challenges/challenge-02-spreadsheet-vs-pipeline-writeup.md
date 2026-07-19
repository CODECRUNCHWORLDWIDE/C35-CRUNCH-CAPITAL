# Challenge 2 — Spreadsheet vs. Pipeline Writeup

**Time:** ~90 minutes. **Difficulty:** Medium. **No single right answer — this is an argument, not a calculation.**

## The scenario

Crunch Machine Co.'s Head of FP&A has run every capital-project analysis in a single, sprawling Excel workbook for six years. It has 40+ tabs, formulas that reference other formulas across a dozen sheets, and at least one senior analyst who is the only person who fully understands how the "Master Discount Engine" tab works. She's heard you're building this course's `cash_flows` pipeline in Postgres and Python, and she's skeptical: *"It's one more thing to learn, and Excel has worked fine for six years. Convince me."*

You are not allowed to answer with vibes ("it's more modern," "spreadsheets are outdated"). You have to answer with **specific, falsifiable claims** about what goes wrong with the workbook approach and what specifically fixes it — grounded in this week's `cash_flows` table and this week's lectures.

## Your task

Write a memo, `challenge-02.md`, 600–900 words, addressed to this skeptical Head of FP&A. It must include all five of the following, each backed by a concrete example (not an abstract claim):

1. **One realistic failure mode** the workbook is exposed to that the pipeline structurally prevents. Don't just name the category (e.g., "formula errors") — construct a specific, plausible scenario using this week's data: e.g., "someone updates the European Market Entry project's hurdle rate in one cell of the summary tab, but three other tabs that reference the old rate in a hardcoded formula don't get updated, and the reported NPV is now wrong by $X — with no visual sign anything is broken." Do the arithmetic for your scenario.

2. **A demonstration query** — an actual `SELECT` against the `cash_flows` table — that answers a real FP&A question ("which projects have a hurdle rate above 10%?" or similar) in one line, and a description of the equivalent multi-step manual process in the spreadsheet (open tab, click filter, select criteria, verify no rows were missed, repeat next quarter). Make the time/effort/error-risk comparison concrete, not hand-wavy.

3. **An honest concession** — name the ONE thing the spreadsheet genuinely does better or more easily than the SQL + Python pipeline (a real tradeoff — e.g., a non-technical stakeholder can open and skim it without any setup; a first draft or true one-off napkin calculation is often faster to bang out in a blank sheet). A writeup that pretends the pipeline is strictly superior at everything is not credible and won't be — because it isn't true.

4. **A migration argument, not a replacement argument.** She has six years of history in that workbook. Argue for *how* the transition would actually happen (e.g., start with new projects in the pipeline while the workbook still runs for existing reporting; migrate the highest-risk/highest-reuse models first) rather than a wave-your-hands "just switch everything."

5. **A one-sentence answer to her literal question**, "It's worked fine for six years — why change now?" — addressing that spreadsheets often *do* work fine right up until the moment they don't (a wrong reference goes unnoticed for months, a departing analyst takes tribal knowledge of "how the Master Discount Engine tab actually works" with them), and that the cost of that failure mode is precisely what's being purchased down by moving to a pipeline.

## Constraints

- **No abstractions without a concrete example.** Every claim needs a number, a query, or a named scenario from this week's data — "spreadsheets are error-prone" is a thesis statement, not an argument.
- **Steelman the other side.** Section 3's concession must be a real, defensible point — if you can't think of one, you haven't understood spreadsheets' actual strengths, and your argument will read as one-sided propaganda rather than a technical case.
- **No "just because the syllabus says so."** Don't cite this course's data tooling rule as your justification — the Head of FP&A doesn't work here and doesn't care what this course requires. Convince her on the merits.

## Hints

<details>
<summary>On finding a concrete failure-mode scenario (item 1)</summary>

Pick any two rows in `cash_flows` that share a `project_id` and imagine a formula error where one row's discount calculation used the *wrong* period or the *wrong* hurdle rate. Compute what the NPV would look like with the error vs. without it — a real dollar gap is far more convincing than "errors can happen."

</details>

<details>
<summary>On the honest concession (item 3)</summary>

Think about who has to interact with the model, not just who builds it. A workbook requires zero setup to open and browse for someone without a terminal. A one-off "let me just sanity-check this number in five minutes before a call" is often genuinely faster in a blank spreadsheet than standing up a script. These are real strengths — name them plainly.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Concreteness | Generic claims about spreadsheets being risky | Specific scenario with an actual dollar figure computed |
| Query evidence | Describes SQL abstractly | Includes a real, runnable query against `cash_flows` |
| Intellectual honesty | Pretends the pipeline is superior in every respect | Names one genuine, defensible spreadsheet advantage |
| Practicality | "Just rewrite everything" | A phased, plausible migration path |
| Persuasion | Reads like a lecture | Reads like it would actually change a skeptical, busy person's mind |

## Submission

Commit `challenge-02.md` to your portfolio under `c35-week-01/challenge-02/`.
