# Session-Memory v2 实施计划（踩坑知识库 + 项目工作流）

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为 session-memory skill 增加全局踩坑知识库（opencode_knowledge.md）与项目内工作流板块（_session_context.md 新增 ## 工作流）。

**Architecture:** 纯 Markdown/配置文件的修改，无代码无测试框架。三份文件联动：`.session-memory-config`（路径配置）→ `SKILL.md`（规则指令）→ `opencode_knowledge.md`（数据）。C 盘 skill 目录为运行时权威，git 仓库（D:\Opencode记忆管理）为版本同步副本，两份 SKILL.md 必须保持字节一致（当前 MD5 相同：`5E6E084C06D8D7724F7F37CD3F1CBA83`）。AGENTS.md 有两处（全局 + 项目），均需同步。

**Tech Stack:** Markdown, Git, PowerShell（校验用）

**参考设计:** `docs/superpowers/specs/2026-08-13-session-memory-v2-design.md`

---

## 文件结构

| 文件 | 职责 | 操作 |
|------|------|------|
| `C:\Users\丧彪\.config\opencode\skills\session-memory\.session-memory-config` | 配置：usage_tracking_path + global_knowledge_path | Modify |
| `C:\Users\丧彪\.config\opencode\skills\session-memory\opencode_knowledge.md` | 全局踩坑知识库数据文件 | Create |
| `C:\Users\丧彪\.config\opencode\skills\session-memory\SKILL.md` | skill 规则：Trigger/模板/知识库章节/工作流章节/Real-Time | Modify |
| `D:\Opencode记忆管理\SKILL.md` | git 仓库同步副本（字节一致） | Modify |
| `D:\Opencode记忆管理\.session-memory-config` | git 仓库同步副本 | Modify |
| `C:\Users\丧彪\.config\opencode\AGENTS.md` | 全局加载指令（步骤 1-4） | Modify |
| `D:\Opencode记忆管理\AGENTS.md` | 项目加载指令（步骤 1-4） | Modify |
| `D:\Opencode记忆管理\_session_context.md` | 本项目会话记忆（新增工作流板块示范） | Modify |
| `D:\Opencode记忆管理\README.md` | 功能清单（可选更新） | Modify |

---

### Task 1: 更新 .session-memory-config 增加知识库路径

**Files:**
- Modify: `C:\Users\丧彪\.config\opencode\skills\session-memory\.session-memory-config`
- Modify: `D:\Opencode记忆管理\.session-memory-config`（仓库同步副本）

- [ ] **Step 1: 修改 C 盘配置文件**

当前内容：
```
usage_tracking_path = D:\Opencode记忆管理\opencode_usage.md
```
改为：
```
usage_tracking_path = D:\Opencode记忆管理\opencode_usage.md
global_knowledge_path = C:\Users\丧彪\.config\opencode\skills\session-memory\opencode_knowledge.md
```

- [ ] **Step 2: 同步仓库副本**

将相同内容写入 `D:\Opencode记忆管理\.session-memory-config`。

- [ ] **Step 3: 验证**

Run: `Get-Content "C:\Users\丧彪\.config\opencode\skills\session-memory\.session-memory-config"; Get-Content "D:\Opencode记忆管理\.session-memory-config"`
Expected: 两文件均含 `global_knowledge_path = ...` 且内容一致。

---

### Task 2: 创建全局知识库初始文件

**Files:**
- Create: `C:\Users\丧彪\.config\opencode\skills\session-memory\opencode_knowledge.md`

- [ ] **Step 1: 写入初始文件**

```markdown
# Opencode 全局知识库

<!-- 跨项目共享的踩坑经验，AI 在会话中自动/手动记录。只进不出。 -->

## 踩坑记录

<!-- 按技术/主题分组，格式：
### <技术/主题>
- 现象/报错：<原始报错或现象>
- 根因：<为什么发生>
- 解决：<具体解法，含命令/配置>
- 技术：<技术栈/语言> | 来源项目：<项目名> | 日期：<yyyy-mm-dd>
-->
```

- [ ] **Step 2: 验证**

Run: `Test-Path "C:\Users\丧彪\.config\opencode\skills\session-memory\opencode_knowledge.md"`
Expected: True

---

### Task 3: 更新 SKILL.md（核心规则）

**Files:**
- Modify: `C:\Users\丧彪\.config\opencode\skills\session-memory\SKILL.md`

- [ ] **Step 1: 更新 frontmatter description**

