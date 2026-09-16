# Revision — R01: Redis (Consolidated Module)

**Format:** Active recall — answer out loud *before* opening the answer.
**Status:** 📦 Consolidated, **not yet taught** — this is a pre-interview quick-revision sheet, not a
record of a completed lesson. No completion date, no confidence score, no spaced-repetition entries
until the module is actually delivered.
**Full lesson:** [`Topics/R01_Redis_Consolidated_Module.md`](../Topics/R01_Redis_Consolidated_Module.md)
**Use it as:** a 25–35 minute pass the night before an interview.

---

## 0. The 60-Second Mental Model

> One very fast clerk, one clipboard, a wall of **differently-shaped** boxes, all in RAM, in a room
> that can burn down.

```
IN RAM       → sub-ms, but bounded by memory and ~50–100× disk cost/GB
DATA STRUCTURES, not just KV → the actual differentiator; picking the right one IS the answer
ONE THREAD executing commands → every command atomic for free
                              → one slow O(n) command blocks EVERYONE
FIXED-SIZE ROOM → eviction policy is a decision you must make (default noeviction = writes FAIL)
CAN BURN DOWN  → persistence makes it RESTARTABLE, not DURABLE

THE SENTENCE: Redis is fast enough to serve AS a cache and coordination layer —
              NOT durable enough to BE the database. Source of truth stays in Postgres/DynamoDB/S3.
```

---

## 1. Data Structure → Use Case Map (recite this cold)

```
String   SET k v EX ttl NX · INCR   → cached blob · counters · rate limit · LOCK · idempotency key
Hash     HSET · HINCRBY             → SESSIONS · carts · objects with per-field updates
List     LPUSH / BRPOP              → simple job queue · capped recent-activity feed
Set      SADD · SISMEMBER · SINTER  → uniqueness · tags · mutual friends · dedup · online set
ZSET     ZADD · ZREVRANGE · ZREVRANK→ LEADERBOARDS · sliding-window rate limit · delayed queue
                                       · presence · time-ordered feeds     ← the workhorse
Stream   XADD · XREADGROUP · XACK   → queue WITH ack + redelivery + consumer groups ("Kafka-lite")
Bitmap   SETBIT · BITCOUNT          → daily-active-users (1 bit per user ID)
HLL      PFADD · PFCOUNT · PFMERGE  → unique counts at scale (≤12 KB/key, 0.81% error)
GEO      GEOADD · GEOSEARCH         → "drivers/restaurants near me"
```

**Self-test — name the structure, no peeking:**
① users who liked this post · ② top 50 players · ③ is this webhook a duplicate ·
④ unique visitors today · ⑤ jobs to run at 3pm · ⑥ drivers within 2 km · ⑦ a user's cart ·
⑧ last 100 messages in a room · ⑨ 100 requests per minute per API key · ⑩ who is online right now

<details>
<summary>Answers</summary>

① Set · ② ZSET · ③ String `SET NX EX` · ④ HyperLogLog (or Bitmap if you need per-user recall) ·
⑤ ZSET, score = run-at timestamp · ⑥ GEO · ⑦ Hash · ⑧ List with `LTRIM`, or ZSET by timestamp ·
⑨ String `INCR` (fixed window) or ZSET (sliding window) · ⑩ key-with-TTL for a boolean, ZSET of
heartbeat timestamps if you need the *list*.

</details>

---

## 2. Active Recall — 12 Questions

### Q1. Give the three real reasons Redis is fast, and state the single-threading nuance correctly.

<details>
<summary>Answer</summary>

(1) **In-memory** — no disk seek; memory read ~100 ns vs SSD ~100 µs. (2) **No query layer** — no
parser/planner/join engine; the command names the exact data-structure operation. (3)
**Single-threaded command execution with I/O multiplexing** — one event loop, commands run one at a
time to completion, so there is zero lock overhead and zero contention. It is fast *because* it is
single-threaded, not despite it.

**The nuance:** since **Redis 6.0**, network I/O can be offloaded to helper threads (`io-threads`), so
an instance can saturate a modern NIC — but **command execution is still single-threaded**, so both the
free-atomicity guarantee and the "one slow command blocks everyone" hazard still hold. Say both halves.

</details>

### Q2. What is the default `maxmemory-policy`, and why does that answer matter?

<details>
<summary>Answer</summary>

