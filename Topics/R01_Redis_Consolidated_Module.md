# Topic R01: Redis — The Consolidated System Design Module

**Track:** R — Technology Deep Dives (cross-module consolidation; not a canonical roadmap number)
**Consolidates:** Topics 017, 028, 031, 032, 033, 034, 035, 036, 037, 039, 040, 041 + previews 042, 056, 059, 060, 061, 070, 071, 082, 089, 090, 092, 098, 099
**Tier:** 🔴 MUST
**Time budget:** ~4.5 h lesson + ~1.0 h revision ≈ **1–1.5 days**
**Status:** 📦 **CONSOLIDATED — READY TO LEARN** (not taught, not completed, not mastered)
**Created:** 2026-08-31
**Confidence:** — (not yet assessed)

---

> ## How to use this file
>
> This is the **single authoritative Redis document** in the repository. Everything Redis-related that
> was previously scattered across the caching module, the load-balancer topic, the database-choice
> topic and `TechChoices.md` is either restated here or explicitly cross-referenced.
>
> **Scope discipline (deliberate):** this is a *System Design interview* module for a ~2-YOE engineer,
> not a Redis administration course. It optimises for **mental model → use cases → architecture →
> trade-offs → failure modes → interview questions**. Command syntax appears only where the command
> name *is* the answer an interviewer expects (`SET NX PX`, `INCR`, `ZADD`, `XREADGROUP`). Anything
> that only matters to a Redis operator is marked ⚫ **beyond scope** and given one line.
>
> **What is NOT re-taught here** (already complete elsewhere — read those first if rusty):
>
> | Already covered in | What it covers |
> |---|---|
> | [032](032_Caching_Fundamentals.md) | Why cache at all, cache layers ladder, hit ratio math |
> | [033](033_Caching_Patterns.md) | Cache-aside / write-through / write-behind mechanics |
> | [034](034_Eviction_Policies.md) | LRU / LFU / FIFO / TTL as *policies* |
> | [035](035_Cache_Problems.md) | Penetration / stampede / avalanche triggers and fixes |
> | [036](036_Distributed_Caching_Redis_Memcached.md) | First pass at Redis vs Memcached, RDB/AOF, hash slots |
> | [037](037_Cache_Consistency_Invalidation.md) | Invalidation ordering, multi-copy invalidation |
> | [039](039_Replication.md) / [040](040_Replication_Lag_Read_Your_Writes.md) / [041](041_Partitioning_and_Sharding.md) | Replication & sharding as general concepts |
>
> This file's job is the **Redis-specific layer on top of those**: what Redis actually does, what it is
> used for *besides* caching, how it fails, and how to defend a Redis choice under interviewer pressure.

---

## 1. Why This Topic Exists

Redis is the single most name-dropped technology in system design interviews, and the single most
*shallowly* defended one. Candidates say "I'll put Redis in front of the database" and stop. The
interviewer's next four questions — *which data structure? what happens when it dies? is that
operation atomic? why not Memcached/Kafka/Postgres?* — are where the actual signal is.

Topic 036 opened Redis as a caching technology. But in real interviews Redis shows up as a **rate
limiter**, a **distributed lock**, a **session store**, a **leaderboard**, a **presence tracker**, an
**idempotency ledger**, a **geospatial index**, and a **lightweight queue** — often several of these in
one design. Those uses are spread across Modules 6, 7, 8 and 9 of the roadmap, which means without
this consolidation the learner meets Redis eight separate times and never assembles one coherent
model of it.

This module assembles that model **once**, so that Redis becomes a component you reason about rather
than a word you sprinkle.

---

## 2. Real Production Problem

**The scenario, drawn from a shape that recurs in real incident write-ups:**

A commerce team puts Redis in front of Postgres for the product catalog. Traffic grows 10×. Then, on
a flash-sale morning, four things happen within ninety seconds:

1. The sale item's cache entry expires. Forty thousand concurrent requests miss the same key and all
   query Postgres at once — **stampede** ([035](035_Cache_Problems.md)).
2. That one key was also being read 200k times/sec from a single Redis shard. That shard's single
   command thread saturates while the other five shards idle — a **hot key / hot shard**. Adding
   Redis nodes does nothing, because one key cannot be split across nodes.
3. An engineer runs `KEYS product:*` on the primary to investigate. Redis executes commands on one
   thread; that O(n) scan blocks **every** other client on that instance for seconds — the
   investigation makes the outage worse.
4. Memory hits `maxmemory`. The eviction policy is the default `noeviction`, so Redis starts
   returning errors on writes instead of evicting — the "cache" now rejects work rather than
   degrading.

Every one of those four failures is a standard interview follow-up. None of them are fixed by "add
more Redis." The fixes are: logical expiration, a local L1 cache in front of the hot key, `SCAN`
instead of `KEYS`, and `allkeys-lru` — four different mechanisms that this module has to give you.

---

## 3. Simple Intuition — the Mental Model

> **Redis is a single, very fast clerk standing at a wall of labelled boxes, holding one clipboard.**

- **The boxes are in RAM, not a filing cabinet.** That is the primary reason it is fast — no disk seek,
  no query planner, no join. Reads are memory-address lookups.
- **The boxes are not all the same shape.** Some hold a single note (String), some a stack of notes
  (List), some a set of unique names (Set), some a form with fields (Hash), some a ranked scoreboard
  (Sorted Set), some an append-only logbook (Stream). *Choosing the right box shape is 80% of using
  Redis well.*
- **There is exactly one clerk, serving one request at a time, to completion.** This is why every
  single command is atomic for free — nobody can interleave. It is also why one slow request
  (`KEYS *`, a huge `SORT`, a 200 MB value) stalls the entire queue behind it.
- **The clerk has a fixed-size room.** When it fills up, something must be thrown out (eviction) —
  and you choose the throw-out rule.
- **The room can burn down.** Redis writes to disk only as an *afterthought*, to help it come back up.
  It is not a vault. **Anything you cannot afford to lose must also live somewhere else.**

**The one sentence to internalise (and the trap logged in
[InterviewMistakes.md](../InterviewMistakes.md) on 2026-08-04):**

> Redis is fast enough to serve **as a cache and as a coordination/state layer** — it is *not*
> durable enough to *be* your database. The source of truth stays in Postgres/DynamoDB/S3.

---

## 4. Core Concepts

### 4.1 What Redis actually is

**REmote DIctionary Server.** An in-memory **data-structure server**: a networked process holding
typed values in RAM, exposing atomic server-side operations on those types, with optional persistence
and optional replication/sharding.

The words that matter, in order:

- **In-memory** → sub-millisecond latency, but capacity is bounded by RAM and cost per GB is ~50–100×
  disk.
- **Data-structure** → not just `get`/`put`. This is the actual differentiator over Memcached.
- **Server** → shared across all your app instances, which is what makes it useful for *distributed*
  state (locks, counters, sessions) that a per-process in-memory map cannot provide.

### 4.2 Why Redis is fast — the four real reasons

| Reason | Mechanism | Interview phrasing |
|---|---|---|
| **1. RAM, not disk** | No seek, no page fetch, no buffer-pool miss. | "The dominant term: a memory read is ~100 ns, an SSD read ~100 µs — three orders of magnitude." |
| **2. No query layer** | No parser/planner/optimiser/join engine. The command names the exact data structure operation. | "You're not asking a question, you're naming an operation." |
| **3. Single-threaded command execution + I/O multiplexing** | One event loop uses `epoll`/`kqueue` to multiplex thousands of connections; commands execute one at a time to completion. No locks, no mutex contention, no context switching between command executions. | "It's fast *because* it's single-threaded, not despite it — zero lock overhead and zero contention." |
| **4. Efficient encodings + a simple protocol** | Small collections use compact memory encodings; RESP is a trivially parseable wire protocol. | Worth one clause, not a paragraph. |

**The nuance that scores points:** since **Redis 6.0**, *network I/O* (reading/writing sockets) can be
offloaded to helper threads (`io-threads`), so a busy instance can saturate a modern NIC. **Command
execution itself is still single-threaded**, so the atomicity guarantee and the "one slow command
blocks everyone" hazard both still hold. Say both halves — candidates who only say "Redis is
single-threaded" sound like they read it in 2015; candidates who only say "Redis is multi-threaded
now" have the guarantee wrong.

⚫ *Beyond scope:* jemalloc tuning, `listpack`/`quicklist` encoding thresholds, RESP2 vs RESP3 framing.

### 4.3 The numbers to carry into an interview

```
Latency (same DC, single simple command) : ~0.1 – 1 ms  (p99 often < 1 ms)
Throughput per instance (simple commands): ~50k – 100k+ ops/sec  (single core bound)
                                            → 500k–1M+/sec with pipelining or io-threads
Max practical dataset per node           : bounded by RAM; keep nodes ≤ ~25–50 GB so that
                                            fork/BGSAVE, failover and resync stay fast
HyperLogLog                              : ≤ 12 KB per key, 0.81% standard error, up to 2^64 items
Redis Cluster                            : 16,384 fixed hash slots; slot = CRC16(key) mod 16384
```

> Compare against [Numbers.md](../Numbers.md). The interview-useful comparison: a Redis hit is
> ~0.5 ms; a well-indexed Postgres query is ~5–10 ms; an uncached page-load chain of 5 DB calls is
> ~50 ms. That ratio is *why* the cache exists — restate it from [032](032_Caching_Fundamentals.md)
> rather than re-deriving it.

### 4.4 When to use Redis / when NOT to

```
✅ USE REDIS WHEN
  - Read-heavy access to a small hot subset of a much larger dataset      → cache
  - You need shared, low-latency, mutable state across many app servers   → counters, sessions,
    that a per-process map cannot provide                                    locks, rate limits
  - The operation maps onto a Redis data structure with an atomic
    server-side command (rank, increment, add-if-absent, pop, range)      → the real differentiator
  - Data is naturally ephemeral or reconstructible                        → TTL is a feature, not a risk
  - You need sub-millisecond p99 and the DB cannot give it

❌ DO NOT USE REDIS WHEN
  - It would be the source of truth for data you cannot afford to lose    → use a durable ACID DB
  - You need ad-hoc queries, joins, secondary-index search, aggregation   → SQL / Elasticsearch
  - You need multi-key ACID transactions with rollback                    → Postgres (see §4.9)
  - The working set doesn't fit in RAM economically (cold/archival/blobs) → disk DB / S3
  - You need durable, replayable, long-retention event history            → Kafka (see §10.4)
  - The data is written far more than it's read, and never re-read        → caching it is pure waste
  - A single in-process map would do (single instance, no sharing needed) → don't add a network hop
```

**Interview sentence:** *"Redis here for the session store and the rate-limit counters, because both
need shared mutable state across all app servers with sub-millisecond access, and both are
reconstructible if lost. Not Redis for the order records — those need durable multi-row transactions,
so they stay in Postgres."*

---

### 4.5 Data Structures — Chosen by Use Case, Not by Syntax

> The interview skill is **"which structure, and why that one"** — never a command list. For each
> structure below: what it is, the operation that makes it special, and the system-design use it
> unlocks.

