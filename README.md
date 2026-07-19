# Opencode Session Memory

> **极轻量 · 低消耗 · 零打扰** — 为 Opencode AI 提供持久化会话记忆的 Skill。

## 概述

Session Memory 是一个 Opencode Skill，让 AI 在跨会话间**自动记住项目上下文**。每次对话开始时静默加载 `_session_context.md`，结束时（或检测到实质进展时）实时更新，无需用户手动干预。

## 核心优势

| 特性 | 说明 |
|------|------|
| **极轻量** | 整个 skill 仅一个 `.md` 文件 + 一个 `.txt` 记录文件，零依赖零安装 |
| **低 token 消耗** | 只记录概要而非原始对话日志，每次加载仅数十行 markdown，对上下文窗口影响极小 |
| **零打扰** | 自动创建、自动更新、自动总结，**从不提问**，用户无感知 |
| **高可定制** | `_session_context.md` 完全在用户项目中，可自由增删改字段结构，扩展为项目 Wiki、决策日志等 |
| **Usage Tracking** | 自动记录所有使用过 opencode 的项目路径及最近活动日期 |

## 工作流程

```
┌─────────────────────────────────────┐
│  对话开始 → 加载 skill               │
│    ├─ _session_context.md 存在？      │
│    │   ├─ 是 → 读取，恢复上下文        │
│    │   └─ 否 → 自动创建（目录名命名）   │
│    └─ Usage Tracking 更新日期         │
├─────────────────────────────────────┤
│  对话中 → 检测实质进展                 │
│    ├─ 任务完成 / 设计决策              │
│    ├─ 方向变更 / Bug 修复             │
│    └─ 新待办出现 → 实时更新记忆文件    │
├─────────────────────────────────────┤
│  对话结束 → 静默等待下一次加载          │
└─────────────────────────────────────┘
```

## 安装

### 1. 放置 skill 文件

将 `session-memory` 文件夹放入 Opencode skills 目录（`~/.claude/skills/` 或对应配置路径），并在 `opencode.json` 中引用。

### 2. 配置自动加载（推荐）

将本仓库的 [`AGENTS.md`](./AGENTS.md) 复制到 `~/.config/opencode/AGENTS.md`（Windows 下为 `C:\Users\<用户名>\.config\opencode\AGENTS.md`）。  
这样每次对话开始时 AI **自动加载** session-memory skill，无需手动操作。

AGENTS.md 内容说明：
```
在每次对话的最开始，必须先加载 session-memory skill。
操作方式：使用 skill 工具，name 参数为 session-memory。
加载后，按顺序：检查 _session_context.md → 不存在则自动创建 → 更新使用记录
```

> AGENTS.md 是 opencode 的全局指令，优先级高于项目级配置，适合放"每次对话都必须做的事"。

## 配置项

在 skill 目录下创建 `.session-memory-config` 文件可自定义行为，**只需设置一次，不增加日常 token 消耗**：

```
usage_tracking_path = D:\自定义路径\opencode_usage.md
```

未配置时默认将 `opencode_usage.md` 存放在 skill 目录下。

## 自定义扩展

`_session_context.md` 的模板结构完全开放，你可以：

- 新增章节（如「技术栈」「接口文档」「已知问题」）
- 修改更新规则
- 集成到 CI/CD 流程中自动记录发布记录

## 许可证

MIT
