---
name: self-validation-driven-development
description: Use when coding tasks involve high complexity (multi-file, multi-layer, cross-module), high uncertainty (exploratory refactoring, unclear requirements, figuring out as you go), complex state (queues, caches, IPC, multi-process, migrations), or high user-feedback friction (same issue reported 2+ times, hard-to-resolve bugs). Also trigger proactively when modifying shared infrastructure where any breakage would cascade silently. SVD provides the self-verification loop (probe → curl baseline → change → self-verify → deliver with evidence) to cut human feedback out of the verification cycle.
---

# Self-Validation Driven Development (SVD)

> TDD asks: "Does my code do what I think?" — hypothetical, unit-level, requires interface contracts.
> **SVD asks: "Does the running system actually work?"** — factual, system-level, one `curl` answers it.

## What is SVD

**Self-Validation Driven Development** is a verification methodology designed for AI-assisted coding. The core principle: the AI doesn't deliver with "I think this should work" — it delivers with runtime evidence from debug endpoints (JSON, counts, state snapshots) that **prove** the change is correct.

```
TDD cycle:  Write test → Watch fail → Write code → Watch pass → Refactor
SVD cycle:  Write probe → curl baseline → Change code → Self-verify → Deliver with evidence
```

**Core principle:** If you can't prove your change works without asking the user, you haven't finished the change.

## Why SVD Beats Pure TDD in Vibe Coding

TDD is powerful but mechanical: it assumes you know the interface upfront, have a test framework wired, and are building something testable in isolation. Real-world vibe coding is messier:

| Scenario | TDD's Struggle | SVD's Solution |
|----------|---------------|----------------|
| **Legacy codebase, zero tests** | Build test infrastructure from scratch — 2+ hours before first assertion | 10-line `GET /debug/queue` → instant state verification |
| **Cross-layer changes** (DB → IPC → state → UI) | Unit tests mock each layer; integration tests are a separate project | One `curl /debug/search/app/test` verifies the entire chain end-to-end |
| **Runtime state** (WAL mode, multi-process, file paths) | Can't unit-test OS-level behavior | `GET /debug/db/:table/count` confirms writes actually landed |
| **Exploratory coding** (final shape unknown) | Can't write tests for code you haven't designed yet | Debug probes inspect any internal state — no contract needed |
| **Regression hunting** ("Fixed A, B broke") | Existing tests pass but real system is broken — test gap | `curl` all related probes → find the regression in 30 seconds |
| **Config/environment issues** (data dir, env var, paths) | Unit tests use fake paths | Probe returns real `getDataDir()` → immediately see if it's wrong |
| **Feedback speed** (need verification in seconds) | Test suite: 30s–5min | Single `curl`: 50ms |

**TDD and SVD are complementary, not competing.** TDD verifies code contracts at the unit level. SVD verifies the running system at the integration level. In mid-to-late project vibe coding, the integration-level bugs are exactly what cause "broken again" feedback loops — and exactly what SVD catches.

## When to Use — Proactive, Not Just for Bugs

SVD isn't just for when things break. These four dimensions should trigger it **before you start coding**:

### Four Trigger Dimensions

```
┌──────────────────────────────────────────────────────┐
│ HIGH COMPLEXITY ←────────────────→ HIGH UNCERTAINTY  │
│ multi-file / multi-layer           fuzzy requirements │
│ cross-module / shared infra        exploratory work   │
│                                                       │
│ COMPLEX STATE ←───────────────────→ HIGH FRICTION     │
│ queue / cache / IPC / WAL          repeated feedback  │
│ migration / config / multi-process  same issue 2×+    │
└──────────────────────────────────────────────────────┘
```

### Trigger Signals

**High Complexity:**
- Change spans 3+ files across 2+ layers (DB, IPC, state, UI)
- Modifying shared infrastructure (schema, migrations, paths, config store)
- Implementing a feature whose correctness depends on multiple modules
- You catch yourself thinking "I'm not sure what else this might affect"

