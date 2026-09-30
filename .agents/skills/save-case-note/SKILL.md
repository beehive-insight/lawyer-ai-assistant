---
name: save-case-note
description: Save case analysis notes from the current conversation into the case materials folder (知识/案件材料/). Use when the lawyer says "把这个案子的分析记下来", "存办案笔记", "记下这个案子的结论", "把刚才的分析存档". Only for case analysis notes, NOT expression preferences or workflows.
---

# Save Case Note（存办案笔记）

## Description

把本次对话中的案件分析、办案结论提炼成分析纪要，写入 `知识/案件材料/<案名>/`。一个案子一个文件夹。产出仅该目录下新建的纪要文件。

## When To Use

- 律师说「把这个案子的分析记下来」「存办案笔记」「记下这个案子的结论」

## When NOT To Use

- 内容是写作 / 沟通风格偏好 → 提示律师改说「记下这个写法」（`save-to-expression`）
- 内容是做事流程 → 提示律师走「帮我建个工作流」（`create-workflow`）
- 律师只说「存一下」没说存什么 → 先反问「想存表达偏好还是办案笔记」，再路由
- 案件名不明确 → 先问清案件名再继续，不自行起名

## Steps

### Step 1: 读护栏（必须最先做）

读 `guards/` 下全部四份文件。对齐 `guards/人工审核.md`：任何写入必须经律师显式确认。

### Step 2: 确认案件名与落点

- 问清案件名（若对话中已明确则复述确认）
- 落点：`知识/案件材料/<案名>/分析纪要-<日期>.md`；文件夹不存在则新建

### Step 3: 提炼纪要

从对话中整理：

- **要点**：本次分析的核心问题、双方主张、证据情况
- **结论**：办案结论 / 下一步动作
- **出处**：标注来自哪天的对话；引用的法条 / 案例按 `guards/来源要求.md` 注明出处

规则：只记对话中实际讨论的内容，不补充律师没说过的判断。

### Step 4: 过目确认（硬规则）

把**将要写入的纪要全文**展示给律师：

- 律师逐段过目、修改
- **律师未确认 = 一律不写**，无例外、无默认放行

### Step 5: 写入与回报

- 确认后写入 `知识/案件材料/<案名>/分析纪要-<日期>.md`
- 回报：落在哪个文件、写了什么

## Failure Strategy

- 案名冲突（已有同名文件夹但似乎是另一个案子）→ 向律师指出，由律师定夺
- 平台无法写文件 → 把纪要全文给律师，让他自己保存
- 对话中没有实质案件分析 → 如实说，不硬凑

## Anti-Patterns

- Do NOT write anywhere except `知识/案件材料/<案名>/` (new note files only)
- Do NOT touch 表达库、guards/、画像.md、工作流/、AGENTS.md
- Do NOT add legal analysis the lawyer did not say in conversation
- Do NOT write the note without showing the full text and getting explicit confirmation
- Do NOT treat "没反对" as confirmation — explicit yes required
