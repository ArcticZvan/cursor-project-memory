# Cursor Project Memory

**[English](#english) | [中文](#中文)**

---

<a id="english"></a>

## English

A file-based memory layer for [Cursor](https://cursor.com/) AI agents. It automatically records problems, solutions, optimizations, and decisions into daily Markdown files — so every new conversation picks up where the last one left off.

### Why

Cursor agents lose context between sessions. This skill gives them persistent, human-readable memory that lives inside your project:

- **Daily logs** — one Markdown file per day (`2026-04-19.md`, `2026-04-20.md`, ...)
- **Project overview** — an evolving summary of architecture, key decisions, and learned patterns
- **Correction tracking** — when a past solution turns out to be wrong, the agent marks it and links to the fix
- **Zero infrastructure** — plain files in `.cursor/memory/`, no database, no API, no cloud

### How It Works

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

#### Memory Entry Example

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

#### Correction Flow

When a previous fix is found to be wrong:

1. The original entry gets marked: `**Status**: ❌ incorrect → see [[2026-04-20#real-fix]]`
2. A new `correction` entry is added in today's file explaining the actual root cause

### Installation

#### Option A: Personal skill (all projects)

```bash
mkdir -p ~/.cursor/skills/project-memory
cp SKILL.md SCHEMA.md ~/.cursor/skills/project-memory/
```

#### Option B: Per-project skill

```bash
mkdir -p .cursor/skills/project-memory
cp SKILL.md SCHEMA.md .cursor/skills/project-memory/
```

That's it. The next time you open a Cursor conversation, the agent will discover the skill automatically.

### File Structure

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

### Entry Types

| Type | When |
|------|------|
| `problem` | A bug or issue is identified |
| `solution` | A fix is applied and verified |
| `optimization` | Performance or quality improvement with before/after metrics |
| `decision` | An architectural or tooling choice is made |
| `correction` | A past entry is found to be wrong — links to the original |

### Status Markers

| Marker | Meaning |
|--------|---------|
| ✅ `verified` | Confirmed working |
| ⚠️ `superseded` | A better approach exists (linked) |
| ❌ `incorrect` | This was wrong (linked to correction) |

### Gitignore

If you want memory to stay local and not be committed:

```bash
echo ".cursor/memory/" >> .gitignore
```

---

<a id="中文"></a>

## 中文

一个基于文件的 [Cursor](https://cursor.com/) AI Agent 记忆层。自动将问题、解决方案、优化和决策记录到按日期分隔的 Markdown 文件中，让每次新对话都能延续上一次的上下文。

### 为什么需要它

Cursor Agent 在不同会话之间会丢失上下文。这个 Skill 为它提供了持久化的、人类可读的记忆，直接存放在你的项目里：

- **按日记录** — 每天一个 Markdown 文件（`2026-04-19.md`、`2026-04-20.md`……）
- **项目概览** — 持续更新的项目摘要，包含架构、关键决策和已总结的经验模式
- **纠错追踪** — 当之前的方案被发现有误时，Agent 会标记错误并链接到正确的修复
- **零依赖** — 纯文件存储在 `.cursor/memory/`，无需数据库、API 或云服务

### 工作原理

```
会话开始                                任务完成
  │                                       │
  ▼                                       ▼
读取今天和                            追加条目到
昨天的日志                            今天的日期文件
  │                                       │
  ▼                                       ▼
作为内部上下文                         如果发现新决策/模式
静默使用                              更新 project-overview.md
```

#### 记忆条目示例

```markdown
---

## [14:30] 修复: Redis 连接池超时

**Type**: solution
**Tags**: `bug`, `performance`
**Related**: [[2026-04-19#redis-连接超时]]

添加了 `retry_on_timeout=True` 并将 `max_connections` 增加到 20。
高并发场景下不再出现 ConnectionError。

**Status**: ✅ verified
```

#### 纠错流程

当发现之前的修复方案有问题时：

1. 原条目状态被标记为：`**Status**: ❌ incorrect → see [[2026-04-20#真正的修复]]`
2. 在今天的文件中添加一条 `correction` 类型的条目，说明真正的根因和正确做法

### 安装

#### 方式 A：个人 Skill（所有项目通用）

```bash
mkdir -p ~/.cursor/skills/project-memory
cp SKILL.md SCHEMA.md ~/.cursor/skills/project-memory/
```

#### 方式 B：项目级 Skill（仅当前项目）

```bash
mkdir -p .cursor/skills/project-memory
cp SKILL.md SCHEMA.md .cursor/skills/project-memory/
```

安装完成。下次在 Cursor 中开启对话，Agent 会自动发现并使用这个 Skill。

### 文件结构

```
~/.cursor/skills/project-memory/   # ← Skill 文件（给 Agent 的指令）
├── SKILL.md                       # 主指令文件
└── SCHEMA.md                      # 条目模板和标签规范

你的项目/
└── .cursor/memory/                # ← 记忆文件（每个项目独立）
    ├── project-overview.md        # 持久化项目概览
    ├── 2026-04-19.md              # 每日记录
    ├── 2026-04-20.md
    └── ...
```

### 条目类型

| 类型 | 使用场景 |
|------|---------|
| `problem` | 发现 bug 或问题 |
| `solution` | 修复并验证通过 |
| `optimization` | 性能/质量优化，包含前后对比指标 |
| `decision` | 做出架构或工具选型决策 |
| `correction` | 发现之前的方案有误，记录纠正并链接原条目 |

### 状态标记

| 标记 | 含义 |
|------|------|
| ✅ `verified` | 已确认有效 |
| ⚠️ `superseded` | 已有更好的方案（附链接） |
| ❌ `incorrect` | 方案有误（附链接指向纠正条目） |

### Gitignore

如果你希望记忆只保留在本地、不提交到 git：

```bash
echo ".cursor/memory/" >> .gitignore
```

---

## License

[MIT](LICENSE)