**High Uncertainty:**
- Requirements are still evolving; final shape unknown
- Multiple implementation approaches, unclear which is optimal
- External dependency behavior may vary (library version, OS, runtime)
- First time touching this module; internal behavior not fully understood

**Complex State:**
- Involves persistent state (DB writes, queues, caches, filesystem)
- Multi-process / WAL mode / concurrent writes
- State migrations (localStorage → SQLite, sites table → documents table)
- Config changes (data dir, model config, environment variables)

**High Friction:**
- User has reported the same issue 2+ times
- User says "broken again", "fixed this but that broke", "was working before"
- Previous fix was rejected or found incomplete
- User is visibly tired or losing patience

### Decision Flow

```
Before starting any coding task, ask yourself:

1. COMPLEXITY: After this change, are you confident it's correct on the first try?
   └─ Not confident → You need an SVD probe

2. UNCERTAINTY: Do you know the system's actual current state?
   └─ Don't know → Write a probe first to check baseline

3. STATE: Does this change affect persistent state?
   └─ Yes → Must verify state was written correctly

4. FRICTION: Has the user rejected a fix for this before?
   └─ Yes → This time you MUST deliver with evidence
```

### When NOT to Use SVD

- Pure UI layout/CSS adjustments (visual — needs human eyes)
- Single-file, single-function changes with clear blast radius
- Well-tested code with comprehensive integration coverage (rare in vibe coding)
- Pure formatting, comments, or typo fixes

## The Core Workflow

```
1. WRITE DEBUG PROBE FIRST (if none exists for the target area)
   └─ Expose the internal state you're about to change
   └─ Return structured JSON (counts, status, key fields)

2. CHECK BASELINE
   └─ curl the probe → see current state
   └─ Document the expected change ("count: 642 → 643")

3. MAKE THE CHANGE
   └─ Edit the production code

4. SELF-VERIFY
   └─ curl the probe again → did it change as expected?
   └─ YES → DONE. Report to user with evidence.
   └─ NO  → The probe output tells you what's wrong. Fix it. Go to 3.

5. EDGE CASE CHECK
   └─ Call the probe with edge case params
   └─ Verify boundary conditions
```

**Iron rule:** Never report "done" without probe output that proves it.

## Patterns by Technology Stack

### Electron / Node.js HTTP Server

Add debug routes to your existing HTTP server (webhook, API, or dedicated debug port):

```typescript
// Pattern: /debug/<module>/<action>/<params...>

// State inspection
if (req.method === 'GET' && req.url === '/debug/queue') {
  const items = db.prepare("SELECT value FROM config WHERE key='launch_queue'").get();
  res.end(JSON.stringify({ count: items.length, items }));
}

// Search verification
if (req.method === 'GET' && req.url?.startsWith('/debug/search/')) {
  const [,,, filter, query] = req.url.split('/');
  const results = await search(query, 'hybrid', filter);
  res.end(JSON.stringify({ count: results.length, first3: results.slice(0, 3) }));
}

// Write-based verification (add test data, check persistence)
if (req.method === 'POST' && req.url === '/debug/queue/add') {
  const { type, target, title } = JSON.parse(body);
  items.push({ id: `${type}-${Date.now()}`, type, target, title });
  db.prepare("INSERT INTO config ... ON CONFLICT ...").run(JSON.stringify(items));
  res.end(JSON.stringify({ ok: true, count: items.length }));
}

// Long-running operation (reindex, rebuild)
if (req.method === 'GET' && req.url === '/debug/rebuild-vectors') {
  ctrl.rebuildVectors(); // fire-and-forget
  res.end(JSON.stringify({ status: 'started' }));
}
```

**Design decisions:**
- `/debug/` prefix to distinguish from production routes
- JSON always — machine-parseable, count-first
- Both read (GET) and write (POST) probes
- Fire-and-forget for long operations (rebuild, reindex)