**`noeviction`** — Redis **rejects writes with an error** rather than evicting. So "what happens when
Redis runs out of memory?" is *not* "it evicts old data" unless someone configured it. For a pure
cache you set **`allkeys-lru`** (or `allkeys-lfu` for skewed popularity). Use the `volatile-*`
family when one instance mixes TTL'd cache entries with must-keep state — it only evicts keys that
have a TTL.

Also: **eviction ≠ expiry.** TTL is "this datum has a lifetime"; eviction is "we're out of room."
Different axes, both can remove your key. And a TTL does **not** refresh on read (sliding TTL is
opt-in via `GETEX`).

</details>

### Q3. Distinguish replication, Sentinel, and Cluster in one line each. Which two are alternatives?

<details>
<summary>Answer</summary>

- **Replication copies** — primary → replicas, giving read scaling and a warm spare. **No automatic
  failover by itself**; a human promotes.
- **Sentinel promotes** — a separate quorum of monitor processes detects a dead primary, elects and
  promotes a replica, and tells Sentinel-aware clients the new address. **No sharding.**
- **Cluster splits (and promotes)** — the keyspace is divided across shards by hash slot; each shard is
  its own primary+replicas doing its own failover. Scales **both memory and write throughput.**

**Sentinel and Cluster are alternatives, not layers** — if you run Cluster you do not run Sentinel.

</details>

### Q4. Redis replication is asynchronous. Give the exact five-step sequence in which an acknowledged write is lost.

<details>
<summary>Answer</summary>

1. Client sends `SET k v` to the **primary**.
2. Primary applies it and returns **"OK"** — *before* any replica has confirmed (async by default).
3. Primary **dies** before the write reaches the replica.
4. Sentinel/Cluster **promotes the replica**, which never saw `k`.
5. Client reads `k` → **nil**. An acknowledged write is gone.

`WAIT numreplicas timeout` reduces the window but — per Redis's own docs — **does not make Redis a CP
system**; acknowledged writes can still be lost on failover. `min-replicas-to-write` /
`min-replicas-max-lag` bound the loss by refusing writes when replicas lag, trading availability for
safety.

**Two consequences to state:** Redis is not a system of record, and a Redis lock held here can be
re-acquired by a second client.

</details>

### Q5. How does Redis Cluster route a key, and what breaks for multi-key operations?

<details>
<summary>Answer</summary>

**16,384 fixed hash slots**; `slot = CRC16(key) mod 16384`. Each slot is owned by exactly one shard
(primary + replicas). Clients are cluster-aware and route directly, following `MOVED`/`ASK` redirects
during resharding. The slot count is **fixed** so rebalancing moves *slots*, not every key — the same
goal as consistent hashing, a different implementation.

**What breaks:** any multi-key operation — `MGET`, `MULTI/EXEC`, a Lua script touching several keys —
only works if all keys are in the **same slot**. Force that with **hash tags**: `{user:123}:profile`
and `{user:123}:cart` hash only on the `{…}` portion, so both land on the same slot.

**Availability caveat:** if a shard's primary and all its replicas die, by default the whole cluster
stops serving (`cluster-require-full-coverage yes`), not just that shard's keys.

</details>

### Q6. A single key gets 200k rps. Why doesn't adding Redis shards fix it, and what are the four real fixes?

<details>
<summary>Answer</summary>

**Why sharding fails:** one key hashes to one slot, which lives on one shard, which is served by one
command thread. Adding shards gives you more *other* slots; the hot slot stays exactly where it was.

**The four fixes:**
1. **In-process L1 cache** in front of it (best first move — absorbs most reads before the network).
2. **Key splitting/replication** — `product:123#0` … `#N`; reads pick a random copy, writes fan out to
   all N (accept N× write cost and brief skew).
3. **Serve from read replicas** if staleness is acceptable.
4. **Push it to the CDN/edge** if it's a public read.

Related: a **hot shard** (uneven slot distribution or one dominant key prefix) is fixed by re-keying to
spread load or isolating the noisy tenant. A **big key** (multi-GB collection) blocks the thread on
access/delete — split it, and delete with `UNLINK`, never `DEL`.

</details>

### Q7. Give the complete distributed-lock recipe, then the caveat that makes it insufficient for correctness.

<details>
<summary>Answer</summary>

```
ACQUIRE : SET lock:resource <random_uuid> NX PX 30000
RELEASE : Lua compare-and-delete — delete ONLY IF the value is still my uuid
```

