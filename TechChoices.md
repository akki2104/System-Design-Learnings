# Technology Decision Playbook — "When to Use What, and Why Not the Others"

> Created 2026-07-08 at the learner's request. This file grows after every lesson that touches a
> technology category. Before any case study, review the relevant sections here.
> The interview goal is never "name the tech" — it's "name the tech, the reason, AND the rejected alternative."

**How to use in an interview — the 3-part sentence:**
> "I'll use **[tech]** because **[the specific problem property]**. I considered **[alternative]** but rejected it because **[what it trades away]**."

---

## The 3 Questions Before Any Tech Choice

```
1. What problem am I actually solving?   (reads? writes? search? consistency? scale?)
2. What does this tech trade away?       (every tool gives something and takes something)
3. What happens if this choice is wrong at 10×?  (can I migrate? blast radius?)
```

---

## Databases — The Decision Table

| Database | Pick it when | Do NOT pick it when | Real users |
|----------|-------------|--------------------|-----------| 
| **PostgreSQL / MySQL** (relational) | You need ACID transactions, joins, structured data with relationships. Money, orders, inventory, bookings. | Data is PB-scale; write throughput exceeds what one primary can take (~10–50K writes/sec); schema changes constantly | Stripe (payments), Amazon (orders), Uber (trips core) |
| **MongoDB** (document) | Flexible/nested schema, product catalogs, user profiles, CMS content. Objects that are read/written whole. | You need multi-record transactions or heavy cross-entity joins. **The classic disaster: payments on Mongo** | eBay (catalog), forums, content platforms |
| **Cassandra / DynamoDB** (wide-column) | Massive write throughput (100K+/sec), known access patterns, time-series-ish data: messages, feeds, sensor data | You need ad-hoc queries, joins, or strong transactions. Query patterns unknown upfront | Discord (messages), Netflix (viewing history), Amazon (cart) |
| **Redis** (in-memory KV) | Hot reads, sessions, leaderboards, rate-limit counters, caching layer | As the primary durable store — it's memory-first; a crash can lose recent writes | Twitter (timeline cache), everyone (sessions) |
| **Elasticsearch** (search) | Full-text search, fuzzy matching, log queries, faceted filters | As the source of truth — it's an *index built from* your real DB, eventual-consistent | Uber Eats (restaurant search), GitHub (code search) |
| **S3 / object storage** (blob) | Images, videos, backups, anything PB-scale and read-whole | Low-latency queries on structured fields — it has no query engine | Netflix (video), Dropbox (files), Instagram (photos) |

**The golden default:** *Start with PostgreSQL. Leave it only when a specific number forces you out* — writes exceed one primary (~10s of K/sec), data exceeds a few TB of hot set, or the data is genuinely unstructured blobs. Saying this in an interview signals maturity; jumping to Cassandra for a 1K-QPS app signals buzzword-matching.

**Interview-ready "why not" phrases:**
- *"Not Mongo here — this is money; I need multi-row ACID transactions."*
- *"Not Cassandra here — our query patterns are ad-hoc and the scale (2K QPS) doesn't justify losing joins."*
- *"Not Postgres for the messages table — 500K writes/sec exceeds a single write primary; Cassandra partitions writes across nodes."*
- *"Redis for the leaderboard, but it's a cache — the source of truth stays in Postgres."*

---

## Why Joins Get Expensive at Scale — the real trigger

**Common misconception:** "huge data volume" is what kills SQL joins. **Wrong.** A single Postgres machine joins tables fine even at hundreds of GB, as long as it's properly indexed — a join there is just a local, in-memory row match.

**The real trigger is SHARDING (spreading data across machines), not data size.**

```
Big data, ONE machine        → joins still work fine (good indexes are enough)
Data SPREAD across machines  → joins become expensive NETWORK calls
                                (every join now needs a round trip to another server)
```

Example — Zomato's `orders` and `restaurants`:
- **Unsharded Postgres:** `orders JOIN restaurants` = fast, local, one machine.
- **Orders sharded by user_id across 10 machines:** `restaurants` lives on one machine only. Every other shard now needs a network hop to fetch restaurant details for a join. Multiply that by every shard, every query — that's the "join overhead" people mean.

**This is why NoSQL documents look "duplicated":** a MongoDB order document embeds the restaurant's name/address directly instead of storing a `restaurant_id` and joining. The duplication is the price paid to avoid a network join. This is called **denormalization**, and it's a deliberate tradeoff — write once, read fast, accept some data duplication and eventual staleness if the restaurant details change.

---

## Estimation → Tech Choice (bridge from Topic 003)

| The number says | The tech conclusion |
|-----------------|--------------------|
| Read:write 100:1 | Redis/Memcached cache in front of the DB; read replicas |
| Read:write ≈1:1, high write QPS | Wide-column store (Cassandra) or sharded SQL — cache won't save you |
| Storage in PBs | S3-style object storage + metadata in a DB; never raw RDBMS |
| p99 latency < 50ms globally | CDN + regional deployments; single-region won't make it |
| Exact money movement | Relational DB + idempotency keys; nothing else is acceptable |

