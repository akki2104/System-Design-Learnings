# Topic 030: Document Stores (MongoDB)

**Module:** 2 — Data Storage Foundations
**Tier:** 🟡 SKIM (per TopicPriority.md — shapes live-session depth only, not documentation depth)
**Completed:** 2026-07-26
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topic 028 introduced Document stores at the survey level (schema-on-read, flexible per-document structure). This topic goes one level deeper on what specifically distinguishes Document stores from both Relational (Topic 021) and Wide-Column (Topic 029) — nesting and flexible secondary indexing — plus a real modeling decision (embedding vs referencing) and MongoDB's actual replication model, which is notably different from Cassandra's.

---

## 2. The Data Model

MongoDB stores **documents** (BSON — a binary form of JSON) inside **collections** (the rough analog of a table, but with no enforced schema across documents). A single document can represent an entire entity, including everything nested inside it:

```json
{
  "user_id": 7,
  "name": "Akash",
  "addresses": [
    {"type": "home", "city": "Pune"},
    {"type": "work", "city": "Bangalore"}
  ],
  "preferences": {
    "notifications": {"email": true, "sms": false}
  }
}
```

Arrays and nested sub-documents are native — this is the first real differentiator from Wide-Column stores, whose "columns" remain flat key-value pairs no matter how many of them a row has.

---

## 3. Differentiator 1 — Nesting vs Flat Columns

Wide-Column (Topic 029) is flat: a row can have thousands of sparse columns, but each column holds one scalar value — no native arrays or sub-objects. Document stores fit naturally hierarchical entities (a product with variants, a user profile with nested settings, an order with embedded line items) that would otherwise require multiple join-heavy relational tables, or multiple denormalized Wide-Column tables just to represent one logical object.

---

## 4. Differentiator 2 — Flexible Secondary Indexing

Wide-Column stores force query-driven modeling: one denormalized table per access pattern, because only the partition key can be efficiently queried (Topic 029, Section 5). MongoDB lets you add a **secondary index on any field** and query by it with reasonable efficiency — no need to pre-build a whole separate collection for every new query shape.

**The tradeoff this buys:** better fit when query patterns are only *mostly* known upfront (not rigidly fixed), at the cost of a lower write-throughput ceiling than a leaderless Wide-Column store — because MongoDB's replication model isn't leaderless (see Section 6).

---

## 5. Embedding vs Referencing — the Real MongoDB Modeling Decision

This is the document-store equivalent of Topic 029's query-driven modeling question, but the tool is different: instead of choosing a partition key, you choose whether related data lives **inside** the parent document (embedding) or in a **separate document referenced by ID** (referencing, MongoDB's rough equivalent of a foreign key, but not enforced or joinable the way SQL foreign keys are).

| | Embed | Reference |
|---|---|---|
| Use when | Child data is always read together with the parent, and doesn't grow unbounded (e.g., a user's addresses) | Child data is large, grows unbounded, or is shared/queried independently (e.g., a user's thousands of orders) |
| Read cost | One document fetch gets everything | Requires a second query (`$lookup`, MongoDB's limited join-like operation) |
| Write cost | Updating deeply nested data can be awkward | Independent documents update independently |

**The rule of thumb:** embed for "always accessed together, bounded size"; reference for "large, unbounded, or independently queried."

---

## 6. MongoDB's Replication Model — a Real Contrast with Cassandra

This is a distinction worth being precise about, since it's easy to blur "NoSQL = leaderless" as a blanket rule after Topic 029: **MongoDB replica sets are NOT leaderless.** A replica set has one **primary** (accepts all writes) and several **secondaries** (replicate from the primary, serve reads if configured to). If the primary fails, the secondaries hold an election to promote a new primary — but at any given moment, there is exactly one node accepting writes for that replica set.

This is a real, meaningful contrast: Cassandra's leaderless replication (Topic 029) is what gives it its very high write-throughput ceiling; MongoDB's single-primary-per-replica-set model caps write throughput lower, in exchange for the simpler consistency model and richer query flexibility described above. **Sharding** (splitting data across multiple replica sets via a shard key) is MongoDB's mechanism for horizontal scale — conceptually similar to Cassandra's partition key, with the same hot-shard risk from a poorly chosen key (Topic 029, Section 6 / preview of Topic 043).

---

## 7. Decision Box — Relational vs Document vs Wide-Column

| | Relational | Document | Wide-Column |
|---|---|---|---|
| Structure | Flat, normalized, fixed schema | Nested, flexible schema | Flat, sparse, fixed access pattern |
| Query flexibility | High (joins) | Medium (secondary indexes, limited `$lookup`) | Low (partition-key-first only) |
| Write throughput ceiling | Lowest (single primary, whole DB) | Medium (single primary per shard) | Highest (leaderless per partition) |
| Replication model | Leader-follower | Leader-follower (per replica set) | Leaderless |
| Best for | Relationships, strong consistency | Nested/evolving entities, mostly-known queries | Write-heavy, rigidly fixed access patterns |

---

## 8. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Assuming all NoSQL stores are leaderless like Cassandra | MongoDB replica sets use a single primary per shard — not leaderless. Leaderless replication is specifically a Wide-Column (Cassandra/DynamoDB) property, not a universal NoSQL property |
| Always embedding related data "because it's a document store" | Embed only for bounded, always-read-together data; reference (and accept a second query) for large/unbounded/independently-queried data |

---

## 9. Revision Questions
See `Revision/Revision_030.md`.

## 10. Summary
- Document stores hold BSON documents in collections; nesting (arrays, sub-documents) is native — the key structural difference from Wide-Column's flat columns.
- Flexible secondary indexing lets you query by any field reasonably efficiently, unlike Wide-Column's rigid partition-key-first access — the tradeoff is a lower write-throughput ceiling.
- Embedding vs referencing is the core MongoDB modeling decision: embed for bounded, always-together data; reference for large/unbounded/independently-queried data.
- MongoDB replica sets are leader-follower (one primary, election on failure), NOT leaderless — this is a genuine, important contrast with Cassandra's leaderless model from Topic 029, not just a smaller-scale version of the same thing.
- Sharding uses a shard key (same concept as a partition key) — same hot-shard risk from a bad key choice.