The three non-negotiables: **TTL (`PX`)** or a crashed holder deadlocks forever · **a unique random
value** or a slow client whose lock expired will delete *someone else's* lock · **atomic Lua
compare-and-delete** because `GET`-then-`DEL` from the app has a race in the gap.

**The caveat (Redlock debate):** async replication means a failover can hand the same lock to two
clients. Redlock (majority of N independent primaries) was antirez's answer; Kleppmann's critique is
that it relies on timing assumptions (a GC pause, clock jump, or network delay can leave a client
believing it still holds an expired lock) and gives no **fencing token**. Practical rule both sides
converge on:

- **Efficiency lock** (double execution is merely wasteful) → single-instance `SET NX PX` is fine.
- **Correctness lock** (two holders corrupt data) → **fencing tokens**, or a consensus-backed service
  (**etcd/ZooKeeper**), or — best — **make the operation idempotent so the lock needn't be perfect.**

Redis's own docs now say: implement fencing tokens if you care about correctness, and note that Redis
TTL expiry does not use a monotonic clock.

</details>

### Q8. `MULTI/EXEC`, `WATCH`, and Lua — what does each give you, and what does `MULTI/EXEC` *not* do?

<details>
<summary>Answer</summary>

- **Every single command is already atomic** (one thread, run to completion) — `INCR`, `ZADD`,
  `HINCRBY`, `SET NX` need no external lock. Most Redis patterns rest on this.
- **`MULTI/EXEC`** queues commands and executes the batch with no other client interleaving. But:
  **no rollback** on runtime errors (the other commands still apply), and **no read-then-branch** —
  results don't exist until `EXEC`. It gives isolation, not the "A" and "D" a SQL engineer expects.
- **`WATCH`** is optimistic concurrency control / compare-and-swap: watch a key, read it outside the
  transaction, `MULTI`…`EXEC`; if the key changed since `WATCH`, `EXEC` returns nil and **the caller
  must retry**. Good under low contention, retry-storms under high contention.
- **Lua (`EVAL`)** runs atomically on the server and — unlike `MULTI` — lets you read, branch, and
  write inside one atomic unit. That's why the sliding-window limiter, token bucket, and
  compare-and-delete lock release are all Lua. Keep scripts short (a long script blocks the thread),
  declare keys via `KEYS[]`, same hash slot in Cluster mode.

**And: pipelining ≠ transaction.** Pipelining batches commands into one round trip for throughput; it
gives no atomicity and no isolation.

</details>

### Q9. RDB vs AOF — what does each lose in a crash, and what's the honest summary?

<details>
<summary>Answer</summary>

- **RDB** = periodic fork+snapshot. Fast restart, compact, great for backups/replica bootstrap.
  **Loses everything since the last snapshot (minutes).** The `fork()` causes a latency spike and a
  copy-on-write memory spike on large datasets.
- **AOF** = append-only log of every write command. Durability is set by `appendfsync`:
  `always` (safest, slowest) · **`everysec` (default — lose ≤ ~1 second)** · `no` (OS flushes, ~30 s).
  Bigger files, slower restart (log replay), periodic rewrite.
- **Redis recommends running both**: RDB for backups and fast restarts, AOF for the small loss window.

**The honest summary:** persistence makes Redis **restartable, not durable**. And even
`appendfsync always` doesn't save you — layer on async replication + failover and an *acknowledged*
write can still be lost (Q4). Also remember the **cold-restart** problem: an empty cache means 100%
misses, which is a cache avalanche against the DB.

</details>

### Q10. Redis Streams vs Kafka vs Redis Pub/Sub — one clean distinction each.

<details>
<summary>Answer</summary>

- **Pub/Sub** — fire-and-forget, at-most-once. **No persistence, no ack, no replay, no consumer
  groups.** A subscriber offline at publish time **loses the message permanently**. Good for cache-
  invalidation fan-out and cross-server WebSocket push; never for anything that must arrive.
- **Streams** — append-only log **in memory**, trimmed by `MAXLEN`/`MINID`. Consumer groups hand each
  entry to exactly one consumer; unacked entries sit in the pending-entries list and can be `XCLAIM`ed
  after a timeout. Job-queue semantics with crash recovery. Retention bounded by RAM.