**Real example — InfoHub** (Electron app with built-in HTTP webhook):

| Endpoint | What It Verifies |
|----------|-----------------|
| `GET /debug/queue` | Queue persistence (SQLite write → read back) |
| `POST /debug/queue/add` | Queue insert works end-to-end |
| `POST /debug/queue/clear` | Queue deletion + empty state |
| `GET /debug/search/:filter/:query` | Search returns expected types/counts |
| `GET /debug/vk/:filter/:k` | Vector search with source_type filtering |
| `GET /debug/vec-test` | vec0 virtual table metadata filtering |
| `GET /debug/rebuild-vectors` | Vector index rebuild triggers correctly |
| `GET /ping` | Server is alive |

### SpringBoot / Java

Dedicated `DebugController` — never ships to production:

```java
@RestController
@RequestMapping("/debug")
@Profile("!prod")  // ← CRITICAL: only active in dev/test
public class DebugController {

    @Autowired private QueueService queueService;
    @Autowired private SearchService searchService;
    @Autowired private DataSource dataSource;

    // State inspection
    @GetMapping("/queue")
    public ResponseEntity<?> getQueue() {
        List<QueueItem> items = queueService.getAll();
        return ResponseEntity.ok(Map.of(
            "count", items.size(),
            "items", items.stream().map(QueueItem::toSummary).toList()
        ));
    }

    // Search verification
    @GetMapping("/search/{type}")
    public ResponseEntity<?> search(
            @PathVariable String type,
            @RequestParam String q) {
        List<SearchResult> results = searchService.search(q, type);
        return ResponseEntity.ok(Map.of(
            "count", results.size(),
            "types", results.stream().map(SearchResult::getType).distinct().toList(),
            "first3", results.stream().limit(3).toList()
        ));
    }

    // Database state
    @GetMapping("/db/{table}/count")
    public ResponseEntity<?> tableCount(@PathVariable String table) {
        if (!table.matches("^[a-zA-Z_][a-zA-Z0-9_]*$")) {
            return ResponseEntity.badRequest().body(Map.of("error", "invalid table"));
        }
        int count = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM " + table, Integer.class);
        return ResponseEntity.ok(Map.of("table", table, "count", count));
    }

    // Write test data
    @PostMapping("/queue/add")
    public ResponseEntity<?> addToQueue(@RequestBody Map<String, Object> body) {
        QueueItem item = queueService.add(
            (String) body.get("type"),
            (String) body.get("target"),
            (String) body.get("title"));
        return ResponseEntity.ok(Map.of("ok", true, "id", item.getId()));
    }
}
```

**Key decisions:**
- `@Profile("!prod")` — Spring won't create this bean in production
- Alternative: `@ConditionalOnProperty("app.debug.enabled")`
- Table name sanitization against SQL injection (even in debug!)
- `Map.of(...)` for clean JSON structure

### Express.js

Separate debug router, conditionally mounted:

```javascript
// routes/debug.js
const router = require('express').Router();

router.get('/queue', (req, res) => {
  const items = req.app.get('queueStore').getAll();
  res.json({ count: items.length, items: items.map(i => i.summary) });
});

router.get('/search/:type', async (req, res) => {
  const results = await req.app.get('searchService').search(req.query.q, req.params.type);
  res.json({
    count: results.length,
    types: [...new Set(results.map(r => r.type))],
    first3: results.slice(0, 3)
  });
});

router.post('/queue/add', (req, res) => {
  const { type, target, title } = req.body;
  const item = req.app.get('queueStore').add(type, target, title);
  res.json({ ok: true, id: item.id, count: req.app.get('queueStore').size() });
});

module.exports = router;

// In app.js — conditionally mount
if (process.env.NODE_ENV !== 'production') {
  app.use('/debug', require('./routes/debug'));
}
```

### Python / Flask

