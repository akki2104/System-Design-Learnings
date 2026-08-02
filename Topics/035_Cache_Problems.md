# Topic 035: Cache Problems (Stampede, Penetration, Avalanche)

**Module:** 3 — Caching
**Tier:** 🔴 MUST
**Completed:** 2026-08-02
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Topics 032-034 built a working mental model of a healthy cache: hot data, good hit ratio, sensible eviction. This topic covers the ways that healthy picture breaks under real-world load — the single most common "let me push on your design" interviewer follow-up after a cache is proposed. There are three distinct, commonly-confused failure modes, and naming them precisely (not just "the cache had a problem") is itself a signal of depth.

---

## 2. The Three Problems — Clearly Distinguished

They get conflated constantly because they all end with "DB gets hammered," but the *trigger* differs for each:

| Problem | Trigger | What's actually happening |
|---|---|---|
| **Cache Penetration** | Queries for data that doesn't exist in cache **OR** the DB | Every request for this key is guaranteed to miss — forever — because there's nothing to ever populate the cache with |
| **Cache Breakdown / Stampede** | One very hot key expires | Thousands of concurrent requests for that *one* key all miss simultaneously and all regenerate it at once |
| **Cache Avalanche** | Many keys expire at once, OR the cache cluster itself goes down | A large fraction of *all* traffic suddenly has no cache to hit, all at once |

Mental shorthand: **penetration** = a hole that was never plugged, **stampede** = one popular door with a crowd rushing it the instant it unlocks, **avalanche** = every door in the building unlocking at once (or the building disappearing).

---

## 3. Cache Penetration

**Mechanism:** A client repeatedly requests a key that exists in neither the cache nor the database — e.g., `GET /users/99999999` where that ID was never created. Cache-aside's logic (Topic 033) says "on miss, query DB, then populate cache" — but the DB has nothing either, so there's nothing to cache. Every request for this key pays the full DB round-trip forever, with zero benefit from caching. This can be accidental (a bad client bug hammering a bad ID) or malicious (an attacker probing random/non-existent IDs) — either way it's a DB-overload vector that completely bypasses the cache's protection.

**Mitigations:**
- **Cache the negative result too.** Store a sentinel value (e.g., `null`/`"NOT_FOUND"`) with a short TTL when the DB confirms a key doesn't exist. Subsequent requests for the same missing key hit the cache and return quickly instead of re-querying the DB every time.
- **Bloom filter in front of the DB** (Topic 089 preview). A probabilistic structure with a one-sided error: it can say "might exist" (false positives possible) or "definitely does not exist" (never a false negative). Check it before touching cache or DB — if it says "definitely not," reject immediately. This asymmetry is exactly why it's safe to use as a hard gate: it never wrongly blocks a real key, it only occasionally lets a nonexistent one through for a normal (cheap, now-cached) miss.

---

## 4. Cache Breakdown / Stampede

**Mechanism:** A single very hot key (a viral post, a flash-sale product, a trending ticker) expires via TTL. In the instant after expiry, potentially thousands of concurrent requests for that exact key all miss at the same moment, and — following cache-aside's normal logic — all of them independently query the DB and try to repopulate the cache. The DB, which was only ever protected from *this one key's* traffic by the cache, suddenly receives the full unfiltered load for that key all at once. For a sufficiently hot key, this alone can take the DB down.

**Mitigations:**
- **Mutex/lock on regeneration.** Only the first request that misses acquires a lock and queries the DB; every other concurrent request for that same key waits briefly for the lock holder to repopulate the cache, then reads the fresh value instead of independently hitting the DB. Converts N concurrent DB queries into 1 — but every waiting request is blocked synchronously until the lock holder finishes, which itself can become a bottleneck under very high concurrency.
- **Logical (soft) expiration.** Never let the entry actually expire from the client's point of view. Store a "logical expiry" timestamp *inside* the value. On read, if that timestamp has passed, serve the (slightly stale) cached value immediately AND kick off a background refresh. No request ever waits on a synchronous DB round-trip, and only one background refresh happens — this decouples every requester from the wait entirely, unlike a mutex where requests still queue.
- **Proactive/staggered refresh:** background-refresh the key a few seconds ahead of its real TTL so it's never actually cold at the moment of peak demand.

---

## 5. Cache Avalanche