| Structure | Mental picture | The operation that matters | Design use it unlocks |
|---|---|---|---|
| **String** | One labelled box holding bytes (text, JSON blob, number, bitmap) | `SET k v EX ttl NX` (set-if-absent with expiry) · `INCR`/`INCRBY` (atomic counter, no read-modify-write race) · `GETSET` | Cached serialized object · **atomic counters** (views, likes, fixed-window rate limits) · **distributed lock** · **idempotency key** · feature flags |
| **Hash** | A form with named fields under one key | `HSET`/`HGET`/`HINCRBY` — update **one field** without fetching/rewriting the whole object | **Session store** (update `last_seen` without rewriting the session) · cached entity with independently-updated fields · shopping cart (`cart:{user} → {sku: qty}`) · memory-efficient storage of many small objects |
| **List** | A stack/queue of notes, ordered by insertion | `LPUSH` + `BRPOP` (**blocking** pop — a worker sleeps until work arrives, no polling) · `LMOVE`/`BLMOVE` for reliable hand-off | **Simple job queue** · recent-activity feed capped with `LTRIM` · producer/consumer between services |
| **Set** | A bag of unique names, unordered | `SADD`/`SISMEMBER` O(1) · `SINTER`/`SUNION`/`SDIFF` server-side | Unique-membership checks ("has user X liked post Y") · tags · **mutual friends** (`SINTER`) · deduplication · online-user set |
| **Sorted Set (ZSET)** | A scoreboard: unique members, each with a numeric score, always kept in score order | `ZADD` O(log n) · `ZRANGE`/`ZREVRANGE` for top-K · `ZRANK` for "my position" · `ZRANGEBYSCORE`/`ZREMRANGEBYSCORE` for windowed slices | **Leaderboards** (top-K *and* a user's own rank, both server-side) · **sliding-window rate limiter** (score = timestamp) · **delayed/priority queue** (score = run-at time) · **presence** (score = last heartbeat) · time-ordered feeds |
| **Stream** | An append-only logbook with IDs, plus bookkeeping of who has read what | `XADD` · `XREADGROUP` + `XACK` (**consumer groups**: each entry goes to exactly one consumer in the group) · pending-entries list + `XCLAIM` for redelivery after a consumer dies | **Durable-ish event log / work queue with acknowledgement and redelivery** — the "Kafka-lite" option. Use when a List is too lossy but Kafka is too heavy (see §10.4) |
| **Bitmap** (on String) | A row of on/off switches, one bit per ID | `SETBIT`/`GETBIT`/`BITCOUNT`/`BITOP` | **Daily-active-users** by user ID (1 bit/user ≈ 1.2 MB per 10M users) · per-user boolean feature matrices · attendance/retention grids |
| **HyperLogLog** | A tiny sketch that remembers *how many distinct* things it saw, not which | `PFADD`/`PFCOUNT`/`PFMERGE` — ≤ **12 KB per key**, **0.81%** standard error, O(1) | **Unique-visitor / unique-viewer counts at massive scale** where exactness isn't required and a Set would cost gigabytes. `PFMERGE` gives "uniques this week" from seven daily keys |
| **Geospatial** (on ZSET) | Locations encoded as geohash scores in a sorted set | `GEOADD` · `GEOSEARCH` by radius or box, sorted by distance | **"Drivers near me" / "restaurants within 2 km"** — the Uber/Swiggy proximity query (see §17.4) |
| ⚫ Others (one line) | — | — | `Bloom filter` (probabilistic membership — the cache-penetration gate from [035](035_Cache_Problems.md)) · `Count-Min Sketch` (frequency) · `Top-K` (heavy hitters) · `t-digest` (percentiles) · `JSON` · `Time Series` · `Vector Set` (similarity search for AI/RAG). Know they exist and roughly what each answers; do not study internals at this level. |

**The decision heuristic to say out loud:**

```
Need a plain blob or a number?         → String
Need to update one field of an object? → Hash
Need FIFO order / blocking hand-off?   → List
Need uniqueness or set math?           → Set
Need ORDER BY a score, rank, or a
  time-window slice?                   → Sorted Set          ← the interview workhorse
Need consumer groups + ack + replay?   → Stream
Need approximate counting at scale?    → HyperLogLog / Bitmap
Need "near me"?                        → Geospatial
```

**Anti-pattern to name:** storing a JSON blob as a String and having the app fetch → deserialize →
mutate one field → serialize → write back. That is a read-modify-write cycle across the network: it
is slow, it wastes bandwidth, and **it reintroduces exactly the lost-update race
[026](026_Concurrency_Control_Locks_2PL_Deadlocks.md) covers**. A Hash with `HSET`/`HINCRBY` does it
in one atomic server-side op.

---

### 4.6 Redis as a Cache — the Redis-specific layer

> Patterns and problems themselves are covered in [033](033_Caching_Patterns.md) and
> [035](035_Cache_Problems.md). Below is only what changes *because it's Redis*.

**Cache-aside in Redis, concretely:**

```
READ   : GET product:123
         miss → SELECT from Postgres → SET product:123 <json> EX 300 → return
WRITE  : UPDATE Postgres  →  DEL product:123        (delete, never overwrite — see 033/037)
```

**TTL — the Redis mechanics.** `EXPIRE`/`SET ... EX` set an absolute countdown from write time; a
read does **not** refresh it (the misconception logged for [034](034_Eviction_Policies.md) —
sliding TTL is opt-in via `GETEX`). Redis removes expired keys two ways: **lazily** (on access) and
**actively** (a background job samples random keys with TTLs many times per second). *Consequence:* an
expired key can still occupy memory until sampled — expiry is prompt logically but not instant
physically.

**Eviction — the config values to name.** When memory reaches `maxmemory`, `maxmemory-policy` decides:

| Policy | Behaviour | Use when |
|---|---|---|
| `noeviction` | **Reject writes with an error** (this is the **default**) | Redis holds data you must not silently drop — but understand it makes a full Redis *fail writes*, not degrade |
| `allkeys-lru` | Evict least-recently-used across all keys | **Pure cache — the usual answer** |
| `allkeys-lfu` | Evict least-frequently-used | Skewed popularity where a one-off scan shouldn't evict genuinely hot keys |
| `volatile-lru` / `volatile-ttl` / `volatile-random` | Evict only among keys that have a TTL | Mixed instance: cache entries (TTL'd) alongside must-keep state (no TTL) |
| `allkeys-random` | Random victim | Rarely the right answer; cheap |

**The interview trap here:** "what happens when Redis runs out of memory?" — the answer is *"it
depends on `maxmemory-policy`, and the default `noeviction` means writes start failing, which is
usually **not** what you want for a cache; I'd set `allkeys-lru`."* Saying "it evicts old data"
assumes a non-default config.

**Eviction ≠ expiry.** TTL is *"this datum has a lifetime"*; eviction is *"we're out of room."*
They are different axes ([034](034_Eviction_Policies.md)) and both can remove your key.

**Cache invalidation in Redis:** `DEL`/`UNLINK` the key (prefer `UNLINK` — it frees memory in a
background thread, so deleting a huge collection doesn't block the command thread). For multi-layer
caches, a bare `DEL` only clears Redis, not each app server's in-process L1 — broadcast via **Pub/Sub**
or Redis **client-side caching** invalidation ([037](037_Cache_Consistency_Invalidation.md)).

**Cache stampede in Redis:** the mutex is `SET lock:product:123 <uuid> NX EX 10` (the lock recipe from
§4.8); logical expiration means storing `{"value":…, "soft_expiry":…}` and refreshing in the
background. Both mechanisms are already reasoned about in [035](035_Cache_Problems.md) — here you can
now name the exact commands.

**Cache penetration in Redis:** `SET user:99999999 "__NULL__" EX 60` for negative caching, plus a
Bloom filter (`BF.EXISTS`) as the hard gate.

**Cache consistency in Redis:** Redis knows nothing about your database. Every consistency guarantee
comes from *your* write path ordering ([037](037_Cache_Consistency_Invalidation.md)), not from Redis.
A dual-write to DB-and-Redis is not atomic; if the process dies between the two, the cache is stale
until TTL. Mitigations: short TTLs, delete-after-commit, delayed double-delete, or CDC/outbox-driven
invalidation ([066] preview).

**Cache failure — what happens if Redis dies:**

```
No HA          → 100% of read traffic falls to the DB instantly = AVALANCHE (035)
Replicas + failover → seconds of unavailability, then a replica takes over, cold-ish but alive
Mitigations     → HA topology (§4.7) + circuit breaker in front of the DB [070]
                  + in-process L1 cache so a Redis outage degrades rather than zeroes caching
                  + FAIL OPEN vs FAIL CLOSED must be a stated decision:
                      cache miss → serve from DB (fail open, correct for caching)
                      rate limiter → allow all traffic (fail open) or reject all (fail closed)?
                        state which, and why — this is a classic follow-up
```

---

### 4.7 Distributed Redis — Replication, Sentinel, Cluster

> General replication/sharding theory: [039](039_Replication.md), [040](040_Replication_Lag_Read_Your_Writes.md),
> [041](041_Partitioning_and_Sharding.md). Below is Redis's specific implementation and — critically —
> the **three-way distinction** interviewers probe.

#### The distinction, stated once and clearly

```
REPLICATION  = copying data.        primary → replicas. Gives read scaling + a warm spare.
                                    By itself: NO automatic failover. A human promotes.
SENTINEL     = automatic failover.  A separate quorum of monitor processes that detects a dead
                                    primary, elects a replica, promotes it, and tells clients.
                                    NO sharding — the dataset still fits on one primary.
CLUSTER      = sharding + failover. Data split across N shards by hash slot; each shard is its own
                                    primary+replicas and does its own failover. Scales BOTH memory
                                    and write throughput.

The one-liner: replication copies, Sentinel promotes, Cluster splits (and promotes).
Sentinel and Cluster are alternatives, not layers: if you run Cluster you do NOT run Sentinel.
```

#### Replication (Redis specifics)

- **Asynchronous by default.** The primary acknowledges the client *before* replicas confirm. This is
  the single most important Redis distributed-systems fact: **an acknowledged write can be lost if
  the primary dies before the write reaches a replica and that replica is promoted.**
- `WAIT numreplicas timeout` asks for N acknowledgements before returning — it *reduces* the loss
  window but, per Redis's own docs, **does not make Redis a CP system**; acknowledged writes can still
  be lost during a failover.
- `min-replicas-to-write` / `min-replicas-max-lag` make the primary **refuse writes** unless N replicas
  are within M seconds — bounding data loss by trading availability for safety.
- **Replicas are read-only by default** and serve **stale** reads (replication lag) — the
  read-your-own-writes problem from [040](040_Replication_Lag_Read_Your_Writes.md) applies verbatim.
  *Never* read a rate-limit counter or a lock from a replica.
- **Partial resync**: replicas that briefly disconnect catch up from a backlog buffer using
  replication-ID + offset; only if the backlog has rolled over does a **full resync** happen (primary
  forks, produces an RDB, ships it) — expensive, and a real source of latency spikes.
- **Danger to name:** a primary with persistence *off* that auto-restarts comes back **empty**, and
  its replicas dutifully replicate the emptiness — wiping the whole dataset. Redis's docs call this
  out explicitly.

#### Sentinel

Three or more Sentinel processes monitor the primary. On a quorum agreeing it is down (subjectively
down → objectively down), they elect a leader Sentinel, promote a replica, reconfigure the other
replicas, and serve the new address to Sentinel-aware clients. Clients must support Sentinel (they ask
Sentinel "who is the primary?" rather than hardcoding a host).

**Use when:** the whole dataset fits comfortably on one node, and you want HA without sharding
complexity. **That is most systems.**

#### Cluster

- Keyspace split into **16,384 fixed hash slots**, `slot = CRC16(key) mod 16384`. Each slot is owned
  by exactly one shard; each shard = primary + replica(s) doing its own failover. Clients are
  cluster-aware and route directly, following `MOVED`/`ASK` redirects during resharding.
- Why a **fixed** slot count rather than hashing straight to nodes: resharding moves *slots*, not
  rehashing every key — the same goal as consistent hashing ([042] preview), different implementation.
- **The sharp edge (already flagged in [036](036_Distributed_Caching_Redis_Memcached.md)):**
  multi-key operations — `MGET` across keys, `MULTI/EXEC`, Lua scripts touching several keys — only
  work if all keys are in the **same slot**. Force that with **hash tags**: `{user:123}:profile` and
  `{user:123}:cart` hash only on the `{…}` part, so both land together.
- **Availability caveat:** if a shard's primary and all its replicas die, the slots it owned are
  unavailable — by default the *whole cluster* stops serving (`cluster-require-full-coverage yes`).
- ⚫ *Beyond scope:* the gossip/cluster-bus protocol, `CLUSTER SETSLOT` migration mechanics, exact
  `node-timeout` election math.

#### Scaling and hot shards

```
Scale reads      → add replicas (accept staleness) OR add an in-process L1 cache
Scale memory     → Cluster (more shards)  OR  bigger node (vertical, 038) OR shorter TTLs / smaller values
Scale writes     → Cluster (more shards) — replicas do NOT scale writes
HOT KEY          → adding shards does NOTHING: one key = one slot = one shard = one thread.
                   Fixes: (a) L1 in-process cache in front of it (best first move),
                          (b) key splitting/replication: product:123#0..#N, read a random copy,
                              write fans out to all N (accepting N× write cost + brief skew),
                          (c) serve it from read replicas if staleness is acceptable,
                          (d) push it to the CDN/edge if it's a public read.
HOT SHARD        → uneven slot distribution or one dominant key prefix. Re-key to spread
                   (041's shard-key reasoning), or isolate the noisy tenant on its own shard.
BIG KEY          → a single multi-GB collection blocks the thread on access/delete and skews
                   memory. Split it; delete with UNLINK, never DEL.
```

---

### 4.8 Redis Beyond Caching — the section that separates candidates

> For each: **why Redis**, **which structure**, and **what breaks**.

#### (a) Rate limiting → String counter or Sorted Set

**Why Redis:** the counter must be **shared across all API servers** (a per-server in-memory counter
lets N servers each allow the full quota) and must be incremented **atomically** under concurrency.

- **Fixed window** — `INCR ratelimit:{user}:{minute}` then `EXPIRE` on first increment. One command,
  atomic, O(1), tiny memory. **Flaw:** boundary burst — 100 requests at 00:59 and 100 at 01:01 is 200
  in two seconds.
- **Sliding window log** — ZSET with score = timestamp: `ZREMRANGEBYSCORE` (drop old) →
  `ZCARD` (count) → `ZADD` (record), wrapped in **one Lua script** so the whole check-and-add is
  atomic. Accurate, no boundary burst. **Costs memory proportional to the request rate** (one member
  per request in-window).
- **Token bucket** — Hash holding `{tokens, last_refill}`, refilled and decremented inside a Lua
  script. Allows controlled bursts; the usual production choice.
- **What breaks:** Redis down → **fail open (allow) or fail closed (reject)?** State the choice.
  Public API abuse protection usually fails open (availability > perfect enforcement); billing quota
  or fraud limits may fail closed. Also: never read the counter from a replica.
- Full algorithm theory arrives in [071] Rate Limiting; this is the Redis implementation layer.

#### (b) Distributed locks → String with `SET NX PX` (+ the Redlock debate)

**Why Redis:** you need mutual exclusion across processes/machines, and Redis gives an atomic
"create-if-absent-with-expiry" in one round trip.

```
ACQUIRE : SET lock:resource <random_uuid> NX PX 30000
RELEASE : Lua compare-and-delete — delete ONLY IF the value is still my uuid
          if redis.call("get",KEYS[1]) == ARGV[1] then return redis.call("del",KEYS[1]) else return 0 end
```

Three non-negotiables to say out loud:

1. **TTL (`PX`) is mandatory** — otherwise a crashed holder deadlocks the resource forever.
2. **A unique random value is mandatory** — otherwise a slow client whose lock already expired will
   `DEL` the lock *a different client now holds*.
3. **Release must be an atomic compare-and-delete (Lua)** — a `GET`-then-`DEL` from the app has a
   race in the gap between the two.

**The Redlock debate (name it; it is a strong signal at any level):** because Redis replication is
asynchronous, a single-instance lock can be lost when the primary fails over — client A holds the
lock, the primary dies before replicating it, a replica is promoted, and client B acquires the *same*
lock. **Redlock** (acquire on a majority of N independent Redis primaries) was antirez's answer.
Martin Kleppmann's critique: Redlock relies on timing assumptions (a GC pause, clock jump, or network
delay can leave a client believing it still holds an expired lock) and provides no **fencing token**.
Both sides effectively converge on the practical rule:

> **Efficiency lock** (avoid duplicate work; double execution is merely wasteful) → a simple
> single-instance `SET NX PX` lock is fine.
> **Correctness lock** (two holders would corrupt data or double-charge) → you need **fencing
> tokens** (a monotonically increasing number the protected resource checks and uses to reject stale
> writers), or a consensus-backed lock service (**ZooKeeper / etcd**), or — best of all — **design
> the operation to be idempotent so you don't need the lock to be perfect.**

Redis's own documentation now states this: implement fencing tokens if you care about correctness, and
note that Redis TTL expiry does not use a monotonic clock (a wall-clock shift can hand the same lock to
two processes). Full treatment lands in [099] Distributed Locks & Leases.

#### (c) Session storage → Hash (or String) with TTL

**Why Redis:** it makes app servers **stateless**, which is what actually enables horizontal scaling
and lets any server serve any request — the correction already logged against
[017](017_Load_Balancers.md) (Redis sessions are *not* "a better sticky session"; they remove the need
for stickiness entirely). Hash lets you update `last_seen` without rewriting the whole session; TTL
gives free session expiry.

**What breaks:** Redis down = everyone logged out (mitigate with HA + replication, or short-lived JWTs
for auth with Redis only for revocation/denylist — [082] preview). Session data is *semi*-durable;
losing it is annoying, not corrupting.

#### (d) Counters → String `INCR` / Hash `HINCRBY`

**Why Redis:** `INCR` is atomic server-side, so a thousand concurrent increments are exact without a
row lock. Absorbs write bursts the DB can't take (view counts, likes), then flushes to the DB
periodically — this is **write-behind** ([033](033_Caching_Patterns.md)) with its data-loss trade-off
stated: a crash loses the un-flushed delta. Fine for view counts, **never** for account balances.

#### (e) Leaderboards → Sorted Set

**Why Redis:** the two queries a leaderboard needs — *top N* (`ZREVRANGE`, O(log n + N)) and *"what's
my rank?"* (`ZREVRANK`, O(log n)) — are both single server-side commands. In SQL, "my rank" is an
`ORDER BY` + window function over the entire table on every request. Update on score change is
`ZADD`, O(log n).

**What breaks:** hot-key on a single global leaderboard (see §4.7); tie-breaking needs encoding into
the score; a truly huge leaderboard should be sharded by segment (country/league) or capped with
`ZREMRANGEBYRANK`.

#### (f) Pub/Sub → fire-and-forget fan-out

**Why Redis:** trivially cheap broadcast to all current subscribers — cache-invalidation fan-out
([037](037_Cache_Consistency_Invalidation.md)), WebSocket fan-out across app servers, live
notifications.

**What breaks — say this before the interviewer does:** Pub/Sub is **at-most-once with no
persistence, no replay, no acknowledgement**. A subscriber that is offline (or restarting, or briefly
disconnected) at publish time **misses the message permanently**. If delivery matters, use Streams or
a real broker. ⚫ Redis 7's *sharded* Pub/Sub limits fan-out to the owning shard in Cluster mode —
one line, no depth needed.

#### (g) Streams → durable-ish log with consumer groups

**Why Redis:** you want a queue with **acknowledgement and redelivery** but not Kafka's operational
weight. `XADD` appends; a consumer group's `XREADGROUP` gives each entry to exactly one consumer;
unacknowledged entries sit in the **pending entries list** and can be `XCLAIM`ed by another consumer
after a timeout — that is the crash-recovery story a List queue lacks.

**What breaks:** retention is bounded by memory (`MAXLEN`/`MINID` trimming), there is no Kafka-style
partition-ordering-plus-massive-throughput model, and durability is Redis's durability (§4.9), not a
replicated commit log. See §10.4.

#### (h) Queues → List (simple) vs Stream (with ack)

```
List  : LPUSH job:queue <payload>  /  BRPOP job:queue 0   (worker blocks until work arrives)
        ✅ dead simple, ✅ fast   ❌ if the worker dies mid-job the message is GONE (already popped)
        Mitigation: BLMOVE into a per-worker processing list, remove on completion, reap stragglers.
Stream: XADD + XREADGROUP + XACK + pending list + XCLAIM
        ✅ ack, redelivery, consumer groups, replay within retention   ❌ more moving parts
Rule of thumb: losing a job is merely annoying → List. Losing a job is a bug → Stream (or a real broker).
```

#### (i) Idempotency → `SET key value NX EX ttl`

**Why Redis:** the canonical "have I already processed request X?" check. The client sends an
idempotency key; the server does `SET idem:{key} <result_or_inflight> NX EX 86400`. If it returns nil,
this is a duplicate — return the stored result instead of re-executing. One atomic command does
check-and-claim with no race. Used for payment retries, webhook redelivery, at-least-once consumers
([056]/[061] preview).

**What breaks:** if Redis loses the key (eviction, failover), a duplicate can slip through — so for
*money*, the idempotency record belongs in the same transactional database as the effect, with Redis
only as a fast pre-filter. Say that.

#### (j) Distributed coordination state → several structures

Feature flags and config (`Hash` + Pub/Sub for change notification), leader-election-ish leases
(`SET NX PX` + renewal — with the correctness caveats from (b)), job dedup (`SET NX`), circuit-breaker
state shared across instances, distributed semaphores (ZSET of holders with timestamps),
"has this user been notified today" gates. The common thread: **small, shared, mutable, hot, and
tolerably ephemeral.**

#### (k) Presence / online status → ZSET or key-with-TTL

**Why Redis:** presence is high-write, high-read, worthless-when-stale — the ideal Redis shape.

- **Key-with-TTL:** each heartbeat does `SET presence:{user} 1 EX 30`; the key existing *is* the
  online status; expiry handles going offline for free. Simplest.
- **ZSET of heartbeats:** `ZADD online <now> {user}` — now you can also answer *"who is online?"*
  (`ZRANGEBYSCORE online (now-30) +inf`) and sweep the stale ones. Use when you need the *list*, not
  just the boolean.

**What breaks:** at very large scale this is a write-per-user-per-heartbeat firehose; batch heartbeats,
widen the interval, and shard by user-id range.

#### (l) Geospatial → GEO commands on a ZSET

**Why Redis:** `GEOADD driver:locations <lon> <lat> driver:42` then
`GEOSEARCH driver:locations FROMLONLAT <lon> <lat> BYRADIUS 3 km ASC COUNT 10` answers "nearest N"
in one server-side command, in memory, at write rates a relational geo-index struggles with.

**What breaks:** it's a flat index — no complex filtering (car type, rating, availability) without
post-filtering in the app; global coverage should be sharded by region/city; a moving-object workload
is write-heavy. Deeper indexing theory (geohash/S2/H3) is [092].

---

### 4.9 Persistence, Atomicity & Concurrency

#### Persistence — exactly as deep as an interview needs

| | **RDB (snapshot)** | **AOF (append-only file)** |
|---|---|---|
| What it does | Point-in-time fork+dump of the whole dataset | Logs every write command as it happens |
| Restart speed | **Fast** (load one compact file) | Slower (replay the log) |
| Data loss on crash | Everything since the last snapshot (**minutes**) | Depends on `appendfsync` (below) |
| Cost | `fork()` + copy-on-write → a **latency spike and a memory spike** on large datasets | Bigger files; continuous disk writes; periodic rewrite (also forks) |
| Good for | Backups, disaster recovery, fast restarts, replica bootstrap | Minimising the loss window |

`appendfsync` — the durability dial:

```
always    → fsync every write batch.  Safest, slowest. (Even this is not ACID: see below.)
everysec  → fsync once per second.    DEFAULT. Lose ≤ ~1 second of writes. Good balance.
no        → let the OS flush (~30s).  Fastest, loosest.
```

**Redis's own recommendation:** run **both** — RDB for backups/fast restart, AOF for the small loss
window. Since Redis 7 the AOF is a multi-part base+incremental file set.

**"What happens if Redis crashes?" — the complete answer:**

```
No persistence          → 100% of data gone. Fine for a pure cache; catastrophic for anything else.
RDB only                → back to the last snapshot; minutes of writes lost.
AOF (everysec)          → up to ~1 second of writes lost.
Any of the above + async replication + failover
                        → a promoted replica may be BEHIND the dead primary: acknowledged writes
                          can still be lost. This is the answer interviewers actually want.
Cold restart at scale   → the cache is EMPTY. Every request misses → the DB takes 100% of load =
                          avalanche (035). Mitigate with cache warming, staged traffic ramp, and a
                          circuit breaker in front of the DB.
```

**The framing that closes the topic:** persistence makes Redis *restartable*, not *durable*. It reduces
data loss; it does not give you ACID durability, and it does not make Redis a system of record.

⚫ *Beyond scope:* AOF rewrite internals, the online `BACKUP` command family, diskless replication tuning.

#### Atomicity and concurrency

**1. Every single command is atomic.** One thread, one command at a time, run to completion. `INCR`,
`SETNX`, `ZADD`, `LPUSH`, `HINCRBY` need no external locking. This is the property that most Redis
patterns are built on.

**2. `MULTI` / `EXEC` — a transaction, but not the one you know.**

```
MULTI          → start queueing commands
<commands>     → queued, NOT executed, no results yet
EXEC           → execute the whole batch atomically: no other client's command interleaves
```

Three differences from a SQL transaction — all of them are trap material:

- **No rollback.** If a command fails *at runtime* (e.g. `INCR` on a string), the other commands in the
  block still apply. Redis treats these as programming bugs that should be caught in development.
  (Syntax errors detected at queue time do abort the whole `EXEC`.)
- **No reads-then-branch.** You cannot read a value inside `MULTI` and decide what to queue next —
  results don't exist until `EXEC`. That's what `WATCH` and Lua are for.
- **Isolation only** — the "A" (rollback) and "D" (durability) a SQL engineer expects are not there.

**3. `WATCH` — optimistic concurrency control (compare-and-swap).**

```
WATCH balance:user1        → "abort if this key changes before my EXEC"
val = GET balance:user1    → read outside the transaction
MULTI
SET balance:user1 (val-10)
EXEC                       → returns nil if balance:user1 was modified by anyone since WATCH
                             → application must RETRY the whole sequence
```

This is [025](025_Isolation_Levels_and_Anomalies.md)/[027](027_MVCC.md)'s optimistic concurrency in
Redis form: no locks held, conflicts detected at commit, **the caller must implement the retry loop.**
Good under low contention, wasteful (retry storms) under high contention.

**4. Lua scripts (and Functions) — the practical answer to read-then-write.**
A script sent with `EVAL` runs **atomically on the server**: no other command interleaves, and unlike
`MULTI` you can read a value and branch on it inside the same atomic unit. That is exactly why the
sliding-window rate limiter, the token bucket, and the compare-and-delete lock release are all Lua
one-liners. It also removes network round trips.

- **Constraints to state:** keep scripts short — a long script blocks the single command thread just
  like any slow command; declare all keys via `KEYS[]`; in Cluster mode all touched keys must be in
  the **same hash slot**; scripts must be deterministic. Redis 7 "Functions" are the persisted,
  named evolution of the same idea. ⚫ Don't go deeper than this.

**5. The race conditions Redis does NOT save you from.**

- **App-side read-modify-write:** `GET` → modify in app → `SET`. Two clients interleave and one update
  is lost. Fix: `INCR`/`HINCRBY`, or `WATCH`, or Lua. This is the most common real Redis bug.
- **Check-then-act across two commands:** `EXISTS` then `SET` is racy; `SET ... NX` is not.
- **Dual writes to DB + Redis:** not atomic; a crash between them leaves them divergent
  ([037](037_Cache_Consistency_Invalidation.md)).
- **Cross-shard "transactions"** in Cluster: not possible without hash tags.
- **Lock loss on failover:** §4.8(b).

**Pipelining ≠ transaction.** Pipelining batches many commands into one network round trip for
throughput; it gives **no** atomicity or isolation. Candidates conflate these constantly.

---

## 5. Visualization — Where Redis Sits

```
                       ┌──────────────────────────────────────────┐
      clients ───────▶ │            Load Balancer / API GW        │
                       └────────────────────┬─────────────────────┘
                                            │
                        ┌───────────────────┼───────────────────┐
                        ▼                   ▼                   ▼
                  ┌───────────┐       ┌───────────┐       ┌───────────┐
                  │ App srv 1 │       │ App srv 2 │       │ App srv 3 │   ← STATELESS
                  │  [L1 map] │       │  [L1 map] │       │  [L1 map] │     (sessions live in Redis)
                  └─────┬─────┘       └─────┬─────┘       └─────┬─────┘
                        └───────────────────┼───────────────────┘
                                            ▼
                    ╔═══════════════════════════════════════════════╗
                    ║                   REDIS                       ║
                    ║  cache · sessions · rate-limit counters       ║
                    ║  locks · leaderboards · presence · queues     ║
                    ║  (hot, shared, mutable, tolerably ephemeral)  ║
                    ╚═══════════════════════┬═══════════════════════╝
                                            │ miss / write-through / async flush
                                            ▼
                    ┌───────────────────────────────────────────────┐
                    │   SOURCE OF TRUTH: Postgres / DynamoDB / S3   │
                    │      (durable, ACID, queryable, slower)       │
                    └───────────────────────────────────────────────┘
```

**Read the picture as one rule:** everything in the Redis band is *reconstructible* from the band
below it. The moment something in Redis is **not** reconstructible, the design has a durability bug.

---

## 6. Animation Frames

### Frames A — cache-aside read, miss then hit

```
Frame 1   App ──GET product:9──▶ Redis                    Redis: (empty)
Frame 2   App ◀──── nil ─────── Redis                     MISS
Frame 3   App ──SELECT id=9───▶ Postgres  ──row──▶ App    ~8 ms
Frame 4   App ──SET product:9 <json> EX 300──▶ Redis      populate
Frame 5   next request: App ──GET product:9──▶ Redis ──json──▶ App    HIT, ~0.4 ms
```

### Frames B — the write that must invalidate

```
Frame 1   App ──UPDATE products SET price=80 WHERE id=9──▶ Postgres    ✅ committed
Frame 2   App ──DEL product:9──▶ Redis                                 ✅ evicted
Frame 3   next read → MISS → repopulates 80 from Postgres              ✅ fresh
          ✗ If Frame 2 is skipped, Frame 3 is a HIT returning the OLD price until TTL. (033/037)
```

### Frames C — asynchronous replication losing an acknowledged write

```
Frame 1   client ──SET k v──▶ PRIMARY                    PRIMARY: k=v
Frame 2   PRIMARY ──"OK"──▶ client        ← acknowledged BEFORE replication (async!)
Frame 3   PRIMARY 💥 dies                  replica never received k=v
Frame 4   Sentinel/Cluster promotes REPLICA (no k)
Frame 5   client reads k ──▶ nil          ← an ACKNOWLEDGED write is GONE.
          This is why Redis is not a system of record, and why a lock held here can be re-acquired.
```

### Frames D — a hot key that sharding cannot fix

```
Frame 1   200k rps on "product:viral"      slot = CRC16(...) mod 16384  → always the SAME slot
Frame 2   that slot lives on Shard B       Shard B thread: 100% ─── Shards A,C,D,E,F: ~idle
Frame 3   "add more shards!"               new shards get OTHER slots. Shard B still 100%. ❌
Frame 4   fix: app-local L1 cache absorbs most reads          Shard B: ~5%   ✅
          or:  product:viral#0..#9, read a random copy, write fans out to all 10 ✅
```

### Frames E — acquiring and safely releasing a lock

```
Frame 1   A: SET lock:x uuidA NX PX 30000  → OK        A holds it
Frame 2   B: SET lock:x uuidB NX PX 30000  → nil       B backs off (jittered retry)
Frame 3   A pauses (GC) past 30 s → key expires        NOBODY holds it
Frame 4   B: SET lock:x uuidB NX PX 30000  → OK        B holds it
Frame 5   A wakes, tries to release:
            plain DEL          → would delete B'S lock  ❌ SAFETY VIOLATION
            Lua compare-and-del(uuidA) → value is uuidB → no-op ✅
          Still unsafe for CORRECTNESS: A may believe it holds the lock and write.
          → fencing token, or an idempotent operation, or etcd/ZooKeeper.
```

---

## 7. Architecture Diagrams — the three deployment topologies

```
① STANDALONE (dev, or a truly disposable cache)
   [App] ──▶ [Redis]                    ✗ SPOF   ✗ no read scaling   ✗ RAM-bound
   Say this only if you also say "and I'd accept losing the cache entirely."

② PRIMARY + REPLICAS + SENTINEL  ← the default answer for most systems
   [App] ──writes──▶ [PRIMARY] ══async══▶ [Replica1] [Replica2]
      └───reads(optional, STALE)──────────────┘
              ▲
        [Sentinel ×3]  monitor → quorum → promote → tell clients
   ✅ HA, ✅ read scaling   ❌ one primary's RAM & write throughput is still the ceiling

③ REDIS CLUSTER  ← when RAM or write throughput exceeds one node
   16,384 hash slots split across shards; cluster-aware client routes by slot
   ┌── Shard A ──┐  ┌── Shard B ──┐  ┌── Shard C ──┐
   │ P + replica │  │ P + replica │  │ P + replica │   each shard fails over independently
   └─────────────┘  └─────────────┘  └─────────────┘
   ✅ scales memory AND writes   ❌ multi-key ops need hash tags   ❌ more ops complexity
   (Managed equivalents: AWS ElastiCache / MemoryDB, GCP Memorystore, Azure Cache for Redis)
```

**Interview move:** propose ② by default and name the *specific number* that would push you to ③
("once the working set passes ~50 GB, or writes pass what one primary core can take"). Jumping
straight to Cluster for a 2k-QPS app is the same buzzword-matching error as reaching for Cassandra —
the same reasoning as [031](031_Choosing_a_Database.md)'s golden default.

---

## 8. Real Engineering Examples

- **Redis Cluster's 16,384 hash slots** — a deliberately *fixed* indirection layer between keys and
  nodes so rebalancing moves slots, not keys. Compare with Cassandra/DynamoDB's consistent hashing
  ([042] preview): same problem, different implementation.
- **`SET key value NX PX ttl`** — one command that is simultaneously the standard distributed-lock
  primitive, the idempotency-key primitive, and the cache-stampede mutex. Recognising that all three
  patterns are the *same* primitive is a strong signal.
- **Redis Sorted Sets as a time index** — score = Unix timestamp turns one structure into a sliding
  window (`ZREMRANGEBYSCORE`), a delayed-job queue (`ZRANGEBYSCORE now`), a presence tracker, and a
  time-ordered feed. Same structure, four systems.
- **Lua as the atomicity escape hatch** — every "read a value, branch, then write" pattern that
  `MULTI/EXEC` cannot express becomes a short server-side script.
- **`UNLINK` vs `DEL`** — a production detail with a real blast radius: `DEL` on a multi-million-member
  collection blocks the single command thread; `UNLINK` frees it in the background.

**Ecosystem note worth one sentence in 2026 (do not over-invest):** Redis moved from BSD to SSPL/RSAL
in **March 2024**, prompting AWS/Google/Oracle to fork **Valkey** (now Linux Foundation-governed,
BSD-licensed, a drop-in replacement). Redis **8.0 (May 2025)** added **AGPLv3** as an option,
returning to OSI-approved open source. Practical takeaway for an interview: *"Redis and Valkey are
API-compatible; managed clouds increasingly default to Valkey. It doesn't change the design
reasoning — it changes the vendor/licensing conversation."* One sentence, then move on.

---

## 9. Industry Examples

| Company / system | Redis role | Structure |
|---|---|---|
| **Twitter/X** | Home-timeline cache (fan-out-on-write pushes tweet IDs into per-user timelines) | List / ZSET |
| **GitHub, Shopify, Stripe-style APIs** | API rate limiting per key/user/IP | String counter or ZSET |
| **Uber / Lyft / Swiggy / Zomato** | Live driver/rider location and proximity search; ride-state hot path | GEO (ZSET) |
| **Gaming (any leaderboard product)** | Global and segment leaderboards with live rank | ZSET |
| **Slack / Discord-style chat** | Presence, typing indicators, per-channel WebSocket fan-out | TTL keys, Pub/Sub |
| **E-commerce (Amazon-style carts)** | Session + cart state, inventory-reservation counters | Hash, String |
| **Nearly every web backend** | Session store enabling stateless app servers | Hash + TTL |

> Cross-reference [TechChoices.md](../TechChoices.md) — the "real users" column there and this table
> are the same evidence, used for different purposes (this one for narrative, that one for defence).

---

## 10. Tradeoffs — Redis vs the Alternatives

> The two questions to answer for every comparison: **"when would I choose Redis instead?"** and
> **"when should I NOT choose Redis?"**

### 10.1 Redis vs Memcached

Already established in [036](036_Distributed_Caching_Redis_Memcached.md). Condensed:

| | Memcached | Redis |
|---|---|---|
| Data model | Strings only | 10+ data structures with atomic server-side ops |
| Threading | Multi-threaded (higher raw throughput/instance on big boxes) | Single-threaded command execution (+ I/O threads since 6.0) |
| Persistence | None | RDB / AOF (restartability, not durability) |
| HA & scaling | Client-side sharding only | Replication, Sentinel, Cluster built in |
| Extras | — | Pub/Sub, Lua, Streams, geo, probabilistic types |

**Choose Redis when** you need anything beyond `get`/`put` — which, in a system-design interview, you
almost always do. **Choose Memcached when** the need is genuinely a simple, large, multi-threaded
string cache and you value the smaller operational surface.

**Interview sentence:** *"Memcached would work for the page-fragment cache, but the leaderboard needs
`ZREVRANK` server-side — with Memcached I'd be sorting in the app layer and reopening the exact race
a cache is supposed to remove."*

### 10.2 Redis vs a database (Postgres / DynamoDB)

| Redis wins | The database wins |
|---|---|
| Sub-ms latency on a hot working set | Durability, ACID, rollback |
| Atomic counters and rank queries | Ad-hoc queries, joins, aggregations, secondary indexes |
| Ephemeral/shared coordination state | Data that must survive a crash, an audit, or a lawyer |
| Absorbing write bursts before they hit the DB | Unbounded data size at sane cost per GB |

**"Why not just use Redis as the database?"** — the trap question. Answer: *"Cost (RAM is ~50–100×
disk per GB), durability (async replication + `everysec` fsync means acknowledged writes can be lost),
and queryability (no joins, no ad-hoc queries, no secondary indexes without extra modules). Redis is a
front-end for hot state, not a system of record."*

**Note the legitimate exception, briefly:** some teams *do* run Redis as a primary store for genuinely
ephemeral data (sessions, presence, live game state) where loss is acceptable — say that you know the
exception rather than pretending it never happens.

### 10.3 Redis vs RabbitMQ (task queue)

| | Redis (List/Stream) | RabbitMQ |
|---|---|---|
| Model | Data structure you use as a queue | Purpose-built broker: exchanges, bindings, queues |
| Routing | You build it | Rich routing (topic/fanout/direct/headers) |
| Delivery | List = at-most-once after pop; Stream = ack + redelivery via pending list | Per-message ack, redelivery, **dead-letter queues**, TTL, priorities, publisher confirms |
| Ops | Already running Redis; nothing new | Another system to run and monitor |

**Choose Redis when** the workload is simple background jobs, you already run Redis, and the queue is
one of several Redis uses in the design. **Choose RabbitMQ when** you need real routing topologies,
per-message reliability guarantees, DLQs, or the queue is central enough to deserve its own system.

**Interview sentence:** *"A Redis List queue is enough for thumbnail generation. For payment-webhook
processing I'd want a real broker — I need dead-letter queues and per-message acks, and building those
on a List means reimplementing a broker badly."*

### 10.4 Redis vs Kafka (streaming)

**This is the most-asked of the four.** They look similar (both have append-only logs and consumer
groups) and are architecturally very different.

| | Redis Streams | Kafka |
|---|---|---|
| Storage | **In memory**, trimmed by `MAXLEN`/`MINID` | **On disk**, retention in days/weeks/forever, replicated commit log |
| Scale | Great for thousands–low millions of msgs/day; bounded by RAM | Built for very high throughput and TB-scale retention |
| Parallelism | Consumer group hands each entry to one consumer (job-queue semantics) | Partitions are the unit of parallelism; same key → same partition → ordering per key |
| Replay | Within retention (memory-bounded) | Full history replay — this is Kafka's defining feature |
| Ordering | Per stream | Per partition, with key-based routing |
| Ops cost | ~zero if you already run Redis | Brokers, partition strategy, replication factor, rebalancing, its own monitoring stack |
| Latency | Sub-ms | Low ms |

**Choose Redis Streams when:** you already run Redis, volume is modest, retention is short, and you
want ack + redelivery without standing up a broker.

**Choose Kafka when:** you need durable replayable history, event sourcing / CDC, many independent
consumer groups reading the same stream, multi-region durability, or throughput beyond one machine's
memory.

**Interview sentence:** *"Redis Streams for the in-app notification fan-out — short retention, we
already run Redis. Kafka for the analytics event pipeline, because those events feed four different
consumers, need replay when a consumer's logic changes, and must survive a week of retention that
would never fit in RAM."*

**Trap:** "Redis Pub/Sub is like Kafka." It is not — Pub/Sub has no persistence, no ack, no replay,
and no consumer groups. Redis *Streams* is the Kafka-shaped thing; Pub/Sub is fire-and-forget.

### 10.5 One-screen decision table

```
Need ...                                        → Pick
────────────────────────────────────────────────────────────────────
Hot-read cache with rich ops                    → Redis
Dead-simple string cache, max raw throughput    → Memcached
Durable record of truth, ACID, joins            → Postgres / DynamoDB
Durable replayable event log, huge throughput   → Kafka
Complex routing + per-message reliability + DLQ → RabbitMQ / SQS
Simple background jobs, Redis already present   → Redis List / Stream
Correctness-critical distributed lock           → etcd / ZooKeeper (or fencing tokens + idempotency)
Full-text / faceted search                      → Elasticsearch
```

---

## 11. Complexity Analysis

```
GET / SET / INCR / SETNX / EXPIRE          O(1)
HGET / HSET / HINCRBY                      O(1)          HGETALL  O(n) in fields  ← big-hash hazard
SADD / SREM / SISMEMBER                    O(1)          SMEMBERS O(n)            ← big-set hazard
LPUSH / RPUSH / LPOP / RPOP / BRPOP        O(1)          LRANGE   O(start+count)
ZADD / ZSCORE / ZRANK / ZINCRBY            O(log n)
ZRANGE / ZREVRANGE / ZRANGEBYSCORE         O(log n + m)  (m = elements returned)
XADD                                       O(1)          XREADGROUP ~O(entries returned)
PFADD / PFCOUNT                            O(1)          ≤ 12 KB/key, 0.81% std error
GEOSEARCH                                  O(n + log m)-ish over the searched area
DEL (big collection)                       O(n) — BLOCKS the thread. Use UNLINK.
KEYS pattern                               O(n) over the ENTIRE keyspace — NEVER in production. Use SCAN.
FLUSHALL / SORT (unbounded)                O(n) — same hazard
```

**The rule that turns this table into an interview answer:** on a single-threaded server, *any* O(n)
command is a latency incident for **every** client, not just the caller. "Is this command O(n), and
how big is n?" is the entire production-safety heuristic.

---

## 12. Scaling Considerations

| Scale | What breaks first | What you do |
|---|---|---|
| **1×** | Nothing | Standalone or primary+replica; cache-aside; TTLs with jitter |
| **10×** | Read throughput; a single point of failure becomes intolerable | Replicas for reads (accept staleness), Sentinel for HA, connection **pooling**, **pipelining** to cut round trips, `MGET` instead of N `GET`s |
| **100×** | One node's RAM and one core's write throughput | **Cluster** (shard); shrink values (compress, store IDs not blobs); shorter TTLs; add an in-process **L1** tier so Redis isn't hit for the hottest keys |
| **1000×** | Hot keys, hot shards, big keys, cross-shard ops, fork pauses on huge datasets | Key splitting for hot keys; re-key to spread hot shards; keep nodes small (~25–50 GB) so fork/failover/resync stay fast; hash tags for co-location; push public hot reads to the CDN; consider client-side caching (RESP3 invalidation) |

**Three scaling truths to state plainly:**

1. **Replicas scale reads, never writes.** Writes scale only by sharding.
2. **Sharding does nothing for a hot key** — one key is one slot is one shard is one thread.
3. **The network round trip often dominates.** 100 sequential `GET`s = 100 RTTs ≈ 50 ms; one `MGET` or
   one pipeline ≈ 0.5 ms. "N+1 queries" is a Redis problem too.

---

## 13. Failure Scenarios

| Failure | What actually happens | Blast radius | Mitigation |
|---|---|---|---|
| **Redis process dies (no HA)** | 100% of cached reads fall through to the DB instantly | Whole system — **avalanche** ([035](035_Cache_Problems.md)) | Replicas + Sentinel/Cluster; circuit breaker [070]; in-process L1; degrade gracefully |
| **Failover after async replication** | Promoted replica is behind → **acknowledged writes lost**; a held lock can be re-acquired by another client | Silent data loss / double execution | `WAIT`, `min-replicas-to-write`; keep correctness-critical state in a durable store; fencing tokens |
| **Memory hits `maxmemory`** | Default `noeviction` → **writes start erroring**; with `allkeys-lru` → silent eviction of hot data | Write failures or a hit-rate collapse | Set the policy deliberately; alert on `evicted_keys` and memory %; size headroom for fork/COW |
| **Hot key** | One shard's thread saturates while others idle; p99 for *every* key on that shard degrades | That shard | L1 cache; key splitting; replicas; CDN |
| **Big key** | O(n) access/delete blocks all clients; memory skew across shards | Whole instance | Split the collection; `UNLINK`; alert on key size |
| **Slow command (`KEYS *`, big `SORT`, huge Lua)** | Every other client waits behind it | Whole instance | Ban `KEYS` in prod (rename/ACL it away); use `SCAN`; `slowlog` alerts |
| **`fork()` for RDB/AOF-rewrite on a large dataset** | Latency spike; memory spike from copy-on-write | Whole instance | Smaller nodes; snapshot on a replica instead of the primary; stagger snapshots across shards |
| **Cold restart / cache warm-up** | Empty cache → 100% miss → DB overload | Whole system | Warm the cache before taking traffic; ramp traffic; circuit breaker |
| **Network partition / client timeouts** | Clients block or pile up; connection-pool exhaustion cascades into app-thread starvation | App tier | Aggressive timeouts, bounded pools, retries **with jitter** [069], bulkheads [070] |
| **Split brain (old primary returns)** | Two primaries briefly accept writes; one side's writes are discarded | Data divergence | Sentinel quorum sizing; `min-replicas-to-write`; fencing at the application layer |
| **Cross-shard multi-key op** | Error, or silently non-atomic | Correctness bug | Hash tags to co-locate related keys |

**The two-sentence answer to "what if Redis goes down?"** — *"Reads fail over to the database, which
means I need a circuit breaker so the DB isn't taken down with it, plus an in-process L1 so caching
degrades rather than disappears. For the non-cache uses I'd state the policy explicitly: the rate
limiter fails open, sessions fail to a re-login, and anything correctness-critical was never
Redis-only to begin with."*

---

## 14. Monitoring

```
HEALTH & SATURATION
  used_memory / maxmemory  (% full)          ← the #1 predictor of an incident
  evicted_keys per sec                       ← non-zero on a "durable" instance = data loss
  expired_keys per sec
  connected_clients, blocked_clients, rejected_connections
  instantaneous_ops_per_sec, CPU of the main thread

EFFECTIVENESS
  keyspace_hits / (hits + misses) = HIT RATIO ← a sudden drop = invalidation storm, restart, or eviction
  latency p50 / p99 / p999  (Redis-side and client-side — measure BOTH)
  SLOWLOG entries            ← O(n) commands caught in the act

REPLICATION & CLUSTER
  master_link_status, master_repl_offset − replica offset = REPLICATION LAG
  connected_slaves, sync_full (full resyncs are expensive — spikes are a smell)
  cluster_state, slot coverage, per-shard key count & ops (detects HOT SHARDS)

PERSISTENCE
  rdb_last_bgsave_status, aof_last_write_status, aof_rewrite_in_progress
  latest_fork_usec           ← fork pauses show up here

KEY-LEVEL
  hot keys and big keys (redis-cli --hotkeys / --bigkeys, or MEMORY USAGE sampling)
```

## 15. Observability

- **Dashboards:** memory %, hit ratio, ops/sec, p99 latency, evictions, replication lag, per-shard key
  distribution (the hot-shard detector), slowlog rate.
- **Alerts that are actually actionable:** memory > 80% of `maxmemory`; hit ratio drops > 20% in 10
  min; replication lag > N seconds; `evicted_keys` > 0 where eviction should never happen;
  `master_link_status: down`; slowlog entries above threshold; failover event.
- **Tracing:** include the Redis call as a span ([078] preview) so a slow page can be attributed to a
  Redis stall rather than guessed at. Tag spans with the command name and key *prefix* — **never the
  full key**, which may contain user identifiers.
- **Runbook entries worth pre-writing:** "hit ratio collapsed," "memory near max," "failover
  occurred," "hot key detected." Each maps to a mitigation already named in §13.

## 16. Security

- **Redis is not internet-facing. Ever.** Historically it shipped with no auth and bound to all
  interfaces, which produced a long tail of publicly-exposed, ransomed instances. Put it in a private
  subnet / VPC with security groups, and bind to internal interfaces only.
- **AUTH + ACLs (Redis 6+):** per-user credentials with command and key-pattern restrictions — e.g. the
  app user can touch `cache:*` but cannot run `FLUSHALL`, `CONFIG`, `KEYS`, or `DEBUG`. Renaming or
  ACL-disabling dangerous commands in production is standard practice.
- **TLS in transit** (Redis 6+) and encryption at rest for RDB/AOF files and backups — the persistence
  files contain your data in near-plaintext.
- **Data-protection reasoning that earns points:** sessions and cached PII in Redis are *personal
  data*. Set TTLs so deletion is automatic, keep PII out of key *names* (key names show up in logs and
  `SLOWLOG`), and remember an RDB snapshot shipped to S3 is a copy of your users' data
  ([088] preview).
- **Abuse/DoS angle:** a Lua script or `KEYS *` from a compromised client can stall the whole instance
  — ACLs are an *availability* control here, not just a confidentiality one.

---

## 17. Interview Discussion — Where Redis Fits in Real Designs

> Seven representative systems. For each: **where Redis sits, why, and the follow-up you will get.**

### 17.1 Rate Limiter ([071] preview)

```
Client ─▶ API Gateway ─▶ [REDIS: counter per {user|API-key|IP} per window] ─▶ Service
                              allow → forward     deny → HTTP 429 + Retry-After
```

**Why Redis:** the limit must be enforced **globally across all gateway instances** — a local counter
means N gateways each allow the full quota. Atomic `INCR`, sub-ms, TTL for free.

**Structure:** String + `INCR` (fixed window), ZSET (sliding window log), Hash + Lua (token bucket).

**Follow-ups you will get:** *"Which algorithm and why?"* (boundary burst vs memory cost) ·
*"What if Redis is down — fail open or closed?"* (state and justify) · *"Is your check-and-increment
atomic?"* (one `INCR`, or a Lua script — not GET-then-SET) · *"How do you handle a distributed limit
where the counter itself is a hot key?"* (shard the limit per-node with a fraction of the quota, or
approximate).

