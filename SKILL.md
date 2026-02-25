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
- **Archive** outdated documents
- **Import** existing documents in bulk
- **Initialize** a new knowledge base repository

## Repository Structure

```
knowledge-base/
├── CLAUDE.md               # Rules & settings (primary entry point)
├── .gitignore              # Ignores local/generated files
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

> **Note:** The primary config file is `CLAUDE.md`. For non-Claude agents, symlink or rename it:
> `ln -s CLAUDE.md CONVENTIONS.md` or `cp CLAUDE.md CONVENTIONS.md`.

## Scaling: Subdirectories

The flat structure works for small teams (< ~100 docs per folder). When a folder grows beyond that, introduce **topic subdirectories**:

```
concepts/
├── security/
│   ├── authentication.md
│   ├── rate-limiting.md
│   └── cors-policy.md
├── infrastructure/
│   ├── deployment.md
│   ├── monitoring.md
│   └── ci-pipeline.md
├── hardware/
│   ├── thermal-management.md
│   └── cell-monitoring.md
└── getting-started.md          # Top-level docs still allowed
```

### When to restructure
- A folder has **>50 files** → group by topic
- Multiple teams contribute → group by **team or domain**
- Hard to find things → the structure isn't serving you anymore

### How to restructure
Ask the agent: *"Reorganize concepts/ by topic"*. The agent will:
1. Analyze all files by tags and content
2. Propose a grouping
3. After confirmation: move files, update all cross-references, commit

### Rules
- **Max 2 levels deep.** `concepts/security/auth.md` ✅ — `concepts/security/oauth/flows/pkce.md` ❌
- Subdirectory names: `kebab-case`
- The agent auto-detects the correct subdirectory from context, or asks if ambiguous
- Update `CLAUDE.md` when adding new subdirectories (so the agent knows the taxonomy)

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
id: caching-strategies      # optional, kebab-case, unique across the KB
title: "Document Title"
author: "Name or Agent"
date: 2026-02-24
status: draft                # draft | review | accepted | archived
tags: [topic, area]
---
```

The `id` field is optional but recommended for cross-referencing. Use kebab-case, keep it unique. Examples: `caching-strategies`, `adr-0012-use-postgres`, `sprint-2026-02-24`.

Status lifecycle: `draft` → `review` → `accepted` → `archived`

## Cross-Referencing

Documents reference each other with relative markdown links:

```markdown
See [Use Postgres](../decisions/0012-use-postgres.md) for the database decision.
Related: [Caching Strategies](../concepts/caching-strategies.md)
```

Conventions:
- Always use **relative paths** from the current file
- Place cross-references in a `## Related` section at the bottom, or inline
- When a user asks "what relates to X?", the agent should scan all documents for links to X, matching tags, and semantic relevance

## Workflows

### 1. Create Document

```
User: "Create a concept article about caching strategies"
```

1. `git pull`
2. Determine type → `concepts/`
3. Generate filename: `concepts/caching-strategies.md`
4. Write file with frontmatter (including `id`) + content
5. `git add concepts/caching-strategies.md`
6. `git commit -m "docs(concepts): add caching strategies"`
7. `git push`
8. Confirm to user with a summary

### 2. Find Document

```
User: "What do we know about authentication?"
```

1. `git pull`
2. **Phase 1 — Metadata scan:** Search titles and tags in frontmatter across all documents
3. **Phase 2 — Content grep:** `grep -ril "authentication" concepts/ decisions/ projects/ meetings/`
4. **Phase 3 — Fuzzy matching:** Also match related terms (e.g., "auth" matches "authentication", "authorization", "OAuth")
5. Deduplicate results, read matching files
6. Summarize findings for the user, cite sources with file paths

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
2. **Phase 1 — Frontmatter scan:** Extract `title` and `tags` from all documents, match against query
3. **Phase 2 — Content search:** `grep -ril "deploy" concepts/ decisions/ projects/ meetings/`
4. **Phase 3 — Semantic expansion:** Also search for related terms ("deploy" → "deployment", "CI/CD", "release", "rollout")
5. Read matching files, understand context
6. Return relevant excerpts with file references, ranked by relevance

