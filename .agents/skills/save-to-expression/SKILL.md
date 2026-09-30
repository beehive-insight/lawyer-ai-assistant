---
name: save-to-expression
description: Save the lawyer's writing and communication style preferences to the expression library (知识/表达库.md). Use when the lawyer says things like "记下这个写法", "这个偏好记下来", "以后都这么写，存一下", "把我的表达习惯记下来". Only for style/expression preferences, NOT case conclusions or workflows.
---

# Save To Expression（记下这个写法）

## Description

把本次对话中律师表达出的写作 / 沟通风格偏好，提炼成条目追加进 `知识/表达库.md`。每条一句话 + 出处。产出仅一个文件的追加。

## When To Use

- 律师说「记下这个写法」「这个偏好记下来」「以后都这么写，存一下」
- 律师在对话中明确了风格偏好并要求记住（如「以后答辩状都结论先行」）

## When NOT To Use

- 内容是案件分析 / 办案结论 → 提示律师改说「存办案笔记」（`save-case-note`）
- 内容是做事流程 → 提示律师走「帮我建个工作流」（`create-workflow`）
- 律师只说「存一下」没说存什么 → 先反问「想存表达偏好还是办案笔记」，再路由
- 无确凿出处（律师没说过、只是推测他可能喜欢）→ 不写入，如实说

## Steps

### Step 1: 读护栏（必须最先做）

读 `guards/` 下全部四份文件。对齐 `guards/人工审核.md`：任何写入必须经律师显式确认。

### Step 2: 提炼候选条目

回顾本次对话，找出律师表达的风格偏好，逐条整理为：

> 一句话偏好（出处：YYYY-MM 存档，对话）

分组参考：文书风格 / 客户沟通 / 分析套路 / 术语习惯（律师可自建组）。

规则：

- 每条必须能指到律师的原话或明确决定，不臆造、不润色成断言
- 拿不准算不算偏好 → 列出来问律师，不自行取舍

### Step 3: 逐条确认（硬规则）

把候选条目**原文**逐条展示给律师：

- 律师逐条点头 / 剔除 / 修改
- **律师未确认 = 一律不写**，无例外、无默认放行

### Step 4: 写入与回报

- 确认后的条目**追加**进 `知识/表达库.md` 对应分组（文件不存在则按模板新建）
- 只追加，不改写、不删除已有条目
- 回报：写了几条、落在哪个文件、原文是什么

## Failure Strategy

- `知识/表达库.md` 不存在 → 按模板新建空骨架再追加，告知律师
- 平台无法写文件 → 把条目全文给律师，让他自己粘贴保存
- 对话中找不到确凿的偏好 → 如实说「这次对话没有可记的表达偏好」，不硬凑

## Anti-Patterns

- Do NOT write to any file other than `知识/表达库.md`
- Do NOT modify or delete existing entries — append only
- Do NOT write any entry the lawyer did not explicitly confirm
- Do NOT write preferences without a traceable source (对齐 `guards/防幻觉.md`、`guards/来源要求.md`)
- Do NOT treat "没反对" as confirmation — explicit yes required
