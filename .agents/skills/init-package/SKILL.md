---
name: init-package
description: Initialize the lawyer's AI assistant package for first use. Guides the lawyer through loading the package context, filling in the profile (画像.md), and learning where to put materials. Use when the user says "初始化", "开始用这个包", "刚拿到这个包", "怎么开始用", or asks how to set up the assistant package. Also use on a new computer or after reinstall.
---

# Initialize Package（初始化我的律师助手包）

## Description

带律师完成包的首次设置：让 agent 加载包上下文、逐项引导填写画像、指明材料存放位置。产出仅一个文件的变更：`画像.md`。

## When To Use

- 律师新拿到包 / 换电脑 / 重装后，说「初始化」「开始用这个包」「这东西怎么用」
- 律师打开 AI 工具后不知道第一句说什么，发出任何「开始 / 设置 / 上手」类请求

## When NOT To Use

- `画像.md` 已有实质内容（非模板原样）→ 包已初始化：提示律师可随时说「更新我的画像」来补充，**不要重跑覆盖**
- 律师直接提出具体业务任务（如「整理这个案子」）→ 直接干活，初始化可事后补
- 律师要建工作流 → 使用 `create-workflow` skill

## Steps

### Step 1: 读护栏（必须最先做）

读 `guards/` 下全部四份文件。向律师用一两句话复述护栏核心（法律依据必须可追溯、产出均为草稿待确认），作为加载成功的信号。

### Step 2: 读包结构

读本包 `AGENTS.md` 与 `画像.md` 模板，确认能力映射与知识目录约定已在上下文中。

### Step 3: 带填画像（一问一答）

逐节引导，**一次只问一节**，律师答多少填多少：

1. 「您主要做哪类业务？在哪个城市执业？」（我是谁）
2. 「干活上有什么习惯想让 AI 知道的？比如文书喜欢详还是略」（我怎么干活，可跳过）
3. 「『知识/』里现在放了什么，或打算放什么？」（我的资料）
4. 「有没有不接的案、不希望 AI 做的事？」（我的边界，可跳过）

规则：

- 律师说「跳过 / 先不填」→ 保留模板原文，不追问
- 不臆造律师没说的内容；律师回答模糊 → 按原话记录，不替他润色成断言

### Step 4: 写画像（唯一写盘动作）

- 把律师确认的内容按模板结构整理成 `画像.md` 全文，**先完整展示给律师**
- 律师点头后才写入 `画像.md`，仅此一个文件；不建工作流、不动 `知识/`、不改其他任何文件

### Step 5: 指路

收尾时告诉律师三件事：

1. 材料放哪：复述 `知识/` 目录速查（案件材料 / 案例库 / 文书库 / 模板库 / 法律资料）
2. 现在能做什么：举两个例子（「描述案情，我帮你找类似案例」「把乱糟糟的材料描述给我，我整理成要素表」）
3. 想固化流程：说「帮我建个工作流」

## Failure Strategy

- 律师中途中断 → 保存已确认部分再退出，告知「下次说继续初始化即可」
- 平台无法写文件 → 把整理好的画像全文给律师，让他自己粘贴保存到 `画像.md`

## Anti-Patterns

- Do NOT write to any file other than `画像.md`
- Do NOT create workflows or touch `知识/` during initialization
- Do NOT fill profile fields with inferred or fabricated content the lawyer did not say
- Do NOT overwrite an existing filled-in profile