### 17.2 URL Shortener ([098] preview — the repo's Case Study #1)

```
POST /shorten → generate ID → write (short→long) to DB → optionally SET url:{short} in Redis
GET /{short}  → REDIS GET url:{short} ── hit ──▶ 302 redirect   (the 99% path)
                     └── miss ──▶ DB ──▶ populate ──▶ 302
```

**Why Redis:** read:write is on the order of 100:1 or worse, the mapping is immutable (so **staleness
is a non-issue** — the ideal cache), and redirect latency is the product.

**Also:** `INCR` on a Redis counter is one of the standard ID-generation strategies; click analytics
can be `INCR`'d in Redis and flushed to the DB in batches (write-behind).

**Follow-ups:** *"What's your cache hit rate and why?"* (Zipfian access — a small hot set serves most
traffic) · *"Cache eviction policy?"* (`allkeys-lru`) · *"What if a short code doesn't exist?"*
(negative caching + Bloom filter — **cache penetration**, [035](035_Cache_Problems.md)) · *"Does the
counter survive a Redis restart?"* (this is where ID-generation-in-Redis gets uncomfortable — say
Snowflake or pre-allocated ID ranges instead if IDs must never repeat).

### 17.3 Chat / Presence ([015](015_WebSockets_SSE_Polling_Long_Polling.md), preview [082])

