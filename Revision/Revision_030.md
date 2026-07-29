# Revision — Topic 030: Document Stores (MongoDB)

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-26

---

## Q1. What's the key structural difference between Document stores and Wide-Column stores?

<details>
<summary>Answer</summary>

Document stores support native nesting — arrays and sub-documents inside a single document. Wide-Column stores stay flat — each column holds one scalar value, no matter how many sparse columns a row has. This makes Document stores a better fit for naturally hierarchical entities (product with variants, nested user preferences).

</details>

---

## Q2. Why can MongoDB add a secondary index on any field, while Wide-Column stores can't efficiently query on anything but the partition key?

<details>
<summary>Answer</summary>

Wide-Column's efficiency depends entirely on the partition key determining physical data location (Topic 029) — querying anything else means scanning every partition. MongoDB's secondary indexes are separate structures built on top of any field, letting it answer queries beyond the primary access pattern without needing a whole separate collection per query shape — at the cost of a lower write-throughput ceiling than a leaderless Wide-Column store.

</details>

---

## Q3. What's the rule of thumb for embedding vs referencing in MongoDB, and what does each cost?

<details>
<summary>Answer</summary>

Embed when child data is always read together with the parent and stays bounded in size (e.g., a user's addresses) — one document fetch gets everything. Reference when child data is large, grows unbounded, or is queried independently (e.g., a user's thousands of orders) — requires a second query (`$lookup`), MongoDB's limited join-like operation.

</details>

---

## Q4. Is MongoDB's replication model leaderless like Cassandra's? Explain.

<details>
<summary>Answer</summary>

No. MongoDB replica sets have one primary (accepts all writes) and several secondaries (replicate from the primary). If the primary fails, secondaries hold an election to promote a new one — but at any moment, exactly one node accepts writes for that replica set. This is a genuine contrast with Cassandra's leaderless model (Topic 029), which is why Cassandra has a higher write-throughput ceiling.

</details>

---

## Q5. What mechanism does MongoDB use for horizontal scale, and what risk does it share with Wide-Column stores?

<details>
<summary>Answer</summary>

Sharding, using a shard key — conceptually the same idea as a Wide-Column partition key. A poorly chosen shard key (low cardinality or skewed access) causes the same hot-shard/hot-partition problem covered in Topic 029.

</details>

---

## 30-Second Elevator Pitch

> Document stores hold BSON documents in collections, with native nesting (arrays, sub-documents) as the key structural difference from Wide-Column's flat columns — a good fit for hierarchical entities. Flexible secondary indexing lets you query by any field, unlike Wide-Column's rigid partition-key-first access, at the cost of a lower write-throughput ceiling. The core modeling decision is embedding (bounded, always-together data) vs referencing (large, unbounded, or independently queried data, requiring a second `$lookup` query). Critically, MongoDB replica sets are leader-follower — one primary, election on failure — NOT leaderless like Cassandra, which is exactly why its write-throughput ceiling is lower. Sharding via a shard key handles horizontal scale, with the same hot-shard risk as a bad Wide-Column partition key.

---

## Weak Areas to Watch

- MongoDB is NOT leaderless — don't generalize Cassandra's leaderless property to all of NoSQL
- Embed vs reference is a real modeling decision with real costs on both sides, not a "just embed everything" default
