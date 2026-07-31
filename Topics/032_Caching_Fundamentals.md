# Topic 032: Caching Fundamentals

**Module:** 3 — Caching
**Tier:** 🔴 MUST
**Completed:** 2026-07-31
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Every system taught so far reads from a database, and databases — even with the page/buffer-pool machinery from Topic 020 — pay overhead per query: locking, WAL bookkeeping, MVCC visibility checks, query planning. As traffic grows, the database becomes the bottleneck long before the app tier does. Caching is the standard fix: keep a copy of hot data in a fast, in-memory store so most reads never touch the database at all. This topic is the foundation for the rest of Module 3 (patterns, eviction, distributed caching, consistency).

---

## 2. What a Cache Actually Is

A cache is a smaller, faster storage layer sitting in front of a larger, slower one. The bet: a small fraction of data accounts for most reads (the **80/20 rule**, the "hot set"). Serve that hot set from RAM and you absorb most read traffic without touching disk.

```
Client → App Server → Cache (Redis/Memcached) → Database
                ↑______________________|
                (on miss: read DB, then populate cache)
```

**Where a cache can live:**
| Type | Speed | Consistency across fleet | Survives restart |
|---|---|---|---|
| In-process (local) | Fastest (no network hop) | No — each instance has its own copy | No |
| Remote (distributed) — Redis/Memcached | One hop slower than local | Yes — shared by all app servers | Yes |
| CDN | Geographically close to user | N/A (edge-specific) | Yes (Topic 018/041) |

Default assumption in system-design answers: "cache" = distributed/remote cache, unless local caching is explicitly called out (e.g., caching config in-process).

---

## 3. Cache Hit, Miss, and Hit Ratio

- **Cache hit**: data found in cache — fast path, no DB query.
- **Cache miss**: data not in cache — slow path: query DB, then (usually) populate cache so the next request hits.
- **Hit ratio** = hits / (hits + misses) — the single most important number for a cache. 95% hit ratio means the DB sees only 5% of read traffic.

Hit ratio depends on: how skewed the access pattern is (hot set size vs total data), memory allocated to the cache, and eviction policy (Topic 034) for what to keep when full.

---

## 4. Why a Cache Hit Is Faster Than Even a Well-Indexed DB Query

Ties back to Topic 020/027. It is NOT simply "RAM is faster than disk" — that's only one factor, and it doesn't even apply once data is sitting in the DB's own buffer pool (also RAM). The real reason a cache hit wins:

A DB query — even served entirely from the buffer pool — still pays for:
- Query parsing/planning
- Lock acquisition (Topic 026)
- MVCC visibility checks (xmin/xmax, Topic 027)
- WAL bookkeeping

A dedicated cache (Redis/Memcached) is typically just a **hashmap lookup by key** — no SQL parsing, no joins, no transaction machinery. That overhead gap, not raw disk-vs-RAM speed, is why a cache hit can be 10-100x faster than even a fast indexed query.

---

## 5. The Cache Invalidation Problem

Caching is easy until data changes. The cache is now a second copy of the truth, and that copy can go **stale** — the DB has new data, the cache still serves the old value. ("There are only two hard things in computer science: cache invalidation and naming things.")

The tension:
- Cache too briefly → low hit ratio, cache barely helps.
- Cache too long → stale reads.

Two blunt tools (full patterns come in Topic 033):
- **TTL (Time-To-Live)**: entries auto-expire after N seconds. Simple, bounds staleness, but doesn't eliminate it.
- **Explicit invalidation**: delete/update the cache entry at write time. Removes staleness immediately, but every write path must remember to do it — miss one and you get a silent stale-cache bug.

**Display cache vs transactional read** (key distinction): a page showing a product's price can tolerate a TTL-bound staleness window just fine. The moment that same value is used to actually **charge** the customer (checkout/payment), it must not be trusted from cache — that read has to hit or revalidate against the source of truth. Conflating "acceptable to display slightly stale" with "acceptable to transact on" is a real interview trap.

---

## 6. What Should You Cache?

**Good candidates:**
- Read-heavy, write-light data (e.g., a user profile read 1000x per write)
- Expensive-to-compute data (aggregations, rendered fragments, search results)
- Data that tolerates some staleness