```
User ══WebSocket══ App-server-3
   presence : SET presence:{user} 1 EX 30  (heartbeat)  OR  ZADD online <ts> {user}
   fan-out  : App-3 PUBLISH channel:{room} <msg>  →  App-1, App-2 push to their own sockets
   recent   : LPUSH/LTRIM or ZSET (score=ts) for the last N messages; full history in the DB
   typing   : SET typing:{room}:{user} 1 EX 5     (expiry IS the feature)
```

**Why Redis:** presence and typing indicators are high-write, high-read, and worthless when stale —
exactly what an in-memory TTL store is for. Pub/Sub solves the cross-server WebSocket fan-out problem
(the recipient's socket is on a different app server than the sender's).

**Follow-ups:** *"What if a subscriber is momentarily disconnected?"* (**Pub/Sub loses it** — use
Streams or store-then-notify) · *"How do you know someone went offline?"* (TTL expiry, not an explicit
event — clients disappear without saying goodbye) · *"Where is message history?"* (durable DB;
Redis holds only the hot tail) · *"Does this scale to 10M concurrent?"* (shard by room; sharded
Pub/Sub; heartbeat batching).

### 17.4 Uber / Ride Matching ([092] preview)

```
Driver app ──location every 4s──▶ [REDIS GEO: driver:locations]     (write firehose)
Rider requests ──▶ GEOSEARCH FROMLONLAT ... BYRADIUS 3 km ASC COUNT 20
                 ──▶ post-filter in app (car type, rating, currently free)
                 ──▶ matching service ──▶ trip record ──▶ POSTGRES (source of truth)
Also in Redis: ride-state hot path, surge multiplier per zone, driver-availability locks.
```

