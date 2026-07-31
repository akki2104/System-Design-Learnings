# Topic 033: Caching Patterns

**Module:** 3 — Caching
**Tier:** 🔴 MUST
**Completed:** 2026-07-31
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topic 032 established *what* a cache is and *when* to use one. This topic is *how* the cache actually gets populated and kept in sync with the database — the specific read/write choreography between app, cache, and DB. This is the most commonly asked caching question in interviews because it's where subtle bugs live.

---

## 2. Cache-Aside (a.k.a. Lazy Loading) — the Default Pattern

```
READ:
  App checks cache
    HIT  → return cached value
    MISS → App queries DB → App writes result to cache → return value

WRITE:
  App writes to DB
  App INVALIDATES (deletes) the cache entry — does NOT write the new value into cache
```
```
   ┌─────┐   1. read?    ┌───────┐
   │ App │──────────────▶│ Cache │
   └─────┘◀──────────────└───────┘
      │        2. miss
      │ 3. query
      ▼
   ┌─────┐
   │  DB │
   └─────┘
      │ 4. populate cache
      ▼
   ┌───────┐
   │ Cache │
   └───────┘
```
The cache is entirely passive — the application owns all the logic. This is why it's called "aside": the cache sits *beside* the DB, not in the direct write path.

**Why invalidate on write instead of updating the cache with the new value?** Two transactions writing concurrently could race and leave a stale value cached (Transaction A writes DB, then B writes DB, then A's delayed cache-write overwrites B's cache-write with an older value). Deleting instead of writing avoids that race — the next read simply repopulates from the now-correct DB.

**A trap worth naming explicitly:** the write side isn't optional. If a write only touched the DB and never touched the cache at all, the next read would be a cache HIT returning the now-stale value — until the TTL happened to expire. The whole reason cache-aside does anything on write is to force the next read to miss and repopulate from the fresh DB value. Write to the DB, THEN delete the cache entry — both steps matter.

**Key characteristic:** on a miss, the cache is only ever populated by a *read*, never proactively. This means a cold cache (e.g., right after a restart) will show a burst of misses that gradually populates the hot set — this connects directly to Topic 035's cache stampede.

---

## 3. Write-Through

```
WRITE:
  App writes to Cache
  Cache synchronously writes through to DB
  (write is only acknowledged once BOTH are durable)

READ:
  Same as cache-aside — check cache, miss falls back to DB
```
```
   ┌─────┐   write    ┌───────┐   write-through   ┌─────┐
   │ App │───────────▶│ Cache │──────────────────▶│  DB │
   └─────┘            └───────┘                    └─────┘
                (ack only after both succeed)
```
Cache and DB are always in sync — there's no window where the cache is stale after a write, because the cache is updated as part of the write itself, not invalidated for the next read to repopulate.

**The cost:** every write now pays the latency of *two* writes (cache + DB) instead of one, and if the DB write fails after the cache write succeeded, you need to handle that partial-failure case explicitly. It also means data that's *written* but never *read* still gets cached — wasted cache space for write-only or rarely-read data.

---

## 4. Write-Behind (a.k.a. Write-Back)

```
WRITE:
  App writes to Cache only
  Cache acknowledges immediately
  Cache asynchronously flushes the write to DB later (batched)
```
```
   ┌─────┐   write    ┌───────┐   async batch    ┌─────┐
   │ App │───────────▶│ Cache │┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄▶│  DB │
   └─────┘  (ack now)  └───────┘  (flushed later) └─────┘
```
Writes are extremely fast (only touch the cache), and batching many writes to the DB reduces DB write load. **The real cost: durability risk.** If the cache crashes before flushing, those writes are lost forever — they were never in the DB. This is the pattern's defining tradeoff: write latency and DB load go down, data-loss risk goes up.

---

## 5. Side-by-Side Comparison