- **Kafka** — **on-disk** replicated commit log, retention in days/weeks/forever, partitions as the
  unit of parallelism with per-key ordering, and **full history replay** — its defining feature. Costs
  brokers, partition strategy, rebalancing, and its own monitoring stack.

**Pick Streams** when you already run Redis, volume is modest, retention is short, and you want ack +
redelivery without a broker. **Pick Kafka** for durable replayable history, event sourcing/CDC, many
independent consumer groups on one stream, or multi-region durability.

**Trap:** "Redis Pub/Sub is like Kafka" — it isn't; Redis **Streams** is the Kafka-shaped thing.

</details>

### Q11. Redis is down. Walk through the consequences and what you'd have designed in advance.

<details>
<summary>Answer</summary>

**Consequences:** 100% of cached reads fall straight to the DB instantly → **cache avalanche**;
sessions gone → everyone logged out; rate-limit counters gone → enforcement decision needed; locks
gone → whatever they guarded is unguarded; queued jobs in a List → lost unless persisted.

**Designed in advance:**
- **HA topology** — replicas + Sentinel (or Cluster), so a node death is seconds of disruption, not an
  outage.
- **Circuit breaker in front of the DB** so the DB isn't taken down by the fall-through load.
- **In-process L1 cache** so a Redis outage means *degraded* caching, not zero caching.
- **An explicit fail-open / fail-closed decision per use case** — the rate limiter usually fails open
  (availability beats perfect enforcement) but a billing quota may fail closed; caching always fails
  open to the DB; anything correctness-critical was never Redis-only to begin with.
- **Cache warming / staged traffic ramp** for the cold-restart case.

The two-sentence version is in §13 of the full lesson — memorise its shape, not its words.

</details>

### Q12. Which O(n) commands are production hazards, and why does "O(n)" matter more in Redis than elsewhere?

<details>
<summary>Answer</summary>

**Hazards:** `KEYS pattern` (O(n) over the **entire keyspace** — never in production, use `SCAN`) ·
`DEL` on a huge collection (use **`UNLINK`**, which frees in a background thread) · unbounded `SORT` ·
`HGETALL` on a giant hash · `SMEMBERS` on a giant set · `FLUSHALL` · a long Lua script.

**Why it matters more here:** commands execute on a **single thread**. An O(n) command isn't slow for
*the caller* — it is a latency incident for **every client connected to that instance**, until it
finishes. So the whole production-safety heuristic collapses to one question: *"is this command O(n),
and how big is n?"*

</details>

---

## 3. Explain From Scratch — the 10-line skeleton

Recite this unaided; each line should expand into 20–40 seconds of speech.

```
1.  Redis = in-memory DATA-STRUCTURE server, not a plain KV cache.
2.  Fast because: RAM · no query layer · single-threaded execution (free atomicity, no locks).
3.  The structures ARE the product: String/Hash/List/Set/ZSET/Stream + Bitmap/HLL/GEO.
4.  As a cache: cache-aside, TTL with jitter, allkeys-lru (default noeviction FAILS writes),
    invalidate-don't-update, and the three problems — penetration/stampede/avalanche.
5.  Beyond caching: rate limits, locks, sessions, counters, leaderboards, pub/sub, streams,
    queues, idempotency, presence, geo — each maps to a specific structure.
6.  Distributed: replication COPIES (async!), Sentinel PROMOTES (HA, no shards),
    Cluster SPLITS via 16,384 hash slots (+ hash tags for multi-key).
7.  Persistence: RDB snapshot (lose minutes) + AOF log (lose ~1s at everysec).
    RESTARTABLE, NOT DURABLE.
8.  Atomicity: single commands free · MULTI/EXEC has no rollback · WATCH = optimistic CAS
    · Lua = atomic read-branch-write. Pipelining is NOT a transaction.
9.  Failure: no HA → avalanche · failover → lost acked writes + lock re-acquisition ·
    maxmemory → errors or eviction · hot key → sharding does NOT help.
10. Therefore: Redis is fast enough to serve AS a cache/coordination layer, not durable enough
    to BE the database. Everything in Redis must be reconstructible from the store beneath it.
```

---

## 4. The Three Diagrams (redraw from memory)

### ① Where Redis sits

