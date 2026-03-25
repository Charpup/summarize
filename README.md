# summarize — Claude Code Skill

A Claude Code skill that extracts key outcomes from your conversation and writes them to a long-term memory file, building a compounding knowledge base across sessions.

## What it does

Every time you finish meaningful work, run `/summarize` to:

- Extract: task topic, key decisions, outputs, reusable insights, pending items
- Write: structured entry to `~/.claude/projects/<project>/memory/MEMORY.md`
- Archive: automatically rotates old entries when the file exceeds 150 lines

No configuration needed — the memory file path is automatically inferred from your current working directory using Claude Code's project path convention.

## Installation

```bash
claude skill install https://github.com/Charpup/summarize
```

Or manually copy `SKILL.md` into your Claude skills directory:

```bash
mkdir -p ~/.claude/skills/summarize
cp SKILL.md ~/.claude/skills/summarize/
```

## Usage

After any conversation with substantive outputs, run:

```
/summarize
```

Claude will analyze the conversation, extract key information, and append a structured entry to your memory file.

### When to use

- After creating or modifying important files
- After making architectural or design decisions
- After completing research or competitive analysis
- After any session where you'd want future-you to remember what happened

### Memory file location

The skill writes to:
```
~/.claude/projects/<project-id>/memory/MEMORY.md
```

Where `<project-id>` mirrors Claude Code's own project path convention (working directory path with separators replaced by `-`).

## Memory format

Each summary entry follows this structure:

```markdown
---
## [YYYY-MM-DD] Task Topic

**成果**: What was created or concluded

**决策**: Key decisions made

**可复用**: Reusable insights or frameworks

**待处理**: Pending follow-ups (omitted if none)
```

## Auto-archiving

When `MEMORY.md` exceeds 150 lines:
1. Lines 51+ are archived to `memory/archive/YYYY-MM.md`
2. An index is maintained at `memory/archive-index.md`
3. The main file is trimmed to keep it fast to load

## License

MIT
