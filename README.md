# Project Memory for AI Agents

**[English](#english) | [中文](#中文)**

---

<a id="english"></a>

## English

A portable, file-based memory layer for AI coding agents. It records problems,
solutions, optimizations, and decisions into daily Markdown files so new
conversations can pick up useful project context without requiring a database,
API, or cloud service.

### Why

AI agents lose context between sessions, devices, and tools. This skill gives
them persistent, human-readable memory that lives inside your project:

- **Daily logs** — one Markdown file per day (`2026-04-19.md`, `2026-04-20.md`, ...)
- **Project overview** — an evolving summary of architecture, key decisions, and learned patterns
- **Correction tracking** — when a past solution turns out to be wrong, the agent marks it and links to the fix
- **Date calibration** — agents use the machine's current date/time before reading or writing, avoiding stale conversation timestamps
- **Zero infrastructure** — plain files in `.cursor/memory/` by default, no database, no API, no cloud

The default memory directory is `.cursor/memory/` because this repository ships
as a Cursor skill, but the protocol itself is tool-agnostic. Other agents can
reuse the same `SKILL.md` and `SCHEMA.md` instructions and keep the same folder,
or adapt the memory directory to their own project convention.

### How It Works

```
Session Start                          Task Complete
     │                                       │
     ▼                                       ▼
 Calibrate today's date             Append entry to
 and read recent logs               today's daily file
     │                                       │
     ▼                                       ▼
 Use as internal                    Re-check clock and update
 context silently                   project-overview.md if needed
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

### Installation / Usage

#### Cursor skill: personal install (all projects)

macOS / Linux:

```bash
mkdir -p ~/.cursor/skills/project-memory
cp SKILL.md SCHEMA.md ~/.cursor/skills/project-memory/
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force "$HOME\.cursor\skills\project-memory"; Copy-Item SKILL.md, SCHEMA.md "$HOME\.cursor\skills\project-memory\"
```

#### Cursor skill: per-project install

macOS / Linux:

```bash
mkdir -p .cursor/skills/project-memory
cp SKILL.md SCHEMA.md .cursor/skills/project-memory/
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force ".cursor\skills\project-memory"; Copy-Item SKILL.md, SCHEMA.md ".cursor\skills\project-memory\"
```

#### Other AI agents

Add the contents of `SKILL.md` to the agent's custom instructions, skill system,
repository guidance file, or equivalent memory/tooling mechanism. Keep
`SCHEMA.md` available beside it so the agent can follow the templates.

The protocol needs only three capabilities:

- Read Markdown files from the project.
- Run a local date command such as `Get-Date -Format "yyyy-MM-dd"` or `date +%Y-%m-%d`.
- Write Markdown files when a task produces a meaningful memory entry.

### File Structure

```
agent-instructions/project-memory/ # The skill/protocol instructions
├── SKILL.md                       # Main skill file — agent reads this
└── SCHEMA.md                      # Entry templates & tag conventions

your-project/
└── .cursor/memory/                # Default project memory location
    ├── project-overview.md        # Persistent project summary
    ├── 2026-04-19.md              # Daily log
    ├── 2026-04-20.md
    └── ...
```

### Entry Types

- `problem` — a bug or issue is identified
- `solution` — a fix is applied and verified
- `optimization` — performance or quality improves, ideally with before/after metrics
- `decision` — an architectural or tooling choice is made
- `correction` — a past entry is found to be wrong and linked to the correction

### Status Markers

- ✅ `verified` — confirmed working
- ⚠️ `superseded` — a better approach exists and is linked
- ❌ `incorrect` — the entry was wrong and links to a correction

### Gitignore

If you want memory to stay local and not be committed:

```bash
echo ".cursor/memory/" >> .gitignore
```

---

<a id="中文"></a>

## 中文

一个可移植的、基于文件的 AI 编程 Agent 记忆层。它会把问题、解决方案、优化和决策记录到按日期分隔的 Markdown 文件中，让新的对话能延续项目上下文，同时不依赖数据库、API 或云服务。

### 为什么需要它

AI Agent 在不同会话、设备和工具之间都会丢失上下文。这个 Skill 为它们提供持久化的、人类可读的记忆，直接存放在你的项目里：

- **按日记录** — 每天一个 Markdown 文件（`2026-04-19.md`、`2026-04-20.md`……）
- **项目概览** — 持续更新的项目摘要，包含架构、关键决策和已总结的经验模式
- **纠错追踪** — 当之前的方案被发现有误时，Agent 会标记错误并链接到正确的修复
- **日期校准** — 读写前使用本机当前日期/时间，避免长会话里的过期时间戳
- **零依赖** — 默认纯文件存储在 `.cursor/memory/`，无需数据库、API 或云服务

默认目录仍是 `.cursor/memory/`，因为这个仓库可以作为 Cursor Skill 使用；但协议本身并不绑定 Cursor。其他 AI Agent 可以复用 `SKILL.md` 和 `SCHEMA.md` 的指令，也可以按自己的项目规范调整记忆目录。

### 工作原理

```
会话开始                                任务完成
  │                                       │
  ▼                                       ▼
校准今天的日期                         追加条目到
并读取近期日志                         今天的日期文件
  │                                       │
  ▼                                       ▼
作为内部上下文                         写入前重新校验时间
静默使用                              必要时更新项目概览
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

### 安装 / 使用

#### Cursor Skill：个人安装（所有项目通用）

macOS / Linux：

```bash
mkdir -p ~/.cursor/skills/project-memory
cp SKILL.md SCHEMA.md ~/.cursor/skills/project-memory/
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force "$HOME\.cursor\skills\project-memory"; Copy-Item SKILL.md, SCHEMA.md "$HOME\.cursor\skills\project-memory\"
```

#### Cursor Skill：项目级安装（仅当前项目）

macOS / Linux：

```bash
mkdir -p .cursor/skills/project-memory
cp SKILL.md SCHEMA.md .cursor/skills/project-memory/
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force ".cursor\skills\project-memory"; Copy-Item SKILL.md, SCHEMA.md ".cursor\skills\project-memory\"
```

#### 其他 AI Agent

把 `SKILL.md` 的内容加入该 Agent 的自定义指令、Skill 系统、仓库指导文件或等价机制中，并让 `SCHEMA.md` 保持可读，方便 Agent 使用模板。

这个协议只要求 Agent 具备三种能力：

- 读取项目中的 Markdown 文件。
- 执行本地日期命令，例如 `Get-Date -Format "yyyy-MM-dd"` 或 `date +%Y-%m-%d`。
- 在任务产生有价值的记忆时写入 Markdown 文件。

### 文件结构

```
agent-instructions/project-memory/ # Skill/协议指令
├── SKILL.md                       # 主指令文件
└── SCHEMA.md                      # 条目模板和标签规范

你的项目/
└── .cursor/memory/                # 默认项目记忆目录
    ├── project-overview.md        # 持久化项目概览
    ├── 2026-04-19.md              # 每日记录
    ├── 2026-04-20.md
    └── ...
```

### 条目类型

- `problem` — 发现 bug 或问题
- `solution` — 修复并验证通过
- `optimization` — 性能或质量优化，最好包含前后对比指标
- `decision` — 做出架构或工具选型决策
- `correction` — 发现之前的方案有误，记录纠正并链接原条目

### 状态标记

- ✅ `verified` — 已确认有效
- ⚠️ `superseded` — 已有更好的方案，并链接到新条目
- ❌ `incorrect` — 方案有误，并链接到纠正条目

### Gitignore

如果你希望记忆只保留在本地、不提交到 git：

```bash
echo ".cursor/memory/" >> .gitignore
```

---

## License

[MIT](LICENSE)
