# Topic 028: NoSQL Overview

**Module:** 2 — Data Storage Foundations
**Tier:** 🔴 MUST
**Completed:** 2026-07-26
**Confidence:** 4-5/5

---

## 1. Why This Topic Exists

Topics 021–027 went deep on relational databases. This topic surveys the broader NoSQL landscape as a whole, before Topic 029 goes deep on Wide-Column stores specifically and Topic 031 gives the actual decision framework. "NoSQL" isn't one technology — it's an umbrella term for anything that departs from the traditional relational model, built to solve problems relational DBs struggle with: massive write throughput, flexible/evolving schemas, and access-pattern-specific optimization.

---

## 2. The Four Main NoSQL Categories

| Category | Examples | Model | Best for |
|---|---|---|---|
| **Key-Value** | Redis, DynamoDB (KV mode) | Simple `get(key)`/`put(key, value)`, no query language | Extremely fast lookups by a known key — sessions, caches, feature flags |
| **Document** | MongoDB, Couchbase | Semi-structured documents (JSON-like), flexible per-document schema | Nested/hierarchical data with an evolving shape — user profiles, product catalogs |
| **Wide-Column / Column-Family** | Cassandra, HBase, Bigtable, DynamoDB | Rows can have different columns; optimized for massive writes with a known access pattern | Write-heavy workloads with predictable query patterns — time-series, messaging, IoT (full depth in Topic 029) |
| **Graph** | Neo4j | Nodes + edges | Traversing relationships — social graphs, recommendation engines |

---

## 3. The Core Tradeoff vs Relational (extends Topics 005 & 021)

| | Relational | NoSQL |
|---|---|---|
| Schema | Fixed, enforced at write time | Flexible, often per-document |
| Consistency | Strong (ACID) | Often eventual (tradeoff for availability — preview of Topic 046, CAP Theorem) |
| Joins | First-class, efficient on one machine | Usually avoided by design — denormalize instead |
| Scaling | Vertical + limited horizontal (sharding bolted on) | Built for horizontal scaling from the ground up |

---

## 4. Why NoSQL Scales Writes Better — the Actual Mechanism

Not "the data model is different" — the real mechanism is **replication architecture**. Many NoSQL stores (Cassandra, DynamoDB) use **leaderless/multi-master replication**: any node can accept a write. A typical relational setup instead routes all writes through a single primary, which becomes the write bottleneck no matter how many read replicas exist. This connects directly to Topic 005's original "leaderless replication" mention (full depth in Topic 039 — Replication).

---

## 5. Schema-on-Write vs Schema-on-Read

A precise technical restatement of Topic 005's "flexible schema" idea:
- **Schema-on-write** (relational) — schema is enforced the moment data is written; every row must conform.
- **Schema-on-read** (document stores) — no enforced structure at write time; the application interprets each document's structure when it reads it. Two documents in the same MongoDB collection can have completely different fields.

---

## 6. Common Trap: "Write-Heavy → Use MongoDB"

This is a category error. MongoDB (Document) targets flexible/nested/unknown schema shape — not write throughput. The category that actually targets write-heavy, known-access-pattern workloads is **Wide-Column** (Cassandra, DynamoDB), because of its leaderless write architecture. Naming the *specific property* a system needs (write throughput vs. schema flexibility) before naming a database is the discipline this program has been building since Topic 005.

---

## 7. When to Use What (ties 005 + 021 + 028 together)

| Use relational when | Use NoSQL when |
|---|---|
| Relationships/joins matter | Access pattern is known and write-heavy (Wide-Column) |
| Strong consistency is required (money, inventory) | Schema is evolving/nested and unknown upfront (Document) |
| Schema is well-understood and stable | Simple, extremely fast key-based lookups are the whole workload (Key-Value) |
| | Relationship traversal IS the primary query (Graph) |

---

## 8. The Trap to Avoid

NoSQL is not "strictly better" or "more scalable" in every dimension — it's a different set of tradeoffs, not an upgrade. Most NoSQL systems trade away strong consistency and joins specifically to gain availability and write throughput. That's a real cost paid, not a free lunch — naming that cost explicitly separates a strong interview answer from a shallow "NoSQL scales better" one-liner.

---

## 9. Common Mistakes
None logged — clean pass across all three checkpoint questions.

## 10. Revision Questions
See `Revision/Revision_028.md`.

## 11. Summary
- NoSQL is an umbrella term for four main categories: Key-Value, Document, Wide-Column, Graph — each solves a different specific problem.
- The real mechanism behind NoSQL's write-throughput advantage is leaderless/multi-master replication, not the data model itself.
- Schema-on-write (relational) enforces structure at write time; schema-on-read (document stores) defers structure interpretation to read time.
- "Write-heavy → MongoDB" is a category error — Wide-Column stores (Cassandra/DynamoDB) are the actual answer for write-heavy known-access-pattern workloads.
- NoSQL trades away strong consistency and joins for availability and write throughput — a real, explicit cost, not a free upgrade over relational.
