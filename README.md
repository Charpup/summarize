# summarize — Claude Code Skill

A Claude Code skill that records **session deliverables** (files created/modified and cross-session decisions) to a long-term memory file, enabling reliable context restoration across sessions.

## Design rationale

Claude Code already has a built-in auto-memory system that automatically captures behavioral patterns — user preferences, feedback corrections, project background, and external references. This skill focuses on what auto-memory does **not** capture: a chronological log of **what was built** and **what was decided** in each session.

| What auto-memory captures | What this skill captures |
|---|---|
| User preferences & habits | File deliverables (paths + summaries) |
| Feedback & corrections | Cross-session decisions (what + why) |
| Project background | Pending follow-ups |
| External resource references | Session date & topic tags |

## When to use

Run `/summarize` when the conversation produced:

- **File outputs** — any files created or meaningfully modified
- **Cross-session decisions** — architectural choices, direction changes, plan adjustments

**Skip** for pure Q&A conversations with no file outputs and no decisions that affect future sessions. Claude will detect this and exit gracefully.

## Installation

```bash
claude skill install https://github.com/Charpup/summarize
```

Or manually:

```bash
mkdir -p ~/.claude/skills/summarize
cp SKILL.md ~/.claude/skills/summarize/
```

## Usage

```
/summarize
```

Claude analyzes the conversation, extracts file deliverables and key decisions, and appends a structured entry to your memory file.

## Memory file location

Writes to:
```
~/.claude/projects/<project-id>/memory/MEMORY.md
```

`<project-id>` mirrors Claude Code's project path convention (working directory path with separators replaced by `-`).

## Entry format

```markdown
---
## [YYYY-MM-DD] Task Topic
**话题**：#tag1 #tag2

**文件产出**：
- `path/to/file.md` — one-line description
(omitted when no files were produced)

**决策**：Cross-session decisions made (omitted if none)

**待处理**：Explicit follow-up items (omitted if none)
```

## Auto-archiving

When `MEMORY.md` exceeds 200 lines:
1. Lines 51+ are archived to `memory/archive/YYYY-MM.md`
2. A searchable index is maintained at `memory/archive-index.md`
3. The main file is trimmed to keep context loading fast

## License

MIT
