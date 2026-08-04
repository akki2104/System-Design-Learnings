# Topic 036: Distributed Caching with Redis/Memcached

**Module:** 3 — Caching
**Tier:** 🔴 MUST
**Completed:** 2026-08-04
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topics 032–035 covered caching theory generically, occasionally name-dropping "Redis" as a placeholder. This topic opens up the technology: what Redis gives beyond a plain hashmap, how Redis and Memcached genuinely differ, how either survives a restart or node failure, and where the sharp edges are. This is the topic where "I'd use Redis" turns from a buzzword into a defensible design decision.

---

## 2. Redis's Real Differentiator: Rich Data Structures

A plain cache is `get(key)` / `put(key, value)`. Redis gives data structures *inside* the value, each with atomic operations server-side:

```
STRING  → simple value, or counters via INCR/DECR (atomic, no read-modify-write race)
LIST    → ordered sequence, push/pop from either end — queues, activity feeds
SET     → unordered unique members — tag sets, "has user X liked post Y" checks
HASH    → field→value map within one key — a whole object's fields in one key
                                            (avoids fetching separate keys per field)
SORTED SET (ZSET) → members ranked by a score, O(log n) insert/range/rank
                     → leaderboards, rate limiters, or anything ordered by a
                       numeric score (e.g. messages in a conversation by timestamp)
```
```
ZADD leaderboard 1500 "alice"
ZADD leaderboard 2100 "bob"
ZREVRANGE leaderboard 0 9    ← top 10, O(log n + 10), no app-side sorting needed
```
This is why Redis shows up constantly in case studies: a rate limiter's sliding window, a leaderboard's top-K, a session's field-level updates — all map onto one of these structures with a single atomic server-side command, instead of the application fetching a blob, deserializing, modifying, and writing it all back (which reintroduces the exact race conditions Topic 026 covered).

---

## 3. Redis vs Memcached — the Actual Difference

```
MEMCACHED                          REDIS
──────────────────────────────     ──────────────────────────────
Pure key-value, strings only       Rich data structures (above)
Multi-threaded                     Single-threaded event loop per core
                                    (simple commands are atomic for free —
                                     no locking needed within one instance)
No persistence — pure cache,       Optional persistence (RDB/AOF, below) —
data gone on restart                survives a restart, though still not a
                                     substitute for a real durable DB
No built-in clustering             Built-in clustering (hash slots, below)
                                     with per-shard replication
No pub/sub, no scripting           Pub/Sub, Lua scripting, transactions
                                     (MULTI/EXEC)
```
**The one-line distinction:** Memcached is a simpler, pure caching layer with a smaller surface area. Redis is a data structure server that happens to be fast enough to also serve **as a cache** — not fast enough to replace a real database. Redis's data lives in memory (with optional disk persistence as a restart aid); the real database's disk-backed storage remains the actual, durable source of truth.

---

## 4. Persistence: Surviving a Restart (Without Becoming a Real DB)

Two mechanisms, same tradeoff shape as Topic 020/024's WAL discussion:

```
RDB (snapshot): periodic point-in-time dump of the whole dataset to disk
  + Fast to restart from (just load the snapshot)
  - Data written since the last snapshot is lost on a crash

AOF (Append-Only File): every write command logged to disk as it happens
  + Much more durable — replay the log to reconstruct near-exact state
  - Larger file, slower to restart (replaying a long log takes time)

Common real setup: BOTH together — RDB for fast bulk restarts, AOF for
  minimizing the loss window between snapshots.
```
**The trap to name explicitly (ties back to Topic 032's Common Mistakes):** even with AOF, Redis is not a substitute for a real durable, ACID database. If Redis is holding data that must never be lost, that data's source of truth belongs in Postgres/MySQL, with Redis as an accelerant in front of it — not the other way around.

---

## 5. Clustering: Scaling Redis Horizontally

A single Redis instance is one thread, one machine's RAM — eventually both become limits. Redis Cluster shards the keyspace across multiple nodes:

```
Keyspace divided into 16,384 fixed "hash slots"
  slot = CRC16(key) % 16384

Each slot is owned by exactly one shard (a shard = primary + N replicas)
Client hashes the key, routes directly to the owning shard
```
```
   Shard A (slots 0-5460)      Shard B (slots 5461-10922)     Shard C (slots 10923-16383)
   ┌──────────┐                ┌──────────┐                   ┌──────────┐
   │ Primary  │                │ Primary  │                   │ Primary  │
   │ + 2 Repl │                │ + 2 Repl │                   │ + 2 Repl │
   └──────────┘                └──────────┘                   └──────────┘
```
Each shard replicates independently for HA (a replica takes over if its primary fails — the cache-outage mitigation Topic 035 previewed). This hash-slot scheme previews Topic 042's Consistent Hashing — same underlying goal (distribute keys, minimize reshuffling on node changes), Redis's specific fixed-slot-count implementation.

**A sharp edge worth naming:** a multi-key operation (transactions, or Lua scripts touching several keys) is only atomic if all those keys land on the *same* shard. Redis supports "hash tags" (`{user123}:profile`, `{user123}:sessions`) to force related keys onto the same slot deliberately — otherwise a transaction spanning shards simply isn't possible the way it would be on a single instance.

---

## Tech Decision Box: Redis vs Memcached vs Managed (ElastiCache/MemoryDB)

```
Use MEMCACHED when:
  - Pure, simple key-value caching is the entire need
  - Multi-threaded throughput per instance matters more than features
  - You don't need persistence, clustering, or rich data types

Use REDIS when:
  - You need data structures beyond plain KV (leaderboards, rate limiters,
    queues, session field-updates) with atomic server-side operations
  - You want built-in HA (replication) and horizontal scaling (clustering)
  - Some persistence across restarts is valuable, even if not full durability

Use a MANAGED service (AWS ElastiCache, MemoryDB) when:
  - You want Redis/Memcached's benefits without operating the cluster
    yourself (failover, patching, backups handled by the provider)
  - Team lacks dedicated ops bandwidth for a self-managed cluster
```
**Interview sentence:** "I'd use Redis here rather than Memcached because the leaderboard needs a sorted set with atomic rank queries — Memcached would force me to fetch, sort, and rewrite the whole thing in the app layer, reopening exactly the race conditions a cache is supposed to avoid. I'd run it as a managed ElastiCache cluster rather than self-hosted, since failover and patching aren't where I want engineering time going."

---

## Common Mistakes

| Mistake | Correction |
|---|---|
| Describing Redis as "fast enough to be used as a database" | Inverts the point: Redis is fast enough to serve AS A CACHE despite being a full data-structure server — it is not fast/durable enough to replace a real database. Data that must never be lost belongs in a proper ACID database with Redis in front of it |
| Treating Redis's persistence (RDB/AOF) as making it a real durable database | It reduces data loss on restart, but doesn't provide the durability guarantees of a real ACID database |
| Assuming Redis and Memcached are interchangeable, picking whichever is more familiar | Redis's data structures, persistence, and clustering are real differentiators when the use case needs them — Memcached is the right, simpler choice when it doesn't |
| Running expensive O(n) commands (`KEYS *`, unbounded `SORT`) in production | Redis is single-threaded per instance — one slow command blocks every other client on that thread until it finishes |
| Assuming a multi-key transaction is atomic in Redis Cluster mode | Only true if all keys hash to the same slot — use hash tags to force related keys together, or the transaction can't span shards at all |

---

## Real Interview Questions

1. "Why would you pick Redis over Memcached for this component?" (tests naming the specific data-structure/persistence/clustering need, not just familiarity)
2. "How does Redis stay fast without needing explicit locks for simple operations?" (single-threaded event loop → free atomicity for single-key ops)
3. "What happens to your cached data if the Redis process restarts? What if the whole box dies?" (RDB/AOF vs total data loss without persistence, tied to replication for HA)
4. "How would you scale a single Redis instance that's outgrown one machine's RAM?" (clustering, hash slots)
5. "Is a Lua script touching three keys guaranteed atomic in a Redis Cluster?" (only if same slot — hash tags)

---

## 6. Revision Questions
See `Revision/Revision_036.md`.

## 7. Summary
- Redis's real differentiator over a plain cache is server-side data structures (String, List, Set, Hash, Sorted Set) with atomic operations — this is why it shows up in leaderboards, rate limiters, and session stores specifically.
- Memcached is the simpler, pure-KV, multi-threaded choice; Redis trades some of that simplicity for data structures, persistence, and built-in clustering/HA.
- Redis is fast enough to serve AS A CACHE, not fast/durable enough to replace a real database — RDB and AOF reduce restart data loss but don't provide ACID durability guarantees.
- Redis Cluster shards via 16,384 fixed hash slots, each shard replicated for HA — a preview of Topic 042's Consistent Hashing.
- Redis is single-threaded per instance: simple ops get free atomicity, but one slow command can block everything else on that thread; multi-key atomicity across a cluster requires hash tags to force same-slot placement.

> **You now can:** name Redis's data structures and their real use cases, explain the actual Redis-vs-Memcached difference (not a vibe), reason about RDB/AOF's durability tradeoff and why it still isn't a real database, and describe how Redis Cluster shards and replicates data.