```
clients → LB → [stateless app servers, each with a small L1 map]
                          ↓
              ╔═══════════════════════════════╗
              ║  REDIS: cache · sessions ·    ║   everything here must be
              ║  counters · locks · ZSETs ·   ║   RECONSTRUCTIBLE from below
              ║  presence · queues            ║
              ╚═══════════════╤═══════════════╝
                              ↓ miss / flush
              ┌───────────────────────────────┐
              │ SOURCE OF TRUTH: Postgres/S3  │
              └───────────────────────────────┘
```

### ② The three topologies

```
① STANDALONE            [App] → [Redis]                    SPOF, RAM-bound
② PRIMARY+REPLICAS      [App] → [PRIMARY] ══async══→ [R1][R2]      ← DEFAULT ANSWER
   + SENTINEL×3          Sentinel: monitor → quorum → promote → tell clients
③ CLUSTER               16,384 slots across shards; each shard P+replica, own failover
                        scales memory AND writes; multi-key needs hash tags
```

### ③ The async-replication write loss

```
SET k v → PRIMARY → "OK" to client   (BEFORE replication)
PRIMARY 💥 → replica (no k) promoted → read k = nil
⇒ not a system of record · a lock here can be held by two clients
```

---

## 5. Traps to Watch (from prior topics + known Redis pitfalls)

**Already logged against you in [InterviewMistakes.md](../InterviewMistakes.md):**

| Prior mistake | The Redis form it will take |
|---|---|
| 2026-08-04 (T036) — "Redis is fast enough to be used as a database" | **Recurring family** with T032's durable-store trap. Watch for it re-emerging as "we could just keep the orders in Redis with AOF." Inverted: fast enough to serve *as* a cache, not to *be* a DB |
| T034 — assumed a TTL refreshes on read | Redis TTL is an absolute countdown from write; sliding TTL is opt-in (`GETEX`) |
| T033 — cache-aside write path omitted invalidation | In Redis this is the missing `DEL product:123` after the Postgres `UPDATE` |
| T017 — conflated IP-hash stickiness with Redis sessions | Redis sessions **remove the need for stickiness** (true statelessness); they are not "a better sticky session" |
| T037 — mixed up an in-process L1 cache with a distributed cluster node | Relevant again: a bare Redis `DEL` clears Redis, not each app server's L1 — that needs Pub/Sub |
| T032 — display-vs-transactional staleness (PERSISTENT ×2) | The e-commerce section: catalog staleness is fine, inventory staleness is not. Redis gates, Postgres commits |

**The eight generic Redis traps:**

1. "Redis is durable now, it has AOF." → **Restartable, not durable**; failover still loses acked writes.
2. "Add more nodes for the hot key." → One key, one slot, one thread.
3. "MULTI/EXEC rolls back on error." → It does not.
4. "Pub/Sub is a reliable queue." → At-most-once, no persistence, no replay.
5. "The lock guarantees one worker." → Not across failover. Efficiency vs correctness.
6. "Redis evicts when memory fills." → Only if `maxmemory-policy` isn't the default `noeviction`.
7. "Read the counter from a replica to scale." → Asynchronously stale; correctness reads → primary.
8. "Streams is basically Kafka." → Memory-bounded retention, different parallelism, no long replay.

**Two habits that fail silently:** doing `GET` → modify in app → `SET` (a lost-update race — use
`INCR`/`WATCH`/Lua), and N sequential `GET`s in a loop (N round trips — use `MGET`/pipelining).

---

## 6. Interview Questions — the ten to have ready

Full list with provenance labels in §20 of the lesson. These are the ones to be fluent on:

1. Why Redis and not Memcached **here**? (name the structure you'd lose)
2. Design a distributed rate limiter. (algorithm + its flaw + atomicity + Redis-down behaviour)
3. Implement a distributed lock in Redis. (`SET NX PX` + Lua release → then the Redlock caveat)
4. Design a leaderboard. (ZSET; `ZREVRANGE` **and** `ZREVRANK`; why SQL `ORDER BY` doesn't scale)
5. What happens if Redis goes down? (avalanche, breaker, L1, fail-open/closed)
6. Is Redis single-threaded, and why does it matter? (free atomicity **and** blocking; I/O threads 6.0)
7. How does Redis Cluster route a key? (16,384 slots, CRC16, hash tags)
8. Redis vs Kafka — when each? (retention/replay/throughput/ops; Pub/Sub ≠ Kafka)
9. How do you keep the cache consistent with the DB? (invalidate-don't-update, ordering, TTL backstop)
10. RDB vs AOF — what do you lose in a crash? (per-mode window **plus** the failover caveat)

**The follow-up chain you will actually face:**

```
"I'd cache it in Redis" → which structure? → what TTL? → two concurrent misses? (stampede)
→ key never exists? (penetration) → 200k rps on it? (hot key) → Redis dies? (avalanche)
→ how does the cache learn about a DB write? (invalidation) → is the update atomic? (INCR/Lua)
→ would Memcached do?
```

**Trade-off questions (reasoning is the score, not the answer):** fixed vs sliding window ·
cache-aside vs write-through for *this* data · one big node vs many shards · `noeviction` vs
`allkeys-lru` on a mixed instance · Streams vs Kafka · Postgres consistency vs Redis latency for
inventory · Sentinel vs Cluster (and the number that pushes you across).

---

## 7. 30-Second Elevator Explanation

> Redis is an **in-memory data-structure server** — not just a key-value cache. Its differentiator is
> typed values with atomic server-side operations: sorted sets for leaderboards and sliding-window
> rate limits, hashes for sessions, streams for queues with acknowledgement, plus geo, bitmaps and
> HyperLogLog. It's fast because everything is in RAM, there's no query layer, and command execution
> is single-threaded — which also gives atomicity for free, at the cost of one slow O(n) command
> blocking every client. For availability you add replicas plus **Sentinel** for automatic failover;
> when one node's RAM or write throughput isn't enough you move to **Cluster**, which shards across
> 16,384 hash slots. Replication is **asynchronous**, so an acknowledged write can be lost on
> failover, and persistence — RDB snapshots and the AOF log — makes Redis *restartable*, not
> *durable*. So the rule is: Redis is fast enough to serve **as** a cache and a coordination layer,
> never durable enough to **be** the database. Everything I put in Redis has to be reconstructible
> from the store beneath it.

---

## 8. Final Cheat Sheet (one screen, last look before you walk in)

```
MODEL     one fast clerk · one clipboard · differently-shaped boxes · RAM · room can burn down
RULE      fast enough to serve AS a cache/coordination layer — NOT durable enough to BE the DB
NUMBERS   0.1–1 ms · 50–100k ops/sec/instance · 16,384 slots · HLL ≤12 KB @ 0.81%

STRUCTURE String(counter/lock/idem) Hash(session/cart) List(queue/feed) Set(unique/tags)
          ZSET(leaderboard/sliding-window/delayed/presence) Stream(ack+redelivery)
          Bitmap(DAU) HLL(unique counts) GEO(near me)

CACHE     cache-aside: GET→miss→DB→SET EX; write: DB then DEL (never overwrite)
          DEFAULT maxmemory-policy = noeviction → WRITES FAIL. Pure cache → allkeys-lru.
          TTL doesn't refresh on read · UNLINK > DEL · jitter TTLs
          stampede→mutex/logical expiry · penetration→negative cache+Bloom · avalanche→jitter+HA
          HOT KEY → L1 / key-split / replicas / CDN.  SHARDING DOES NOT HELP.

BEYOND    rate limit · lock (SET NX PX + Lua CAS) · session · counter · leaderboard · pub/sub
          · stream · queue · idempotency (SET NX EX) · presence · geo

DIST      replication COPIES (ASYNC → acked writes can be LOST) · Sentinel PROMOTES (no shards)
          Cluster SPLITS + promotes · multi-key needs SAME SLOT → hash tags {user:1}:*
          replicas scale READS only · never read locks/counters from a replica

PERSIST   RDB snapshot (lose minutes, fork spike) · AOF (everysec → lose ~1s) · run both
          RESTARTABLE ≠ DURABLE · cold restart = 100% miss = avalanche

ATOMIC    single commands free · MULTI/EXEC: no rollback, no read-then-branch
          WATCH = optimistic CAS (caller retries) · Lua = atomic read-branch-write
          pipelining ≠ transaction · app-side GET→modify→SET = lost update

VS        Memcached(simpler KV) · Postgres(durable/queryable) · Kafka(durable replay, throughput)
          RabbitMQ(routing/DLQ/per-msg acks) · etcd-ZK(correctness locks)

ALWAYS SAY   which structure · what TTL · what happens when it dies · fail open or fail closed
NEVER SAY    "Redis can be the database" · "add shards for the hot key" · "Pub/Sub is like Kafka"
```
