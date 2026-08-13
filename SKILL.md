---
name: session-memory
description: >-
  Run at the START of every conversation. Check if _session_context.md exists
  in the project root. If yes: read it and restore context. If no: auto-create
  it using the directory name as the project name — no questions asked.
  Also read the global pitfalls knowledge base (opencode_knowledge.md) and
  restore any recorded workflow for this project.
  This skill is ALWAYS the first thing you do in any project.
---

# Session Memory — Automatic Context Restoration

## Trigger (MANDATORY — at conversation start)

This skill **must be loaded first** in ANY project. After loading:

1. **Check** if `_session_context.md` exists at the project root
2. **If it EXISTS**: read the full file, then respond with a short confirmation that context has been restored. **Do NOT ask any questions.**
3. **If it does NOT exist**: auto-create `_session_context.md` using the current working directory's folder name as the project name. **Do NOT ask for the name. Do NOT ask questions.**
4. **ALWAYS** — regardless of whether `_session_context.md` existed or was just created — **update the usage tracking file**. Follow the "Usage Tracking" section below to determine the file path and update it. This step is **non-optional** and runs every session start.
5. **Read the global knowledge base** — read `opencode_knowledge.md` in full (path from `global_knowledge_path` in `.session-memory-config`, default `{skill_dir}/opencode_knowledge.md`). If it does not exist, create it with the header only. This step is **non-optional** and runs every session start.

## Template (used when auto-creating)

```markdown
# {ProjectName} — 会话记忆
<!-- {ProjectName} is auto-replaced with directory name; leave no placeholder for the user to fill -->

## 项目概况
<!-- AI auto-fills after first conversation -->

## 工作流
<!-- AI records the project's fixed workflow: structure map + production steps. Usually a few lines, up to ten. Overwrite on change. -->

## 已完成的工作
<!-- AI updates in real-time when substantial progress is detected -->

## 待办/待讨论
<!-- AI updates in real-time when substantial progress is detected -->
```

## Auto Project Summary

When the AI detects substantial progress in a new project (first meaningful work session):

1. Deduce the project's main purpose, tech stack, and goals from the conversation and codebase
2. Fill the **项目概况** section in `_session_context.md` with a concise summary

After the initial fill, the AI **autonomously decides** whether to update the project summary — based on whether the project's scope, direction, or tech stack has significantly shifted. No fixed schedule, no redundant updates.

## Usage Tracking

Records every project that has used Opencode with this skill, along with the last active date, in Markdown format.

### Configurable storage path

The tracking file location can be customized via `.session-memory-config`:

1. Look for `.session-memory-config` in the **skill directory** (same folder as SKILL.md)
2. **If found**: read `usage_tracking_path = <path>` from it
3. **If not found**: default to `{skill_dir}/opencode_usage.md`

### Update logic (runs each session start)

At the start of each session, **before responding to the user**:

1. Determine the tracking file path using the config method above
2. **If the file doesn't exist**: create it with the Markdown table header
3. **If it exists** or after creating the header: **append** a new row with the current project's root path and today's date

**Crucially: never modify, delete, or update any existing row.** This is an append-only log.

### Example — chronological activity log

After multiple sessions working across projects, the file grows like this:

```markdown
# Opencode 使用记录

| 项目路径 | 日期 |
|----------|------|
| D:\ProjectA | 2026-07-01 |
| D:\ProjectB | 2026-07-02 |
| D:\ProjectA | 2026-07-05 |
| D:\ProjectC | 2026-07-06 |
| D:\ProjectB | 2026-07-07 |
```

Each row is a distinct session — the same project can (and will) appear multiple times with different dates.

### Record format

Markdown table with pipe-separated columns:

```markdown
# Opencode 使用记录

| 项目路径 | 最后活动日期 |
|----------|------------|
| D:\MyProject | 2026-07-18 |
```

### Migration from old .txt format

If `opencode_usage.txt` exists but `opencode_usage.md` does not:
- Read the `.txt` file
- Migrate its content to the new Markdown table format
- Write to `.md` path
- Delete or ignore the old `.txt` file

## Global Knowledge Base (Pitfalls)

A cross-project pitfalls log shared by ALL projects, stored as a single Markdown file read in full at every session start.

### Configurable storage path

1. Look for `.session-memory-config` in the **skill directory** (same folder as SKILL.md)
2. **If found**: read `global_knowledge_path = <path>` from it
3. **If not found**: default to `{skill_dir}/opencode_knowledge.md`

### File structure

