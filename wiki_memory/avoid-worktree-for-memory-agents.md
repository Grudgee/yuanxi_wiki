---
name: avoid-worktree-for-memory-agents
description: 读取本地 memory 的 wiki agent 任务不要使用 worktree 隔离
metadata:
  type: feedback

对 `wiki_assistant`、`wiki_professor` 或其他只读本地 memory 的任务，直接使用当前会话的绝对路径 `/home/yuanxi/.claude/projects/-/memory/`，不要启动 `isolation: "worktree"`。此前 worktree 启动曾错误尝试创建 `/.claude`，原因是运行环境把仓库根目录解析成 `/`，不是 agent prompt 内容导致的。

**Why:** worktree 隔离在当前环境可能把路径解析到 `/`，导致 `EACCES: permission denied, mkdir '/.claude'`。

**How to apply:** 优先使用 Read 等直接文件工具；必须启动 agent 时省略 isolation 参数，并在 prompt 中指定绝对 memory 路径。仅在用户明确要求且确认仓库根目录解析正确时使用 worktree。
