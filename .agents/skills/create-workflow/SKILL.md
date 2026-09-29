---
name: create-workflow
description: Guide the lawyer to create a new workflow file (工作流/*.md) through a conversational five-step process. Use when the user says "帮我建个工作流", "建工作流", "把我的流程写下来", "整理一个流程", or describes a recurring work process and wants it saved as a repeatable workflow.
---

# Create Workflow（帮我建工作流）

## Description

通过五步对话引导，把律师的某类业务流程写成一份可复用的工作流文件，存入 `工作流/`。全程以律师说的做法为准，不臆造步骤或能力。

## When To Use

- 律师说「帮我建个工作流」「把这个流程写下来」「我每次都这么干，让它照着做」
- 律师描述某类重复性工作并希望固化成流程

## When NOT To Use

- 律师只是想当场完成某个任务（不固化）→ 直接按任务处理
- 律师想改已有工作流 → 读该文件按律师意见修改，不走新建引导
- 律师想初始化包 / 填画像 → 使用 `init-package` skill

## Steps

### Step 1: 读护栏（必须最先做）

读 `guards/` 下全部四份文件，尤其 `人工审核.md`（落盘前确认的依据）。

### Step 2: 问场景

「这类活儿一般什么时候开始？当事人找你要什么？」→ 得到一句话场景，作为工作流的「这个流程解决什么」。

### Step 3: 问步骤

「你拿到材料后第一步做什么？然后呢？」逐步追问直至收尾。记录每个步骤：做什么、产出什么。律师说的做法照录，不替他优化重构。

### Step 4: 标确认点

「哪些步骤的产出你需要亲自过目？」→ 记为「律师确认点」。

### Step 5: 映射能力

把每个步骤对到已有能力（按包内 `AGENTS.md` 能力映射表，如类案检索、案件要素整理）或标记「纯人工步骤」：

- 没有能力支撑的步骤 → **明说**，问律师是保留为人工步骤还是接受不自动化
- **不臆造能力**：映射表里没有的，一律视为无能力

### Step 6: 成稿确认后写入

- 按 `工作流/_模板.md` 生成 `工作流/<名称>.md` 全文（名称用律师听得懂的中文）
- **完整展示给律师，逐节确认；律师点头后才写入**
- 写入后在 `工作流/README.md` 清单加一行（流程名 + 一句话场景）

## Constraints

- 一次只引导一个流程，不捆绑生成
- 生成内容不超出模板约定；步骤中能力引用用中文名，不写 Skill 英文目录名
- 律师说的做法与护栏冲突 → 以护栏为准，并当场提醒律师
- 写盘范围仅：新工作流文件 + `工作流/README.md` 加一行

## Failure Strategy

- 律师中途中断 → 告知「下次说继续建 XX 工作流即可」，未确认的内容不落盘
- 平台无法写文件 → 把工作流全文给律师，让他自己保存到 `工作流/`

## Anti-Patterns

- Do NOT write the workflow file before the lawyer confirms the full content
- Do NOT invent steps or capabilities the lawyer did not mention
- Do NOT create multiple workflows in one session
- Do NOT reference skills by their English directory names in workflow content
