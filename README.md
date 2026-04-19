# Cursor Project Memory

[中文文档](README_CN.md)

A file-based memory layer for [Cursor](https://cursor.com/) AI agents. It automatically records problems, solutions, optimizations, and decisions into daily Markdown files — so every new conversation picks up where the last one left off.

## Why

Cursor agents lose context between sessions. This skill gives them persistent, human-readable memory that lives inside your project:

- **Daily logs** — one Markdown file per day (`2026-04-19.md`, `2026-04-20.md`, ...)
- **Project overview** — an evolving summary of architecture, key decisions, and learned patterns
- **Correction tracking** — when a past solution turns out to be wrong, the agent marks it and links to the fix
- **Zero infrastructure** — plain files in `.cursor/memory/`, no database, no API, no cloud

## How It Works

```
Session Start                          Task Complete
     │                                       │
     ▼                                       ▼
 Read today's &                     Append entry to
 yesterday's logs                   today's daily file
     │                                       │
     ▼                                       ▼
 Use as internal                    Update project-overview.md
 context silently                   if pattern/decision found
```

### Memory Entry Example

```markdown
---

## [14:30] Fix: Redis connection pool timeout

**Type**: solution
**Tags**: `bug`, `performance`
**Related**: [[2026-04-19#redis-connection-timeout]]

Added `retry_on_timeout=True` and increased `max_connections` to 20.
High-concurrency ConnectionError no longer occurs.

**Status**: ✅ verified
```

### Correction Flow

When a previous fix is found to be wrong:

1. The original entry gets marked: `**Status**: ❌ incorrect → see [[2026-04-20#real-fix]]`
2. A new `correction` entry is added in today's file explaining the actual root cause

## Installation

### Option A: Personal skill (all projects)

```bash
mkdir -p ~/.cursor/skills/project-memory
cp SKILL.md SCHEMA.md ~/.cursor/skills/project-memory/
```

### Option B: Per-project skill

```bash
mkdir -p .cursor/skills/project-memory
cp SKILL.md SCHEMA.md .cursor/skills/project-memory/
```

That's it. The next time you open a Cursor conversation, the agent will discover the skill automatically.

## File Structure

```
~/.cursor/skills/project-memory/   # ← The skill (instructions for the agent)
├── SKILL.md                       # Main skill file — agent reads this
└── SCHEMA.md                      # Entry templates & tag conventions

your-project/
└── .cursor/memory/                # ← The memory (created per project)
    ├── project-overview.md        # Persistent project summary
    ├── 2026-04-19.md              # Daily log
    ├── 2026-04-20.md
    └── ...
```

## Entry Types

| Type | When |
|------|------|
| `problem` | A bug or issue is identified |
| `solution` | A fix is applied and verified |
| `optimization` | Performance or quality improvement with before/after metrics |
| `decision` | An architectural or tooling choice is made |
| `correction` | A past entry is found to be wrong — links to the original |

## Status Markers

| Marker | Meaning |
|--------|---------|
| ✅ `verified` | Confirmed working |
| ⚠️ `superseded` | A better approach exists (linked) |
| ❌ `incorrect` | This was wrong (linked to correction) |

## Gitignore

If you want memory to stay local and not be committed:

```bash
echo ".cursor/memory/" >> .gitignore
```

## License

[MIT](LICENSE)
