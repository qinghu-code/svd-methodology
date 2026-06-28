---
name: cross-session-delivery
description: Use when ending a session with unfinished work, starting a session that continues prior work, or when stuck on a problem that resists multiple fix attempts. Triggers include: user says "continue" / "next session" / "keep working on this", a NEXT_SESSION.md file exists in the project, the same bug has been fixed 3+ times without success, or the user wants to package context for a future session.
---

# Cross-Session Delivery

## Overview

Package unfinished work so the next session can continue without the user repeating context. Different project phases need different handoff formats — a v0.1 prototype handoff looks nothing like a v0.9 polish handoff.

## When to Use

**Ending a session:**
- User says "continue next time" / "next session" / "keep working on this"
- You've completed a phase but there are follow-up tasks
- The user explicitly asks you to write NEXT_SESSION.md

**Starting a session:**
- NEXT_SESSION.md or MEMORY.md exists in the project
- User says "continue" / "pick up where we left off"

**Resetting thinking:**
- Same bug fixed 3+ times and still broken
- Solutions getting more complex instead of simpler
- User says "不对" more than twice on the same issue
- You realize the current approach has a fundamental flaw

**When NOT to use:**
- The task is trivial and fits in one session
- User explicitly says not to bother
- You're mid-flow and the session naturally continues

## The Three-Phase Model

Projects mature through phases. The handoff format must match the phase.

| Phase | Maturity | Handoff Format | Key Principle |
|-------|----------|---------------|---------------|
| **初期** (0→6) | Core prototype | Light: reference the Spec | Spec IS the handoff |
| **中期** (6→9) | Architecture refinement | Standard: structured task list | Precise scope + references |
| **后期** (9→10) | Polish & verification | Minimal: specific bug/feedback | One item at a time |

### How to Identify the Phase

Ask yourself (or the user):

```
初期 signals: "I want to build X" (but details not decided)
             "Add feature Y" (core functionality, not refinement)
             Spec/plan docs exist and are the primary reference

中期 signals: "Refactor X to use Y pattern"
             "X and Y are redundant, merge them"
             "Add debug endpoints for X"
             Architecture-level changes with clear direction

后期 signals: "The icon is gray" (UI detail)
             "This text should be aligned" (visual polish)
             "The count is 642 but should be 643" (precise expected value)
             Documentation polish, theme compatibility, packaging
```

## The Core Handoff File: NEXT_SESSION.md

### Template (初期 — Light)

Used when a comprehensive Spec exists:

```markdown
# 新会话引导词：<one-line summary>

请继续执行以下任务：

## 背景
<1-2 sentences why>

## 详细计划
参考 spec 文件：`docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

## 执行步骤
1. 先读 <key files, 3-5 max>
2. 按照 spec 中的「涉及文件与改动」逐文件修改
3. 重点：<2-3 critical gotchas>
4. 参照 <existing pattern to follow>（已完成，可参考）

## 验收标准
- `npm run build` 无错误
- <3-7 verifiable checkpoints>
```

### Template (中期 — Standard)

Used for targeted architecture changes:

```markdown
# 新会话引导词：<one-line summary>

## 背景
<Why this change. What's the current problem.>

## 涉及文件
1. `path/to/file1.ts` — <what to change, which method/function>
2. `path/to/file2.ts` — <what to change>
3. `path/to/file3.ts` — <what to change>

## 执行步骤
1. 先读 <all involved files>
2. <Step 1 with specific SQL/code changes>
3. <Step 2>
4. 参照 <reference pattern/commit>
5. 构建通过后重启验证

## 重点
- <Critical thing that's easy to miss>
- <Another gotcha>

## 验收标准
- [ ] `npm run build` 无错误
- [ ] <Verification point 1>
- [ ] <Verification point 2>
```

### Template (后期 — Minimal)

Used for bug fixes and polish:

```markdown
# 新会话引导词：<one-line summary>

## 现象
<User's exact words describing the issue>

## 排查方向
- <Likely cause area 1>
- <Likely cause area 2>

## 涉及文件
- `path/to/file.ts`

## 验收
- [ ] <Specific expected behavior>
```

### What NOT to Include

- ❌ Full code dumps (they're in git)
- ❌ Session transcripts ("then I said... then you said...")
- ❌ Exploration dead ends that were abandoned
- ❌ Spec content that's already in the spec file
- ❌ Generic advice like "be careful"

### What to ALWAYS Include

- ✅ File paths (absolute or from project root)
- ✅ Specific method/function names to change
- ✅ Exact commit hashes to reference as patterns
- ✅ Verifiable acceptance criteria (build passes, specific behavior)
- ✅ Critical gotchas that are easy to miss

## Fresh Session Reset

Starting a new session isn't just about context limits. It's a tool for breaking out of wrong approaches.

### The Trap

In long sessions, AI forms mental inertia. Once committed to an approach (say, using `max()` for count merging), subsequent reasoning defends it — even when it's wrong. Each fix patches symptoms without questioning the root approach. The solution gets more complex, not simpler.

### The Reset

A fresh session:
- Re-reads the code from scratch — no cached assumptions
- Evaluates the design objectively — not defending prior choices
- Spots contradictions the previous session rationalized away

### When to Call for a Reset

| Signal | Action |
|--------|--------|
| Same bug fixed 3+ times, still broken | **Stop. Request fresh session.** |
| Fixes are getting more complex, not simpler | **Stop. Request fresh session.** |
| User says "不对" more than twice | **Stop. Request fresh session.** |
| You realize the approach has a fundamental flaw | **Stop. Write NEXT_SESSION.md for fresh session.** |

### How to Initiate a Reset

```markdown
I think we should start a fresh session for this.

