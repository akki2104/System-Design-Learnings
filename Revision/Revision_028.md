# Revision — Topic 028: NoSQL Overview

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-26

---

## Q1. What's wrong with "we're write-heavy, so let's use MongoDB"? Which category actually fits?

<details>
<summary>Answer</summary>

This conflates schema flexibility (MongoDB/Document's actual strength) with write throughput. The category that targets write-heavy, known-access-pattern workloads is Wide-Column (Cassandra, DynamoDB), due to its leaderless write architecture.

</details>

---

## Q2. What's the actual mechanism behind why many NoSQL stores handle massive write throughput better than a typical relational setup?

<details>
<summary>Answer</summary>

Leaderless/multi-master replication — any node can accept a write. A typical relational setup routes all writes through a single primary, which becomes the write bottleneck regardless of how many read replicas exist.

</details>

---

## Q3. What's the difference between schema-on-write and schema-on-read?

<details>
<summary>Answer</summary>

Schema-on-write (relational): structure is enforced the moment data is written. Schema-on-read (document stores): no enforced structure at write time — the application interprets each document's structure when reading, so two documents in the same collection can have completely different fields.

</details>

---

## Q4. Name the four main NoSQL categories and one example database for each.

<details>
<summary>Answer</summary>

Key-Value (Redis, DynamoDB KV mode), Document (MongoDB, Couchbase), Wide-Column/Column-Family (Cassandra, HBase, Bigtable), Graph (Neo4j).

</details>

---

## Q5. Why is "NoSQL is more scalable than SQL" an incomplete/misleading interview answer?

<details>
<summary>Answer</summary>

NoSQL isn't strictly better — it's a different set of tradeoffs. Most NoSQL systems trade away strong consistency and joins specifically to gain availability and write throughput. Naming that explicit cost (not just the benefit) is what separates a strong answer from a shallow one.

</details>

---

## 30-Second Elevator Pitch

> NoSQL is an umbrella term covering four categories — Key-Value (fast lookups), Document (flexible/nested schema), Wide-Column (write-heavy known access patterns), and Graph (relationship traversal). The real reason many NoSQL stores handle massive write throughput is leaderless/multi-master replication — any node accepts writes, unlike a relational setup's single-primary bottleneck. Document stores are schema-on-read (structure interpreted at read time, so records can differ); relational is schema-on-write (structure enforced at write time). The common trap is treating NoSQL as strictly "more scalable" — it's really a tradeoff: giving up strong consistency and joins to gain availability and write throughput, which is a real cost, not a free upgrade.

---

## Weak Areas to Watch

None — clean pass, no corrections needed this lesson.
