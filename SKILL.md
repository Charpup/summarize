---
name: summarize
description: Summarize — Record session deliverables (files created/modified and cross-session decisions) to a long-term memory file. This skill should be used when the conversation has produced file outputs or decisions with cross-session impact. Skip for pure Q&A conversations. Trigger on "/summarize" command.
---

# Summarize — Session 交付物归档技能

将当前对话产生的**文件交付物**和**跨 session 决策**写入长期记忆，供后续 session 恢复上下文使用。

> **适用条件（满足任一即触发）：**
> - 对话中创建或修改了文件
> - 做出了影响后续 session 的决策（架构选型、方向判断、计划调整）
>
> **跳过条件：** 纯问答/解释性对话，无文件产出，无跨 session 决策。

---

## Path Resolution（路径推断）

Memory 文件路径遵循 Claude Code 项目路径约定：

```
~/.claude/projects/<project-id>/memory/MEMORY.md
```

`<project-id>` 由当前工作目录路径转换而来（路径分隔符和特殊字符替换为 `-`）：

- `/home/user/my-project` → `home-user-my-project`
- `D:\ClaudeCodeWorkspace` → `D--ClaudeCodeWorkspace`

若 `memory/` 目录或 `MEMORY.md` 不存在，写入前先创建。

---

## Step 1 — 评估是否需要归档

检查当前对话是否满足触发条件：

- **有文件产出**（创建/修改了文件）→ 继续执行
- **有跨 session 决策**（方向选择、架构判断、计划调整）→ 继续执行
- **仅为问答/解释**，无文件、无决策 → 输出提示后退出：
  > "ℹ️ 本次对话为纯问答，无需归档。"

---

## Step 2 — 读取现有记忆

- 推断并读取当前项目的 `MEMORY.md` 文件
- 如文件不存在，先创建空文件再继续
- 统计当前总行数

---

## Step 3 — 提取交付物摘要

从当前对话中提取以下信息（**仅记录文件和决策，不重复 auto-memory 已捕获的偏好/反馈类内容**）：

- **任务主题**：本次对话处理了什么任务（1 句话）
- **话题标签**：从固定标签表中选 1-3 个最相关的标签：
  `#game-industry` `#ai-product` `#tools-workflow` `#personal-career` `#workspace-setup` `#research-analysis`
- **文件产出**：列出本次创建或修改的文件路径（无则省略此字段）
  - 格式：`path/to/file.md — 一句话说明内容`
- **关键决策**：做出了哪些跨 session 影响的判断或方向选择（无则省略此字段）
  - 仅记录**决策本身**（是什么 + 为什么），不记录实现细节
- **待处理**：有哪些明确的后续跟进事项（无则省略此字段）

> **不记录的内容：** 用户偏好、操作习惯、反馈修正、外部资源引用——这些由 Claude Code auto-memory 自动捕获，无需在此重复。

---

## Step 4 — 检查记忆容量

- 若当前 MEMORY.md **≤ 200 行**：直接追加摘要，跳至 Step 5
- 若当前 MEMORY.md **> 200 行**：

  ### Step 4a — 归档旧内容

  1. 确定归档文件名（格式：`YYYY-MM.md`，使用当前年月）
  2. 将 MEMORY.md **第 51 行至末尾**的内容追加写入：
     `~/.claude/projects/<project-id>/memory/archive/YYYY-MM.md`
     （若 `archive/` 目录不存在，写入时自动创建）
  3. 截断 MEMORY.md 至前 50 行（保留顶部固定说明）

  ### Step 4b — 更新归档索引

  从归档内容中提取所有以 `## [` 开头的 session 标题行及其 `**话题**` 标签，写入：
  `~/.claude/projects/<project-id>/memory/archive-index.md`

  **写入规则：**
  - 若 `archive-index.md` **不存在**：创建文件并写入：
    ```markdown
    # 归档内容索引
    > 自动维护，按需读取。完整内容见 memory/archive/ 对应文件。

    ## YYYY-MM.md
    - [YYYY-MM-DD] 标题 | #tag1 #tag2
    ```
  - 若 `archive-index.md` **已存在**：
    - 已有该月份区块 → 在区块末尾追加条目
    - 无该月份区块 → 在文件末尾追加新区块 + 条目

---

## Step 5 — 写入摘要

向 MEMORY.md 末尾追加以下格式：

```markdown
---
## [YYYY-MM-DD] <任务主题>
**话题**：#tag1 #tag2

**文件产出**：
- `path/to/file.md` — 说明
（无文件产出时省略整个字段）

**决策**：<跨 session 影响的决策，无则省略>

**待处理**：<明确的后续跟进事项，无则省略>
```

---

## Step 6 — 确认完成

输出："✅ 摘要已写入记忆，本次工作成果已存档。"

若触发了归档（Step 4），额外输出：
"📦 已归档 N 条历史记录至 memory/archive/YYYY-MM.md，索引已更新。"
