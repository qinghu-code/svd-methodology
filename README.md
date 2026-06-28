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

## 📖 目录 / TOC

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

```
TDD (1999)                             人类驱动
──────────────────────────────────────────────
 写测试 → 看失败 → 写代码 → 看通过 → 重构

 🧑‍💻 人类驱动      📐 需要接口契约
 🔬 单元级          ⏱️ 30s~5min 反馈
 ❓ 假设性验证
```

</td>
<td width="50%">

```
SVD (2026)                             AI 驱动
──────────────────────────────────────────────
 写探针 → curl基线 → 改代码 → 自校验 → 附证交付

 🤖 AI 驱动         🔓 无需契约
 🌐 系统级          ⚡ 50ms 反馈
 ✅ 事实性验证
```

</td>
</tr>
</table>

| | TDD 🧑‍💻 | SVD 🤖 |
|---|---|---|
| **为谁设计** | 人类开发者 | AI agents |
| **验证对象** | 代码逻辑（单元） | 运行系统（全栈） |
| **前置条件** | 接口契约 + 测试框架 | 一个 HTTP/CLI 入口 |
| **启动成本** | 30分 ~ 2小时 | **2 分钟**（5 行） |
| **反馈速度** | 30秒 ~ 5分钟 | **50 毫秒**（一次 curl） |
| **覆盖层级** | 仅单元级 | DB → IPC → State → UI |
| **遗留项目** | 从零补测试 | 即插即用 |
| **探索式开发** | 没设计完写不了测试 | 探针随时看状态 |

---

## 📊 直观对比：用数据说话

> 数据基于 InfoHub 项目 30+ 个真实 session，保守估算。

### 反馈时间

```
TDD  ████████████████████████████████████████████████████  5 分钟
SVD  ██  50 毫秒  (−99%)
```

### 人肉往返次数 / session

```
TDD  ████████████████████████████████████████████████████  3 ~ 4 次
SVD  ████  0 ~ 1 次  (−75%)
```

### AI 自发现问题率

```
TDD  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  0%
SVD  ████████████████████████████████████████████████████  ~80%
```

### 启动成本

```
TDD  ████████████████████████████████████████████████████  30分 ~ 2小时
SVD  █  2 分钟  (−95%)
```

### 探针代码量

```
TDD  ████████████████████████████████████████████████████  完整测试套件
SVD  █  5~20 行
```

### 一个典型 2 小时 session 的账本

```
                    不用 SVD                 用 SVD
改动次数            10                       10
引发回归            3~4                      3~4 (AI 自己发现)
人发现的 bug        3~4                      0~1  ← 关键差异
每次回归耗时        5 min (报→查→修→验)       30 sec (curl→看→修)
人肉耗时/session    ~18 min                  ~2 min
净节省              —                        ~13 min / session  ✅
```

---

## 🔍 工作原理

<table>
<tr>
<td width="50%">

### ❌ 以前：人肉验证循环

```
AI 改代码
  ↓
"应该没问题了"
  ↓
👤 "不对，XXX 坏了"       ← 人发现
  ↓
AI: "我看一下..."
  ↓
AI 再改
  ↓
"这回应该好了"
  ↓
👤 "XXX 好了但 YYY 又坏了" ← 人又发现
  ↓
AI 再再改
  ↓
👤 "终于好了"             ← 3轮 ~15分钟
```

</td>
<td width="50%">

### ✅ 现在：SVD 自校验循环

```
AI 写探针（5行）
  ↓
AI curl 探针，记基线
  ↓
AI 改代码
  ↓
AI curl 探针，对比基线
  ↓
AI 发现 XXX 缺失 → 追溯到3处 ← AI自发现
  ↓
全修、全验、全通过 ✅
  ↓
"搞定。证据：icon_data ✅" ← 1轮 ~2分钟
```

</td>
</tr>
</table>

---

## 📋 实战验证

> 全部数据来自 InfoHub — Electron + SQLite 桌面应用，纯 vibe coding 构建。