### 8. Initialize Knowledge Base

```
User: "Set up a new knowledge base"
```

1. Create directory structure: `concepts/`, `decisions/`, `projects/`, `meetings/`, `templates/`
2. Write `CLAUDE.md` from template (see below)
3. Write `.gitignore` (see below)
4. Write template files (see below)
5. `git init && git add -A && git commit -m "docs: initialize knowledge base"`
6. If remote provided: `git remote add origin <url> && git push -u origin main`

> For non-Claude agents: `cp CLAUDE.md CONVENTIONS.md` or `ln -s CLAUDE.md CONVENTIONS.md`

### 9. Archive Document

```
User: "Archive the old deployment guide"
```

1. `git pull`
2. Find the document
3. Update frontmatter: set `status: archived`, add `archived_date: YYYY-MM-DD`
4. If superseded, add `superseded_by: path/to/new-doc.md` to frontmatter
5. `git add <file>`
6. `git commit -m "docs(<scope>): archive <title>"`
7. `git push`

```
User: "What's outdated?"
```

1. Scan all documents with `status: accepted`
2. Check `date` field and last git modification: `git log -1 --format=%ci <file>`
3. Flag documents not updated in >6 months
4. Present list with last-modified dates, suggest review or archival

### 10. Bulk Import

```
User: "Migrate existing docs from ./old-docs/"
```

1. Scan the source directory for `.md`, `.txt`, `.docx` files
2. For each file:
   - Convert to markdown if needed (`.docx` → use `pandoc` if available, else extract text)
   - Analyze content to determine type (concept, decision, project, meeting)
   - Generate proper filename following naming conventions
   - Add frontmatter (infer title from first heading, set `status: draft`, add date)
3. Place files in the correct folders
4. `git add -A && git commit -m "docs: bulk import from <source>"`
5. `git push`
6. Report: X files imported, broken down by type

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
docs: update CLAUDE.md
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

1. Check `CLAUDE.md` for a `language` setting
2. If set, write all documents in that language
3. If not set, match the language of the user's query
4. Frontmatter keys are always in English

## .gitignore Template

```
# Local/personal
CLAUDE.local.md
.claude/

# OS
.DS_Store
Thumbs.db

# Editor
*.swp
*.swo
*~

# Locks
*.lock
```

## CLAUDE.md Template

When initializing a knowledge base, create this file at the repo root:

````markdown
# Knowledge Base Conventions

> This is the primary configuration file. For non-Claude agents, symlink or rename:
> `ln -s CLAUDE.md CONVENTIONS.md`

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

## Cross-Referencing

Link between documents using relative paths:
```markdown
See [Use Postgres](decisions/0012-use-postgres.md) for context.
```

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
id: 
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
id: 
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
id: 
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
id: 
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

This skill works naturally with Claude Code's auto-memory system. `CLAUDE.md` is loaded automatically at session start — no extra configuration needed.

### Recommended setup

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Project rules, structure, naming — loaded every session |
| `.claude/rules/*.md` | Modular topic rules (e.g., `review-process.md`, `adr-format.md`) |
| `CLAUDE.local.md` | Personal preferences, not committed (in `.gitignore`) |

For non-Claude agents, symlink: `ln -s CLAUDE.md CONVENTIONS.md`

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
- When searching, use the 3-phase approach: metadata → content → semantic expansion
- Keep summaries concise but link to source files
- If the user asks something vague, search broadly and present options
- Auto-detect document type from context (e.g., "meeting notes" → `meetings/`)
- For ADRs, always check the latest number and increment
- Consider splitting complex conventions into `.claude/rules/*.md` for modularity
- For "what relates to X?" queries, check cross-references, tags, and content mentions