**Mechanism — two distinct causes, same symptom (mass DB overload):**
1. **Mass simultaneous TTL expiry:** many *different* keys were cached around the same time with the same TTL (e.g., an entire product catalog warmed at 9:00am with TTL=1hr, all expiring at 10:00am simultaneously). A large fraction of all traffic misses at once, and the DB gets hit with the load of the *entire warmed set*, not just one key.
2. **Cache cluster outage:** the distributed cache itself (all of Redis, say) crashes or becomes unreachable. 100% of traffic that used to be absorbed by the cache falls straight through to the DB instantaneously, with zero warning.

**Mitigations:**
- **For cause 1 (mass expiry): jitter every TTL.** Instead of `TTL = 3600s` for every key in a batch, use `TTL = 3600 + random(0, 300)` — spreading expirations across a window instead of one instant. The single most important, cheapest fix; should be a default habit whenever bulk-populating a cache.
- **For cause 2 (cache outage): high availability for the cache itself.** Run the distributed cache as a replicated cluster (Topic 036 preview) instead of a single node, so one node's failure doesn't take the whole cache down.
- **Circuit breaker in front of the DB** (Topic 070 preview). If the DB starts timing out/erroring under sudden load, stop sending it more requests for a cooldown window and fail fast (or serve degraded/stale data) instead of piling on and worsening the outage.
- **Multi-tier caching** (e.g., an in-process L1 cache in front of the distributed L2 cache) so a distributed-cache outage means degraded caching, not zero caching.

---

## 6. Decision Box: Matching the Defense to the Problem

```
Cache Penetration  → cache negative results (short TTL) + Bloom filter in front of DB
Cache Stampede     → mutex/lock on regeneration, OR logical expiry + background refresh
Cache Avalanche    → jittered TTLs (mass-expiry cause) + HA cache cluster + circuit
                      breaker + multi-tier caching (outage cause)
```

**Interview sentence:** "For this hot-key risk, I'd use logical expiration so no request ever synchronously waits on the DB during a refresh. Separately, since we're bulk-warming the catalog cache, I'd jitter the TTLs so we don't get an avalanche when they all expire at once — and I'd put a circuit breaker in front of the DB as a last-resort safety net regardless of which failure mode actually hits."

---

## 7. Worked Example

A news site caches article pages with `TTL=600s`, all warmed at deploy time. Three independent things could go wrong:
- **One article goes viral** and its individual cache entry expires under heavy concurrent load → **stampede** (fix: lock or logical expiry on that key).
- **The whole catalog was warmed together** and all articles' TTLs expire around the same 10-minute mark → **avalanche by mass expiry** (fix: jitter each article's TTL individually).
- **A bot scrapes `/articles/-1`, `/articles/999999999`**, etc. (IDs that never existed) → **penetration** (fix: cache the negative result + Bloom filter).

Three different problems, three different fixes — the interview signal is picking the *right* one for the *stated* trigger, not reaching for "add a lock" or "add jitter" reflexively for all three.

---

## 8. Common Mistakes

| Mistake | Correction |
|---|---|
| Using "cache stampede" as a catch-all term for any cache-related DB overload | Penetration (never-existed key), stampede (one hot key expiring), and avalanche (mass expiry or total outage) have different triggers and different fixes — naming the wrong one signals shallow understanding |
| Proposing a Bloom filter to fix stampede or avalanche | Bloom filters solve penetration specifically — they answer "does this key exist at all," which is irrelevant once a key that *does* exist expires (stampede/avalanche's actual problem) |
| Fixing avalanche only by jittering TTLs, ignoring the cache-outage cause | Jitter only fixes mass-simultaneous-expiry; it does nothing if the cache cluster itself goes down — that needs HA/replication + a circuit breaker as a separate mitigation |
| Treating a lock/mutex fix as free | Every waiting request blocks synchronously until the lock holder finishes — under very high concurrency this can itself become a bottleneck; logical/soft expiration (serve stale + background refresh) avoids blocking entirely |

---

## 9. Revision Questions
See `Revision/Revision_035.md`.

## 10. Summary
- Three distinct cache failure modes, each with a different trigger: **penetration** (key never existed anywhere), **stampede** (one hot key expires under concurrent load), **avalanche** (many keys expire together, or the cache cluster itself dies).
- Penetration fix: cache negative results + a Bloom filter's one-sided error (never a false negative) as a hard gate in front of the DB.
- Stampede fix: mutex/lock (converts N DB queries to 1, but requests still wait) or logical/soft expiration (serve stale immediately + background refresh, no request ever waits).
- Avalanche fix depends on the cause: jittered TTLs for mass-expiry, HA cache cluster + circuit breaker + multi-tier caching for a total cache outage.
- The interview skill is matching the specific fix to the specific trigger, not reaching for one fix (lock, jitter, Bloom filter) reflexively for all three problems.
