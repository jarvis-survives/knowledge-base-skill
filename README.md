# 📚 Knowledge Base Skill

**AI-powered knowledge management via git + markdown.**

Drop a single `SKILL.md` into your agent setup and let AI manage your team's knowledge base. Users chat naturally — the agent creates, finds, updates, and versions documents. Nobody needs to know git or markdown.

## How It Works

```
You:   "Create a decision record for switching to PostgreSQL"
Agent: ✅ Created decisions/0012-switch-to-postgres.md, committed and pushed.

You:   "What do we know about caching?"
Agent: Found 3 relevant documents:
       - concepts/caching-strategies.md (accepted)
       - decisions/0008-use-redis.md (accepted)
       - meetings/2026-02-10-performance-review.md

You:   "What changed this week?"
Agent: 4 documents updated:
       - New ADR proposed: switch to GraphQL (#0013)
       - Caching article updated with Redis Cluster section
       - 2 meeting notes added
```

## Quick Start

1. **Copy `SKILL.md`** into your agent's skill directory
2. **Point it at a git repo** — new or existing
3. **Start chatting** — the agent handles the rest

To initialize a fresh knowledge base, just ask: *"Set up a new knowledge base"*

## What's Included

| File | Purpose |
|------|---------|
| `SKILL.md` | Complete skill definition — all workflows, templates, and conventions |
| `README.md` | This file |
| `LICENSE` | MIT license |

## Repository Structure

The skill creates and manages this structure:

```
your-knowledge-base/
├── CLAUDE.md           ← Settings & rules (primary entry point)
├── .gitignore          ← Ignores local/generated files
├── concepts/           ← Knowledge articles
├── decisions/          ← Architecture Decision Records
├── projects/           ← Project logs
├── meetings/           ← Meeting notes
└── templates/          ← Document templates
```

> **Using a non-Claude agent?** Symlink or rename: `ln -s CLAUDE.md CONVENTIONS.md`

## What the Agent Does

| You say | Agent does |
|---------|-----------|
| "Document our auth approach" | Creates `concepts/authentication.md` with frontmatter, commits, pushes |
| "Find info about deployments" | 3-phase search: metadata → content → semantic expansion |
| "Update the caching guide" | Edits the file, commits with a meaningful message, pushes |
| "Show history of the API doc" | Runs `git log`, summarizes changes in plain language |
| "What changed since Monday?" | Reads recent commits, summarizes by category |
| "Propose an ADR for GraphQL" | Creates branch, writes ADR in review status, suggests MR |
| "Archive the old deploy guide" | Sets status to archived, adds archived_date, commits |
| "What's outdated?" | Finds accepted docs not updated in >6 months |
| "Migrate docs from ./old-docs/" | Bulk imports with auto-classification and frontmatter |

## Features

- **Automatic git workflow** — pull, commit, push with conventional commit messages
- **Document frontmatter** — YAML metadata with optional `id` for cross-referencing
- **Status lifecycle** — draft → review → accepted → archived
- **Smart search** — 3-phase: metadata scan → content grep → semantic/fuzzy matching
- **Cross-referencing** — relative links between documents, agent resolves "what relates to X?"
- **Multi-language** — set `language` in CLAUDE.md, or auto-detect from user
- **Conflict resolution** — agent reads both versions and merges semantically
- **Archive workflow** — archive outdated docs, find stale content
- **Bulk import** — migrate existing .md/.txt/.docx files with auto-classification
- **Templates** — included for all document types
- **.gitignore** — sensible defaults for local files

## Configuration

All settings live in `CLAUDE.md` at the repo root. Key options:

- **Language** — what language to write documents in
- **Default author** — used when no author specified
- **Tags** — recommended tag categories
- **Review process** — how ADRs flow through approval

## Compatibility

This skill is a plain markdown file. It works with any agent framework that reads skill/instruction files:

- [OpenClaw](https://github.com/AiClaw/OpenClaw)
- Claude Code / Claude with tool use
- Any LLM agent with file system + git access

## License

MIT — see [LICENSE](LICENSE).
