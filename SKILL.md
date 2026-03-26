---
name: summarize
description: Summarize — Extract key outcomes from the current conversation and write them to a long-term memory file, building a compounding knowledge base. Use when the conversation produces substantive outputs such as new files, important decisions, research conclusions, or design proposals. Trigger on "/summarize" command.
---

# Summarize — 对话成果总结技能

从当前对话中提取关键信息，生成结构化摘要并追加至长期记忆文件，构建复利知识库。

---

## Path Resolution（路径推断）

This skill automatically determines the memory file path using Claude Code's project path convention:

```
~/.claude/projects/<project-id>/memory/MEMORY.md
```

Where `<project-id>` is derived from the current working directory by replacing path separators and special characters with `-`. For example:

- `/home/user/my-project` → `home-user-my-project`
- `D:\ClaudeCodeWorkspace` → `D--ClaudeCodeWorkspace`

**To determine the path at runtime:** Read the current working directory, apply the transformation above, and construct the full path. If the `memory/` directory or `MEMORY.md` does not exist yet, create it before writing.

---

## Step 1 — 读取现有记忆

- 推断并读取当前项目的 `MEMORY.md` 文件（路径见上方 Path Resolution）
- 如文件不存在，先创建空文件，再继续
- 统计当前总行数

---

## Step 2 — 提取对话摘要

分析当前完整对话，提取以下信息：

- **任务主题**：本次对话处理了什么问题/需求（1-2 句话）
- **话题标签**：从以下固定话题表中选 2-4 个最相关的标签：
  `#game-industry` `#ai-product` `#tools-workflow` `#personal-career` `#workspace-setup` `#research-analysis`
  （可同时打多个标签；标签保持英文以便检索一致性）
- **关键决策**：做出了哪些重要判断或方向选择
- **输出成果**：创建/修改了哪些文件，得出了哪些结论
- **可复用知识**：对未来工作有参考价值的洞察、框架或模式
- **待处理事项**：有哪些未完成或待跟进的问题（如无则省略此项）

---

## Step 3 — 检查记忆容量

- 若当前 MEMORY.md **≤ 200 行**：直接追加摘要，跳至 Step 4
- 若当前 MEMORY.md **> 200 行**：

  ### Step 3a — 归档旧内容

  1. 确定归档文件名（格式：`YYYY-MM.md`，使用当前年月）
  2. 将 MEMORY.md 中**第 51 行至末尾**的内容，追加写入：
     `~/.claude/projects/<project-id>/memory/archive/YYYY-MM.md`
     （若 `archive/` 目录不存在，写入时自动创建）
  3. 截断 MEMORY.md 至前 50 行（保留顶部固定说明）

  ### Step 3b — 更新归档索引

  从刚归档的内容中，提取所有以 `## [` 开头的 session 标题行（格式：`## [YYYY-MM-DD] 标题`），以及紧随其后的 `**话题**：` 行中的标签。

  将这些标题+标签写入索引文件：
  `~/.claude/projects/<project-id>/memory/archive-index.md`

  **写入规则：**

  - 若 `archive-index.md` **不存在**：创建文件，写入表头后再写入本次条目：
    ```markdown
    # 归档内容索引
    > 自动维护，按需读取。完整内容见 memory/archive/ 对应文件。

    ## YYYY-MM.md
    - [YYYY-MM-DD] 标题1 | #tag1 #tag2
    - [YYYY-MM-DD] 标题2 | #tag1
    ```
  - 若 `archive-index.md` **已存在**：
    - 检查文件中是否已有该月份的区块（`## YYYY-MM.md`）
    - 若无：在文件末尾追加新月份区块 + 条目
    - 若有：在该月份区块末尾追加新条目
  - **标签格式**：若某条目有 `**话题**` 行，在标题后追加 ` | #tag1 #tag2`；若无则省略 `|` 部分（向后兼容旧格式）

---

## Step 4 — 写入摘要

向 MEMORY.md 末尾追加以下格式内容：

```markdown
---
## [YYYY-MM-DD] <任务主题>
**话题**：#tag1 #tag2

**成果**：<输出成果>

**决策**：<关键决策>

**可复用**：<可复用知识>

**待处理**：<待处理事项，若无则省略整行>
```

---

## Step 5 — 确认完成

输出："✅ 摘要已写入记忆，本次工作成果已存档。"

若本次触发了归档（Step 3），额外输出：
"📦 已归档 N 条历史记录至 memory/archive/YYYY-MM.md，索引已更新至 memory/archive-index.md"