```python
# debug_routes.py
from flask import Blueprint, jsonify, request

debug_bp = Blueprint('debug', __name__, url_prefix='/debug')

@debug_bp.route('/queue')
def get_queue():
    items = queue_store.get_all()
    return jsonify({
        'count': len(items),
        'items': [{'id': i.id, 'type': i.type, 'title': i.title} for i in items]
    })

@debug_bp.route('/search/<filter_type>')
def search(filter_type):
    q = request.args.get('q', '')
    results = search_service.search(q, filter_type)
    return jsonify({
        'count': len(results),
        'types': list(set(r.type for r in results)),
        'first3': [{'title': r.title, 'type': r.type} for r in results[:3]]
    })

@debug_bp.route('/queue/add', methods=['POST'])
def add_to_queue():
    data = request.get_json()
    item = queue_store.add(data['type'], data['target'], data['title'])
    return jsonify({'ok': True, 'id': item.id, 'count': queue_store.size()})

# In app.py — conditionally register
if app.config.get('DEBUG') or os.environ.get('APP_DEBUG'):
    app.register_blueprint(debug_bp)
```

### Go / net/http

```go
// debug/handler.go
package debug

import (
    "encoding/json"
    "net/http"
    "strings"
)

type DebugHandler struct {
    Queue  QueueStore
    Search SearchService
}

func (h *DebugHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")

    switch {
    case r.Method == "GET" && r.URL.Path == "/debug/queue":
        items := h.Queue.GetAll()
        json.NewEncoder(w).Encode(map[string]interface{}{
            "count": len(items), "items": items,
        })

    case r.Method == "GET" && strings.HasPrefix(r.URL.Path, "/debug/search/"):
        parts := strings.Split(r.URL.Path, "/")
        results := h.Search.Search(r.URL.Query().Get("q"), parts[3])
        json.NewEncoder(w).Encode(map[string]interface{}{
            "count": len(results), "first3": results[:min(3, len(results))],
        })

    case r.Method == "POST" && r.URL.Path == "/debug/queue/add":
        var body struct {
            Type   string `json:"type"`
            Target string `json:"target"`
            Title  string `json:"title"`
        }
        json.NewDecoder(r.Body).Decode(&body)
        item := h.Queue.Add(body.Type, body.Target, body.Title)
        json.NewEncoder(w).Encode(map[string]interface{}{
            "ok": true, "id": item.ID, "count": h.Queue.Size(),
        })

    default:
        w.WriteHeader(http.StatusNotFound)
        json.NewEncoder(w).Encode(map[string]string{"error": "unknown debug route"})
    }
}

// In main.go — conditionally mount
if os.Getenv("APP_DEBUG") != "" || !isProduction {
    http.Handle("/debug/", &debug.DebugHandler{Queue: qs, Search: ss})
}
```

## Designing Good Self-Validation Probes

### Debug Endpoints vs Self-Validation Probes

Traditional debug endpoints are designed for **humans** — they need readable error messages, UIs, documentation, usability. Self-validation probes are designed for **AI** — the only consumer is another model reading JSON. This changes everything:

| | Debug Endpoint (Human) / 调试端点 | Self-Validation Probe (AI) / 自校验探针 |
|------|------|------|
| **Consumer / 消费者** | Human developer / 人类开发者 | AI agent / AI |
| **Format / 格式** | Needs readable messages, maybe HTML / 需要可读消息 | JSON only, machine-parseable / 纯 JSON |
| **Usability / 易用性** | Requires docs, error guidance / 需要文档 | Just needs `count` first + predictable shape / 结构一致即可 |
| **Security / 安全** | Must guard against accidental exposure / 需要防护 | Same production guard, but simpler internally |
| **Cost / 成本** | Hours to build proper debug UI / 几小时 | 5–20 lines per probe / 5–20 行 |

**Key insight:** You're not building a debug panel for humans. You're building a machine-readable state snapshot for an AI to `curl` and parse. Don't over-engineer it. JSON. Count first. Done.

### The JSON Contract

