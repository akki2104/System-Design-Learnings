# Revision — Topic 036: Distributed Caching with Redis/Memcached

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-04

---

## Q1. Name Redis's four extra data structures beyond a plain string value, and give one real use case for each.

<details>
<summary>Answer</summary>

List (queues/activity feeds), Set (membership checks — "has user X liked post Y"), Hash (an object's fields in one key), Sorted Set/ZSET (ranked by score — leaderboards, rate limiters, or anything ordered by a numeric value like timestamp).

</details>

---

## Q2. What's the actual difference between Redis and Memcached, and where does Redis's data sit relative to a real database?

<details>
<summary>Answer</summary>

Memcached is a pure key-value store, multi-threaded, no persistence, no clustering — the simpler choice when that's all you need. Redis is a data-structure server (rich types, single-threaded event loop, optional persistence, built-in clustering/replication) that's fast enough to serve AS A CACHE — not fast or durable enough to replace a real database. Redis's data lives in memory (with optional disk persistence as a restart aid); the real database's disk-backed storage remains the actual source of truth.

</details>

---

## Q3. Compare RDB and AOF — what does each protect against, and what's the tradeoff?

<details>
<summary>Answer</summary>

RDB is a periodic point-in-time snapshot to disk — fast to restart from, but loses everything written since the last snapshot on a crash. AOF logs every write command as it happens — much more durable (replay reconstructs near-exact state), but the file is larger and restart is slower since the log must be replayed. Common real setups use both together.

</details>

---

## Q4. Why can a single slow command in Redis block every other client, and why doesn't a single-key `INCR` need an explicit lock?

<details>
<summary>Answer</summary>

Redis processes commands on a single thread per instance — one command runs to completion before the next starts, so a slow command (e.g. `KEYS *` or an unbounded `SORT`) blocks every other client until it finishes. That same single-threaded execution is exactly why `INCR` (and other single-key ops) are atomic for free — no other command can interleave mid-operation, so no explicit lock is needed.

</details>

---

## Q5. How does Redis Cluster decide which node owns a given key, and what has to be true for a multi-key operation to stay atomic under clustering?

<details>
<summary>Answer</summary>

The keyspace is divided into 16,384 fixed hash slots; a key's slot is `CRC16(key) % 16384`, and each slot is owned by exactly one shard (primary + replicas). A multi-key operation is only atomic if all the keys involved hash to the same slot — achieved deliberately via hash tags (e.g. `{user123}:profile`, `{user123}:sessions`) that force related keys onto the same slot. Without that, a transaction spanning multiple shards isn't possible the way it would be on a single instance.

</details>

---

## 30-Second Elevator Pitch

> Redis's real differentiator over a plain cache is server-side data structures — String, List, Set, Hash, Sorted Set — each with atomic operations, which is why it shows up in leaderboards, rate limiters, and session stores. Memcached is the simpler pure-KV, multi-threaded alternative with no persistence or clustering. Redis is fast enough to serve as a cache, not fast or durable enough to replace a real database — RDB (snapshot, fast restart, lossy) and AOF (write log, durable, slower restart) reduce restart data loss but don't provide ACID guarantees. Redis Cluster shards via 16,384 fixed hash slots, each shard replicated for HA; multi-key atomicity across shards requires hash tags to force same-slot placement. Redis is single-threaded per instance, which gives free atomicity for simple ops but means one slow command blocks everyone else.

---

## Weak Areas to Watch

- First-pass framing of the Redis-vs-Memcached distinction drifted toward "Redis is fast enough to be used as a database" — the inverse of the real point (fast enough to serve as a cache, not to replace a database). Self-corrected on a direct nudge; worth a quick re-check next revision pass.
