<p align="center">
  <br>
  <picture>
    <source media="(prefers-color-scheme: dark)">
    <img alt="SVD" src="https://img.shields.io/badge/SVD-Self--Validation%20Driven%20Development-6366f1?style=for-the-badge&logo=checkmarx&logoColor=white">
  </picture>
</p>

<p align="center">
  <b>TDD for the AI Coding Era</b><br>
  <sub>AI 编码时代的 TDD</sub>
</p>

<p align="center">
  <i>TDD taught humans how to verify code. SVD teaches AI how to verify itself.<br>TDD 教会了人类如何验证代码。SVD 教会 AI 如何自己验证自己。</i>
</p>

<br>

---

## 📖 目录 / TOC

- [TDD vs SVD](#-tdd-vs-svd-对比一览)
- [What SVD Saves You](#-what-svd-saves-you-省多少)
- [How It Works](#-how-it-works-怎么做到的)
- [Real-World Impact](#-real-world-impact-实战验证)
- [Quick Start](#-quick-start-快速上手)
- [Patterns by Stack](#-patterns-by-stack-各技术栈模式)
- [Repository Structure](#-repository-structure-仓库结构)
- [Why Not Just TDD?](#-why-not-just-tdd-为什么不能只用-tdd)
- [Contributing](#-contributing-贡献)

---

## ⚡ TDD vs SVD / 对比一览

<table>
<tr>
<td width="50%" valign="top">

### TDD `(1999)`

```
 Write Test  →  Watch Fail  →  Write Code  →  Watch Pass  →  Refactor
   写测试         看失败          写代码          看通过           重构
```

> 🧑‍💻 Human-driven · 📐 Needs contracts · 🔬 Unit-level · ❓ Hypothetical · ⏱️ 30s~5min loop
>
> 人类驱动 · 需要接口契约 · 单元级 · 假设性 · 30s~5min 反馈

</td>
<td width="50%" valign="top">

### SVD `(2026)`

```
 Write Probe →  curl Baseline →  Change Code →  curl Verify  →  Deliver
   写探针         curl 基线         改代码         自证校验          附证交付
```

> 🤖 AI-driven · 🔓 No contracts · 🌐 System-level · ✅ Factual · ⚡ 50ms loop
>
> AI 驱动 · 无需契约 · 系统级 · 事实性 · 50ms 反馈

</td>
</tr>
</table>

| Dimension / 维度 | TDD 🧑‍💻 | SVD 🤖 |
|---|---|---|
| **Designed for** / 为谁设计 | Human developers / 人类开发者 | AI agents / AI |
| **Verifies** / 验证对象 | Code logic (unit) / 代码逻辑 | Running system (full stack) / 运行系统 |
| **Prerequisites** / 前置条件 | Interface contracts + test framework / 接口契约 + 测试框架 | One HTTP/CLI entry / 一个入口即可 |
| **Setup cost** / 启动成本 | 30 min ~ 2 hr / 30分~2小时 | **2 min** (5 lines) / **2 分钟**（5 行） |
| **Feedback speed** / 反馈速度 | 30 s ~ 5 min | **50 ms** (one curl) / **50 毫秒**（一次 curl） |
| **Coverage** / 覆盖层级 | Unit only / 仅单元级 | DB → IPC → State → UI / 全链路 |
| **Legacy projects** / 遗留项目 | Build test infra from scratch / 从零补测试 | Plug and play / 即插即用 |
| **Exploratory coding** / 探索式开发 | Can't test before design / 没设计完写不了测试 | Probe inspects any state / 探针随时看状态 |

---

## 💰 What SVD Saves You / 省多少

> Data from 30+ real vibe coding sessions on the InfoHub project. Conservative estimates.
> 数据基于 InfoHub 项目 30+ 个真实 session，保守估算。

**A typical 2-hour vibe coding session / 一个典型的 2 小时 session：**

| | Without SVD / 不用 SVD | With SVD / 用 SVD |
|---|---|---|
| **Changes** / 改动次数 | 10 | 10 |
| **Regressions** / 引发回归 | 3 ~ 4 | 3 ~ 4 *(AI catches them)* |
| **Human-found bugs** / 人发现 | 3 ~ 4 | **0 ~ 1** |
| **Time per regression** / 每次回归耗时 | 5 min (report → check → fix → re-verify) | **30 sec** (curl → see → fix) |
| **Human fix time** / 人肉耗时 | ~18 min / session | **~2 min / session** |
| **AI self-check** / AI 自检 | 0 | ~3 min / session |
| **Net saved** / 净节省 | — | **~13 min / session** |

| Metric / 指标 | Number / 数值 |
|---|---|
| 🕐 **Regression feedback time** / 回归反馈时间 | 5 min → **30 sec (`−90%`)** |
| 🔁 **Human round-trips / session** / 人肉往返 | 3~4 → **0~1 (`−75%`)** |
| 🎯 **AI self-catch rate** / AI 自发现率 | 0% → **~80%** |
| 🧵 **Probe code needed** / 探针代码量 | **5~20 lines each** / 行 |
| 📉 **"Broken again" complaints** / "又坏了" | ~4/session → **~1/session** |

---

## 🔍 How It Works / 怎么做到的

<table>
<tr>
<td width="50%" valign="top">

### ❌ Before / 以前
**Human verification loop (one bug's journey) / 人肉验证循环**

```
AI changes code / AI 改代码
  ↓
"Should be fine now" / "应该没问题了"
  ↓
👤 "No, XXX is broken"          ← 人类发现
  ↓
AI: "Let me check..."
  ↓
AI changes again / AI 再改
  ↓
"Should be fixed now" / "这回应该好了"
  ↓
👤 "XXX works but YYY broke now" ← 又发现
  ↓
AI: "Checking again..."
  ↓
AI changes again / AI 再再改
  ↓
👤 "Finally works"              ← 3 轮 ~15 min
```

</td>
<td width="50%" valign="top">

### ✅ After / 现在
**SVD self-verification loop / 自校验循环**

```
AI writes probe (5 lines) / AI 写探针（5行）
  ↓
AI curls probe, records baseline / curl 探针，记基线
  ↓
AI changes code / AI 改代码
  ↓
AI curls probe, compares to baseline / curl 探针，对比基线
  ↓
AI finds XXX missing, traces to 3 paths ← AI 自发现
  ↓
AI fixes all 3, curls all 3, confirms ✅
  ↓
"Done. Evidence: icon_data ✅"  ← 1 轮 ~2 min
```

</td>
</tr>
</table>

---

## 📊 Real-World Impact / 实战验证

> All data from InfoHub — Electron + SQLite desktop app built entirely via vibe coding.
> 全部数据来自 InfoHub 项目。

| Probe / 探针 | Bug Caught / 抓到的 Bug | Without It / 没有它的话 |
|---|---|---|
| `GET /debug/queue` | Queue persisted but invisible (multi-process WAL) / 持久化成功但 UI 不显示 | 4 rounds of "not working" |
| `POST /debug/queue/add` | IPC chain missing handler / IPC 链路缺 handler | Root cause never isolated |
| `GET /debug/search/:type/:query` | 3 search paths all missing `icon_data` / 三路全漏 | Fix one, miss two |
| `GET /debug/vk/:filter/:k` | vec0 `IN (...)` vs `=` filter difference | "Wrong results" + no debug path |
| `GET /debug/vec-test` | vec0 version-specific behavior / 版本差异 | Pure black-box guessing |

---

## 🚀 Quick Start / 快速上手

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

> That's SVD. 5 lines. Zero human feedback loops.
> 这就是 SVD。5 行代码。零次人类反馈循环。

---

## 🧩 Patterns by Stack / 各技术栈模式

> All patterns include production safeguards. / 所有模式默认带生产环境保护。

| Stack / 技术栈 | Pattern / 模式 | Guard / 防护 |
|---|---|---|
| **Electron / Node** | Probe routes on webhook server | `app.isPackaged` |
| **Spring Boot** | `DebugController` | `@Profile("!prod")` |
| **Express** | Probe router | `NODE_ENV !== 'production'` |
| **Flask** | Probe blueprint | `app.config['DEBUG']` |
| **Go** | `SelfValidationHandler` | `os.Getenv("APP_DEBUG")` |
| **CLI** | `--probe` flag | Entrypoint check |

> 📂 Full implementations → [`skills/self-validation-driven-development/SKILL.md`](skills/self-validation-driven-development/SKILL.md)

---

## 📁 Repository Structure / 仓库结构

```
svd-methodology/
├── README.md                                          ← You are here / 你在这里
├── skills/
│   ├── self-proving-delivery/SKILL.md                 ← Meta-pattern (all domains)
│   ├── self-validation-driven-development/SKILL.md    ← SVD for coding / 开发领域
│   └── cross-session-delivery/SKILL.md                ← Session handoff / 跨会话交付
├── rules/
│   └── self-proving-delivery.md                       ← Drop-in AGENTS.md / 复制即用
└── examples/
    └── infohub-debug-endpoints.md                      ← Real probe catalog / 真实探针目录
```

| I want to... / 我想... | Read this / 看这个 |
|---|---|
| Understand SVD in 3 min / 3 分钟搞懂 SVD | This README / 本页 |
| Use SVD in coding / 写代码时用 SVD | [`skills/self-validation-driven-development/SKILL.md`](skills/self-validation-driven-development/SKILL.md) |
| Apply to docs, data, ops / 用到文档、数据、运维 | [`skills/self-proving-delivery/SKILL.md`](skills/self-proving-delivery/SKILL.md) |
| Make it default every session / 每次会话自动生效 | Copy `rules/self-proving-delivery.md` → `AGENTS.md` |
| See real probes / 看真实探针实现 | [`examples/infohub-debug-endpoints.md`](examples/infohub-debug-endpoints.md) |

---

## ❓ Why Not Just TDD? / 为什么不能只用 TDD？

TDD is a human methodology, designed in 1999 for human-paced iteration. It assumes:
1. You know the interface upfront
2. You have a test framework ready
3. You can build and test things in isolation

**Vibe coding with AI breaks all three assumptions.**

TDD 是 1999 年为人类迭代节奏设计的方法论。它假设你开工前知道接口、测试框架就绪、构建物可隔离测试。**AI vibe coding 打破了所有三个假设。**

SVD is built for the reality of AI-assisted development: exploratory, multi-layer, runtime-verified.
SVD 为 AI 编码的现实而生：探索性、跨层、运行时验证。

> **TDD and SVD are complementary, not competing.** TDD 在单元级验证代码契约。SVD 在系统级验证运行时真相。
>
> TDD verifies code contracts at the unit level. SVD verifies the running system at the integration level.

---

## 🤝 Contributing / 贡献

SVD is a new methodology — patterns, examples, and critiques are welcome.
SVD 是一个新方法论——欢迎模式、实例、批判。

- 🐛 Caught a bug with a self-validation probe? → Add it to `examples/`
- 🧩 Implemented SVD in a new stack (Rust, Ruby, Kotlin)? → PR the pattern
- 💬 Disagree? → Open an issue — adversarial feedback sharpens methodology

---

<br>

<p align="center">
  <b>Every human correction of an AI-catchable error is a self-validation failure.</b><br>
  <sub>每一次人类帮 AI 发现的、AI 本可以自己发现的错误，都是一次自校验失败。</sub>
</p>

<p align="center">
  <i>Don't let the user be your test suite. You have curl. Use it.</i><br>
  <sub>别让用户当你的测试套件。你有 curl。用起来。</sub>
</p>

<br>
