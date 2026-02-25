# Knowledge Base Skill

> AI-powered knowledge management via git + markdown.

## Description

This skill turns an AI agent into a knowledge base manager. Users chat naturally — the agent creates, finds, updates, and versions markdown documents in a git repository. No one needs to know git or markdown.

## Triggers

Activate this skill when the user wants to:

- **Create** a document, article, decision record, meeting notes, or project log
- **Find** information ("What do we know about X?", "Find the ADR for Y")
- **Update** existing knowledge ("Update the deployment guide", "Add a section about Z")
- **Summarize** recent changes ("What changed this week?", "What's new?")
- **Show history** of a document ("Who changed X?", "What's the history of Y?")
- **Review** a document (branching workflow)
- **Search** across the knowledge base
- **Initialize** a new knowledge base repository

## Repository Structure

```
knowledge-base/
├── CONVENTIONS.md          # Rules & settings (the heart of the system)
├── concepts/               # Knowledge articles, technical concepts, theory
│   └── kebab-case-name.md
├── decisions/              # Architecture Decision Records (ADRs)
│   └── NNNN-title.md
├── projects/               # Project logs, dated
│   └── YYYY-MM-DD-name.md
├── meetings/               # Meeting notes
│   └── YYYY-MM-DD-topic.md
└── templates/              # Document templates
    ├── concept.md
    ├── decision.md
    ├── project.md
    └── meeting.md
```

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Concept | `kebab-case.md` | `concepts/rate-limiting.md` |
| Decision | `NNNN-title.md` | `decisions/0012-use-postgres.md` |
| Project | `YYYY-MM-DD-name.md` | `projects/2026-02-24-api-redesign.md` |
| Meeting | `YYYY-MM-DD-topic.md` | `meetings/2026-02-24-sprint-planning.md` |

For decisions, auto-increment the number by checking the highest existing `NNNN-*.md` in `decisions/`.

## Document Frontmatter

Every document starts with YAML frontmatter:

```yaml
---
title: "Document Title"
author: "Name or Agent"
date: 2026-02-24
status: draft          # draft | review | accepted | archived
tags: [topic, area]
---
```

Status lifecycle: `draft` → `review` → `accepted` → `archived`

## Workflows

### 1. Create Document

```
User: "Create a concept article about caching strategies"
```

1. `git pull`
2. Determine type → `concepts/`
3. Generate filename: `concepts/caching-strategies.md`
4. Write file with frontmatter + content
5. `git add concepts/caching-strategies.md`
6. `git commit -m "docs(concepts): add caching strategies"`
7. `git push`
8. Confirm to user with a summary

### 2. Find Document

```
User: "What do we know about authentication?"
```

1. `git pull`
2. Search: `grep -ril "authentication" concepts/ decisions/ projects/ meetings/`
3. Also check frontmatter tags and titles
4. Read matching files
5. Summarize findings for the user, cite sources

### 3. Update Document

```
User: "Update the caching article to mention Redis"
```

1. `git pull`
2. Find the file: `concepts/caching-strategies.md`
3. Read current content
4. Apply the edit
5. `git add concepts/caching-strategies.md`
6. `git commit -m "docs(concepts): add Redis section to caching strategies"`
7. `git push`
8. Show what changed

### 4. Show History

```
User: "What's the history of the caching article?"
```

1. `git log --oneline concepts/caching-strategies.md`
2. For detail: `git log -p -3 concepts/caching-strategies.md`
3. Summarize changes in plain language

### 5. Summarize Recent Changes

```
User: "What changed this week?"
```

1. `git log --since="7 days ago" --name-status --oneline`
2. Read changed files
3. Produce a human-friendly summary grouped by category

### 6. Review Process

```
User: "I want to propose a new ADR for switching to GraphQL"
```

1. `git pull`
2. Create branch: `git checkout -b docs/NNNN-switch-to-graphql`
3. Write the decision document with status `review`
4. `git add decisions/NNNN-switch-to-graphql.md`
5. `git commit -m "docs(decisions): propose switching to GraphQL"`
6. `git push -u origin docs/NNNN-switch-to-graphql`
7. Suggest creating a merge/pull request if applicable

### 7. Search

```
User: "Search for everything about deployment"
```

1. `git pull`
2. `grep -ril "deploy" concepts/ decisions/ projects/ meetings/`
3. Read matching files, understand context
4. Return relevant excerpts with file references

### 8. Initialize Knowledge Base

```
User: "Set up a new knowledge base"
```

1. Create directory structure: `concepts/`, `decisions/`, `projects/`, `meetings/`, `templates/`
2. Write `CONVENTIONS.md` from template (see below)
3. Write template files (see below)
4. `git init && git add -A && git commit -m "docs: initialize knowledge base"`
5. If remote provided: `git remote add origin <url> && git push -u origin main`

## Git Conventions

### Before any edit
```bash
git pull --rebase
```

### Commit messages
Use conventional commits with `docs` type:
```
docs(concepts): add caching strategies
docs(decisions): propose switching to GraphQL
docs(meetings): add sprint planning notes 2026-02-24
docs(projects): update API redesign progress
docs: update CONVENTIONS.md
```

### After every meaningful change
```bash
git push
```

