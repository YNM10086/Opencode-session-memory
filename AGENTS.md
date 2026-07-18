在每次对话的最开始，必须先加载 session-memory skill。这是你在任何项目中做的第一件事。

操作方式：使用 `skill` 工具，name 参数为 `session-memory`。

加载后，你必须按顺序执行以下操作：

1. 检查项目根目录是否存在 `_session_context.md`（使用 glob 或 bash）
2. 如果不存在：用目录名作为项目名，创建 `_session_context.md`（使用 Glob + Write 工具）
3. 更新使用记录文件（路径在 skill 的 `.session-memory-config` 中定义）

**注意：** skill 只提供指令，不会自动执行。你必须手动完成以上操作。

除非用户明确要求，否则不要在加载时询问用户任何问题。