原：
```
  Run at the START of every conversation. Check if _session_context.md exists
  in the project root. If yes: read it and restore context. If no: auto-create
  it using the directory name as the project name — no questions asked.
  This skill is ALWAYS the first thing you do in any project.
```
改为：
```
  Run at the START of every conversation. Check if _session_context.md exists
  in the project root. If yes: read it and restore context. If no: auto-create
  it using the directory name as the project name — no questions asked.
  Also read the global pitfalls knowledge base (opencode_knowledge.md) and
  restore any recorded workflow for this project.
  This skill is ALWAYS the first thing you do in any project.
```

- [ ] **Step 2: Trigger 步骤 3 后新增步骤 4（原步骤 4 变 5）**

原：
```
4. **ALWAYS** — regardless of whether `_session_context.md` existed or was just created — **update the usage tracking file**. Follow the "Usage Tracking" section below to determine the file path and update it. This step is **non-optional** and runs every session start.
```
改为：
```
4. **ALWAYS** — regardless of whether `_session_context.md` existed or was just created — **update the usage tracking file**. Follow the "Usage Tracking" section below to determine the file path and update it. This step is **non-optional** and runs every session start.
5. **Read the global knowledge base** — read `opencode_knowledge.md` in full (path from `global_knowledge_path` in `.session-memory-config`, default `{skill_dir}/opencode_knowledge.md`). If it does not exist, create it with the header only. This step is **non-optional** and runs every session start.
```

- [ ] **Step 3: 更新模板（新增 ## 工作流 板块）**

原模板代码块：
````markdown
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
````
改为：
````markdown
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
````

- [ ] **Step 4: 在 "Usage Tracking" 章节之后插入 "Global Knowledge Base" 章节**

插入完整章节（位置：`### Migration from old .txt format` 小节结束之后、`## Real-Time Auto-Update` 之前）：

````markdown
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
| Unconventional fix | Standard solutions failed, a non-obvious fix worked |
| Significant bug fixed | Long root-cause effort finally resolved |
| User manual marker | User says "记住这个坑" / "记一下" |

### Deduplication

Before recording, search the knowledge base for the same topic. If an existing entry covers the same problem with an updated solution, **update that entry in place** — do not append a duplicate. Only genuinely new pitfalls are appended.

### Grouping

File each entry under the matching technology/topic subsection in `## 踩坑记录`; create a new subsection if none matches.

### When NOT to record

- Casual Q&A, research without conclusion
- Pitfall without a root cause (symptom only, unsolved)
````

- [ ] **Step 5: 在 "Global Knowledge Base" 章节之后插入 "Workflow Record" 章节**

````markdown
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
````

- [ ] **Step 6: 扩展 Real-Time Auto-Update 触发条件表**

原表：
```
| Trigger | Example |
|---------|---------|
| A feature/task is completed | "Implemented user login" |
| A design decision is made | "Chose PostgreSQL over MySQL for X reason" |
| Project scope or direction changes | "Pivoted from CLI tool to web service" |
| A significant bug is fixed | "Root-caused and fixed the memory leak" |
| New actionable items emerge | "Need to add tests for module Y" |
```
改为：
```
| Trigger | Example |
|---------|---------|
| A feature/task is completed | "Implemented user login" |
| A design decision is made | "Chose PostgreSQL over MySQL for X reason" |
| Project scope or direction changes | "Pivoted from CLI tool to web service" |
| A significant bug is fixed | "Root-caused and fixed the memory leak" |
| New actionable items emerge | "Need to add tests for module Y" |
| A pitfall with root cause is resolved | "Fixed the build error; record it in the global knowledge base" |
| A workflow pattern is recognized | "This project has a fixed output flow; write it to ## 工作流" |
```

- [ ] **Step 7: 更新 "How to update" 小节**

原：
```
- **已完成的工作**: Append a new bullet point with a concise description of what was done and the date
- **待办/待讨论**: Replace the entire section with the current list of pending items (remove completed ones, add new ones)
- **项目概况**: Only update if the project's direction or scope has significantly changed (AI judges)
- Keep entries concise and structured — no raw chat logs
```
改为：
```
- **已完成的工作**: Append a new bullet point with a concise description of what was done and the date
- **待办/待讨论**: Replace the entire section with the current list of pending items (remove completed ones, add new ones)
- **项目概况**: Only update if the project's direction or scope has significantly changed (AI judges)
- **工作流**: Overwrite the section when the project's workflow changes (per "Workflow Record" rules)
- **全局知识库** (`opencode_knowledge.md`): Append/update pitfall entries per "Global Knowledge Base (Pitfalls)" rules — never delete existing entries
- Keep entries concise and structured — no raw chat logs
```

