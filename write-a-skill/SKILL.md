
---
name: write-a-skill
description: Create new agent skills with proper structure, progressive disclosure, and bundled resources. Use when user wants to create, write, or build a new skill.
---

# Writing Skills

## Process

1. **Gather requirements** - ask user about:
   - What task/domain does the skill cover?
   - What specific use cases should it handle?
   - Does it need executable scripts or just instructions?
   - Any reference materials to include?

2. **Draft the skill** - create:
   - SKILL.md with concise instructions
   - Additional reference files if content exceeds 500 lines
   - Utility scripts if deterministic operations needed

3. **Review with user** - present draft and ask:
   - Does this cover your use cases?
   - Anything missing or unclear?
   - Should any section be more/less detailed?

## Skill Structure

```
skill-name/
├── SKILL.md           # Main instructions (required)
├── REFERENCE.md       # Detailed docs (if needed)
├── EXAMPLES.md        # Usage examples (if needed)
└── scripts/           # Utility scripts (if needed)
    └── helper.js
```

## SKILL.md Template

```md
---
name: skill-name
description: Brief description of capability. Use when [specific triggers].
---

# Skill Name

## Quick start

[Minimal working example]

## Workflows

[Step-by-step processes with checklists for complex tasks]

## Advanced features

[Link to separate files: See [REFERENCE.md](REFERENCE.md)]
```

## Description Requirements

The description is **the only thing your agent sees** when deciding which skill to load. It's surfaced in the system prompt alongside all other installed skills. Your agent reads these descriptions and picks the relevant skill based on the user's request.

**Goal**: Give your agent just enough info to know:

1. What capability this skill provides
2. When/why to trigger it (specific keywords, contexts, file types)

**Format**:

- Max 1024 chars
- Write in third person
- First sentence: what it does
- Second sentence: "Use when [specific triggers]"

**Good example**:

```
Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when user mentions PDFs, forms, or document extraction.
```

**Bad example**:

```
Helps with documents.
```

The bad example gives your agent no way to distinguish this from other document skills.

## When to Add Scripts

Add utility scripts when:

- Operation is deterministic (validation, formatting)
- Same code would be generated repeatedly
- Errors need explicit handling

Scripts save tokens and improve reliability vs generated code.

## When to Split Files

Skills load in three tiers: the **description** is always in context (every skill, every session); the **SKILL.md body** loads when the skill triggers; **referenced files** load only when the agent follows the link. The goal is to push content down to the cheapest tier it can live in — keeping SKILL.md lean both saves tokens and *raises instruction adherence*, since the agent follows a tight, scannable body far more reliably than a long one.

Split into separate files when:

- SKILL.md exceeds ~100 lines
- Content has distinct domains (finance vs sales schemas)
- Advanced features are rarely needed

**But the line count is a proxy, not the goal — and splitting is only free for content the agent looks up *on demand*** (formats, schemas, examples, reference detail). The link-follow is triggered exactly when that content is needed, so it costs nothing until then.

It is **not** free for **core decision-making logic the agent must weigh on every pass** — behavioral rules, guardrails, judgment calls that shape how the skill acts continuously. If you exile those to a reference file, you're betting the agent reliably opens it every time; it won't, and a rule that doesn't load is a rule that doesn't fire. Such logic belongs in the body **even if it pushes past 100 lines.** When a skill is over the limit, cut elaboration and examples to a reference file first — not the rules.

## Review Checklist

After drafting, verify:

- [ ] Description includes triggers ("Use when...")
- [ ] SKILL.md lean (~100 lines) — over only if the excess is core behavioral logic, not reference detail
- [ ] No time-sensitive info
- [ ] Consistent terminology
- [ ] Concrete examples included
- [ ] References one level deep
