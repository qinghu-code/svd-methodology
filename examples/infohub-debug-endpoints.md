# InfoHub Self-Validation Probes — A Real-World SVD Catalog

All probes from the [InfoHub](https://github.com/example/infohub) project — an Electron + SQLite desktop app built entirely via vibe coding. Each probe is a real HTTP endpoint on the app's built-in webhook server (`http://127.0.0.1:37231`).

## Probe Reference

### `GET /debug/queue`

**Purpose:** Verify queue persistence and IPC round-trip.

```bash
$ curl http://127.0.0.1:37231/debug/queue
{"count":3,"items":[
  {"id":"web-1735123456789","type":"web","title":"GitHub"},
  {"id":"app-1735123456790","type":"app","title":"VS Code"},
  {"id":"file-1735123456791","type":"file","title":"README.md"}
]}
```

**What it caught:**
- Queue persisted to SQLite but invisible in UI due to multi-process WAL mode (different Electron process wrote to DB — current process couldn't see it)
- IPC handler chain (`preload.ts → ipcMain.handle → configStore`) was missing the `queue:set` handler

**Without it:** User reports "加不进去了" → 4 rounds of back-and-forth trying different fixes without isolating IPC vs DB root cause.

---

### `POST /debug/queue/add`

**Purpose:** Verify write path end-to-end (HTTP → SQLite → read back).

```bash
$ curl -X POST http://127.0.0.1:37231/debug/queue/add \
  -H 'Content-Type: application/json' \
  -d '{"type":"web","target":"https://example.com","title":"Test"}'
{"ok":true,"count":4}
```

**What it caught:**
- Proved the DB write path works in isolation — isolated the bug to the IPC bridge
- Caught a regression where config store upsert silently failed on first insert (missing ON CONFLICT clause)

**Without it:** Would have guessed the DB was broken and spent time debugging SQLite instead of IPC.

---

### `POST /debug/queue/clear`

**Purpose:** Verify queue deletion and empty-state handling.

```bash
$ curl -X POST http://127.0.0.1:37231/debug/queue/clear
{"ok":true}
$ curl http://127.0.0.1:37231/debug/queue
{"count":0,"items":[]}
```

**What it caught:**
- Empty queue state rendered correctly (no stale data left in memory)
- Clear wasn't propagating to the shared cache module (`queueStore.ts`)

**Without it:** Users would clear queue, see items still displayed, and report "clear doesn't work."

---

### `GET /debug/search/:filter/:query`

**Purpose:** Verify search returns correct types and structure across all three search engines (FTS5, LIKE, vector).

```bash
$ curl "http://127.0.0.1:37231/debug/search/all/test"
{"count":5,"types":"web,app,file,site",
 "first3":[
   {"t":"web","title":"Example Page","d":0.0234},
   {"t":"app","title":"Test App","d":0.0456}
 ]}
```

**What it caught:**
- All three search paths (`searchFts`, `searchLike`, `searchVector`) were missing `d.icon_data AS icon_data` in SELECT
- Fixing one would have left the other two broken — probe caught all three at once
- First version of site→documents merge returned duplicate results (missing DISTINCT)

**Without it:** User reports "icons are gray in search" → fix one path → user: "still broken in some searches" → fix another → repeat. 3 fixes for one bug.

---

### `GET /debug/vk/:filter/:k`

**Purpose:** Verify vec0 vector search with metadata column filtering.

```bash
$ curl "http://127.0.0.1:37231/debug/vk/app/5"
{"k":5,"filter":"app","count":3,
 "results":[
   {"t":"app","title":"VS Code","d":"0.0123"},
   {"t":"app","title":"Terminal","d":"0.0234"}
 ]}
```

**What it caught:**
- vec0 `source_type` metadata filtering: `=` worked but `IN (...)` didn't return all expected results
- Temporary vec0 table creation succeeded but the embedding dimension didn't match (384 vs 512)

**Without it:** "Vector search results have wrong types" with no way to inspect what vec0 is actually returning.

---

### `GET /debug/vec-test`

**Purpose:** Self-contained vec0 unit test — creates temp table, inserts test vectors, queries with filters, verifies metadata column behavior.

```bash
$ curl "http://127.0.0.1:37231/debug/vec-test"
{"all":["1:aaa","2:bbb","3:aaa"],
 "filtered_eq":["1:aaa","3:aaa"],
 "filtered_in":["1:aaa","2:bbb","3:aaa"],
 "message":"ALL should have 3, filtered_eq should have [1,3], filtered_in should have all 3"}
```

**What it caught:**
- vec0 version-specific behavior: some versions treat metadata columns differently
- Confirmed the `AND dtype = ?` and `AND dtype IN (?,?)` patterns work before using them in production tables

**Without it:** vec0 behavior is a black box. Every "vector search is wrong" bug would involve guessing.

---

### `GET /debug/rebuild-vectors`

**Purpose:** Trigger vector index rebuild and confirm it started (fire-and-forget).

```bash
$ curl "http://127.0.0.1:37231/debug/rebuild-vectors"
{"status":"started"}
# Check logs for completion: [debug] rebuild vectors done: {...}
```

**What it caught:**
- Rebuild could hang silently if embedding model wasn't loaded
- Fire-and-forget pattern confirmed the trigger works even for long operations

**Without it:** "Why are there no vector search results?" with no way to check if rebuild even started.

---

### `GET /ping`

**Purpose:** Verify the webhook server is alive.

```bash
$ curl "http://127.0.0.1:37231/ping"
{"ok":true}
```

**What it's good for:** After restart, config change, or port change — confirms the server is up before testing other probes.

---

## Probe Design Principles (Learned from InfoHub)

1. **Count first, items second.** Every response starts with `count` — the AI reads this first to verify cardinality.
2. **Truncate items to first 3-5.** Never return full datasets. `count` tells you the total; `first3` lets you inspect shape.
3. **Include type/category summaries.** `"types": "app,web,file"` catches type confusion bugs instantly.
4. **Write operations return new state.** `{ok: true, count: 18}` not just `{ok: true}` — verifies persistence.
5. **Error returns JSON, not HTML.** A 500 page is useless to an AI. `{error: "table not found: xyz"}` is actionable.
6. **Test probes exist as standalone units.** `/debug/vec-test` creates its own temp table — doesn't depend on production data.

## Adding a New Probe (Template)

```typescript
// In your Electron webhook server (server.ts):

// NEW: Verify <module> state
if (req.method === 'GET' && req.url === '/debug/<module>') {
  try {
    const items = db.prepare("SELECT ...").all();
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      count: items.length,
      first3: items.slice(0, 3).map(i => ({ /* key fields */ })),
    }));
  } catch (e: any) {
    res.writeHead(500);
    res.end(JSON.stringify({ error: e.message }));
  }
  return;
}
```

**Checklist before deploying:**
- [ ] Returns JSON with `count` first
- [ ] Items truncated to max 5
- [ ] Catches errors and returns `{error: message}`
- [ ] Doesn't expose secrets or full file paths
- [ ] Conditionally mounted (not in production)
