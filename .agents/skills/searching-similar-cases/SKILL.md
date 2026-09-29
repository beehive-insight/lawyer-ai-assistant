---
name: searching-similar-cases
description: Search for similar court cases from the People's Court Case Database (rmfyalk.court.gov.cn). Use when the user describes a legal case and asks to find similar cases, search for precedents, or look up analogous rulings. Triggers on phrases like "找类似案例", "类案检索", "相似案件", "类似判例", or when a case description (≥50 chars) is pasted with a search intent.
---

# Searching Similar Cases

## Description

Search the People's Court Case Database (人民法院案例库, rmfyalk.court.gov.cn) for cases similar to the user's case description. Return structured case cards with matching scores and reasons.

## When To Use

- User describes a legal case and asks to find similar cases / precedents
- User pastes a case description (≥50 chars) with search intent
- User asks "有没有类似的案例" / "类案检索" / "找类似判例"

## When NOT To Use

- User only wants to organize case elements without searching (use `organizing-case-elements` skill instead)
- User asks about a specific known case by case number (just search directly)
- User asks legal questions without a case description

## Inputs

- `caseDescription`: string (required) — free-text description of the case, including parties, facts, disputes, and the legal question

## Outputs

- `queries`: array of search keyword groups extracted from the case
- `cases`: array of similar case cards, each containing:
  - `title`: case title
  - `caseNumber`: case number (e.g. （2021）最高法民申2084号)
  - `court`: court name
  - `procedure`: trial procedure (一审/二审/再审)
  - `gist`: referee gist (裁判要旨, ≤300 chars)
  - `legalBasis`: array of cited laws/articles
  - `matchScore`: 0-100, relevance to input case
  - `matchReason`: one sentence explaining why this case matches
  - `detailUrl`: link to full text
  - `verified`: whether legalBasis was verified against the source

## Steps

### Step 1: Extract Search Queries

From the case description, extract 2-3 keyword groups for searching rmfyalk. Each group should cover: cause of action + dispute focus + key legal concept.

Output JSON:
```json
{
  "caseType": "民事 | 刑事 | 行政 | 国家赔偿 | 执行",
  "cause": "precise legal cause of action",
  "focusPoints": ["focus 1", "focus 2"],
  "queries": [
    { "keyword": "keyword1 keyword2 keyword3", "reason": "why this group", "priority": "high|medium|low" }
  ]
}
```

Rules:
- Use precise legal terms, not colloquial language (e.g. "民间借贷纠纷" not "借钱不还")
- Avoid overly broad queries (e.g. just "民间借贷") or overly narrow (e.g. with specific names/amounts)
- 2-3 groups, each with 2-4 keywords separated by spaces

### Step 2: Check Login State

Before searching, verify the browser is logged into rmfyalk:

1. Navigate to `https://rmfyalk.court.gov.cn/`
2. Wait 2 seconds
3. Check if the page shows a phone number in the top-right corner (indicates logged in)

If NOT logged in:
- Call `browser_waiting_for_user_interaction` with reason "请在浏览器内登录 rmfyalk.court.gov.cn（手机号+密码 / 支付宝 / 钉钉）"
- Wait for user to complete login
- After user confirms, re-check login state

### Step 3: Build Search URL

```javascript
function buildSearchUrl(keyword, options = {}) {
  const base = "https://rmfyalk.court.gov.cn/view/list.html";
  const params = new URLSearchParams({
    key: "qw",
    keyName: "全文",
    value: keyword,
    isAdvSearch: "0",
    searchType: options.fuzzy ? "2" : "1",  // 1=exact, 2=fuzzy
    lib: options.lib || "cpwsAl_qb"          // cpwsAl_qb=all, cpwsAl_zd=guiding, cpwsAl_ck=reference
  });
  return `${base}?${params.toString()}`;
}
```

- Default to fuzzy search (searchType=2) for multi-word queries — higher hit rate
- Use exact search (searchType=1) only for single keywords or precise phrases

### Step 4: Execute Search and Extract Results

1. Navigate to the search URL
2. Wait 4 seconds for results to load
3. Use `browser_evaluate` to extract results:

```javascript
JSON.stringify({
  civilCount: (document.body.innerText.match(/民事\s*[（(](\d+)[）)]/) || ['','0'])[1],
  cases: Array.from(document.querySelectorAll('.al-list li'))
    .filter(li => li.textContent.includes('裁判要旨'))
    .slice(0, 5)
    .map(li => ({
      type: li.querySelector('.list-lib')?.textContent?.trim(),
      title: li.querySelector('.list-title a')?.textContent?.replace(/\s+/g, ' ')?.trim(),
      detailUrl: li.querySelector('.list-title a')?.href,
      attr: li.querySelector('.list-attribute')?.textContent?.trim(),
      gist: li.querySelector('.list-content-value')?.textContent?.replace(/\s+/g, ' ')?.trim()?.slice(0, 400)
    }))
})
```

### Step 5: Extract Legal Basis from Detail Pages (Optional)

For the top 1-2 most relevant cases, open the detail page to extract legal basis:

1. Navigate to `detailUrl`
2. Wait 3 seconds
3. Use `browser_evaluate` to extract:

```javascript
var t = document.body.innerText;
var i4 = t.indexOf('关联索引');
JSON.stringify({ laws: t.slice(i4, i4 + 700) })
```

The "关联索引" section contains cited laws and trial case numbers in this format:
```
《中华人民共和国民法典》第577条
《最高人民法院关于审理买卖合同纠纷案件适用法律问题的解释》第18条第4款

一审：XXX法院（20XX）XXX号民事判决（20XX年XX月XX日）
```

Mark extracted laws as `verified: true` since they come from the official case database.

### Step 6: Rank and Score

Score each case (0-100) based on:
- Cause of action match (30%)
- Dispute focus match (30%)
- Legal basis relevance (20%)
- Case type match (10%)
- Procedure match (10%)

For each case, write one sentence explaining the match reason.

### Step 7: Output Case Cards

Render as Markdown cards:

```markdown
### 相似案例卡片

#### 1. {title}
- **匹配度**：{matchScore}%
- **案号**：{caseNumber}
- **法院**：{court} · {procedure}
- **裁判要旨**：{gist}
- **关联法条**：
  - ✓ 已验证：{law1}
  - ⚠ 未校验：{law2}
- **匹配理由**：{matchReason}
- **原文链接**：[查看详情]({detailUrl})
```

## Failure Strategy

- **Login fails / session expired**: Call `browser_waiting_for_user_interaction` to prompt re-login. If user declines, fall back to WebSearch.
- **rmfyalk returns 0 results**: Try a broader query (fewer keywords) or switch from exact to fuzzy search. If still 0, fall back to WebSearch with `site:rmfyalk.court.gov.cn {keywords}`.
- **Anti-crawl / CAPTCHA triggered**: Stop, inform user, fall back to WebSearch.
- **WebSearch fallback**: Inform user "已切换到搜索引擎兜底，精度可能下降". Results will lack full referee gist and verified legal basis.

## Anti-Patterns

- Do NOT fabricate case numbers — all case numbers must come from rmfyalk results
- Do NOT invent referee gist — only use text extracted from rmfyalk pages
- Do NOT mark laws as "verified" unless they were extracted from rmfyalk detail pages
- Do NOT search with specific names, amounts, or dates — use legal terms only