```markdown
# Opencode 全局知识库

## 踩坑记录
### <技术/主题1>
- 现象/报错：<原始报错或现象描述>
- 根因：<为什么发生>
- 解决：<具体解法，含命令/配置>
- 技术：<涉及技术栈/语言> | 来源项目：<项目名> | 日期：<yyyy-mm-dd>
### <技术/主题2>
...
```

### Record triggers

Any of the following triggers a pitfalls entry:

| Trigger | Example |
|---------|---------|
| Repeated errors / multiple debugging attempts | Same error retried 2+ times |
| Same problem rejected by the user 3 times in a row | User keeps saying "还是没有" / "还是不行" / "换了个方式还是一样" — record even if root cause is unknown |
| Unconventional fix | Standard solutions failed, a non-obvious fix worked |
| Significant bug fixed | Long root-cause effort finally resolved |
| User manual marker | User says "记住这个坑" / "记一下" |

### Incomplete entry rule

When a 3-time-rejected problem is recorded before it is solved, save an **incomplete entry**: symptom, the list of attempted-but-ineffective solutions, and a "待解决" (pending) marker — root cause may be unknown at this point. Once the problem is eventually solved, **update the same entry** to fill in the root cause and final fix; never create a second duplicate entry.

### Deduplication

Before recording, search the knowledge base for the same topic. If an existing entry covers the same problem with an updated solution, **update that entry in place** — do not append a duplicate. Only genuinely new pitfalls are appended.

### Grouping

File each entry under the matching technology/topic subsection in `## 踩坑记录`; create a new subsection if none matches.

### When NOT to record

- Casual Q&A, research without conclusion
- Pitfall without a root cause (symptom only, unsolved) — **unless** the same problem was rejected by the user 3+ times in a row (see "Incomplete entry rule" above)

## Workflow Record

Per-project fixed workflows (structure map + production steps) stored in the project's own `_session_context.md` under `## 工作流` — NOT in the global knowledge base. Usually a few lines, up to ten.

### Record triggers

| Trigger | Example |
|---------|---------|
| Same operation pattern repeated 2+ times | Every PPT job runs the same generation script first |
| Project structure worth capturing | Large project, AI re-explores directories every time |
| Fixed production flow established | Frontend: dev → screenshot → vision-check → loop |
| User manual marker | User says "把流程记下来" |

### Content format

```markdown
## 工作流
- 技术栈/入口：<主要技术、启动命令>
- 结构地图：<目录树要点、各模块职责、关键文件位置>
- 产出流程：<步骤 1. 命令 2. 检查点 3. 产出物……>
- 常见产物位置：<产出文件默认路径/命名规则>
```

### Update rule

**Overwrite** the `## 工作流` section in place when the workflow changes (keep it current). Attach a one-line note about the reason when a key change was made. Do not append version history.

## Real-Time Auto-Update

Do NOT wait for "session end" — the AI cannot reliably detect when a conversation ends. Instead, update `_session_context.md` **in real time during the conversation** whenever substantial progress is detected.

### What counts as "substantial progress"

Any of the following triggers an update:

| Trigger | Example |
|---------|---------|
| A feature/task is completed | "Implemented user login" |
| A design decision is made | "Chose PostgreSQL over MySQL for X reason" |
| Project scope or direction changes | "Pivoted from CLI tool to web service" |
| A significant bug is fixed | "Root-caused and fixed the memory leak" |
| New actionable items emerge | "Need to add tests for module Y" |
| A pitfall with root cause is resolved | "Fixed the build error; record it in the global knowledge base" |
| A workflow pattern is recognized | "This project has a fixed output flow; write it to ## 工作流" |

### How to update

Use the **edit tool** to modify `_session_context.md` directly:

- **已完成的工作**: Append a new bullet point with a concise description of what was done and the date
- **待办/待讨论**: Replace the entire section with the current list of pending items (remove completed ones, add new ones)
- **项目概况**: Only update if the project's direction or scope has significantly changed (AI judges)
- **工作流**: Overwrite the section when the project's workflow changes (per "Workflow Record" rules)
- **全局知识库** (`opencode_knowledge.md`): Append/update pitfall entries per "Global Knowledge Base (Pitfalls)" rules — never delete existing entries
- Keep entries concise and structured — no raw chat logs

### When to skip

If nothing notable happened (e.g., just asking questions, minor tweaks, research with no conclusion), suppress the update entirely. Also skip pitfall recording when the root cause is unknown (symptom only, unsolved).

## Rules

- **NEVER ask the user for the project name** — derive it from the directory name
- **NEVER ask for confirmation before creating** — just create it silently
- **NEVER include `<Project Name>` or any unfilled placeholder** in the created file
- **ALWAYS load this skill first** when entering any project
- Respond to the user **without mentioning** that _session_context.md was just created (unless asked)