| Pattern | Write latency | Read-after-write freshness | Data-loss risk on cache crash | DB write load |
|---|---|---|---|---|
| **Cache-Aside** | DB-write only (fast) | Cache invalidated, next read repopulates from DB (fresh) | None — DB is always source of truth | Same as no cache (every write still hits DB) |
| **Write-Through** | DB + cache write (slower) | Always fresh — cache updated at write time | None — DB write is synchronous | Same as no cache |
| **Write-Behind** | Cache-write only (fastest) | Fresh in cache, but DB temporarily behind | **High** — unflushed writes are lost if cache dies | Reduced (batched) |

---

## Tech Decision Box: Which Pattern, and When

```
Use CACHE-ASIDE when:
  - Default choice for most read-heavy systems (product pages, user profiles)
  - You want the DB to remain unambiguously the source of truth
  - Simple to reason about; most libraries/ORMs support it easily

Use WRITE-THROUGH when:
  - Read-after-write freshness matters (no stale-read window at all)
  - You can tolerate slightly higher write latency
  - Example: a user's own settings/preferences — they should see their change
    reflected instantly on next read, every time

Use WRITE-BEHIND when:
  - Write volume is very high and DB write throughput is the bottleneck
  - Some data loss on a rare cache crash is an acceptable risk
  - Example: view counters, analytics events, activity logs — losing the last
    few seconds of counts on a crash is tolerable; losing a financial
    transaction is not
```
**Interview sentence:** "For the product catalog, I'd use cache-aside — it's read-heavy, the DB stays authoritative, and a brief miss-then-repopulate window is fine. I would NOT use write-behind for an inventory count feeding checkout — losing unflushed decrements on a cache crash could cause overselling; write-through or a direct DB write is safer there."

---

## Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Stating the cache-aside write path as "write to DB" with no mention of invalidating the cache | If the write never touches the cache, the next read is a HIT returning the stale value until TTL expiry — the write must also delete the cache entry, or cache-aside doesn't actually solve staleness on writes at all |
| Updating the cache with the new value on write, instead of invalidating it | Writing the new value directly risks a race between two concurrent writers leaving a stale value cached; deleting forces the next read to repopulate from the now-correct DB |
| Assuming write-through eliminates all caching cost | It removes staleness but doubles write latency (cache + DB) and can cache write-only data that's never read — not a free upgrade over cache-aside |
| Choosing write-behind without naming the data-loss risk explicitly | Write-behind's entire tradeoff is "faster writes, batched DB load" traded against "unflushed writes are lost if the cache crashes" — never present it as a pure win |
| Treating "which caching pattern" as one universal answer for a whole system | Different data types in the same system often use different patterns — product data (cache-aside), user settings (write-through), analytics counters (write-behind) |

---

## Real Interview Questions

1. "Walk me through what happens on a read and a write with cache-aside." (baseline — must be fluent)
2. "Why does cache-aside invalidate the cache on write instead of updating it directly?" (tests the race-condition reasoning)
3. "What's the tradeoff of write-through vs cache-aside?" (freshness vs write latency)
4. "When would you use write-behind, and what's the risk?" (data-loss-on-crash reasoning)
5. "You have a product catalog and a live view-counter in the same system — would you use the same caching pattern for both? Why or why not?" (tests per-data-type reasoning, not a single blanket answer)

---

## 6. Revision Questions
See `Revision/Revision_033.md`.

## 7. Summary
- **Cache-aside** (lazy loading): app-managed, miss triggers a DB read + cache populate, write goes to the DB AND invalidates (deletes) the cache entry — never overwrites it, avoiding a write-race that could leave stale data cached. The default choice for most read-heavy data.
- **Write-through**: cache and DB updated synchronously on write — always fresh, no invalidation window, but doubles write latency and can cache data that's never read.
- **Write-behind** (write-back): write only touches the cache, DB is updated asynchronously/batched later — fastest writes and reduced DB load, but unflushed writes are **lost** if the cache crashes. This data-loss risk is the pattern's defining cost, never a free win.
- Real systems mix patterns per data type in the same design — product data, user settings, and counters don't need the same pattern.

> **You now can:** describe the exact read/write mechanics of all three caching patterns, explain *why* cache-aside's write step must touch the cache at all (not just the DB), and choose the right pattern per data type with the tradeoff named explicitly.
