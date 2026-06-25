---
name: improve-codebase-architecture
description: Find deepening opportunities in a codebase, informed by the project's domain language and recorded architecture decisions where they exist. Use when the user wants to improve architecture, find refactoring opportunities, consolidate tightly-coupled modules, or make a codebase more testable and AI-navigable.
---

# Improve Codebase Architecture

Surface architectural friction and propose **deepening opportunities** — refactors that turn shallow modules into deep ones. The aim is testability and AI-navigability.

## Glossary

Use these terms exactly in every suggestion. Consistent language is the point — don't drift into "component," "service," "API," or "boundary." Full definitions in [LANGUAGE.md](LANGUAGE.md).

- **Module** — anything with an interface and an implementation (function, class, package, slice).
- **Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, config. Not just the type signature.
- **Implementation** — the code inside.
- **Depth** — leverage at the interface: a lot of behaviour behind a small interface. **Deep** = high leverage. **Shallow** = interface nearly as complex as the implementation.
- **Seam** — where an interface lives; a place behaviour can be altered without editing in place. (Use this, not "boundary.")
- **Adapter** — a concrete thing satisfying an interface at a seam.
- **Leverage** — what callers get from depth.
- **Locality** — what maintainers get from depth: change, bugs, knowledge concentrated in one place.

Key principles (see [LANGUAGE.md](LANGUAGE.md) for the full list):

- **Deletion test**: imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.
- **The interface is the test surface.**
- **One adapter = hypothetical seam. Two adapters = real seam.**

This skill is _informed_ by the project's domain model where one is written down. A domain glossary (often a `CONTEXT.md`) gives names to good seams; architecture decision records (often in `docs/adr/`) record decisions the skill should not re-litigate. Neither is required — use them if the project has them, and fall back to names inferred from the code if it doesn't.

## Process

### 1. Explore

If the project has a domain glossary (e.g. a `CONTEXT.md`) or architecture decision records (e.g. in `docs/adr/`), read the ones covering the area you're touching first. If it has neither, skip straight to walking the code.

Then use the Agent tool with `subagent_type=Explore` to walk the codebase. Don't follow rigid heuristics — explore organically and note where you experience friction:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules **shallow** — interface nearly as complex as the implementation?
- Where have pure functions been extracted just for testability, but the real bugs hide in how they're called (no **locality**)?
- Where do tightly-coupled modules leak across their seams?
- Which parts of the codebase are untested, or hard to test through their current interface?

Apply the **deletion test** to anything you suspect is shallow: would deleting it concentrate complexity, or just move it? A "yes, concentrates" is the signal you want.

### 2. Present candidates

Present a numbered list of deepening opportunities. For each candidate:

- **Files** — which files/modules are involved
- **Problem** — why the current architecture is causing friction
- **Solution** — plain English description of what would change
- **Benefits** — explained in terms of locality and leverage, and also in how tests would improve

**If the project has a domain glossary, use its vocabulary for the domain, and [LANGUAGE.md](LANGUAGE.md) vocabulary for the architecture.** If the glossary defines "Order," talk about "the Order intake module" — not "the FooBarHandler," and not "the Order service." With no glossary, use the clearest domain names the code itself already establishes.

**ADR conflicts**: if a candidate contradicts an existing ADR, only surface it when the friction is real enough to warrant revisiting the ADR. Mark it clearly (e.g. _"contradicts ADR-0007 — but worth reopening because…"_). Don't list every theoretical refactor an ADR forbids.

Do NOT propose interfaces yet. Ask the user: "Which of these would you like to explore?"

### 3. Grilling loop

Once the user picks a candidate, drop into a grilling conversation. Walk the design tree with them — constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive. To work out how to test across the seam, classify the candidate's dependencies (in-process, local-substitutable, remote-but-owned, true-external) — see [DEEPENING.md](DEEPENING.md), which maps each category to a testing strategy and to whether a port/adapter is actually warranted.

Side effects happen inline as decisions crystallize:

- **Naming a deepened module after a concept the project's glossary doesn't yet cover?** If the project keeps a domain glossary (e.g. `CONTEXT.md`), add the term: a tight one-or-two-sentence definition of what it _is_, with any aliases to avoid. Create the file lazily if it doesn't exist and the project would benefit from one; don't force a glossary on a project that has no other use for it.
- **Sharpening a fuzzy term during the conversation?** Update the glossary right there if one exists.
- **User rejects the candidate with a load-bearing reason?** If the project records architecture decisions (e.g. in `docs/adr/`), offer one, framed as: _"Want me to record this as an ADR so future architecture reviews don't re-suggest it?"_ Only offer when the reason is hard to reverse, surprising without context, and the result of a real trade-off — and would actually be needed by a future explorer to avoid re-suggesting the same thing. Skip ephemeral reasons ("not worth it right now") and self-evident ones. Keep the ADR to a sentence or three: what was decided and why.
- **Want to explore alternative interfaces for the deepened module?** See [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md).
