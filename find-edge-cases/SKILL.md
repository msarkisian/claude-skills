---
name: find-edge-cases
description: Systematically audit a codebase for edge cases the code doesn't sufficiently handle, by inferring intended behavior and finding the inputs and states that violate it. Reports findings only — no code or test changes. Use when the user wants to find uncovered edge cases, hunt for gaps in boundary or error handling, audit a function/module/codebase for missing cases, or asks "what edge cases aren't handled here?"
---

# Finding Uncovered Edge Cases

Hunt for inputs and states the code *should* handle but doesn't. The output is a report — you do not change code or write tests unless the user asks separately.

## What makes something a finding

A candidate is only worth reporting when all three hold, each backed by evidence you actually checked:

1. **It can occur** — a real call site, input source, or reachable state can produce it. Not theoretical.
2. **Intended behavior differs from actual** — you can point to *something* (a type, name, docstring, existing validation, test, or call-site expectation) that says this case should be handled differently than it is.
3. **The code doesn't already handle it** — you read the actual code path and confirmed no guard, branch, or upstream check covers it.

Drop any candidate that fails any of the three. This is the spine of the skill: an unverified "what if X is null?" that the code already guards, or that nothing can ever trigger, is noise that buries the real gaps.

## Process

1. **Scope.** Default to the whole codebase. If the user named a file, module, or function, audit that instead.
2. **Prioritize, don't sweep uniformly.** For a whole-codebase audit, rank surfaces by blast radius and attack first: trust boundaries (user/network/file input), parsing and deserialization, money/auth/permission logic, concurrency and shared mutable state, resource lifecycles (handles, connections, transactions), and external I/O. A bug in a parser outranks a bug in a log formatter. For large codebases, fan out with the Explore agent or subagents to map these surfaces before reading deeply.
3. **Reconstruct the intended contract.** Before judging a unit, infer what it's *supposed* to do from signatures, types, names, docstrings, existing tests, and how call sites use it. Note explicitly where intent is genuinely ambiguous — that's a question, not a finding.
4. **Probe the input/state space** against that contract (see lenses below) to surface candidates.
5. **Verify each candidate** against the three-part test above. Read the real code path; never assume a guard is or isn't there.
6. **Report** (format below).

## Where gaps hide (lenses, not a checklist)

Use these to interrogate the code — but every candidate they surface still has to pass the three-part test:

- **Boundaries** — 0, 1, empty, single-element, max, off-by-one, overflow/underflow
- **Absence** — null, undefined, missing key, empty string, default-when-unset
- **Malformed / adversarial** — wrong type, injection, huge input, unexpected encoding
- **Concurrency & ordering** — races, reentrancy, stale reads, out-of-order events
- **Partial failure** — an operation half-completes; is it rolled back or left inconsistent?
- **Resource limits** — exhaustion, leaks, unclosed handles, unbounded growth
- **Time & environment** — clock skew, timezones, DST, locale, platform differences

## Don't assert a gap you can't ground

Never report an unhandled case without citing what says it *should* be handled — otherwise you flood the report with deliberate omissions and the user can't separate a real bug from your guess. When the contract is ambiguous (the code could plausibly intend either behavior), surface it as a question to the user rather than asserting a bug. A short, trustworthy report beats an exhaustive one the user has to re-verify.

## Report format

Order findings by severity × likelihood. For each:

- **Location** — `file:line`
- **Trigger** — the specific input or state that exposes the gap
- **Reachability** — how that input/state actually arises (the evidence for point 1)
- **Now vs. intended** — what the code does today and what it should do, with the evidence of intent (point 2)
- **Suggested handling** — one line; do not implement it

Close with any **ambiguous contracts** as open questions. If you bounded the audit (e.g. only covered high-risk surfaces, or capped findings), say so plainly so the absence of findings elsewhere isn't read as a clean bill of health.
