# Tracker backends

Two backends. Choose per repo: **GitHub Issues** when `git remote get-url origin` points at github.com and `gh auth status` succeeds; otherwise **local markdown**. Whichever backend a map was created on is the one every later session uses for it.

## GitHub Issues (via `gh`)

`gh api` auto-fills `{owner}/{repo}` from the current repo. Sub-issue and dependency endpoints take the issue's **database id** (`.id`), not its number — fetch it first:

```bash
ID=$(gh api repos/{owner}/{repo}/issues/<number> --jq .id)
```

| Operation | Command |
|---|---|
| Create map | `gh issue create --title "<name>" --label wayfinder:map --body-file map.md` (create the label first if missing: `gh label create wayfinder:map`) |
| Create ticket | `gh issue create --title "<name>" --label "wayfinder:<type>" --body-file ticket.md`, then attach as child: `gh api repos/{owner}/{repo}/issues/<map-number>/sub_issues -F sub_issue_id=<ticket-id>` |
| Wire blocking | `gh api repos/{owner}/{repo}/issues/<blocked-number>/dependencies/blocked_by -F issue_id=<blocker-id>` |
| List children | `gh api repos/{owner}/{repo}/issues/<map-number>/sub_issues --jq '.[] \| {number, title, state, assignees: [.assignees[].login]}'` |
| Check blockers | `gh api repos/{owner}/{repo}/issues/<number>/dependencies/blocked_by --jq '[.[] \| select(.state == "open")] \| length'` — 0 means unblocked |
| Claim | `gh issue edit <number> --add-assignee @me` |
| Zoom a ticket | `gh issue view <number> --comments` |
| Resolve | `gh issue comment <number> --body "<answer>"` then `gh issue close <number>` |
| Update map body | `gh issue edit <map-number> --body-file map.md` (fetch current body first: `gh issue view <map-number> --json body --jq .body`) |

**Frontier query**: list children, keep open + unassigned, then drop any with open blockers.

If the sub-issue or dependency endpoints aren't available on this GitHub instance (404), fall back to body conventions — first line of each ticket body: `Map: #<map-number>`, plus `Blocked by: #<n> #<n>` when blocked — and query children by searching open issues for the `Map:` line.

## Local markdown

Lives in `.wayfinder/` at the repo root:

```
.wayfinder/
├── map.md               # the map body
└── tickets/
    └── 003-<slug>.md    # one file per ticket; the number is its id
```

Ticket file shape — frontmatter is the tracker state, body is the question:

```markdown
---
title: <ticket name>
type: research | prototype | grilling | task
status: open | closed
claimed-by:              # git user.name; empty = unclaimed
blocked-by: []           # ticket numbers, e.g. [1, 4]
---

## Question

<the decision or investigation this ticket resolves>

## Resolution

<appended on close — the answer>
```

| Operation | How |
|---|---|
| Create map / ticket | Write the file; next ticket number = highest existing + 1 |
| Wire blocking | Add numbers to the blocked ticket's `blocked-by` |
| Claim | Set `claimed-by` to `git config user.name` |
| Resolve | Append `## Resolution`, set `status: closed` |
| Frontier | Tickets with `status: open`, empty `claimed-by`, and every `blocked-by` entry closed |
| Links | Relative paths, e.g. `[Pick the queue backend](tickets/003-queue-backend.md)` |

Commit `.wayfinder/` changes as part of resolving a ticket — the working tree is the shared state, so an uncommitted resolution is invisible to parallel sessions and collaborators.
