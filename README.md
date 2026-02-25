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

## Repository Structure

```
your-knowledge-base/
├── CONVENTIONS.md      ← Settings & rules
├── concepts/           ← Knowledge articles
├── decisions/          ← Architecture Decision Records
├── projects/           ← Project logs
├── meetings/           ← Meeting notes
└── templates/          ← Document templates
```

## What the Agent Does

| You say | Agent does |
|---------|-----------|
| "Document our auth approach" | Creates `concepts/authentication.md` with frontmatter, commits, pushes |
| "Find info about deployments" | Searches files, reads matches, summarizes findings |
| "Update the caching guide" | Edits the file, commits with a meaningful message, pushes |
| "Show history of the API doc" | Runs `git log`, summarizes changes in plain language |
| "What changed since Monday?" | Reads recent commits, summarizes by category |
| "Propose an ADR for GraphQL" | Creates branch, writes ADR in review status, suggests MR |

## Features

- **Automatic git workflow** — pull, commit, push with conventional commit messages
- **Document frontmatter** — YAML metadata (title, author, date, status, tags)
- **Status lifecycle** — draft → review → accepted → archived
- **Smart search** — searches content, titles, and tags
- **Multi-language** — set `language` in CONVENTIONS.md, or auto-detect from user
- **Conflict resolution** — agent reads both versions and merges semantically
- **Templates** — included for all document types

## Configuration

All settings live in `CONVENTIONS.md` at the repo root. Key options:

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

MIT — use it however you want.
