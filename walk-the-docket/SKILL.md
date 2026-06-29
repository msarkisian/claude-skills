---
name: walk-the-docket
description: Walk through a project's planning docs (DESIGN.md, TODO.md, ROADMAP.md, PLAN.md, etc.) and surface the consequential choices and buried assumptions baked into them, plus places where the doc has drifted from the actual code, so the user can correct misconceptions and resync the docs before stale text drives work. Use when the user wants to review a plan or design doc for smuggled-in assumptions, asks "what's on the docket," wants to check whether the plan still matches their intent or the current code, suspects their opinions have drifted from what's written down, or wants the docs resynced to reality.
---

# Walk the Docket

Plans go wrong two ways. They **accrete assumptions**: a "we could consider X" becomes load-bearing fact three sections later, a default Claude reached for gets written down as a decision, an opinion the user has since changed sits unchallenged. And they **drift from reality**: the code gets updated when the user asks for a change, the doc that described the old approach doesn't, and the text now misdescribes the system it's supposed to plan. Either way the doc stops matching the truth and starts misdirecting work. This skill walks the planning docs, surfaces those commitments and divergences, and resyncs the docs once the user says what's actually true.

## Find the docs

Look for `DESIGN.md`, `TODO.md`, `PLAN.md`, `ROADMAP.md`, `ARCHITECTURE.md`, `NOTES.md`, `SPEC.md`, and anything similar in the repo root, `docs/`, and `.github/`. Include any file the user names. If you find several, list them and confirm scope before diving in.

## What to surface

Review **consequential choices, not line items.** A chore like "write tests for the parser" needs no review; "the parser is recursive-descent" is a choice that forecloses an alternative and shapes everything downstream. Walking every checkbox makes the review long and low-value *and* buries the real commitments among the chores — so filter for items where being wrong would invalidate work that depends on them. A long doc collapses to a handful of these.

Within those, two kinds of problem are worth catching — one judged from the text, one verified against the code.

### Smuggled assumptions (doc vs. intent)

You almost never have the conversation that produced the doc, so you **cannot know what the user actually decided versus what got assumed.** Don't pretend to: asserting "you decided X" or "I assumed X" fabricates a trace and either falsely reassures or falsely alarms. Instead, judge from the **text**, which is what you can actually inspect. Scan for these tells:

- **Specificity beyond rationale** — a named library, threshold, schema, or format stated flatly with no "because." The confidence of the prose outruns any reason given for it.
- **Hardened hedges** — a "maybe / consider / probably / TBD / one option is" in one place that's treated as settled elsewhere in the doc.
- **Conventional defaults** — choices that look like your own house style (Postgres, REST, a standard folder layout) rather than a project-specific call. This is exactly where you tend to smuggle your taste in as the user's.
- **Cascading coupling** — an item that silently rests on an earlier choice, so if the earlier one is wrong, several others fall with it. Surface the root, and name what cascades.

### Drift (doc vs. code)

Where the doc makes a checkable claim about how the system works, **verify it against the code instead of taking the doc's word.** The common case: the user asked for a change, it landed in the code, and the doc still describes the old shape. So when doc and code disagree, the doc is the more likely suspect — but not the certain one. Tells:

- A named module, file, endpoint, or function the doc relies on that's absent or renamed in the repo.
- A "we will use X" / "X works by Y" where the code now uses something else.
- A described flow, data shape, or default the implementation contradicts.
- A decision the doc still presents as open that the code has already settled — or one the doc states as settled that the code went the other way on.

You have ground truth you can point to here, so this isn't a guess: cite both sides — what `DESIGN.md:line` claims and what the code at `path:line` actually does. But a mismatch doesn't *automatically* mean the doc loses. The code may be what drifted from the intended plan, or the doc may describe a target not yet built. Surface the divergence and let the user name which side is truth; lead with the likelihood that the doc is stale, but don't assume it.

## How to present each one

For **assumptions**, surface, don't lead. **Never phrase it so the question answers itself** — "You're going with Postgres, right?" invites a rubber-stamp and the assumption survives the review. Give:

1. **What the doc commits to** — quote or cite it (`file:line`).
2. **What it forecloses or rests on** — the alternative it rules out, or the earlier choice it depends on.
3. **Why it might not be what you want** — the wording was tentative, it reads as my default rather than your decision, it's a hedge that hardened, or it couples to something else you might revisit.

For **drift**, you have ground truth, so state the mismatch factually rather than softening it into a leading question: cite the doc's claim and the contradicting code, say which you believe is current (usually the code), and ask the user to confirm which side is truth.

Then let the user react. Concentrate on the consequential items so the walk stays short; for a doc with many, go in themed passes rather than one undifferentiated wall.

Examples:

> **`DESIGN.md:42` — events stored as an append-only log, not mutable rows.**
> This rules out in-place edits and makes every read a fold over history. The doc states it flatly with no rationale, and it's the kind of default I reach for — was this your call, or worth reconsidering for this project?

> **`DESIGN.md:88` says ingestion is synchronous, but `ingest.py:31` enqueues to a worker and returns immediately.**
> The doc looks stale — the code went async. Want me to resync the doc to match, or did the code drift from the plan?

## After the user reacts

You confirmed — edit the docs to match:

- **Correction** (assumption was wrong): rewrite the item to reflect actual intent.
- **Tentative / undecided**: demote it from a stated decision to an open question — move it under an "Open questions" heading or annotate it `(undecided — option A vs. B)` — so it stops reading as settled.
- **Drift, doc is stale**: rewrite the item to match what the code actually does, citing the code that's now the source of truth.
- **Drift, code is wrong**: leave the doc and flag the divergence as work — the implementation needs to catch up to the plan (e.g. add it to TODO), not the other way around.

Leave confirmed items as they are. Show the user the edits as you go.
