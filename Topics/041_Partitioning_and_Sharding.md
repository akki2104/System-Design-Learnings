# Topic 041: Partitioning & Sharding

**Module:** 4 — Scaling & Distributing Data
**Tier:** 🔴 MUST
**Completed:** 2026-08-22
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topic 039 (Replication) solved availability (a replica survives a leader's death) and read scaling (spread reads across followers) — but every write still funnels through one leader, and that leader's disk still has to hold the *entire* dataset. Replication makes copies; it doesn't split the data up. Two walls replication can't fix: **storage capacity** (a single machine's disk is finite) and **write throughput** (one leader, no matter how many followers it has, can only absorb so many writes). Partitioning (sharding) is the answer — split the dataset itself across multiple machines so each holds and writes only a fraction of it. This is a top-3 highest-signal topic in the curriculum and a direct prerequisite for Case Study #1 (TinyURL).

---

## 2. Terminology

"Partitioning" and "sharding" are used near-interchangeably in interviews. If a distinction is drawn: partitioning is the general concept of splitting data into subsets; sharding specifically means those partitions live on separate physical machines/nodes. In practice, either term is understood to mean the same thing.

---

## 3. Partitioning + Replication Are Orthogonal (and Composed Together)

The single most important structural fact in this topic: partitioning and replication solve **different** problems and are used **together**, not as alternatives.
```
Partitioning → solves: storage capacity, write throughput (spread data/writes across N machines)
Replication  → solves: availability, read scaling (copy each machine's data onto followers)

REAL SYSTEMS DO BOTH:
Shard 1 (data range A-M) → [Leader] → [Follower] → [Follower]
Shard 2 (data range N-Z) → [Leader] → [Follower] → [Follower]
```
Each shard is *itself* a leader-follower replica set. Sharding decides *which machine group* owns a piece of data; replication decides *how many copies* of that piece exist and how available it is. Treating one as a substitute for the other is the most common shallow answer on this topic — in particular, without per-shard replication, each shard becomes a brand-new single point of failure for its own slice of the data: if that one machine dies, that entire slice is unavailable (or lost) until recovered, with no fallback. Sharding divides the load; replication protects each divided piece from being a new SPOF.

---

## 4. Three Partitioning Strategies

### Range-Based Partitioning
Split data by ranges of the partition key.
```
Shard 1: users A-M
Shard 2: users N-Z
```
**Pro:** range queries stay efficient — "all users from Da to Db" hits exactly one shard.
**Con — hot spotting:** if access or write patterns are skewed toward a specific range, that shard gets overloaded while others sit idle. Classic failure case: partitioning by timestamp/date — all NEW writes always land on the "latest" shard, since new records always have the newest timestamp, making it permanently hot while older shards go cold.

### Hash-Based Partitioning
`shard = hash(key) % N`
**Pro:** hashing spreads keys near-uniformly across shards regardless of the key's natural distribution — no hot spotting from skewed key ranges.
**Con:** range queries become expensive — "all users from Da to Db" now has to query *every* shard and merge results, since consecutive keys are scattered randomly. Also: naively changing N (adding/removing a shard) reshuffles the shard assignment for almost every key (not just a fair share of them), forcing a massive, disruptive data migration — this exact problem is what Consistent Hashing (Topic 042, next) is built to solve.

### Directory-Based (Lookup-Service) Partitioning
A separate lookup/directory service holds an explicit mapping of key → shard, rather than computing it from a formula.
**Pro:** maximum flexibility — rebalancing means updating map entries, not recomputing a hash function; can put a specific "hot" key on its own dedicated shard.
**Con:** the lookup service itself is a new critical dependency — every request now needs an extra hop to it, and it becomes a new single point of failure/bottleneck unless it's made highly available itself.

---

## 5. What Partitioning Costs You

Splitting data across machines reintroduces problems that were trivial on one machine:
- **Cross-partition queries/joins get expensive** (Topic 005's insight made concrete): a join across two shards means a network round-trip, not an in-memory lookup.
- **Cross-partition transactions are hard.** A single-node ACID transaction (Topic 024) is easy because everything lives on one machine with one WAL. A transaction touching two shards needs distributed coordination — two-phase commit (Topic 054 preview) or a Saga (Topic 055 preview).
- **Hot partitions can still occur** even with hashing, if the *chosen key* itself is low-cardinality or one value dominates traffic (Topic 029's hot-partition concept, generalized).
- **Rebalancing is operationally hard** — adding/removing shards means physically moving data between machines while the system stays live (full topic in 044).

---

## 6. Decision Box: Do You Even Need to Shard?

```
Replication alone is enough when:
  - Read-heavy, and the dataset fits comfortably on one (replicated) machine
  - Write volume is well within one machine's capacity

Shard when:
  - Data volume will exceed what any single machine's disk can hold
  - Write throughput exceeds what a single leader can process, even projecting
    growth — this is a WRITE problem specifically, not a read problem
  - You've already maxed out vertical scaling (Topic 038) on the write node
```
**Interview sentence:** "I'd start with a single leader-follower setup — replication alone handles our read scaling and availability. I'd only introduce sharding once write throughput or data volume is projected to exceed a single machine's capacity, since sharding adds real complexity: cross-shard queries, distributed transactions, and rebalancing overhead a single-leader system never has to deal with."

---

## 7. Worked Example — A Messaging App's Message Table

Requirements: billions of messages, extremely write-heavy, most queries are "get all messages in conversation X."
- **Partition key:** `conversation_id` — every message in the same conversation lands on the same shard, so the common query never needs to cross shards.
- **Strategy:** hash-based on `conversation_id`, avoiding hot-spotting on any particular conversation range.
- **Composed with replication:** each shard is itself a leader-follower set — a single shard's leader dying doesn't lose that slice of data, and read-heavy conversations can still be served by followers within that shard.
- **What this doesn't solve:** "find all messages containing word X across all conversations" is a genuinely cross-shard query — full-text search needs its own dedicated system (Elasticsearch, Topic 093/094 preview) rather than querying the sharded message store directly.

---

## 8. Common Mistakes

| Mistake | Correction |
|---|---|
| Treating replication and partitioning as alternative solutions to the same problem | They solve different problems and are composed together — replication (availability/read-scaling) is applied *per shard*, not instead of sharding |
| Assuming hash-based partitioning has no downsides | It sacrifices efficient range queries and makes naive rebalancing expensive — motivates Consistent Hashing (next topic) |
| Choosing a partition key without checking actual query patterns | A key that spreads writes evenly but forces common queries to cross shards defeats much of sharding's benefit — query-driven key choice (Topic 043) matters as much as even distribution |
| Sharding prematurely, "just in case" | Sharding adds real operational complexity a single well-replicated node doesn't have — only shard once storage or write-throughput limits are actually being approached |
| Crediting per-shard replication only with read-scaling, omitting availability | The more fundamental reason for per-shard replication is fault-tolerance — without it, each shard is a brand-new single point of failure for its own slice of data; read-scaling is a real but secondary benefit |

---

## 9. Real Interview Questions

1. "Why can't replication alone solve a write-throughput problem?" (tests the core distinction this topic exists for)
2. "What's the downside of hash-based partitioning, and when does it show up?" (range queries + rebalancing cost)
3. "Your team wants to add a shard to a hash-partitioned cluster — what goes wrong with a naive `% N` approach?" (tests the rebalancing-cost mechanism, direct setup for Topic 042)
4. "A system shards by customer_id and replicates each shard 3x. What problem does sharding solve, and what problem does the replication solve?" (tests that both are needed and for different reasons — write-throughput/storage vs availability+read-scaling)
5. "When would you choose NOT to shard, even at meaningful scale?" (tests judgment about premature complexity)

---

## 10. Revision Questions
See `Revision/Revision_041.md`.

## 11. Summary
- Replication solves availability and read scaling; it cannot solve storage-capacity or write-throughput limits, because every write still funnels through one leader and one leader's disk still holds all the data.
- Partitioning/sharding splits the dataset itself across machines, solving exactly those two problems.
- Partitioning and replication are orthogonal and composed together — each shard is typically its own leader-follower replica set.
- Three strategies: range-based (efficient range queries, risks hot-spotting), hash-based (even distribution, sacrifices range queries and cheap rebalancing), directory-based (flexible, but the lookup service is a new dependency).
- Sharding costs: expensive cross-shard queries/joins, hard distributed transactions, possible hot partitions even with hashing, and operationally difficult rebalancing.
- Per-shard replication's primary role is protecting each shard from becoming a new single point of failure — read-scaling is a real secondary benefit, not the main reason.
