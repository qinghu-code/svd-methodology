# SVD — Self-Validation Driven Development / 自校验驱动开发

<p align="center">
  <b>TDD for the AI Coding Era / AI 编码时代的 TDD</b><br>
  <i>TDD taught humans how to verify code. SVD teaches AI how to verify itself.<br>TDD 教会了人类如何验证代码。SVD 教会 AI 如何自己验证自己。</i>
</p>

---

## TDD vs SVD / 对比一览

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              TDD (1999)                                       │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌─────────┐ │
│  │ Write test│───→│Watch fail│───→│Write code│───→│Watch pass│───→│Refactor │ │
│  │  写测试    │    │ 看失败    │    │ 写代码    │    │ 看通过    │    │  重构    │ │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘    └─────────┘ │
│  Human-driven · Needs contracts · Unit-level · Hypothetical · 30s~5min loop  │
│  人类驱动 · 需要接口契约 · 单元级 · 假设性 · 30s~5min 反馈                       │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│                              SVD (2025)                                       │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌─────────┐ │
│  │Write probe│───→│curl base │───→│Change code│───→│curl verify│───→│Deliver  │ │
│  │  写探针    │    │ curl基线  │    │  改代码    │    │ 自证校验   │    │ 附证交付 │ │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘    └─────────┘ │
│  AI-driven · No contracts · System-level · Factual · 50ms loop               │
│  AI 驱动 · 无需契约 · 系统级 · 事实性 · 50ms 反馈                                │
└──────────────────────────────────────────────────────────────────────────────┘
```

| Dimension / 维度 | TDD 🧑‍💻 | SVD 🤖 |
|------|---------|------|
| **Designed for / 为谁设计** | Human developers / 人类开发者 | AI agents / AI |
| **Verifies / 验证对象** | Code logic (unit) / 代码逻辑（单元） | Running system (full stack) / 运行系统（全栈） |
| **Prerequisites / 前置条件** | Interface contracts + test framework / 接口契约 + 测试框架 | One HTTP/CLI entry / 一个 HTTP/CLI 入口 |
| **Setup cost / 启动成本** | 30min ~ 2hr / 30 分~2 小时 | 2min (5 lines) / 2 分钟（5 行） |
| **Feedback speed / 反馈速度** | 30s ~ 5min / 30 秒~5 分钟 | 50ms (one curl) / 50 毫秒（一次 curl） |
| **Coverage / 覆盖层级** | Unit only / 仅单元级 | DB → IPC → State → UI / 全链路 |
| **Legacy projects / 遗留项目** | Build test infra from scratch / 从零补测试 | Plug and play / 即插即用 |
| **Exploratory coding / 探索式开发** | Can't test before design / 没设计完写不了测试 | Probe inspects any state / 探针随时看状态 |

---

## What SVD Saves You / 省多少

> Data from 30+ real vibe coding sessions on the InfoHub project. Conservative estimates.
> 数据基于 InfoHub 项目 30+ 个真实 session。保守估算。

```
A typical 2-hour vibe coding session / 一个典型的 2 小时 session：

                         │  Without SVD / 不用    │  With SVD / 用 SVD
═════════════════════════╪════════════════════════╪═══════════════════════════
 Changes / 改动次数       │  10                     │  10
 Regressions / 引发回归   │  3~4                    │  3~4 (AI catches them)
 Human-found bugs / 人发现│  3~4                    │  0~1
 Time per regression / 次 │  5 min (report→check→   │  30 sec (curl→see→fix)
                          │       fix→re-verify)    │
 Human fix time / 人肉耗时 │  ~18 min / session      │  ~2 min / session
 AI self-check / AI自检   │  0                      │  ~3 min / session
═════════════════════════╪════════════════════════╪═══════════════════════════
 Net saved / 净节省       │  —                      │  ~13 min / session
 User feels / 用户感受    │  "Broken again" ×4      │  "Works. No issues."
                          │  "怎么又坏了" ×4          │  "挺好，没毛病"
