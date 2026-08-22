# Revision — Topic 041: Partitioning & Sharding

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-22

---

## Q1. Why can't replication alone solve a write-throughput or storage-capacity problem?

<details>
<summary>Answer</summary>

Replication makes copies of the same data — it doesn't split the data up. Every write still funnels through one leader, so more followers don't help write throughput at all. And the leader's disk still has to hold the entire dataset, so replication doesn't help storage capacity either. Both require actually dividing the data across machines — partitioning/sharding.

</details>

---

## Q2. A hash-partitioned cluster (`hash(key) % N`) wants to add a shard. What goes wrong with the naive approach?

<details>
<summary>Answer</summary>

Changing N in `hash(key) % N` reshuffles the shard assignment for almost every key, not just a fair share of them (e.g., going from %4 to %5 changes most keys' target shard). This forces a massive, disruptive data migration across the whole cluster just to add one shard. This is the exact motivation for Consistent Hashing (Topic 042).

</details>

---

## Q3. Why does range-based partitioning by timestamp tend to create a hot partition?

<details>
<summary>Answer</summary>

All new writes always have the newest timestamp, so they always land on the same "latest" shard — that shard is permanently hot while older shards, which only get occasional reads, sit comparatively idle.

</details>

---

## Q4. A system shards its `orders` table by `customer_id`, and each shard is itself a 3-node leader-follower replica set. What problem does sharding solve, and what problem does the per-shard replication solve?

<details>
<summary>Answer</summary>

Sharding solves the write-throughput/storage problem — dividing customers across shards divides the write load and total data volume across machines. Per-shard replication's primary role is availability/fault-tolerance — without it, each shard would be a brand-new single point of failure for its own slice of customers' data (that shard's machine dying would make that data unavailable with no fallback). Read-scaling within a shard is a real secondary benefit of the replication, but availability is the more fundamental reason it's needed. Both are required because they solve different problems — dividing load vs. protecting each divided piece.

</details>

---

## Q5. What's the tradeoff of hash-based partitioning versus range-based partitioning?

<details>
<summary>Answer</summary>

Hash-based: even distribution, no hot-spotting from skewed key ranges, but range queries become expensive (must query every shard and merge) and naive rebalancing is disruptive. Range-based: efficient range queries (one shard per range), but risks hot-spotting if access/write patterns skew toward a particular range (e.g., timestamp-based keys).

</details>

---

## 30-Second Elevator Pitch

> Replication solves availability and read scaling but can't fix storage-capacity or write-throughput limits, since every write still goes through one leader and one leader's disk holds all the data. Partitioning/sharding splits the dataset itself across machines to solve exactly those two problems — and it's composed WITH replication, not instead of it: each shard is typically its own leader-follower set, so sharding divides the load while replication protects each divided piece from being a new single point of failure. Three strategies: range-based (efficient ranges, risks hot-spotting), hash-based (even distribution, sacrifices cheap range queries and cheap rebalancing — motivating Consistent Hashing next), directory-based (flexible, but the lookup service is a new dependency). Sharding's real costs are expensive cross-shard queries/joins, hard distributed transactions, and difficult rebalancing — reasons to shard only once you actually need to, not preemptively.

---

## Weak Areas to Watch

- Crediting per-shard replication only with read-scaling and omitting its primary role: protecting each shard from becoming a new single point of failure (availability)
