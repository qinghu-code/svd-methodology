<p align="center">
  <img alt="SVD" src="https://img.shields.io/badge/SVD-Self--Validation%20Driven%20Development-6366f1?style=for-the-badge&logo=checkmarx&logoColor=white">
</p>

<p align="center">
  <b>TDD for the AI Coding Era</b> · <sub>AI 编码时代的 TDD</sub>
</p>

<p align="center">
  <i>TDD taught humans how to verify code. SVD teaches AI how to verify itself.<br>TDD 教会了人类如何验证代码。SVD 教会 AI 如何自己验证自己。</i>
</p>

---

## 📖 TOC / 目录

- [TDD vs SVD](#-tdd-vs-svd)
- [直观对比：用数据说话](#-直观对比用数据说话)
- [工作原理](#-工作原理)
- [实战验证](#-实战验证)
- [快速上手](#-快速上手)
- [各技术栈模式](#-各技术栈模式)
- [仓库结构](#-仓库结构)
- [为什么不能只用 TDD](#-为什么不能只用-tdd)
- [贡献](#-贡献)

---

## ⚡ TDD vs SVD

<table>
<tr>
<td width="50%">

<pre><code>TDD (1999)       Human-driven
TDD (1999)          人类驱动
──────────────────────────────────
 写测试 → 看失败 → 写代码
 看通过 → 重构

 🧑‍💻 Human-driven    人类驱动
 📐 Needs contracts 需要接口契约
 🔬 Unit-level      单元级
 ⏱️ 30s~5min loop   30s~5min
 ❓ Hypothetical    假设性验证
</code></pre>

</td>
<td width="50%">

<pre><code>SVD (2026)          AI-driven
SVD (2026)           AI 驱动
──────────────────────────────────
 写探针 → curl基线 → 改代码
 自校验 → 附证交付

 🤖 AI-driven        AI 驱动
 🔓 No contracts    无需契约
 🌐 System-level    系统级
 ⚡ 50ms loop        50ms
 ✅ Factual         事实性验证
</code></pre>

</td>
</tr>
</table>

| Dimension / 维度 | TDD 🧑‍💻 | SVD 🤖 |
|---|---|---|
| **Designed for** / 为谁设计 | Human developers / 人类开发者 | AI agents / AI |
| **Verifies** / 验证对象 | Code logic (unit) / 代码逻辑 | Running system (full stack) / 运行系统 |
| **Prerequisites** / 前置条件 | Interface contracts + test framework / 接口契约+测试框架 | One HTTP/CLI entry / 一个入口即可 |
| **Setup cost** / 启动成本 | 30 min ~ 2 hr / 30分~2小时 | **2 min** (5 lines) / **2 分钟**（5 行） |
| **Feedback speed** / 反馈速度 | 30 s ~ 5 min | **50 ms** (one curl) |
| **Coverage** / 覆盖层级 | Unit only / 仅单元级 | DB → IPC → State → UI / 全链路 |
| **Legacy projects** / 遗留项目 | Build test infra from scratch / 从零补测试 | Plug and play / 即插即用 |
| **Exploratory coding** / 探索式开发 | Can't test before design | Probe inspects any state |

---

## 📊 Side-by-Side Data / 直观对比用数据说话

> Data from 30+ real vibe coding sessions on the InfoHub project. Conservative estimates.
> 数据基于 InfoHub 项目 30+ 个真实 session，保守估算。
> Bars render perfectly in monospace — zero CSS dependency. / 等宽字体下比例精确，零 CSS 依赖。

### Feedback Time / 反馈时间

```
TDD  ████████████████████████████████████████████████  5 min     / 5 分钟
SVD  █▎                                                50 ms     −99%
```

### Human Round-Trips / Session / 人肉往返

```
TDD  ████████████████████████████████████████████████  3 ~ 4
SVD  ██████▋                                           0 ~ 1     −75%
```

### AI Self-Catch Rate / AI 自发现问题率

```
TDD  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  0%
SVD  ████████████████████████████████████████▊          ~80%      from zero / 从零到有
```

### Setup Cost / 启动成本

```
TDD  ████████████████████████████████████████████████  30 min ~ 2 hr
SVD  █▎                                                 2 min    −95%
```

### Probe Code Needed / 探针代码量

```
TDD  ████████████████████████████████████████████████  Full test suite / 完整测试套件
SVD  █▎                                                5~20 lines / 行    minimal / 极简
```

### One Typical 2-Hour Session / 典型 2 小时 session 的账本

| | Without SVD / 不用 | With SVD / 用 | |
|---|---|---|---|
| Changes / 改动次数 | 10 | 10 | |
| Regressions / 引发回归 | 3 ~ 4 | 3 ~ 4 | AI catches them / AI 自己发现 |
| **Human-found bugs** / **人发现的** | **3 ~ 4** | **0 ~ 1** | ← key difference / 关键差异 |
| Time per regression / 每次回归耗时 | 5 min | **30 sec** | |
| Human fix time / 人肉耗时 | ~18 min | **~2 min** | |
| **Net saved** / **净节省** | — | **~13 min / session** ✅ | |

---

## 🔍 How It Works / 工作原理

<table>
<tr>
<td width="50%">

### ❌ Before / 以前
**Human verification loop / 人肉验证循环**

<pre><code>AI changes code / AI 改代码
  ↓
"Should be fine" / "应该没问题了"
  ↓
👤 "No, XXX is broken"    ← human finds
👤 "不对，XXX 坏了"         ← 人发现
  ↓
AI: "Let me check..." / 我看下
  ↓
AI changes again / AI 再改
  ↓
👤 "XXX works but YYY broke"
👤 "XXX 好了但 YYY 又坏了"  ← again
  ↓
AI changes again / AI 再再改
  ↓
👤 "Finally works" / "终于好了"
            ← 3 rounds ~15 min
</code></pre>

</td>
<td width="50%">

### ✅ After / 现在
**SVD self-verification loop / 自校验循环**

<pre><code>AI writes probe (5 lines)
AI 写探针（5行）
  ↓
AI curls probe, records baseline
AI curl 探针，记录基线
  ↓
AI changes code / AI 改代码
  ↓
AI curls probe, compares to baseline
AI curl 探针，对比基线
  ↓
AI finds XXX missing, traces 3 paths
AI 发现 XXX 缺失，追溯到 3 处
  ↓
AI fixes all 3, curls all 3 ✅
全修、全验、全通过 ✅
  ↓
"Done. Evidence: icon_data ✅"
"搞定。证据：icon_data ✅"
            ← 1 round ~2 min
</code></pre>

</td>
</tr>
</table>

---

## 📋 Real-World Impact / 实战验证

> All data from InfoHub — Electron + SQLite desktop app built entirely via vibe coding.
> 全部数据来自 InfoHub 项目。

| Probe / 探针 | Bug Caught / 抓到的 Bug | Without It / 没有它的话 |
|---|---|---|
| `GET /debug/queue` | Queue persisted but invisible (multi-process WAL) / 持久化成功但 UI 不显示 | 4 rounds of "not working" / 4 轮"加不进去了" |
| `POST /debug/queue/add` | IPC chain missing handler / IPC 链路缺 handler | Root cause never isolated / 根因永远定不到 |
| `GET /debug/search/:type/:query` | 3 search paths all missing `icon_data` / 三路搜索全漏 | Fix one, miss two / 修一个漏两个 |
| `GET /debug/vk/:filter/:k` | vec0 `IN (...)` vs `=` filter difference | "Wrong results" + no debug path |
| `GET /debug/vec-test` | vec0 version-specific behavior / 版本差异 | Pure black-box guessing / 纯黑盒盲猜 |

---

## 🚀 Quick Start / 快速上手

**5 lines of code. Zero human feedback loops. / 5 行代码，零人类反馈循环。**

### ① Add a probe / 加探针

```typescript
// In your HTTP server / 在你的 HTTP 服务里:
if (req.method === 'GET' && req.url === '/debug/queue') {
  const items = db.prepare("SELECT value FROM config WHERE key='queue'").get();
  res.end(JSON.stringify({ count: items.length, items }));
}
```

### ② Check baseline / 查基线

```bash
$ curl http://localhost:37231/debug/queue
{"count":0, "items":[]}          # ← baseline / 基线
```

### ③ Change → Self-verify → Deliver / 改代码 → 自校验 → 交付

```bash
$ curl http://localhost:37231/debug/queue
{"count":1, "items":[...]}       # ✅ verified / 已验证
```

```
✅ Added queue persistence / 添加队列持久化
✅ Verified: POST /debug/queue/add → count:1, GET /debug/queue → persisted
```

---

## 🧩 Patterns by Stack / 各技术栈模式

> All patterns include production safeguards. / 所有模式默认带生产环境保护。

| Stack / 技术栈 | Pattern / 模式 | Guard / 防护 |
|---|---|---|
| **Electron / Node** | Probe routes on webhook server / 探针路由 | `app.isPackaged` |
| **Spring Boot** | `DebugController` | `@Profile("!prod")` |
| **Express** | Probe router / 探针路由 | `NODE_ENV !== 'production'` |
| **Flask** | Probe blueprint / 探针蓝图 | `app.config['DEBUG']` |
| **Go** | `SelfValidationHandler` | `os.Getenv("APP_DEBUG")` |
| **CLI** | `--probe` flag | Entrypoint check / 入口检查 |

> 📂 Full implementations → [`skills/self-validation-driven-development/SKILL.md`](skills/self-validation-driven-development/SKILL.md)

---

## 📁 Repository Structure / 仓库结构

```
svd-methodology/
├── README.md                                          ← You are here / 你在这里
├── skills/
│   ├── self-proving-delivery/SKILL.md                 ← Meta-pattern (all domains) / 通用方法论
│   ├── self-validation-driven-development/SKILL.md    ← SVD for coding / SVD 开发领域
│   └── cross-session-delivery/SKILL.md                ← Session handoff / 跨会话交付
├── rules/
│   └── self-proving-delivery.md                       ← Drop-in → AGENTS.md / 复制即用
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
TDD 是 1999 年为**人类迭代节奏**设计的方法论。它假设：

1. You know the interface upfront / 你开工前知道接口
2. You have a test framework ready / 测试框架就绪
3. You can build and test things in isolation / 构建物可隔离测试

**Vibe coding with AI breaks all three. / AI vibe coding 打破了所有三个假设。**

SVD is built for the reality of AI-assisted development: exploratory, multi-layer, runtime-verified.
SVD 为 AI 编码的现实而生：**探索性、跨层、运行时验证**。

> **TDD and SVD are complementary, not competing.**
> **TDD 和 SVD 互补，不是替代。**
> TDD verifies code contracts at the unit level. SVD verifies the running system at the integration level.
> TDD 在单元级验证代码契约。SVD 在系统级验证运行时真相。

---

## 🤝 Contributing / 贡献

SVD is a new methodology — patterns, examples, and critiques are welcome.
SVD 是一个新方法论——欢迎模式、实例、批判。

- 🐛 Caught a bug with a self-validation probe? / 用探针抓到了 bug？→ Add it to `examples/`
- 🧩 Implemented SVD in a new stack? / 在新栈上实现了 SVD？→ PR the pattern / PR 模式
- 💬 Disagree? / 不同意？→ Open an issue — adversarial feedback sharpens methodology / 开 issue

---

<br>

<p align="center">
  <b>Every human correction of an AI-catchable error is a self-validation failure.</b><br>
  <b>每一次人类帮 AI 发现的、AI 本可以自己发现的错误，都是一次自校验失败。</b>
</p>

<p align="center">
  <i>Don't let the user be your test suite. You have curl. Use it.</i><br>
  <i>别让用户当你的测试套件。你有 curl。用起来。</i>
</p>

<br>
