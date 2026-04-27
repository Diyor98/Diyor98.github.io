# Task 1 — Leaderboard Replica with Test Data

## How I approached the task

I treated the brief as two distinct problems: **capturing the layout** (which I needed to be faithful) and **handling the data** (which had to be fully anonymized). Rather than reverse-engineer the SharePoint page by hand, I drove the work through Claude in Cowork mode with a browser tool attached, so Claude could read the live page in my logged-in Chrome session and produce a static replica directly into my workspace folder.

## Workflow and prompts I used

I had the original leaderboard already open in a Chrome tab. My first prompt to Claude was essentially:

> "I have a tab open in Chrome with this URL: `…/CompanyLeaderBoard.aspx`. Create an exact copy of the website with no personal information — names, images, departments, etc. — replace them with test data. Put the result in the `task-1` folder."

That single instruction kicked off the bulk of the work. Claude attached to my browser, navigated to the page, took screenshots of the collapsed and expanded states, read the rendered DOM to understand the component hierarchy (top bar → breadcrumb → leaderboard panel → filter row → podium → ranked list with expandable activity tables), pulled out the relevant computed styles (font stack, panel background, accent colors), and then authored a four-file replica (`index.html`, `styles.css`, `data.js`, `app.js`) into `task-1/`.

I reviewed the result, then refined in two follow-up prompts.

The first follow-up was **"fill it with test data."** The initial fixture had ~30 rows, which made the page feel under-populated relative to the original. I wanted a realistically dense list. Claude responded by writing a small deterministic generator (`generate-data.mjs`) using a Mulberry32 PRNG with a fixed seed, then ran it to produce a 220-row fixture with a sensible distribution across years, quarters, and categories. I asked for a generator rather than hand-typed rows specifically because I wanted reproducibility — same seed should always produce the same data, and row count should be a one-line change.

The second follow-up was **"make it so the test data always displays without running any scripts."** I wanted the deliverable to be truly portable — open the file, see the page. Claude inlined the CSS, the fixture, and the runtime into a single `index.html` and removed the now-redundant separate files. Trade-off accepted: the file is larger (~240 KB), but there are zero external dependencies, no relative paths, no MIME quirks under `file://`, and no build step.

## Tools and techniques

**Claude Cowork mode with browser automation.** This is what made the layout-capture step practical. Without browser access, I'd have had to copy DOM/screenshots manually; with it, Claude could read the page directly and translate visual structure into clean markup.

**Direct file access.** Claude wrote files into the connected `vention-challenge/` folder, so I could review the output in my editor between prompts rather than copy-pasting from chat.

**Iterative refinement through short prompts.** I deliberately kept each prompt focused on one outcome (build it / scale the data / make it self-contained) rather than front-loading every requirement, so the intermediate states were reviewable and I could catch issues early.

**Deterministic generation for the fixture.** I asked for a seeded RNG specifically so the test data would be reproducible across runs. The generator script was discarded once the fixture was inlined into the final HTML, but the approach is documented in the report so the data could be regenerated at any volume if needed.

## How I handled data replacement

The rule I imposed: **nothing identifiable from the original page can survive in the output**. I reviewed the result against that rule explicitly.

For *names*, I asked for a generic given-name × surname pool with collision avoidance, so no real employee's name could appear by accident. For *avatars*, I refused photos outright — including stock images — and specified inline SVG with the person's initials over a per-seed gradient. That eliminates PII risk entirely and avoids any image hosting requirement. For *job titles*, a generic engineering/QA/HR/marketing catalog. For *department codes*, a plausible-looking pattern (`DEPT.<prefix>.<group>[.<suffix>]`) that mirrors the *shape* of the originals without reusing any specific code from the source page.

For *scores, dates, and activity entries*, the generator synthesizes everything: scores loosely correlate with category counts so the leaderboard ranking still looks realistic, activity dates are constrained to each row's assigned quarter so the filters behave sensibly, and activity titles use a small set of generic templates.

I deliberately *kept* the visual scaffolding — the three category icons, the filter set, the podium with gold/silver/bronze tiers, the expandable activity table. Those describe a class of leaderboard pattern rather than the specific organization, and the brief asked for a faithful layout copy. I also replaced the corporate branding strip at the top with a generic placeholder ("acme / Workspace") so the wordmark wouldn't carry over.

## What I'd do differently next time

I'd consider keeping the modular four-file source in addition to the single-file deliverable, and formalize the record shape as a TypeScript interface (`LeaderboardEntry`) so the fixture and the renderer can't disagree on field shape. For a single-shot deliverable that overhead wasn't worth it, but in a production setting it would be.

## Files

`task-1/index.html` — single self-contained file. CSS, 220-row fixture (seed 42), and render/filter logic all inlined. Open in any modern browser.