### Merge conflicts
If `git pull` produces a conflict:
1. Read both versions (`<<<<<<<` markers)
2. Merge semantically — combine knowledge from both sides
3. Remove conflict markers
4. `git add <file> && git commit -m "docs: resolve merge conflict in <file>"`
5. `git push`

## Language Handling

1. Check `CONVENTIONS.md` for a `language` setting
2. If set, write all documents in that language
3. If not set, match the language of the user's query
4. Frontmatter keys are always in English

## CONVENTIONS.md Template

When initializing a knowledge base, create this file at the repo root:

````markdown
# Knowledge Base Conventions

## General

- **Language:** en  <!-- Language for all documents (e.g., en, de, fr, es) -->
- **Default author:** Team  <!-- Used when no author specified -->

## Document Status Lifecycle

1. **draft** — Work in progress
2. **review** — Ready for team review
3. **accepted** — Approved and current
4. **archived** — Outdated or superseded

## Structure

| Folder | Purpose | Naming |
|--------|---------|--------|
| `concepts/` | Knowledge articles, theory | `kebab-case.md` |
| `decisions/` | Architecture Decision Records | `NNNN-title.md` |
| `projects/` | Project logs | `YYYY-MM-DD-name.md` |
| `meetings/` | Meeting notes | `YYYY-MM-DD-topic.md` |
| `templates/` | Document templates | `type.md` |

## Tags

Use lowercase tags. Recommended categories:
- **Area:** `backend`, `frontend`, `infra`, `security`, `data`
- **Type:** `guide`, `reference`, `tutorial`, `proposal`
- **Priority:** `critical`, `important`, `nice-to-have`

## Commit Messages

Format: `docs(scope): description`

Scopes: `concepts`, `decisions`, `meetings`, `projects`, or omit for root files.

## Review Process

- Decisions (ADRs) start as `draft`, move to `review` for team discussion
- Use branches for proposals: `docs/NNNN-short-title`
- Accepted decisions should not be modified — create a new ADR that supersedes
````

## Templates

### concepts/template → `templates/concept.md`

```markdown
---
title: "Concept Title"
author: ""
date: YYYY-MM-DD
status: draft
tags: []
---

# Concept Title

## Summary

Brief overview of this concept.

## Detail

In-depth explanation.

## Related

- Links to related concepts or decisions
```

### decisions/template → `templates/decision.md`

```markdown
---
title: "ADR NNNN: Decision Title"
author: ""
date: YYYY-MM-DD
status: draft
tags: []
---

# ADR NNNN: Decision Title

## Context

What is the issue we're facing?

## Decision

What did we decide?

## Consequences

What are the trade-offs?

## Alternatives Considered

What other options were evaluated?
```

### projects/template → `templates/project.md`

```markdown
---
title: "Project Name"
author: ""
date: YYYY-MM-DD
status: draft
tags: []
---

# Project Name

## Goal

What are we trying to achieve?

## Progress

### YYYY-MM-DD
- Initial entry

## Open Questions

- 

## Links

- 
```

### meetings/template → `templates/meeting.md`

```markdown
---
title: "Meeting: Topic"
author: ""
date: YYYY-MM-DD
status: accepted
tags: []
---

# Meeting: Topic

**Date:** YYYY-MM-DD  
**Attendees:** 

## Agenda

1. 

## Notes

- 

## Action Items

- [ ] 
```

## Claude Code Auto-Memory Integration

This skill works naturally with [Claude Code's auto-memory](https://code.claude.com/docs/en/memory#auto-memory) system. Claude Code has two kinds of persistent memory:

- **Auto memory** (`~/.claude/projects/<project>/memory/`) — Claude automatically saves project patterns, debugging insights, and architecture notes. The first 200 lines of `MEMORY.md` are loaded at session start.
- **CLAUDE.md files** — Loaded hierarchically at session start. The `CONVENTIONS.md` in this skill serves as the project-level `CLAUDE.md`.

### Recommended setup with Claude Code

| File | Purpose |
|------|---------|
| `CLAUDE.md` or `CONVENTIONS.md` | Project rules, structure, naming — loaded every session |
| `.claude/rules/*.md` | Modular topic rules (e.g., `review-process.md`, `adr-format.md`) |
| `CLAUDE.local.md` | Personal preferences, not committed (auto-added to `.gitignore`) |

**Tip:** Rename `CONVENTIONS.md` to `CLAUDE.md` when using Claude Code directly — it loads automatically. For other agents, keep `CONVENTIONS.md` (more universal). You can also symlink: `ln -s CONVENTIONS.md CLAUDE.md`.

### What auto-memory learns over time

Claude Code automatically remembers:
- Which folders your team uses most
- Common search patterns and queries
- Project-specific terminology and acronyms
- Preferred document structures beyond the templates
- Team members and their areas of expertise

This means the knowledge base gets smarter with use — no configuration needed.

## Tips

- When creating documents, fill in as much content as possible from the user's input
- When searching, also look at tags and titles — not just body text
- Keep summaries concise but link to source files
- If the user asks something vague, search broadly and present options
- Auto-detect document type from context (e.g., "meeting notes" → `meetings/`)
- For ADRs, always check the latest number and increment
- Consider splitting complex conventions into `.claude/rules/*.md` for modularity
