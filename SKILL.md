---
name: session-memory
description: >-
  Run at the START of every conversation. Check if _session_context.md exists
  in the project root. If yes: read it and restore context. If no: auto-create
  it using the directory name as the project name — no questions asked.
  This skill is ALWAYS the first thing you do in any project.
---

# Session Memory — Automatic Context Restoration

## Trigger (MANDATORY — at conversation start)

This skill **must be loaded first** in ANY project. After loading:

1. **Check** if `_session_context.md` exists at the project root
2. **If it EXISTS**: read the full file, then respond with a short confirmation that context has been restored. **Do NOT ask any questions.**
3. **If it does NOT exist**: auto-create `_session_context.md` using the current working directory's folder name as the project name. **Do NOT ask for the name. Do NOT ask questions.**
4. **ALWAYS** — regardless of whether `_session_context.md` existed or was just created — **update the usage tracking file**. Follow the "Usage Tracking" section below to determine the file path and update it. This step is **non-optional** and runs every session start.

## Template (used when auto-creating)

```markdown
# {ProjectName} — 会话记忆
<!-- {ProjectName} is auto-replaced with directory name; leave no placeholder for the user to fill -->

## 项目概况
<!-- AI auto-fills after first conversation -->

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

### How to update

Use the **edit tool** to modify `_session_context.md` directly:

- **已完成的工作**: Append a new bullet point with a concise description of what was done and the date
- **待办/待讨论**: Replace the entire section with the current list of pending items (remove completed ones, add new ones)
- **项目概况**: Only update if the project's direction or scope has significantly changed (AI judges)
- Keep entries concise and structured — no raw chat logs

### When to skip

If nothing notable happened (e.g., just asking questions, minor tweaks, research with no conclusion), suppress the update entirely.

## Rules

- **NEVER ask the user for the project name** — derive it from the directory name
- **NEVER ask for confirmation before creating** — just create it silently
- **NEVER include `<Project Name>` or any unfilled placeholder** in the created file
- **ALWAYS load this skill first** when entering any project
- Respond to the user **without mentioning** that _session_context.md was just created (unless asked)
