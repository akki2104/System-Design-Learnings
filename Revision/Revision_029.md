# Revision — Topic 029: Wide-Column Stores

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-26

---

## Q1. What does "leaderless replication" precisely mean, and why isn't it "any node in the cluster can accept any write"?

<details>
<summary>Answer</summary>

Consistent hashing maps each partition key to a specific set of replica nodes (e.g., N=3). Any of THOSE replica nodes can accept a write for that specific partition — not any node in the whole cluster. Precision matters because it clarifies that writes are still localized to the relevant replica set, just without a single leader among them.

</details>

---

## Q2. Why does Cassandra's write throughput depend on BOTH the LSM-Tree engine AND leaderless replication?

<details>
<summary>Answer</summary>

Losing the LSM-Tree engine would mean falling back to random-position disk writes, losing the sequential-write speed advantage. Losing leaderless replication would reintroduce a single-primary-per-partition bottleneck — the same outage/availability risk relational master-slave setups have. The two mechanisms solve different halves of the problem: per-node write speed (LSM-Tree) and cluster-wide write distribution (leaderless replication).

</details>

---

## Q3. Why can't a single Wide-Column table efficiently serve two different access patterns (e.g., "messages by conversation" and "conversations by user")?

<details>
<summary>Answer</summary>

A partition key determines how data is physically distributed and located. A table partitioned by `conversation_id` has no way to efficiently answer "all conversations for a user" — that would require scanning every partition across the cluster. A second table, partitioned by `user_id`, is needed to physically co-locate that access pattern's data. This is why Wide-Column stores use one denormalized table per query pattern instead of joins.

</details>

---

## Q4. What is a hot partition, and what causes it?

<details>
<summary>Answer</summary>

A hot partition occurs when a partition key has low cardinality or skewed access (e.g., partitioning by `country` when most traffic comes from one country) — overwhelming one replica set while others sit idle. Caused by a poorly-chosen partition key relative to the actual traffic distribution.

</details>

---

## Q5. What are the three tunable consistency levels in Cassandra, and what's the tradeoff?

<details>
<summary>Answer</summary>

ONE (fastest, weakest — only 1 replica acks), QUORUM (majority of replicas — balanced), ALL (every replica acks — strongest, slowest, least available). Tradeoff: speed/availability vs. consistency strength, chosen per query.

</details>

---

## 30-Second Elevator Pitch

> Wide-Column stores organize data by partition key (distribution) and clustering key (order within a partition), with sparse columns per row. Write throughput comes from two compounding mechanisms: an LSM-Tree storage engine per node (fast sequential writes) and leaderless replication across the cluster (any replica of a partition's specific replica set can accept a write — not any node in the whole cluster). The cost is tunable, weaker consistency and no joins — which forces query-driven data modeling: one denormalized table per access pattern, because a partition key can only efficiently serve one query shape. Picking a bad partition key causes hot partitions, overwhelming one replica set while others idle.

---

## Weak Areas to Watch

None — clean pass, precise distinctions made throughout (leaderless mechanism, LSM+replication interdependence, partition-key reasoning).