```

| Metric / 指标 | Number / 数值 |
|------|------|
| 🕐 **Regression feedback time / 回归反馈时间** | 5 min → **30 sec** (**−90%**) |
| 🔁 **Human round-trips / session / 人肉往返** | 3~4 → **0~1** (**−75%**) |
| 🎯 **AI self-catch rate / AI 自发现率** | 0% → **~80%** |
| 🧵 **Probe code needed / 探针代码量** | **5~20 lines each / 行** |
| 📉 **"Broken again" complaints / "又坏了"** | ~4/session → **~1/session** |

---

## How It Works / 怎么做到的

```
❌ Before / 以前 — Human verification loop (one bug's journey) / 人肉验证循环:
   AI changes code / AI 改代码
   → "Should be fine now / 应该没问题了"
   → 👤 "No, XXX is broken"                           ← Human finds / 人类发现
   → AI: "Let me check..."
   → AI changes again / AI 再改
   → "Should be fixed now / 这回应该好了"
   → 👤 "XXX works but YYY broke now"                 ← Human finds again
   → AI: "Checking again..."
   → AI changes again / AI 再再改
   → 👤 "Finally works"                                ← 3 rounds, ~15 min / 3 轮 ~15 分钟

✅ After / 现在 — SVD self-verification loop / 自校验循环:
   AI writes probe (5 lines) / AI 写探针（5行）
   → AI curls probe, records baseline / AI curl 探针，记录基线
   → AI changes code / AI 改代码
   → AI curls probe, compares to baseline / AI curl 探针，对比基线
   → AI finds XXX missing, traces to 3 search paths missing icon_data  ← AI self-finds
   → AI fixes all 3, curls all 3, confirms icon_data returned
   → "Done. Evidence: curl /debug/search/all/test → icon_data ✅"       ← 1 round, ~2 min
```

---

## Real-World Impact / 实战验证

All data from InfoHub — Electron + SQLite desktop app built entirely via vibe coding.
全部数据来自 InfoHub 项目。

| Probe / 探针 | Bug Caught / 抓到的 Bug | Without It / 没有它的话 |
|------|-----------|----------|
| `GET /debug/queue` | Queue persisted but invisible (multi-process WAL) / 持久化成功但 UI 不显示 | 4 rounds of "not working" / 4 轮"加不进去了" |
| `POST /debug/queue/add` | IPC chain missing handler / IPC 链路缺 handler | Root cause never isolated / 根因永远定不到 |
| `GET /debug/search/:type/:query` | 3 search paths all missing `icon_data` / 三路全漏 | Fix one, miss two / 修一个漏两个 |
| `GET /debug/vk/:filter/:k` | vec0 `IN (...)` vs `=` filter difference | "Wrong results" + no debug path |
| `GET /debug/vec-test` | vec0 version-specific behavior / 版本差异 | Pure black-box guessing / 纯黑盒 |

---

## Quick Start / 快速上手

### 1. Add a probe (5 lines) / 加探针（5 行）

```typescript
// In your HTTP server / 在你的 HTTP 服务里:
if (req.method === 'GET' && req.url === '/debug/queue') {
  const items = db.prepare("SELECT value FROM config WHERE key='queue'").get();
  res.end(JSON.stringify({ count: items.length, items }));
}
```

### 2. Check baseline / 查基线

```bash
$ curl http://localhost:37231/debug/queue
{"count":0, "items":[]}          # ← baseline / 基线
```

### 3. Change → Self-verify → Deliver / 改代码 → 自校验 → 交付

```bash
$ curl http://localhost:37231/debug/queue
{"count":1, "items":[...]}       # ✅ verified / 已验证
```

```
✅ Added queue persistence / 添加队列持久化
✅ Verified: POST /debug/queue/add → count:1, GET /debug/queue → persisted
```

That's SVD. 5 lines. Zero human feedback loops.
这就是 SVD。5 行代码。零次人类反馈循环。

---

## Patterns by Stack / 各技术栈模式

All patterns include production safeguards. / 所有模式默认带生产环境保护。

| Stack / 技术栈 | Pattern / 模式 | Guard / 防护 |
|-------|------|------|
| **Electron/Node** | Probe routes on webhook server / 探针路由 | `app.isPackaged` |
| **SpringBoot** | `DebugController` | `@Profile("!prod")` |
| **Express** | Probe router / 探针路由 | `NODE_ENV !== 'production'` |
| **Flask** | Probe blueprint / 探针蓝图 | `app.config['DEBUG']` |
| **Go** | `SelfValidationHandler` | `os.Getenv("APP_DEBUG")` |
| **CLI** | `--probe` flag | Entrypoint check / 入口检查 |

Full implementations → `skills/self-validation-driven-development/SKILL.md` / 完整实现 →

---

## Repository Structure / 仓库结构

```
svd-methodology/
├── README.md                                          ← You are here / 你在这里
├── skills/
│   ├── self-proving-delivery/SKILL.md                 ← Meta-pattern (all domains) / 通用方法论
│   ├── self-validation-driven-development/SKILL.md    ← SVD for coding / 开发领域
│   └── cross-session-delivery/SKILL.md                ← Session handoff / 跨会话交付
├── rules/
│   └── self-proving-delivery.md                       ← Drop-in AGENTS.md / 复制即用
└── examples/
    └── infohub-debug-endpoints.md                      ← Real probe catalog / 真实探针目录
```

| I want to... / 我想... | Read this / 看这个 |
|--------|--------|
| Understand SVD in 3 min / 3 分钟搞懂 SVD | This README / 本页 |
| Use SVD in coding / 写代码时用 SVD | `skills/self-validation-driven-development/SKILL.md` |
| Apply to docs, data, ops / 用到文档数据运维 | `skills/self-proving-delivery/SKILL.md` |
| Make it default every session / 每次会话自动生效 | Copy `rules/self-proving-delivery.md` → `AGENTS.md` |
| See real probes / 看真实探针实现 | `examples/infohub-debug-endpoints.md` |

---

## Why Not Just TDD? / 为什么不能只用 TDD？

TDD is a human methodology, designed in 1999 for human-paced iteration. It assumes you know the interface upfront, have a test framework, and build testable things in isolation.

TDD 是 1999 年为人类迭代节奏设计的方法论。它假设你开工前知道接口、测试框架就绪、构建物可隔离测试。

Vibe coding with AI breaks all three. SVD is built for the reality of AI-assisted development: exploratory, multi-layer, runtime-verified.

AI vibe coding 打破了所有三个假设。SVD 为 AI 编码的现实而生：探索性、跨层、运行时验证。

**TDD and SVD are complementary, not competing.** TDD verifies code contracts at the unit level. SVD verifies the running system at the integration level.

**TDD 和 SVD 互补，不是替代。** TDD 在单元级验证代码契约。SVD 在系统级验证运行时真相。

---

## Contributing / 贡献

SVD is a new methodology — patterns, examples, and critiques are welcome.

SVD 是一个新方法论——欢迎模式、实例、批判。

- Caught a bug with a self-validation probe? Add it to `examples/`. / 用自校验探针抓到了 bug？加到 `examples/`。
- Implemented SVD in a new stack (Rust, Ruby, Kotlin)? PR the pattern. / 在新栈上实现了 SVD？PR 模式。
- Disagree? Open an issue — adversarial feedback sharpens methodology. / 不同意？开 issue——反面意见打磨方法论。

---

<p align="center">
  <b>Every human correction of an AI-catchable error is a self-validation failure.</b><br>
  <b>每一次人类帮 AI 发现的、AI 本可以自己发现的错误，都是一次自校验失败。</b><br><br>
  <i>Don't let the user be your test suite.<br>别让用户当你的测试套件。<br>你有 curl。用起来。You have curl. Use it.</i>
</p>
