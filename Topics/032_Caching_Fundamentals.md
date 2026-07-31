# Topic 032: Caching Fundamentals

**Module:** 3 — Caching
**Tier:** 🔴 MUST
**Completed:** 2026-07-31
**Confidence:** 4/5

> Note: this topic was independently taught in two parallel sessions (one on this machine, one
> on another). This file merges both — the fuller in-chat lesson as the base, plus the other
> session's DB-internals mechanism, invalidation framing, read-your-own-write concept, and cache
> stampede coverage folded in as their own sections so nothing from either pass is lost.

---

## 1. Why This Topic Exists

Every system taught so far reads from a database, and databases — even with the page/buffer-pool machinery from Topic 020 — pay overhead per query: locking, WAL bookkeeping, MVCC visibility checks, query planning. As traffic grows, the database becomes the bottleneck long before the app tier does. Caching is the standard fix: keep a copy of hot data in a fast, in-memory store so most reads never touch the database at all. This topic is the foundation for the rest of Module 3 (patterns, eviction, distributed caching, consistency, stampede/avalanche).

---

## 2. What a Cache Actually Is

```
Client → [ CACHE — small, fast ] → [ SOURCE OF TRUTH — big, slow ]
              (RAM, ~1ms)              (disk-backed DB, ~10-50ms)
```
A cache is a smaller, faster copy of data sitting in front of a slower system of record. It trades **space** (you can't fit everything) and **freshness** (the copy can go stale) for **latency**. Nothing new is created — a cache is not a new source of truth, just a temporary, disposable accelerant in front of one.

---

## 3. Why Caching Works: Locality of Reference

Caching is a bet that pays off because of a real statistical property of workloads:
```
TEMPORAL LOCALITY — data accessed recently is likely to be accessed again soon
  (a trending tweet gets read a million times in the next minute)

SPATIAL LOCALITY — data near recently-accessed data is likely to be accessed too
  (reading product_id=42 usually means its reviews/images/price get read next)
```
This is the mechanism behind the informal **80/20 rule**: a small fraction of data (the "hot set") accounts for most reads. Caching just that hot set captures most of the benefit without caching everything.

---

## 4. Where Caches Live — the Full Ladder

```
User's Browser
   │  Browser Cache (HTTP cache headers, Topic 010 preview)
   ▼
CDN / Edge (Topic 004)
   │  Caches static assets, sometimes API responses, closest to the user
   ▼
Reverse Proxy / API Gateway
   │  Can cache whole HTTP responses
   ▼
Application Layer
   │  In-process cache — fastest (no network hop), but each instance has
   │  its own copy (no fleet-wide consistency) and is lost on restart
   ▼
Distributed Cache (Redis / Memcached — full depth Topic 036)
   │  One network hop slower than in-process, but shared by every app
   │  server and survives an individual server restart
   ▼
Database Buffer Pool / Query Cache
   │  The DB's own internal caching of hot pages/query results
   ▼
Disk (the actual source of truth)
```
| Type | Speed | Consistency across fleet | Survives restart |
|---|---|---|---|
| In-process (local) | Fastest (no network hop) | No — each instance has its own copy | No |
| Remote (distributed) — Redis/Memcached | One hop slower | Yes — shared by all app servers | Yes |
| CDN | Geographically close to user | N/A (edge-specific) | Yes |

**Default assumption in interview answers:** "cache" means a distributed/remote cache unless in-process caching is explicitly called out (e.g., caching static config in-process). The closer to the user a cache sits, the faster and cheaper it is — but also the harder to keep consistent. Naming the *specific right layer* for a given problem, not just "add a cache," is the actual interview-relevant skill.

---

## 5. The Read Path: Hit vs Miss, and Hit Ratio

```
CACHE HIT                          CACHE MISS
─────────────                      ─────────────
Request → Cache                    Request → Cache (not found)
         ↓ found                            ↓
      Return (~1ms)                     Query DB (~20ms)
                                             ↓
                                       Store in cache
                                             ↓
                                       Return to client
```
**Hit ratio** = hits / (hits + misses) — the single most important number for a cache. A 95% hit ratio means the DB sees only 5% of read traffic. Hit ratio depends on how skewed the access pattern is, how much memory is allocated to the cache, and the eviction policy (Topic 034) for what gets kept when the cache is full.

---

## 6. Why a Cache Hit Beats Even a Well-Indexed DB Query

This is easy to get wrong: it is **not** simply "RAM is faster than disk" — that factor doesn't even apply once data is already sitting in the DB's own buffer pool (also RAM, Topic 020). A DB query, even served entirely from the buffer pool, still pays for:
- Query parsing/planning
- Lock acquisition (Topic 026)
- MVCC visibility checks — xmin/xmax (Topic 027)
- WAL bookkeeping

A dedicated cache (Redis/Memcached) is typically just a **hashmap lookup by key** — no SQL parsing, no transaction machinery. That overhead gap, not raw disk-vs-RAM speed, is why a cache hit can be 10-100x faster than even a fast indexed query.

---

## 7. Worked Example — Hit Rate → Latency

```
Without cache: every request costs 20ms (DB round-trip)

With cache at 90% hit rate:
  90% of requests: ~1ms  (cache hit)
  10% of requests: ~21ms (miss + DB fetch + cache write-back)

  Average latency = (0.9 × 1ms) + (0.1 × 21ms) = 0.9 + 2.1 = 3ms
```
A 90% hit rate turns a 20ms average into a 3ms average — roughly a 6-7x latency improvement — and the DB now sees only 10% of the original traffic. This is the kind of number to actually estimate out loud in an interview to justify adding a cache.

---

## 8. TTL and the Invalidation Problem

Caching is easy until data changes. The cache is now a second copy of the truth, and that copy can go stale. ("There are only two hard things in computer science: cache invalidation and naming things.") Two blunt tools:
- **TTL (Time-To-Live):** entries auto-expire after N seconds — `SET product:42 {...} EX 60`. Simple, bounds staleness, but doesn't eliminate it. Shorter TTL = fresher but lower hit rate; longer TTL = higher hit rate but staler. There's no universal "correct" TTL — it depends on how staleness-tolerant that specific data is.
- **Explicit invalidation:** delete/update the cache entry at write time. Removes staleness immediately, but every write path must remember to do it — miss one and you get a silent stale-cache bug. (Full patterns in Topic 033.)

**Display read vs transactional read (a real interview trap):** a product page showing a price can tolerate a TTL-bound staleness window just fine. The moment that same value is used to actually **charge** the customer at checkout, it must not be trusted from cache — that read has to bypass the cache or revalidate against the source of truth. The same data can be acceptably stale for one use and unacceptably stale for another — staleness tolerance is a property of the *read*, not just the data.

---

## 9. What Should You Cache? Good and Bad Candidates

**Good candidates:** read-heavy/write-light data, expensive-to-compute data (aggregations, rendered fragments, search results), data that tolerates some staleness.

**Bad candidates:** write-heavy data (invalidating almost as often as you'd just query the DB — no net win), data requiring strong consistency or **read-your-own-write** (the user must see their own edit immediately — e.g., "did my payment go through," a cart being actively edited), and huge/rarely-accessed data (wastes memory for near-zero hit-ratio payoff).

A candidate can fail more than one criterion at once — see the worked example below.

### Worked Example — Shopping Cart (a bad caching candidate on two counts)
Requirements: cart is viewed after nearly every add/remove; user expects to see their own edits immediately before checkout.
1. Read:write ratio ≈ 1:1 — fails "read-heavy, write-light."
2. Read-your-own-write — the user must see their own change instantly, failing "tolerates staleness."

**Conclusion:** don't cache the cart itself (or if caching for performance, invalidate synchronously on every write — no TTL-only strategy). It fails two separate bad-candidate criteria simultaneously, not just one — and naming *which specific criteria* it fails is what separates a strong answer from a vague "that would be frustrating for users."

---

## 10. Cache Stampede (preview — full depth Topic 035)

A hot key's TTL expires, and at that exact instant thousands of concurrent requests all miss simultaneously and all hammer the DB trying to refill it — potentially taking the DB down. Called a **cache stampede** ("thundering herd"). Mitigations: locking so only one request repopulates while others wait, or jittered TTLs so keys don't all expire at the same moment. Knowing the name and shape of this failure is itself an interview signal, even before the full mitigation toolkit (Topic 035).

---

## Tech Decision Box: When to Cache (and When Not To)

| Use a cache when... | Don't reach for a cache when... |
|---|---|
| Read:write ratio is heavily read-skewed | Workload is write-heavy |
| Access pattern is skewed (hot set << total data) | Access is uniformly random (low hit ratio regardless) |
| Some staleness is tolerable | Strong consistency / read-your-own-write required |
| DB is becoming the read bottleneck | DB isn't the bottleneck yet — premature caching adds complexity for no measured win |

**Interview justification format:** "I'd add a distributed cache here because reads outnumber writes roughly 100:1, product data changes infrequently, and a 30-second staleness window is acceptable for the display read — this should push hit ratio above 90% and take most read load off the primary DB. I would NOT apply the same TTL to the checkout price read — that has to revalidate against the source of truth."

---

## Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Attributing cache-hit speed purely to "RAM is faster than disk" | Even DB data already resident in the buffer pool (RAM) is slower to query than a cache hit — the real gap is the DB's per-query overhead (parsing, locking, MVCC checks, WAL bookkeeping) that a cache skips entirely |
| Treating "acceptable staleness" as one fixed yes/no property of the data | Split by use: a display read (browsing) can tolerate a TTL window; a transactional read (checkout/payment) must not trust the cache — same data, different acceptability depending on where it's read |
| Justifying "don't cache X" with vague UX language ("frustrating") instead of the named framework criteria | Name the specific criterion failed: read:write ratio, or read-your-own-write/strong-consistency — often both at once (the shopping cart example) |
| Caching everything by default | Write-heavy or rarely-read data wastes cache memory or even hurts (thrashing) — justify per data type |
| Ignoring cache stampede | A single hot-key expiry under load can cascade into a DB outage — mention TTL jitter or locking when discussing very hot keys |
| Thinking "cache" only means Redis/Memcached | Caching happens at every layer — browser, CDN, reverse proxy, app-local, distributed cache, DB buffer pool. Naming the right layer for the problem is the actual skill |

---

## Real Interview Questions

1. "Where would you add caching in this system, and why there specifically?" (tests the layer-ladder model)
2. "How do you decide what TTL to use for a cached value?" (tests staleness-tolerance reasoning, not a memorized number)
3. "Why is a cache hit faster than even a well-indexed DB query, given the buffer pool is already RAM?" (tests the DB-internals mechanism, not the naive RAM-vs-disk answer)
4. "Walk me through what happens on a cache miss."
5. "If your cache hit rate dropped from 95% to 60% overnight, what would you investigate?" (ties to Topic 076 Monitoring)
6. "What data in this system would you explicitly NOT cache, and why?" (tests judgment — read-your-own-write, write-heavy)
7. "What happens when a very hot key's cache entry expires under heavy load?" (cache stampede)

---

## 11. Revision Questions
See `Revision/Revision_032.md`.

## 12. Summary
- A cache is a fast, small, disposable copy of hot data in front of a slower source of truth — trading space and freshness for latency, never a free win.
- Caching works because of locality of reference (temporal + spatial) — the ~80/20 rule means caching the hot set captures most of the benefit.
- Caching exists at every layer, from the browser down to the DB's own buffer pool — naming the specific right layer is the real skill.
- Hit ratio is the metric that justifies a cache's complexity (worked example: 90% hit rate → ~6-7x latency improvement).
- A cache hit beats even a well-indexed DB query not because "RAM beats disk" but because it skips the DB's per-query overhead entirely (parsing, locking, MVCC checks, WAL bookkeeping).
- TTL and explicit invalidation are the two blunt tools against staleness; staleness tolerance depends on the *read* (display vs transactional), not just the data.
- Good candidates: read-heavy/write-light, expensive-to-compute, staleness-tolerant. Bad candidates: write-heavy, read-your-own-write/strong-consistency, huge-and-rarely-read — a shopping cart fails two criteria at once.
- Cache stampede: a hot key's expiry under load can cascade into a DB outage — jittered TTLs or locking mitigate it (full depth Topic 035).

> **You now can:** explain what a cache is and why it works, name the full caching layer ladder, calculate expected latency improvement from a hit rate, explain the real mechanism behind cache-hit speed, and reason about when caching does and doesn't apply — including the display-vs-transactional staleness distinction.