Every probe response MUST include `count` first — the AI reads this to verify cardinality before inspecting details:

```json
{
  "count": 17,           // ALWAYS first — quick cardinality check
  "ok": true,            // For write operations
  "items": [...],        // Truncate to first 3-5 for inspection
  "types": ["app","web"] // Distinct categories — catches type confusion bugs
}
```

### Quality Checklist

| Quality | Bad | Good |
|---------|-----|------|
| **Count first** | `{items: [...]}` — must parse array | `{count: 17, items: [...]}` |
| **Truncated items** | Returns 5000 items | Returns first 3-5 + total count |
| **Type summary** | No type info | `"types": "app,web,file"` |
| **Write returns new state** | `{ok: true}` only | `{ok: true, count: 18}` — verify persistence |
| **Error returns JSON** | HTML error page | `{error: "table not found: xyz"}` |
| **Idempotent reads** | GET modifies state | GET only returns state |

### What to Expose (Tiers)

```
TIER 1 (Always):     Queue state, search results, DB row counts, config values
TIER 2 (Often):      Cache contents, active connections, file paths, model info
TIER 3 (Sometimes):  Vector test tables, rebuild triggers, simulate errors
```

## Verification in Practice

### Example: Fix Queue Persistence Bug

**Bug:** "Right-click → Add to queue doesn't work"

```
1. CHECK BASELINE
   $ curl http://127.0.0.1:37231/debug/queue
   {"count":0,"items":[]}
   → Queue is empty. Can we add to it?

2. REPRODUCE VIA PROBE
   $ curl -X POST http://127.0.0.1:37231/debug/queue/add \
     -H 'Content-Type: application/json' \
     -d '{"type":"web","target":"https://example.com","title":"Test"}'
   {"ok":true,"count":1}
   → Write works! Bug is in the IPC path, not the DB.

3. TRACE THE GAP
   → preload.ts → ipcMain.handle → controller chain was missing queue:set handler
   → Add the handler

4. VERIFY FIX
   $ curl http://127.0.0.1:37231/debug/queue
   {"count":1,"items":[{"id":"web-1735123456789","type":"web","title":"Test"}]}
   → Queue persists after IPC write! Bug fixed.

5. EDGE CASE: CLEAR
   $ curl -X POST http://127.0.0.1:37231/debug/queue/clear
   {"ok":true}
   $ curl http://127.0.0.1:37231/debug/queue
   {"count":0,"items":[]}
   → Clear works. No stale data.
```

### Example: Fix Missing Search Icons

**Bug:** "All search results have gray icons"

```
1. CHECK BASELINE
   $ curl "http://127.0.0.1:37231/debug/search/all/test"
   {"count":5,"types":"web,app,file","first3":[...]}
   → No icon_data in output. Confirmed missing.

2. FIND THE GAP
   → Grep searchFts/searchLike/searchVector → all three missing icon_data in SELECT

3. FIX ALL THREE
   → Add d.icon_data AS icon_data to each search method

4. VERIFY
   $ curl "http://127.0.0.1:37231/debug/search/all/test"
   {"count":5,"types":"web,app,file,site",
    "first3":[{"t":"web","title":"Example","icon_data":"data:image/png;base64,..."}]}
   → icon_data now present!

5. REGRESSION CHECK
   $ curl "http://127.0.0.1:37231/debug/search/web/test"
   $ curl "http://127.0.0.1:37231/debug/search/app/test"
   $ curl "http://127.0.0.1:37231/debug/search/file/test"
   → All three filters return icon_data. No regression.
```

## When There's No HTTP Server

Not every app has HTTP. Alternatives:

### CLI → `--debug` flag

```bash
./my-tool --debug queue:get        # JSON to stdout
./my-tool --debug db:count:users   # Single value
./my-tool --debug search:test      # Structured output
```

### Desktop apps (no HTTP) → IPC debug channel