Current approach: [what we've been trying]
Problem: [why it's not working]
Files involved: [list]

I'll write NEXT_SESSION.md with the current state and what to investigate.

In the fresh session, the AI will re-read the code without our assumptions
and may find a simpler approach.
```

## Phase Transition Detection

The AI should **proactively detect** when the project has transitioned phases and adjust handoff format:

| Signal | Phase Change |
|--------|-------------|
| User starts giving specific UI feedback ("the icon is gray") | 中期 → 后期 |
| User says "refactor X" or "X and Y are redundant" | 初期 → 中期 |
| User asks to write documentation/README | 中期 → 后期 |
| User asks to package/release | 后期 |

When you detect a phase transition, mention it:

```
I notice we've moved from [old phase] to [new phase].
For this handoff, I'll use the [new phase] format which is [more/less] detailed.
```

## The Self-Check

Before writing NEXT_SESSION.md, verify:

1. **Can a fresh AI understand this without reading our entire conversation?**
   - If no: add more context (file paths, method names, spec references)
   - If yes: you're good

2. **Are the acceptance criteria objectively verifiable?**
   - "Search works" → ❌ ambiguous
   - "`npm run build` 无错误 + 搜索面板站点图标非灰色" → ✅ verifiable

3. **Are there reference commits/patterns listed?**
   - If a similar change was done before, reference its commit hash
   - "参照 apps 迁移 (commit 5a94515)" is infinitely better than "do it like apps"

4. **Is the phase appropriate?**
   - 初期 handoff with 后期-level detail = over-engineered, wastes tokens
   - 后期 handoff with 初期-level vagueness = useless, AI will guess wrong

## MEMORY.md Integration

NEXT_SESSION.md is for the NEXT session. MEMORY.md is for ALL future sessions.

### What goes in MEMORY.md

Facts that remain true across sessions:
```markdown
- [apps 表融入 documents 计划](apps-merge-into-documents.md)
  — 删除 apps/app_usage_log，统一到 documents(source_type='app')
```

### What goes in NEXT_SESSION.md

The specific task to do next:
```markdown
# 新会话引导词：sites 表融入 documents
## 执行步骤
1. 先读 migrations.sql, launcher.ts, webhook/server.ts, controller/home.ts
2. 按照 spec 逐文件修改
```

### The Split

| MEMORY.md | NEXT_SESSION.md |
|-----------|----------------|
| "X happened" (facts) | "Do X next" (tasks) |
| Persistent across sessions | Consumed by next session, then archived |
| Accumulates over time | Replaced each session |

## Common Mistakes

### 1. Writing NEXT_SESSION.md like a transcript
```
❌ "Then you said the icon was gray, and I checked the code..."
✅ "搜索面板站点图标灰色 — 排查方向: searchFts/searchLike/searchVector 是否选了 icon_data"
```

### 2. Not referencing existing patterns
```
❌ "Merge sites into documents"
✅ "参照 apps 迁移模式 (commit 5a94515): 删除 sites 表, 统一到 documents(source_type='site')"
```

### 3. Vague acceptance criteria
```
❌ "Make sure it works"
✅ "- `npm run build` 无错误
   - 首页站点列表正常（浏览器历史 + 自追踪合并）
   - 搜索站点无重复结果
   - sites 表已自动删除"
```

### 4. Using wrong phase format
```
❌ 初期 phase: 700-line NEXT_SESSION with every method signature
✅ 初期 phase: "参考 spec → 按涉及文件改动 → build → 验证"
```

### 5. Including abandoned approaches
```
❌ "We tried max() but it didn't work, then we tried add() but that was wrong too..."
✅ "合并逻辑：浏览器 visit_count + 自追踪 access_count（add 语义，非 max）"
```

## Quick Reference

| I want to... | Use this |
|-------------|----------|
| End session, big feature in progress | 初期 template → reference the Spec |
| End session, targeted refactor | 中期 template → file list + steps |
| End session, bug fix | 后期 template → symptom + investigation direction |
| Start session, pick up work | Read NEXT_SESSION.md → identify phase → execute |
| Stuck on same bug 3+ times | **Call for reset** → write NEXT_SESSION → new session |
| Save fact for all future sessions | MEMORY.md |
| Save task for next session only | NEXT_SESSION.md |