| 探针 | 抓到的 Bug | 没有它的话 |
|---|---|---|
| `GET /debug/queue` | 持久化成功但 UI 不显示（多进程 WAL） | 4 轮"加不进去了" |
| `POST /debug/queue/add` | IPC 链路缺 handler | 根因永远定不到 |
| `GET /debug/search/:type/:query` | 三路搜索全漏 `icon_data` | 修一个漏两个 |
| `GET /debug/vk/:filter/:k` | vec0 `IN (...)` vs `=` 差异 | "结果不对" + 无调试路径 |
| `GET /debug/vec-test` | vec0 版本差异行为 | 纯黑盒盲猜 |

---

## 🚀 快速上手

**5 行代码，零人类反馈循环。**

### ① 加探针

```typescript
if (req.method === 'GET' && req.url === '/debug/queue') {
  const items = db.prepare("SELECT value FROM config WHERE key='queue'").get();
  res.end(JSON.stringify({ count: items.length, items }));
}
```

### ② 查基线

```bash
$ curl http://localhost:37231/debug/queue
{"count":0, "items":[]}          # ← 基线
```

### ③ 改代码 → 自校验 → 交付

```bash
$ curl http://localhost:37231/debug/queue
{"count":1, "items":[...]}       # ✅ 已验证
```

```
✅ 添加队列持久化
✅ 验证通过: POST /debug/queue/add → count:1, GET /debug/queue → persisted
```

---

## 🧩 各技术栈模式

> 所有模式默认带生产环境保护。

| 技术栈 | 模式 | 防护 |
|---|---|---|
| **Electron / Node** | 探针路由挂 webhook server | `app.isPackaged` |
| **Spring Boot** | `DebugController` | `@Profile("!prod")` |
| **Express** | 探针 router | `NODE_ENV !== 'production'` |
| **Flask** | 探针 blueprint | `app.config['DEBUG']` |
| **Go** | `SelfValidationHandler` | `os.Getenv("APP_DEBUG")` |
| **CLI** | `--probe` flag | 入口检查 |

> 📂 完整实现 → [`skills/self-validation-driven-development/SKILL.md`](skills/self-validation-driven-development/SKILL.md)

---

## 📁 仓库结构

```
svd-methodology/
├── README.md                                          ← 你在这里
├── skills/
│   ├── self-proving-delivery/SKILL.md                 ← 通用方法论（全领域）
│   ├── self-validation-driven-development/SKILL.md    ← SVD 开发领域
│   └── cross-session-delivery/SKILL.md                ← 跨会话交付
├── rules/
│   └── self-proving-delivery.md                       ← 复制即用 → AGENTS.md
└── examples/
    └── infohub-debug-endpoints.md                      ← 真实探针目录
```

| 我想... | 看这个 |
|---|---|
| 3 分钟搞懂 SVD | 本页 |
| 写代码时用 SVD | [`skills/self-validation-driven-development/SKILL.md`](skills/self-validation-driven-development/SKILL.md) |
| 用到文档/数据/运维 | [`skills/self-proving-delivery/SKILL.md`](skills/self-proving-delivery/SKILL.md) |
| 每次会话自动生效 | 复制 `rules/self-proving-delivery.md` → `AGENTS.md` |
| 看真实探针实现 | [`examples/infohub-debug-endpoints.md`](examples/infohub-debug-endpoints.md) |

---

## ❓ 为什么不能只用 TDD？

TDD 是 1999 年为**人类迭代节奏**设计的方法论。它假设：

1. 你开工前知道接口
2. 测试框架就绪
3. 构建物可隔离测试

**AI vibe coding 打破了所有三个假设。**

SVD 为 AI 编码的现实而生：**探索性、跨层、运行时验证**。

> TDD 和 SVD 互补，不是替代。TDD 在单元级验证代码契约；SVD 在系统级验证运行时真相。

---

## 🤝 贡献

SVD 是一个新方法论——欢迎模式、实例、批判。

- 🐛 用自校验探针抓到了 bug？→ 加到 `examples/`
- 🧩 在新栈上实现了 SVD？→ PR 模式
- 💬 不同意？→ 开 issue，反面意见打磨方法论

---

<br>

<p align="center">
  <b>每一次人类帮 AI 发现的、AI 本可以自己发现的错误，都是一次自校验失败。</b>
</p>

<p align="center">
  <i>别让用户当你的测试套件。你有 curl。用起来。</i>
</p>

<br>