**Why Redis:** a location write firehose plus a "nearest N" query at sub-ms — a relational geo-index
would collapse on the write rate, and the *current* location is disposable (only the trip is durable).

**Follow-ups:** *"Why not Postgres/PostGIS?"* (write rate + latency) · *"How do you shard?"* (by
city/region — a hash tag per city keeps a city's drivers co-located) · *"How do you prevent two riders
matching the same driver?"* (a short-lived `SET NX PX` lock or an atomic Lua claim — and then the
**lock correctness** discussion from §4.8(b)) · *"What if Redis loses all locations?"* (drivers
re-report within seconds — say this; it's why the choice is safe).

### 17.5 E-commerce ([031](031_Choosing_a_Database.md)'s polyglot persistence, made concrete)

```
Product catalog     → Redis cache-aside over Postgres           (read-heavy, staleness OK)
Session + cart      → Redis Hash + TTL                          (shared, mutable, semi-ephemeral)
Flash-sale inventory→ Redis DECR as a fast reservation gate, reconciled against Postgres
Orders / payments   → POSTGRES ONLY. Never Redis.               (ACID, durable, auditable)
Search / facets     → Elasticsearch
```

**The follow-up that separates candidates:** *"Can you decrement inventory in Redis?"* — the honest
answer is: `DECR` is atomic so it's a good **admission-control gate** for a flash sale, but the
authoritative decrement must be a transaction in Postgres, because a Redis crash between the two
oversells. Say the two-tier design out loud: **Redis gates, Postgres commits.** Claiming Redis alone
is safe for inventory is a durability error; refusing to use Redis at all wastes the one mechanism
that survives a 100k-rps flash sale.

### 17.6 Notification System ([059]/[060] preview)

```
Event ──▶ [Redis Stream OR Kafka] ──▶ worker pool ──▶ push / email / SMS providers
Redis also holds:  dedup/idempotency  SET notif:{event_id} 1 NX EX 86400
                   per-user rate cap  INCR notif:rate:{user}:{day}
                   user preferences   Hash (cached, source of truth in the DB)
                   scheduled sends    ZSET with score = send-at timestamp (poll ZRANGEBYSCORE)
```

**Why Redis:** the delayed/scheduled-send ZSET and the "have I already notified about this?" set are
both perfect Redis shapes.

**Follow-ups:** *"Redis Streams or Kafka?"* (§10.4 — retention, replay, consumer count) · *"How do you
guarantee a user isn't notified twice?"* (idempotency key + `SET NX`, and the caveat that Redis-only
dedup can slip on failover) · *"How do you schedule 'send in 3 days'?"* (ZSET by timestamp, or a real
scheduler — don't pretend Redis is a durable scheduler for critical sends).

### 17.7 Leaderboard

```
Score change → ZADD leaderboard:global <score> {user}          O(log n)
Top 10       → ZREVRANGE leaderboard:global 0 9 WITHSCORES     O(log n + 10)
My rank      → ZREVRANK leaderboard:global {user}              O(log n)
Around me    → ZREVRANGE (rank-5) (rank+5)
```

**Why Redis:** in SQL, "what's my rank among 50M rows" is a full ordering per request. In Redis both
queries are single O(log n) commands.

**Follow-ups:** *"Ties?"* (encode a timestamp into the score's fractional part) · *"50M users?"*
(ZSET memory is on the order of tens of bytes per entry — fine; segment by league/country to reduce
hot-key pressure) · *"Is the score durable?"* (source of truth in the DB, ZSET rebuildable) ·
*"Time-windowed leaderboards?"* (one ZSET per period + `ZUNIONSTORE`).

### 17.8 What the interviewer is actually scoring

Against the 7-dimension rubric ([SYSTEM_DESIGN_MASTER_GUIDE.md](../SYSTEM_DESIGN_MASTER_GUIDE.md) §2.1):
**Architecture** (does Redis sit in the right place, or is it decoration?) · **Data Model** (did you
name the *structure* and why?) · **Deep Dive** (atomicity, hot keys, eviction) · **Tradeoffs** (why
Redis and not Memcached/Kafka/Postgres) · **Operations** (what happens when it dies). Redis touches
five of the seven — which is why one weak Redis answer is disproportionately expensive.

---

## 18. Common Mistakes

| Mistake | Correction |
|---|---|
| "Redis is fast enough to be used as a database" | **Inverted.** Fast enough to serve *as a cache and a coordination layer*; not durable enough to *be* the database. (Logged 2026-08-04 in [InterviewMistakes.md](../InterviewMistakes.md); same family as [032](032_Caching_Fundamentals.md)'s durable-store trap) |
| "I'll add Redis" with no structure named | The structure *is* the answer. "A sorted set keyed by score, so top-K and my-rank are both single server-side commands." |
| "Redis is single-threaded" (full stop) | Command *execution* is single-threaded (that's the atomicity guarantee); **network I/O can be threaded since 6.0**. Say both halves. |
| Treating replication, Sentinel, and Cluster as the same thing | Replication copies · Sentinel promotes (HA, no sharding) · Cluster shards **and** promotes. Sentinel and Cluster are alternatives, not layers. |
| "Add more Redis nodes to fix the hot key" | One key = one slot = one shard = one thread. Sharding cannot split a single key. Fix with an L1 cache, key splitting, replicas, or the CDN. |
| Assuming Redis evicts when full | Only if configured. **Default is `noeviction` → writes fail.** Name `allkeys-lru`. |
| Assuming a TTL refreshes on read | It doesn't by default (sliding TTL is opt-in via `GETEX`). Already logged against [034](034_Eviction_Policies.md). |
| Treating `MULTI/EXEC` as a SQL transaction | **No rollback** on runtime errors, and you cannot read-then-branch inside it. Use `WATCH` (optimistic CAS, caller retries) or Lua (atomic read-branch-write). |
| Confusing pipelining with transactions | Pipelining = fewer round trips. It provides **no** atomicity or isolation. |
| Doing `GET` → modify → `SET` from the app | A distributed read-modify-write race. Use `INCR`/`HINCRBY`, `WATCH`, or Lua. |
| Releasing a lock with a plain `DEL` | You may delete **someone else's** lock after yours expired. Unique value + Lua compare-and-delete. And a TTL-less lock deadlocks forever. |
| Claiming a Redis lock guarantees correctness | Async replication + failover can hand the same lock to two clients. Efficiency locks: fine. Correctness locks: fencing tokens, etcd/ZooKeeper, or idempotency. |
| Calling Redis Pub/Sub "like Kafka" | No persistence, no ack, no replay, no consumer groups. Offline subscribers lose messages permanently. **Streams** is the Kafka-shaped one. |
| Running `KEYS *` (or unbounded `SORT`, or `DEL` on a huge collection) in prod | O(n) on a single thread blocks *every* client. Use `SCAN`, bounded queries, `UNLINK`. |
| Forgetting cross-shard multi-key ops break in Cluster | `MULTI`/Lua/`MGET` need all keys in one slot — use hash tags `{user:123}:*`. |
| Reading counters or locks from a replica | Replicas are asynchronously stale ([040](040_Replication_Lag_Read_Your_Writes.md)). Correctness-critical reads go to the primary. |
| Never stating what happens when Redis dies | This is a guaranteed follow-up. Have the answer ready: fail-open/fail-closed per use, circuit breaker, L1 fallback, HA topology. |
| Sequential `GET`s in a loop | N round trips. Use `MGET`/pipelining — "N+1 queries" applies to Redis too. |

---

## 19. Advanced Topics (recognise, don't study)

One line each — enough to name if the interviewer raises it, not worth time at this level.

- **Client-side caching / tracking (RESP3):** Redis notifies clients when a cached key changes,
  making a safe in-process L1 tier possible. The principled fix for hot keys.
- **Fencing tokens:** monotonic counter issued with a lock; the protected resource rejects lower
  tokens. The real fix for correctness locks (§4.8b).
- **Redis Functions (7.0+):** persisted, named, versioned server-side libraries — Lua's grown-up form.
- **Sharded Pub/Sub (7.0+):** confines fan-out to the owning shard in Cluster mode.
- **Modules / Redis Stack:** RediSearch (secondary indexes + full-text), RedisJSON, RedisBloom,
  TimeSeries, Vector Sets (HNSW similarity search for RAG). They blur the "Redis can't query" line —
  mention only if directly relevant.
- **Alternatives in the same niche:** Valkey (BSD fork), DragonflyDB and KeyDB (multi-threaded
  Redis-compatible servers), AWS MemoryDB (Redis API with a **durable** multi-AZ transaction log — the
  interesting one, because it removes the durability objection at a cost).
- **Active-active geo-replication (CRDT-based, Redis Enterprise):** links to [057] CRDTs.
- ⚫ Not worth your time now: RESP protocol internals, `listpack`/`quicklist` encodings, cluster gossip,
  jemalloc/defrag tuning, `OBJECT ENCODING`-level trivia.

---

## 20. Interview Questions

> **Provenance is labelled honestly.** ✅ **REPORTED** = this question (or a near-verbatim variant)
> appears in publicly documented interview-experience write-ups and established interview-prep
> collections for this kind of round. ⚙️ **PRACTICE** = generated for this curriculum to test
> understanding. Neither category is a guarantee of what any specific interviewer will ask; treat ✅ as
> *"documented as commonly asked"*, not as a leaked question list.

### ✅ Reported / commonly documented

1. **"Why Redis and not Memcached here?"** — the single most frequent Redis question in HLD rounds.
   *Wants:* a specific data-structure/persistence/clustering need, not familiarity.
2. **"Design a distributed rate limiter."** — Redis-centred by default in most published breakdowns.
   *Wants:* algorithm choice with its flaw named, atomicity of check-and-increment, behaviour when
   Redis is down.
3. **"How would you implement a distributed lock with Redis?"** — *Wants:* `SET NX PX`, unique value,
   Lua release, TTL; then the failover/Redlock/fencing caveat. Interviewers explicitly probe worker
   crashes, early expiry, and Redis restarts.
4. **"Design a leaderboard / top-K ranking."** — *Wants:* sorted set, both `ZREVRANGE` and `ZREVRANK`,
   and why SQL `ORDER BY` doesn't scale here.
5. **"What happens if the cache/Redis goes down?"** — *Wants:* avalanche, circuit breaker, fail-open
   vs fail-closed, HA topology.
6. **"Is Redis single-threaded? Why does that matter?"** — *Wants:* free atomicity **and** the
   blocking hazard; bonus for I/O threading since 6.0.
7. **"How does Redis Cluster decide which node holds a key?"** — *Wants:* 16,384 hash slots,
   `CRC16 mod 16384`, and the multi-key/hash-tag consequence.
8. **"Redis vs Kafka — when would you use each?"** — *Wants:* retention/replay/throughput/ops, and
   the Pub/Sub-is-not-Kafka distinction.
9. **"How do you keep the cache consistent with the database?"** — *Wants:* invalidate-don't-update,
   ordering, TTL as a backstop, and honesty that it's eventually consistent.
10. **"What are RDB and AOF, and what do you lose in a crash?"** — *Wants:* the loss window per mode
    *plus* the async-replication failover caveat.

### ⚙️ Practice (generated for this curriculum)

11. Two servers each run a rate limiter with a local in-memory counter. What's wrong, and what
    exactly does Redis fix?
12. You store a user profile as a JSON string and update `last_login` on every request. Name two
    distinct problems and the Redis structure that fixes both.
13. A `ZADD`-based sliding-window limiter uses more memory than an `INCR`-based fixed window. Explain
    roughly why, and say when the extra memory is worth it.
14. Redis is at 100% CPU. Ops/sec is *lower* than yesterday. `evicted_keys` is 0. What do you check
    first, and what are the three most likely causes?
15. Your cache hit ratio dropped from 94% to 61% in ten minutes with no deploy. Give three candidate
    causes and how you'd distinguish them.
16. A payment service uses a Redis lock to prevent double-charging. Explain the exact sequence in
    which a customer gets charged twice, and give two fixes that don't depend on the lock being
    perfect.
17. You must show "number of unique viewers" for 50M videos. Compare a Set against a HyperLogLog on
    memory and accuracy, and state which you'd ship.
18. In Redis Cluster, `MULTI` with `user:1:profile` and `user:1:cart` fails. Why, and what is the
    one-character-class fix?
19. Design the Redis layer of a flash sale: 500k users, 1000 units. Which structures, and where
    exactly does Postgres still have to be involved?
20. Your team proposes Redis as the primary store for user-uploaded documents. Give the three
    strongest objections, in the order you'd raise them.

### Follow-up chains to rehearse (the pressure is always in the follow-up)

```
"I'd cache it in Redis."
  → "Which data structure?"                     → "What TTL, and why that number?"
  → "What if two requests miss at the same time?" (stampede)
  → "What if the key never exists?"              (penetration)
  → "What if that key gets 200k rps?"            (hot key — sharding won't help)
  → "What if Redis dies?"                        (avalanche, circuit breaker, fail-open/closed)
  → "How does the cache learn about a DB write?" (invalidation, ordering, dual-write race)
  → "Is your update atomic?"                     (INCR / WATCH / Lua — not GET-then-SET)
  → "Would Memcached do?"                        (name the structure you'd lose)
```

### Interview traps (the questions designed to catch you)

1. *"Redis is durable now, right? It has AOF."* → Restartable, not durable. Even `appendfsync always`
   plus async replication can lose acknowledged writes on failover.
2. *"So you'd just add more Redis nodes for that hot key?"* → No: one key, one slot, one thread.
3. *"MULTI/EXEC is a transaction, so it rolls back on error?"* → It does not roll back runtime errors.
4. *"Pub/Sub gives you a reliable queue, right?"* → At-most-once, no persistence, no replay.
5. *"Your lock guarantees only one worker runs?"* → Not across failover; efficiency vs correctness.
6. *"You said Redis evicts old keys when memory fills."* → Only if `maxmemory-policy` isn't the
   default `noeviction`.
7. *"Just read the counter from a replica to scale."* → Asynchronously stale; correctness reads go to
   the primary.
8. *"Redis Streams is basically Kafka."* → Memory-bounded retention, different parallelism model,
   no long-horizon replay.

### Trade-off questions (no single right answer — the reasoning is the score)

- Fixed-window (cheap, burst-prone) vs sliding-window (accurate, memory-hungry) rate limiting?
- Cache-aside (simple, brief staleness) vs write-through (fresh, doubled write latency) for *this* data?
- One large Redis node vs many small shards? (fork time, failover blast radius, hot-shard risk)
- `noeviction` vs `allkeys-lru` when the instance holds both cache entries and session data?
- Redis Streams (zero new infra) vs Kafka (durable replay) for this event flow?
- Strong consistency via Postgres vs sub-ms latency via Redis for the inventory counter?
- Sentinel (simpler) vs Cluster (scales) — what number pushes you across?

---

## 21. Exercises

1. **Structure-mapping drill (10 min).** For each, name the structure and one alternative you reject:
   ① "users who liked this post" ② "top 50 players this week" ③ "is this webhook a duplicate?"
   ④ "how many unique visitors today" ⑤ "jobs to run at 3pm" ⑥ "drivers within 2 km"
   ⑦ "a user's cart" ⑧ "the last 100 messages in a room."
2. **Capacity math.** 10M sessions × 2 KB = ? GB. Does it fit one 64 GB node with headroom for
   fork/COW? What if sessions are 20 KB? At what point do you shard, and what else could you do first?
3. **Write the failure paragraph.** In five sentences, describe what happens to a 3-shard Redis Cluster
   when one primary dies: to in-flight writes, to that shard's keys, to the other shards, to clients,
   and to any lock that lived on it.
4. **Atomicity audit.** Take `views = GET post:9:views; SET post:9:views (views+1)`. Show the exact
   interleaving that loses a count, then rewrite it three ways (`INCR`, `WATCH`, Lua) and say when
   each is the right choice.
5. **Design snippet.** Add the Redis layer to a ticket-booking system (seat holds, inventory, sessions,
   rate limiting). Mark every Redis use as *reconstructible* or *not* — and fix anything in the second
   category.
6. **Defend a rejection.** Write the three-part sentence ([TechChoices.md](../TechChoices.md)) for
   rejecting Redis in favour of Kafka for an analytics pipeline, and for rejecting Kafka in favour of
   Redis Streams for in-app notifications.

---

## 22. Revision

See [`Revision/Revision_R01_Redis.md`](../Revision/Revision_R01_Redis.md) — the pre-interview
quick-revision file (mental model, structure→use-case map, caching patterns, non-cache uses, scaling,
failure modes, alternatives, questions, traps, cheat sheet).

Spaced repetition (+1/+3/+7/+15/+30/+60/+90) is scheduled in
[`RevisionSchedule.md`](../RevisionSchedule.md) **only once this module is actually completed** — it is
not scheduled today, because nothing has been taught yet.

---

## 23. Cheat Sheet

```
MENTAL MODEL
  One fast clerk · one clipboard · a wall of differently-shaped boxes · in RAM · room can burn down.
  Fast enough to serve AS a cache/coordination layer — NOT durable enough to BE the database.

WHY FAST : RAM · no query planner · single-threaded execution (free atomicity, no locks)
           · I/O threads since 6.0 · efficient encodings + simple protocol
NUMBERS  : ~0.1–1 ms · ~50–100k ops/sec/instance (500k–1M+ pipelined) · 16,384 slots
           · HLL ≤12 KB @ 0.81% error

STRUCTURE → USE CASE
  String  INCR/SET NX EX → counters, rate limits, LOCKS, idempotency, cached blob
  Hash    HSET/HINCRBY   → sessions, carts, objects with independently-updated fields
  List    LPUSH/BRPOP    → simple job queue, capped recent-activity feed
  Set     SADD/SINTER    → uniqueness, tags, mutual friends, dedup
  ZSET    ZADD/ZRANK     → LEADERBOARDS, sliding-window limiter, delayed queue, presence, feeds
  Stream  XADD/XREADGROUP/XACK → queue WITH ack + redelivery ("Kafka-lite")
  Bitmap  SETBIT/BITCOUNT→ daily-active-users (1 bit/user)
  HLL     PFADD/PFCOUNT  → unique counts at scale, approximate
  GEO     GEOADD/GEOSEARCH → "near me"

CACHING (Redis layer over 032–037)
  cache-aside: GET → miss → DB → SET EX ttl ;  write: DB then DEL (never overwrite)
  maxmemory-policy DEFAULT = noeviction → WRITES FAIL. Pure cache → allkeys-lru.
  TTL doesn't refresh on read. Expiry = lazy + active sampling. UNLINK > DEL for big keys.
  stampede→mutex/logical expiry · penetration→negative cache+Bloom · avalanche→jitter+HA+breaker
  hot key→L1 cache / key splitting / replicas / CDN (SHARDING DOES NOT HELP)

BEYOND CACHING
  rate limit(String INCR | ZSET sliding | Lua token bucket) · lock(SET NX PX + Lua CAS release)
  session(Hash+TTL → stateless app servers) · counters(INCR, flush to DB)
  leaderboard(ZSET) · pub/sub(fire-and-forget, LOSES offline subscribers) · streams(ack+replay)
  queue(List simple | Stream reliable) · idempotency(SET NX EX) · presence(TTL key or ZSET)
  geo(GEOSEARCH)

DISTRIBUTED : replication COPIES · Sentinel PROMOTES (HA, no shards) · Cluster SPLITS + promotes
  Replication is ASYNC → an ACKNOWLEDGED write can be LOST on failover. WAIT reduces, never removes.
  Replicas = read scaling + stale reads. Writes scale only by sharding. Never read locks from replicas.
  Cluster multi-key ops need SAME SLOT → hash tags {user:123}:profile

PERSISTENCE : RDB snapshot (fast restart, lose minutes, fork spike) · AOF log (appendfsync
  always/EVERYSEC/no → lose ≤1s by default) · run both · RESTARTABLE ≠ DURABLE

ATOMICITY : every single command atomic · MULTI/EXEC = no rollback, no read-then-branch
  WATCH = optimistic CAS, caller retries · Lua = atomic read-branch-write (keep it short, same slot)
  Pipelining ≠ transaction. App-side GET→modify→SET = lost-update race.

VS : Memcached(simpler pure KV) · DB(durable/queryable) · Kafka(durable replayable, huge throughput)
     RabbitMQ(routing, DLQ, per-msg reliability) · etcd/ZK(correctness locks)

DEATH : no HA→avalanche · failover→lost acked writes + lock re-acquisition · maxmemory→errors or
  eviction · hot key→one shard saturates · KEYS */big DEL→blocks everyone · cold restart→100% miss
  ALWAYS STATE: fail open or fail closed, per use case.
```

---

## 24. Summary

- **Redis is an in-memory data-structure server**, not a key-value cache that happens to be fast. The
  structures (String, Hash, List, Set, **Sorted Set**, Stream, Bitmap, HLL, Geo) with atomic
  server-side operations *are* the product; naming the right one is most of the interview answer.
- **It is fast because of RAM + no query layer + single-threaded command execution.** That same
  single thread gives free atomicity and makes any O(n) command (`KEYS *`, big `DEL`, huge `SORT`) an
  instance-wide latency incident. I/O threading since 6.0 changes the network path, not the guarantee.
- **Redis is fast enough to serve as a cache and a coordination layer; it is not durable enough to be
  the system of record.** Persistence (RDB/AOF) makes it *restartable*, and asynchronous replication
  means an acknowledged write can still be lost on failover.
- **Beyond caching is where interviews live:** rate limiting, distributed locks, sessions, counters,
  leaderboards, Pub/Sub, Streams, queues, idempotency, presence, geospatial — each with a specific
  structure, a specific reason, and a specific failure mode.
- **Replication copies, Sentinel promotes, Cluster splits.** Replicas scale reads (staleness);
  only sharding scales writes; **nothing shards a single hot key** — that needs an L1 cache, key
  splitting, replicas, or the CDN.
- **Atomicity: single commands are free; `MULTI/EXEC` has no rollback and no read-then-branch;
  `WATCH` is optimistic CAS with a caller-side retry; Lua is the practical read-branch-write tool.**
  Pipelining is throughput, not a transaction.
- **Every Redis answer ends with "and when it dies…"** — avalanche, circuit breaker, L1 fallback, and
  an explicit fail-open/fail-closed decision per use case.

> **You will be able to:** place Redis correctly in a design, name the exact data structure and why,
> defend it against Memcached/Postgres/Kafka/RabbitMQ, explain its atomicity and durability
> guarantees precisely (including what it does *not* guarantee), and answer the hot-key, eviction,
> failover, and cache-death follow-ups without being surprised by any of them.

---

## Sources consulted during consolidation (2026-08-31)

**Official Redis documentation** — [Data types overview](https://redis.io/docs/latest/develop/data-types/) ·
[Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) ·
[Replication](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/) ·
[Distributed Locks with Redis](https://redis.io/docs/latest/develop/clients/patterns/distributed-locks/) ·
[High availability with Redis Sentinel](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/) ·
[Redis cluster specification](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/) ·
[HyperLogLog](https://redis.io/docs/latest/develop/data-types/probabilistic/hyperloglogs/) ·
[Rate-limiting how-tos](https://redis.io/tutorials/howtos/ratelimiting/)

**Engineering discussion & analysis** — Martin Kleppmann,
[How to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html) ·
antirez's [counterpoint](http://antirez.com/news/101) ·
[Redis streams vs. Kafka](https://mattwestcott.org/blog/redis-streams-vs-kafka) (Matt Westcott) ·
[Redis Streams vs Apache Kafka](https://www.instaclustr.com/blog/redis-streams-vs-apache-kafka/) (Instaclustr) ·
[Horizontal scaling with ElastiCache Redis — hot keys and shards](https://www.gomomento.com/blog/horizontal-scaling-with-elasticache-redis-stop-getting-burned-by-hot-keys-and-shards/) (Momento)

**Interview-calibration sources (basis for the ✅ REPORTED labels)** —
[Ultimate Guide to Redis in System Design (Interview Edition)](https://www.designgurus.io/blog/redis-guide) ·
[Hot keys, cache stampedes and thundering herds](https://designgurus.substack.com/p/hot-keys-cache-stampedes-and-thundering) (DesignGurus) ·
[Design a Distributed Rate Limiter](https://www.hellointerview.com/learn/system-design/problem-breakdowns/distributed-rate-limiter) (Hello Interview) ·
[Redis interview questions for system design](https://oneuptime.com/blog/post/2026-03-31-redis-interview-questions-system-design/view)

**Ecosystem/licensing context** — [Valkey (Linux Foundation)](https://en.wikipedia.org/wiki/Valkey) ·
[Redis reverts to open source (AGPL, Redis 8)](https://www.thestack.technology/redis-reverts-to-open-source/)
