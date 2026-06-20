---
name: walk-the-docket
description: Walk through a project's planning docs (DESIGN.md, TODO.md, ROADMAP.md, PLAN.md, etc.) and surface the consequential choices and buried assumptions baked into them, so the user can correct misconceptions before they harden into work. Use when the user wants to review a plan or design doc for smuggled-in assumptions, asks "what's on the docket," wants to check whether the plan still matches their intent, or suspects their opinions have drifted from what's written down.
---

# Walk the Docket

Plans accrete assumptions. A "we could consider X" becomes load-bearing fact three sections later; a default Claude reached for gets written down as a decision; an opinion the user has since changed sits unchallenged. This skill walks the planning docs and surfaces those commitments so the user can correct them before they drive work.

## Find the docs

Look for `DESIGN.md`, `TODO.md`, `PLAN.md`, `ROADMAP.md`, `ARCHITECTURE.md`, `NOTES.md`, `SPEC.md`, and anything similar in the repo root, `docs/`, and `.github/`. Include any file the user names. If you find several, list them and confirm scope before diving in.

## What to surface

Review **consequential choices, not line items.** A chore like "write tests for the parser" needs no review; "the parser is recursive-descent" is a choice that forecloses an alternative and shapes everything downstream. Walking every checkbox makes the review long and low-value *and* buries the real commitments among the chores — so filter for items where being wrong would invalidate work that depends on them. A long doc collapses to a handful of these.

You almost never have the conversation that produced the doc, so you **cannot know what the user actually decided versus what got assumed.** Don't pretend to: asserting "you decided X" or "I assumed X" fabricates a trace and either falsely reassures or falsely alarms. Instead, judge from the **text**, which is what you can actually inspect. Scan for these tells of a smuggled assumption:

- **Specificity beyond rationale** — a named library, threshold, schema, or format stated flatly with no "because." The confidence of the prose outruns any reason given for it.
- **Hardened hedges** — a "maybe / consider / probably / TBD / one option is" in one place that's treated as settled elsewhere in the doc.
- **Conventional defaults** — choices that look like your own house style (Postgres, REST, a standard folder layout) rather than a project-specific call. This is exactly where you tend to smuggle your taste in as the user's.
- **Cascading coupling** — an item that silently rests on an earlier choice, so if the earlier one is wrong, several others fall with it. Surface the root, and name what cascades.

## How to present each one

Surface, don't lead. **Never phrase it so the question answers itself** — "You're going with Postgres, right?" invites a rubber-stamp and the assumption survives the review. For each choice, instead give:

1. **What the doc commits to** — quote or cite it (`file:line`).
2. **What it forecloses or rests on** — the alternative it rules out, or the earlier choice it depends on.
3. **Why it might not be what you want** — name the specific reason: the wording was tentative, it reads as my default rather than your decision, it's a hedge that hardened, or it couples to something else you might revisit.

Then let the user react. Concentrate on the consequential choices so the walk stays short; for a doc with many, go in themed passes rather than one undifferentiated wall.

Example:

> **`DESIGN.md:42` — events stored as an append-only log, not mutable rows.**
> This rules out in-place edits and makes every read a fold over history. The doc states it flatly with no rationale, and it's the kind of default I reach for — was this your call, or worth reconsidering for this project?

## After the user reacts

You confirmed: edit the docs to match. For a **correction**, rewrite the item to reflect actual intent. For something the user marks **tentative or undecided**, demote it from a stated decision to an open question (e.g. move it under an "Open questions" heading or annotate it `(undecided — option A vs. B)`) so it stops reading as settled. Leave confirmed items as they are. Show the user the edits as you go.
