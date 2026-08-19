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
