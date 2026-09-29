---
name: organizing-case-elements
description: Extract structured case elements from a free-text case description and organize them into a standardized case summary table. Use when the user describes a legal case and asks to organize, summarize, or analyze the case elements, or when the user pastes a complaint / case facts / hearing notes and wants structured output. Triggers on phrases like "整理案件", "案件要素", "案件分析", "整理要素", or when a case description (≥50 chars) is pasted with an organize intent.
---

# Organizing Case Elements

## Description

Extract structured case elements from a free-text case description and output a standardized case summary table with risk alerts and pending items.

## When To Use

- User describes a case and asks to organize / summarize / analyze its elements
- User pastes a complaint, case facts, or hearing notes and wants structured output
- User asks "整理案件要素" / "帮我分析这个案件" / "案件要素表"

## When NOT To Use

- User only wants to search for similar cases (use `searching-similar-cases` skill instead)
- User asks a general legal question without a specific case
- User asks to draft a legal document (complaint, contract, etc.)

## Inputs

- `caseDescription`: string (required) — free-text case description, can be a complaint, case facts, hearing notes, or informal narrative

## Outputs

A Markdown case element table with these sections:
1. Basic Info (case type, cause, parties, amount)
2. Claims (type, amount, description)
3. Key Timeline (dates and events)
4. Contract Terms (clause and content)
5. Evidence List (name, type, proves, status)
6. Legal Basis (law, article, verified status)
7. Risk Alerts (type, description, suggestion, severity)
8. Pending Items (checklist)

## Steps

### Step 1: Extract Case Elements

Parse the case description and extract structured elements. Output strict JSON:

```json
{
  "caseType": "民事 | 刑事 | 行政 | 国家赔偿 | 执行",
  "cause": "precise legal cause of action",
  "caseNumber": "case number or null",
  "court": "court name or null",
  "procedure": "一审 | 二审 | 再审 | null",
  "plaintiff": "plaintiff (desensitized)",
  "defendant": "defendant (desensitized)",
  "thirdParty": [],
  "plaintiffType": "自然人 | 法人 | 其他组织",
  "defendantType": "自然人 | 法人 | 其他组织",
  "claims": [
    { "type": "本金 | 利息 | 违约金 | 赔偿金 | 解除合同 | 确认 | 其他", "amount": "number or null", "description": "claim description" }
  ],
  "totalAmount": "number or null",
  "facts": "basic facts summary",
  "keyDates": [
    { "date": "YYYY-MM-DD", "event": "event description" }
  ],
  "contractTerms": [
    { "clause": "clause name", "content": "clause content" }
  ],
  "evidence": [
    { "name": "evidence name", "type": "书证 | 物证 | 电子数据 | 证人证言 | 鉴定意见 | 其他", "proves": "what it proves", "status": "已有 | 待补充" }
  ],
  "legalBasis": [
    { "law": "law name", "article": "article number", "content": "article content", "verified": false }
  ],
  "risks": [
    { "type": "法律风险 | 证据风险 | 程序风险 | 其他", "description": "risk description", "suggestion": "mitigation suggestion", "severity": "高 | 中 | 低" }
  ],
  "pendingItems": ["pending item 1", "pending item 2"]
}
```

### Step 2: Apply Extraction Rules

**Desensitization**:
- Replace real names with "客户A / 客户B" or "原告 / 被告"
- Replace company names with "某公司"
- Keep roles and relationships

**Legal terms**:
- Use precise legal terminology, not colloquial language
- Example: "民间借贷纠纷" not "借钱不还"; "买卖合同纠纷" not "买卖纠纷"

**Claims**:
- Break down compound claims into separate items
- Include amount if stated, otherwise null
- Mark calculation-dependent amounts (like "按年利率24%计算的利息") as null with description

**Evidence**:
- List all evidence mentioned in the input
- Mark evidence that should exist but wasn't mentioned as "待补充"
- Common missing evidence: bank transfer records, contracts, chat logs

**Legal basis**:
- Default all `verified` to `false` — do NOT mark as verified unless cross-checked against rmfyalk or flk.npc.gov.cn
- Include article numbers when identifiable
- Add a `content` field with a brief description of the article's topic

**Risks**:
- Be specific and actionable, not generic ("证据不足" is bad; "借条存在但无银行转账流水证明款项交付，可能被认定借款未实际发生" is good)
- Assign severity: 高 (may lose case / claim unsupported), 中 (partial impact), 低 (minor issue)
- Include a suggestion for each risk

**Pending items**:
- List all missing information needed to complete the case
- Include documents to collect, facts to confirm, amounts to calculate

### Step 3: Hallucination Prevention

**Law citation rules**:
- All laws default to `verified: false`
- Only mark `verified: true` if the law was found in rmfyalk's "关联索引" section or flk.npc.gov.cn
- If unsure whether an article exists, omit it rather than fabricate

**Case number validation**:
- Regex: `（\d{4}）\w+民\w+\d+号`
- If extracted case number doesn't match, mark as "案号格式异常"

**Amount validation**:
- Compare extracted amounts with the input text
- If inconsistent, add a note "金额与原文不符，请核对"

### Step 4: Output Markdown Table

Render the extracted elements as Markdown:

```markdown
## 案件要素整理

### 基础信息
| 要素 | 内容 | 状态 |
|---|---|---|
| 案由 | {cause} | 已识别 |
| 案件类型 | {caseType} | 已识别 |
| 原告 | {plaintiff} | 脱敏 |
| 被告 | {defendant} | 脱敏 |
| 标的额 | {totalAmount} 元 | 已识别 |

### 诉讼请求
| # | 类型 | 金额 | 描述 |
|---|---|---|---|
| 1 | {type} | {amount} 元 | {description} |

### 关键时间线
| 日期 | 事件 |
|---|---|
| {date} | {event} |

### 合同条款
| 条款 | 内容 |
|---|---|
| {clause} | {content} |

### 证据清单
| 证据 | 类型 | 证明事项 | 状态 |
|---|---|---|---|
| {name} | {type} | {proves} | {status} |

### 法条依据
| 法条 | 条款 | 状态 |
|---|---|---|
| 《{law}》 | {article} | ⚠ 未校验 |

### ⚠ 风险提示
1. **【{severity}风险】** {description}
   - 建议：{suggestion}

### 待补项
- [ ] {pending item}
```

Use these status markers:
- ✓ 已验证 (for verified laws)
- ⚠ 未校验 (for unverified laws)
- ⚠ 待补充 (for missing evidence)
- 脱敏 (for desensitized names)

## Failure Strategy

- **Input too short (<50 chars)**: Ask user to provide more details
- **No legal elements identifiable**: Inform user "未识别到法律案件要素，请提供更详细的案件描述"
- **LLM uncertain about a field**: Set to null and add to pending items, do NOT fabricate

## Anti-Patterns

- Do NOT fabricate law citations — if unsure, omit
- Do NOT mark laws as "verified" without checking against a real data source
- Do NOT use real names — always desensitize
- Do NOT generate generic risk alerts like "证据不足" — be specific about what evidence is missing and why it matters
- Do NOT include amounts not stated in the input — mark as "待计算" or null