- [ ] **Step 8: 更新 "When to skip" 小节**

原：
```
If nothing notable happened (e.g., just asking questions, minor tweaks, research with no conclusion), suppress the update entirely.
```
改为：
```
If nothing notable happened (e.g., just asking questions, minor tweaks, research with no conclusion), suppress the update entirely. Also skip pitfall recording when the root cause is unknown (symptom only, unsolved).
```

- [ ] **Step 9: 验证修改后文件**

Run: `Get-Content "C:\Users\丧彪\.config\opencode\skills\session-memory\SKILL.md" | Measure-Object -Line`
Expected: 约 220-240 行，含新章节标题 `## Global Knowledge Base (Pitfalls)`、`## Workflow Record`、模板中的 `## 工作流`。

---

### Task 4: 同步 SKILL.md 与配置文件到 git 仓库

**Files:**
- Modify: `D:\Opencode记忆管理\SKILL.md`
- Modify: `D:\Opencode记忆管理\.session-memory-config`

- [ ] **Step 1: 复制 C 盘修改后的文件到仓库**

Run:
```powershell
Copy-Item "C:\Users\丧彪\.config\opencode\skills\session-memory\SKILL.md" "D:\Opencode记忆管理\SKILL.md" -Force
Copy-Item "C:\Users\丧彪\.config\opencode\skills\session-memory\.session-memory-config" "D:\Opencode记忆管理\.session-memory-config" -Force
```

- [ ] **Step 2: 校验字节一致**

Run: `(Get-FileHash "D:\Opencode记忆管理\SKILL.md" -Algorithm MD5).Hash -eq (Get-FileHash "C:\Users\丧彪\.config\opencode\skills\session-memory\SKILL.md" -Algorithm MD5).Hash`
Expected: True

- [ ] **Step 3: 提交**

```bash
git add SKILL.md .session-memory-config
git commit -m "feat: add global pitfalls knowledge base and per-project workflow record to session-memory"
```

---

### Task 5: 更新两个 AGENTS.md（加载步骤增加知识库读取）

**Files:**
- Modify: `C:\Users\丧彪\.config\opencode\AGENTS.md`
- Modify: `D:\Opencode记忆管理\AGENTS.md`

- [ ] **Step 1: 更新全局 AGENTS.md 步骤 2 之后插入新步骤**

当前（两文件相同）：
```
2. 如果不存在：用目录名作为项目名，创建 `_session_context.md`（使用 Glob + Write 工具）
3. **无论第 1 步结果如何（文件已存在或刚创建），都必须**更新使用记录文件（路径在 skill 的 `.session-memory-config` 中定义）
```
改为：
```
2. 如果不存在：用目录名作为项目名，创建 `_session_context.md`（使用 Glob + Write 工具）
3. **无论第 1 步结果如何（文件已存在或刚创建），都必须**更新使用记录文件（路径在 skill 的 `.session-memory-config` 中定义）
4. **读取全局踩坑知识库** `opencode_knowledge.md`（路径在 `.session-memory-config` 的 `global_knowledge_path` 中定义），全量读入并恢复坑经验
```

注意：全局版 AGENTS.md 后续还有"注意"段落和图片理解规则，编号"3."后直接插一行"4."，不要改动其余内容。项目版 AGENTS.md 的"注意"段落同理。

- [ ] **Step 2: 更新项目 AGENTS.md 相同内容**

- [ ] **Step 3: 验证**

Run: `Select-String -Path "C:\Users\丧彪\.config\opencode\AGENTS.md","D:\Opencode记忆管理\AGENTS.md" -Pattern "全局踩坑知识库" | Select-Object Path,LineNumber`
Expected: 两个文件各匹配 1 处（含"读取全局踩坑知识库"步骤行）。注意：全局 AGENTS.md 的"工作流内部截图强制识图规则"段落可能也含"全局"字样，需确认匹配行是新增的第 4 步。

- [ ] **Step 4: 提交项目 AGENTS.md（全局版不在 git 仓库内，无需提交）**

```bash
git add AGENTS.md
git commit -m "docs: add global knowledge base reading step to AGENTS.md"
```

---

### Task 6: 本项目 _session_context.md 添加工作流板块（示范验证）

**Files:**
- Modify: `D:\Opencode记忆管理\_session_context.md`

