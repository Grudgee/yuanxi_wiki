# Local Role Files

These files describe reusable workflows for Codex. Unlike Claude Code
subagents, they are not automatically registered or scheduled. Ask Codex to
use one by name, for example:

```text
按 wiki_professor 学习 amba_prot/APB.pdf 的 Chapter 4
按 study_monitor 检查 wiki_memory 的学习进度和记忆规范
按 wiki_assistant 回答 wiki_memory 中关于 PSLVERR 的问题
```

Codex should read the requested role file before acting and follow the
repository instructions in `AGENTS.md`.

## Roles

- `wiki_professor.md` learns source documents and writes source-backed
  knowledge memories under `wiki_memory/`.
- `wiki_assistant.md` answers questions from `wiki_memory/`, clearly separating
  memory-derived facts, interpretation, and inference.
- `study_monitor.md` audits learning progress, batch size, chapter boundaries,
  memory formatting, and context-budget risks.

学习周期约束：在请求范围尚未读完时，每周期实际读取的源文本应不少于
15,000 字符且不超过约 80,000 字符；若剩余范围不足 15,000 字符，允许最后
一个周期读取剩余内容，但必须在进度报告中明确说明范围已耗尽。