```typescript
// Electron: IPC-based debug alongside or instead of HTTP
ipcMain.handle('debug:queue:get', async () => { ... });
ipcMain.handle('debug:search:test', async (_e, filter, query) => { ... });
// In DevTools: window.__debug.queue.get()
```

### Library/SDK → debug export

```typescript
// Tree-shakeable in production
export const __debug = process.env.NODE_ENV !== 'production' ? {
  getQueue: () => queueStore.getAll(),
  getSearchIndex: () => index.stats(),
  rebuildIndex: () => index.rebuild(),
} : undefined;
```

## Quick Reference

| I want to verify... | Probe pattern |
|---------------------|---------------|
| Queue/item persistence | `GET /debug/<module>` → count + items |
| Search results | `GET /debug/search/:type/:query` → count + types + first3 |
| DB migration applied | `GET /debug/db/:table/count` |
| Config/state changed | `GET /debug/config/:key` |
| Vector index working | `GET /debug/vec-test` → insert → query → verify |
| Long operation (reindex) | `GET /debug/rebuild-*` → fire-and-forget |
| IPC round-trip | POST add → GET verify count incremented |
| Model loaded/active | `GET /debug/models` → list + active status |

## Common Mistakes

### 1. GET modifies state
```
❌ GET /debug/search → also increments a counter
✅ GET is read-only. POST/PUT for writes. Always.
```

### 2. Returning too much data
```
❌ res.end(JSON.stringify(all5000Items))
✅ res.end(JSON.stringify({ count: 5000, first5: items.slice(0, 5) }))
```

### 3. No count field
```
❌ {items: [...]}  — must parse array to know there are 17 items
✅ {count: 17, items: [...]}  — instant cardinality verification
```

### 4. Debug routes in production
```
❌ if (true) { app.use('/debug', router); }
✅ if (process.env.NODE_ENV !== 'production') { ... }
   // SpringBoot: @Profile("!prod")
   // Feature flag: if (config.debugEnabled) { ... }
```

### 5. Writing probes AFTER the fix
```
❌ Change code → hope it works → "done" → user: "still broken"
✅ Write probe FIRST → reproduce via probe → fix → verify via probe → deliver
```

### 6. No evidence in delivery
```
❌ "I fixed the queue persistence bug" (no evidence)
✅ "Fixed. Verified: POST /debug/queue/add → count:1, GET /debug/queue → persisted.
    curl http://127.0.0.1:37231/debug/queue → {"count":1,"items":[...]}"
```

## Red Flags — STOP and Self-Verify

- "Should be fine now" → **curl the probe. Verify.**
- "This change is simple" → **Simple changes cause the most regressions. Verify.**
- "Same pattern as before" → **Same pattern ≠ same behavior. Verify.**
- "User hasn't complained" → **User hasn't tested yet. You test first.**
- "Just restart and it'll work" → **If restart fixes it, it'll break again. Probe catches root cause.**

## Relationship to Self-Proving Delivery

SVD is the **coding-domain implementation** of the broader [Self-Proving Delivery](../self-proving-delivery/SKILL.md) meta-pattern:

```
Self-Proving Delivery (all domains)
├── Coding → SVD (this skill) — self-validation probes + curl
├── Docs → self-audit chapters + links + terminology
├── Data → self-audit row counts + aggregates + nulls
├── Ops → self-audit config + health checks
└── ...
```

**REQUIRED BACKGROUND:** Read [Self-Proving Delivery](../self-proving-delivery/SKILL.md) first — it establishes the mindset. SVD gives you the concrete tools for code.

## The Bottom Line

**TDD is for when you know what correct looks like before you start. SVD is for when you're figuring it out as you go — which is what vibe coding is.**

Every round-trip to the user for verification is a self-validation failure. A self-validation probe costs 5–20 lines of code and prevents 5–20 minutes of back-and-forth per bug. In mid-to-late project phases, this is the difference between smooth progress and death by a thousand regressions.

**Remember:** The user shouldn't be your test suite. You have `curl`. Use it.