- [ ] **Step 1: 在 ## 项目概况 之后插入工作流板块**

当前文件前 5 行：
```
# Opencode记忆管理 — 会话记忆

## 项目概况
专门用于优化和改进 session-memory skill 的项目。skills 文件位于 `C:\Users\丧彪\.claude\skills\session-memory\`。
```
改为：
```
# Opencode记忆管理 — 会话记忆

## 项目概况
专门用于优化和改进 session-memory skill 的项目。skills 文件位于 `C:\Users\丧彪\.claude\skills\session-memory\`。

## 工作流
- 技术栈/入口：Markdown + Git，无测试框架；skill 运行时权威在 C 盘 `C:\Users\丧彪\.config\opencode\skills\session-memory\`
- 结构地图：`SKILL.md`（规则）/ `.session-memory-config`（路径配置）/ `opencode_knowledge.md`（知识库数据）；git 仓库 `D:\Opencode记忆管理` 为同步副本，两份 SKILL.md 须字节一致
- 产出流程：1. 修改 C 盘 skill 文件 → 2. 同步复制到仓库 → 3. MD5 校验一致 → 4. 提交 git（含 AGENTS.md/.gitignore 联动）→ 5. 在真实项目中验证表现
- 常见产物位置：知识库数据 `opencode_knowledge.md`；使用记录 `opencode_usage.md`（追加式）
```

- [ ] **Step 2: 验证板块存在**

Run: `Select-String -Path "D:\Opencode记忆管理\_session_context.md" -Pattern "## 工作流"`
Expected: 匹配 1 处

- [ ] **Step 3: 更新使用记录与待办**

在 `opencode_usage.md` 追加一行：`|D:\Opencode记忆管理|2026-08-13|`（若本会话已追加则跳过）。

在 `_session_context.md` 的"已完成的工作"追加：
```
- 2026-08-13: 设计并实施 session-memory v2——新增全局踩坑知识库（opencode_knowledge.md，结构化条目+去重+自动/手动触发）与项目工作流板块（_session_context.md 新增 ## 工作流，覆盖更新）
```
在"待办/待讨论"追加：
```
- 在多个真实项目中运行，验证踩坑自动记录与工作流恢复的实际效果（坑是否被正确识别与归类、工作流是否随项目恢复）
```

---

### Task 7: 端到端验证

**Files:**
- 无（仅验证）

- [ ] **Step 1: 模拟完整启动流程**

在临时目录新建测试项目并手动执行加载流程：
Run:
```powershell
$test = "C:\Users\丧彪\AppData\Local\Temp\opencode\memory-v2-test"
New-Item -ItemType Directory -Path $test -Force | Out-Null
$cfg = Get-Content "C:\Users\丧彪\.config\opencode\skills\session-memory\.session-memory-config" -Raw
Write-Output $cfg
Test-Path "C:\Users\丧彪\.config\opencode\skills\session-memory\opencode_knowledge.md"
```
Expected: 配置输出含 `global_knowledge_path`，知识库文件存在。确认新文件 `opencode_knowledge.md` 可被正常读取（全量内容仅头部注释）。

- [ ] **Step 2: 清理测试目录**

Run: `Remove-Item "C:\Users\丧彪\AppData\Local\Temp\opencode\memory-v2-test" -Recurse -Force`
Expected: 无报错

- [ ] **Step 3: 最终提交**

```bash
git add -A
git status
git commit -m "feat: session-memory v2 — global pitfalls knowledge base + per-project workflow record" --dry-run
```

实际提交按需拆分为已完成的各任务提交（Task 4/5/6 的 commit 已包含）。最终确认 `git log --oneline` 显示 v2 相关提交、`git status` clean。

---

## 自检（self-review）

**Spec 覆盖：**
- 全局知识库文件+配置 → Task 1, 2
- 踩坑触发/格式/去重/归类/跳过 → Task 3 Step 4（章节）+ Step 6, 8
- 工作流板块/触发/格式/覆盖更新 → Task 3 Step 3, 5 + Task 6
- 启动流程（Trigger 5 步）→ Task 3 Step 2
- AGENTS.md 联动 → Task 5
- 写入分工/轻量原则 → Task 3 Step 7

**占位符扫描：** 无 TBD/TODO；所有修改给出完整前后文与具体内容。

**类型一致性：** 配置键名统一 `global_knowledge_path`；文件名统一 `opencode_knowledge.md`；板块名统一 `## 工作流`。
