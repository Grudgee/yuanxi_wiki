# Wiki Memory

This directory is the repository-local replacement for the former Claude Code
project memory. It stores source-backed study notes that can be reused by both
Claude and Codex.

## Layout

- `MEMORY.md` is the migrated top-level index.
- `knowledge_*.md` files contain topic or chapter notes.
- Files preserve their original YAML frontmatter and source references.

## Conventions

- Knowledge filenames use lowercase kebab-case or snake_case and end in `.md`.
- `MEMORY.md` and `README.md` are reserved infrastructure filenames and are
  exceptions to the knowledge-file naming and frontmatter rules.
- Each new memory file should include YAML frontmatter with at least `name`
  and `description`.
- Prefer structured, reusable conclusions over large copies of source text.
- Record the source document, version, section, page, URL, and reliability or
  unresolved questions when available.
- Update the relevant index memory after adding or materially changing notes.

## Origin

The initial contents were migrated from:

`/home/mt/.claude/projects/-home-mt-----yuanxi-cc-wiki-files/memory`

No source files were removed or modified during migration.