---

## Caching Patterns — Which One, and When (Topic 033)

| Pattern | Pick it when | Do NOT pick it when | Real users |
|---|---|---|---|
| **Cache-Aside** (lazy loading) | Default for read-heavy data; DB stays unambiguous source of truth; simple to reason about | You need zero read-after-write staleness window | Product catalogs, user profiles |
| **Write-Through** | Read-after-write freshness matters; can tolerate slightly higher write latency | Write volume is very high and every extra ms of write latency compounds | User settings/preferences |
| **Write-Behind** (write-back) | Write volume is very high, DB write throughput is the bottleneck, some data loss on a rare crash is acceptable | Losing unflushed writes would be a real problem (money, inventory) | View counters, analytics/activity events |

**Interview-ready "why not" phrases:**
- *"Not write-behind for inventory decrements feeding checkout — losing unflushed writes on a cache crash could cause overselling."*
- *"Not write-through for the view counter — doubling write latency on every view for data that's fine to lose a few seconds of isn't worth it."*
- *"Cache-aside stays the default for the catalog — the DB remains authoritative and a brief miss-then-repopulate window is fine."*

---

## Caching Infrastructure — Redis vs Memcached vs Managed (Topic 036)

| Option | Pick it when | Do NOT pick it when | Real users |
|---|---|---|---|
| **Memcached** | Pure key-value caching is the entire need; multi-threaded per-instance throughput matters more than features | You need rich data structures, persistence, or built-in clustering/HA | Simple page-fragment/session caches |
| **Redis** | Need data structures beyond plain KV (leaderboards, rate limiters, queues, session field-updates) with atomic server-side ops; want built-in HA + horizontal scaling | Data must never be lost — Redis is fast enough to serve as a cache, not to replace a real database | Twitter (timeline cache), rate limiters everywhere, leaderboards |
| **Managed (ElastiCache/MemoryDB)** | Want Redis/Memcached's benefits without operating the cluster (failover, patching, backups handled) | Team has strong ops bandwidth and needs config control the managed layer restricts | Most cloud-native product teams |

**Interview-ready "why not" phrases:**
- *"Not Memcached here — the leaderboard needs a sorted set with atomic rank queries; Memcached would force sorting in the app layer, reopening the exact race conditions a cache should avoid."*
- *"Not Redis as the source of truth — even with AOF, it's not an ACID-durable database; the real record stays in Postgres."*

---

## Invalidation Broadcast Mechanism (Topic 037)

| Option | Pick it when | Do NOT pick it when |
|---|---|---|
| **Redis Pub/Sub** | Already running Redis; need a lightweight, low-latency fan-out to app-server local caches | Guaranteed delivery matters — Pub/Sub is fire-and-forget, a subscriber offline at publish time misses the message |
| **Kafka topic** | Need durable, replayable invalidation events, or many heterogeneous consumers (analytics + cache invalidation off the same stream) | Latency must be sub-millisecond and the extra infra/ops overhead isn't justified for "just" cache invalidation |
| **CDN purge API** | Invalidating edge-cached content specifically | Invalidating app-server-local or distributed-cache copies — a CDN purge doesn't touch those |

**Interview-ready "why not" phrase:** *"Not a bare Redis DEL for the multi-layer case — that only clears the shared cache. I'd broadcast the invalidation over Pub/Sub so every app server's local cache evicts the key too, since it was never part of Redis's own replication to begin with."*

---

## Replication Topology (Topic 039)

| Option | Pick it when | Do NOT pick it when |
|---|---|---|
| **Leader-Follower** | Single-region system, strong write consistency matters, reads can tolerate slight staleness | Multiple regions need low-latency local writes |
| **Multi-Leader** | Multiple regions each need local writes, some write-conflict resolution is acceptable | Conflicts would be unacceptable/hard to resolve (e.g., financial ledgers) |
| **Leaderless** | Massive write throughput with tunable consistency is the priority (Cassandra/DynamoDB) | A single, unambiguous write path is required |

**Interview-ready "why not" phrase:** *"Not leaderless for the orders database — orders need a clear, unambiguous write path, not tunable eventual consistency; leader-follower with read replicas for reporting load is the right default."*

---

## Sections added by later topics

<!-- Topic 008 will add: TCP vs UDP decision box -->
<!-- Topic 015 will add: WebSockets vs SSE vs polling decision box -->
<!-- Topic 017 will add: L4 vs L7 load balancer decision box -->
<!-- Topic 059-060 will add: Kafka vs RabbitMQ vs SQS decision box -->
<!-- Topic 031 will consolidate the full DB decision framework -->