**Bad candidates:**
- Write-heavy data — invalidating/rewriting nearly as often as you'd just query the DB, no net win
- Data requiring strong consistency / **read-your-own-write** (e.g., "did my payment go through", a shopping cart the user is actively editing)
- Huge, rarely-accessed data — wastes memory for near-zero hit-ratio benefit (Topic 034: eviction)

A candidate can fail on more than one criterion at once — e.g., a shopping cart is both roughly 1:1 read:write AND a read-your-own-write case, which is why it's a clear "don't cache" example.

---

## 7. Cache Stampede

A hot key's TTL expires, and at that exact instant thousands of concurrent requests all miss simultaneously and all hammer the DB trying to refill it — potentially taking the DB down. Called a **cache stampede** (or "thundering herd"). Mitigations (locking so only one request repopulates while others wait, or jittered TTLs so keys don't all expire at once) are advanced detail, but knowing the name and shape of the failure is itself an interview signal.

---

## 8. Decision Box: When to Introduce a Cache

| Use a cache when... | Don't reach for a cache when... |
|---|---|
| Read:write ratio is heavily read-skewed | Workload is write-heavy |
| Access pattern is skewed (hot set << total data) | Access is uniformly random (low hit ratio regardless) |
| Some staleness is tolerable | Strong consistency / read-your-own-write required |
| DB is becoming the read bottleneck | DB isn't the bottleneck yet — premature caching adds complexity for no measured win |

**Interview justification format:** "I'd add a distributed cache here because reads outnumber writes roughly 100:1, product data changes infrequently, and a 30-second staleness window is acceptable — this should push hit ratio above 90% and take most read load off the primary DB."

---

## 9. Worked Example — Shopping Cart (why it's a bad caching candidate)

Requirements: cart is viewed after nearly every add/remove; user expects to see their own edits immediately before checkout.

**Applying the criteria:**
1. Read:write ratio ≈ 1:1 — fails "read-heavy, write-light."
2. Consistency requirement: read-your-own-write — user must see their own change instantly. Fails the "tolerates staleness" criterion too.

**Conclusion:** don't cache the cart itself (or if caching for performance reasons, invalidate synchronously on every write — no TTL-only strategy). This is a case that fails on two separate bad-candidate criteria simultaneously, not just one.

---

## 10. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Justifying "don't cache X" only by vague UX language ("frustrating") instead of the named framework criteria | Name the specific criterion failed: read:write ratio, or read-your-own-write/strong-consistency requirement — often both at once |
| Treating "acceptable staleness" as a single yes/no property of a piece of data | Split by use: a display read (browsing) can tolerate a TTL window; a transactional read (checkout/payment) must not trust the cache — same data, different acceptability depending on where it's read |
| Attributing cache-hit speed purely to "RAM is faster than disk" | Even DB data already resident in the buffer pool (RAM) is slower to query than a cache hit — the real gap is the DB's per-query overhead (parsing, locking, MVCC checks, WAL bookkeeping) that a cache skips entirely |
| Caching everything by default | Write-heavy or rarely-read data wastes cache memory or even hurts (thrashing) — justify per data type |
| Ignoring cache stampede | A single hot-key expiry under load can cascade into a DB outage — mention TTL jitter or locking when discussing very hot keys |

---

## 11. Revision Questions
See `Revision/Revision_032.md`.

## 12. Summary
- A cache is a fast, small copy of hot data in front of a slower source of truth, justified by a skewed access pattern (80/20 rule); value measured by hit ratio.
- A cache hit beats even a well-indexed DB query because it skips per-query DB overhead (parsing, locking, MVCC checks, WAL bookkeeping) — not just because RAM beats disk.
- The real cost of caching is invalidation: staleness is the price of speed. TTL and explicit invalidation are the two blunt tools; Topic 033 covers the real patterns.
- Display-cache staleness and transactional-read staleness are different acceptability questions for the same data.
- Good candidates: read-heavy/write-light, expensive-to-compute, staleness-tolerant. Bad candidates: write-heavy, strong-consistency/read-your-own-write, huge-and-rarely-read.
- Cache stampede: a hot key's expiry under load can cascade into a DB outage — jittered TTLs or locking mitigate it.
