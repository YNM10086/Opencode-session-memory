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

## Template (used when auto-creating)

```markdown
# {ProjectName} — 会话记忆
<!-- {ProjectName} is auto-replaced with directory name; leave no placeholder for the user to fill -->

## 项目概况
<!-- AI auto-fills after first conversation -->

## 已完成的工作
<!-- AI updates at session end -->

## 待办/待讨论
<!-- AI updates at session end -->
```

## Auto Project Summary

After the first meaningful work session in a project:

1. Deduce the project's main purpose, tech stack, and goals from the conversation and codebase
2. Fill the **项目概况** section in `_session_context.md` with a concise summary
3. Update this summary whenever the project's scope or direction significantly changes

This ensures the AI maintains directional awareness and doesn't drift off-topic in future sessions.

## Usage Tracking (opencode_usage.txt)

At the start of each session, **before responding to the user**:

1. Look for a file named `opencode_usage.txt` in the `D:\Opencode记忆管理` directory
2. **If it doesn't exist**: create it with the current project's entry
3. **If it exists**: check whether the current project's root path is already recorded
   - **If already present**: update its date to today (YYYY-MM-DD)
   - **If not present**: append a new line

Record format (one entry per line):
```
<project_root_path> | YYYY-MM-DD
```

Example:
```
D:\Opencode记忆管理 | 2026-07-17
D:\SomeOtherProject | 2026-07-15
```

## Session-End Auto-Update

At the end of every session (before the conversation ends or when the user signals a session boundary):

- Update **已完成的工作** with significant progress made
- Update **待办/待讨论** with any pending items or decisions
- Keep entries concise and structured — no raw chat logs
- If nothing notable happened, skip the update

## Rules

- **NEVER ask the user for the project name** — derive it from the directory name
- **NEVER ask for confirmation before creating** — just create it silently
- **NEVER include `<Project Name>` or any unfilled placeholder** in the created file
- **ALWAYS load this skill first** when entering any project
- Respond to the user **without mentioning** that _session_context.md was just created (unless asked)
