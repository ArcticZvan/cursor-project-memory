# Cursor Project Memory

一个基于文件的 [Cursor](https://cursor.com/) AI Agent 记忆层。自动将问题、解决方案、优化和决策记录到按日期分隔的 Markdown 文件中，让每次新对话都能延续上一次的上下文。

## 为什么需要它

Cursor Agent 在不同会话之间会丢失上下文。这个 Skill 为它提供了持久化的、人类可读的记忆，直接存放在你的项目里：

- **按日记录** — 每天一个 Markdown 文件（`2026-04-19.md`、`2026-04-20.md`……）
- **项目概览** — 持续更新的项目摘要，包含架构、关键决策和已总结的经验模式
- **纠错追踪** — 当之前的方案被发现有误时，Agent 会标记错误并链接到正确的修复
- **零依赖** — 纯文件存储在 `.cursor/memory/`，无需数据库、API 或云服务

## 工作原理

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

### 记忆条目示例

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

### 纠错流程

当发现之前的修复方案有问题时：

1. 原条目状态被标记为：`**Status**: ❌ incorrect → see [[2026-04-20#真正的修复]]`
2. 在今天的文件中添加一条 `correction` 类型的条目，说明真正的根因和正确做法

## 安装

### 方式 A：个人 Skill（所有项目通用）

```bash
mkdir -p ~/.cursor/skills/project-memory
cp SKILL.md SCHEMA.md ~/.cursor/skills/project-memory/
```

### 方式 B：项目级 Skill（仅当前项目）

```bash
mkdir -p .cursor/skills/project-memory
cp SKILL.md SCHEMA.md .cursor/skills/project-memory/
```

安装完成。下次在 Cursor 中开启对话，Agent 会自动发现并使用这个 Skill。

## 文件结构

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

## 条目类型

| 类型 | 使用场景 |
|------|---------|
| `problem` | 发现 bug 或问题 |
| `solution` | 修复并验证通过 |
| `optimization` | 性能/质量优化，包含前后对比指标 |
| `decision` | 做出架构或工具选型决策 |
| `correction` | 发现之前的方案有误，记录纠正并链接原条目 |

## 状态标记

| 标记 | 含义 |
|------|------|
| ✅ `verified` | 已确认有效 |
| ⚠️ `superseded` | 已有更好的方案（附链接） |
| ❌ `incorrect` | 方案有误（附链接指向纠正条目） |

## Gitignore

如果你希望记忆只保留在本地、不提交到 git：

```bash
echo ".cursor/memory/" >> .gitignore
```

## 许可证

[MIT](LICENSE)
