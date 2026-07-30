# Topic 031: Choosing a Database

**Module:** 2 — Data Storage Foundations
**Tier:** 🔴 MUST
**Completed:** 2026-07-30
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topics 020–030 gave deep internals of individual database types. This topic is the synthesis skill: given a set of requirements, choose a database and justify it in under two minutes, then defend it under interviewer pushback. This is graded directly on the Tradeoffs dimension of the interview rubric — it closes out Module 2.

---

## 2. The Decision Framework — Questions to Ask (in order)

1. **Access pattern:** known and fixed upfront, or ad-hoc/evolving?
2. **Read:write ratio** (Topic 003): read-heavy, write-heavy, or balanced?
3. **Relationships:** joins across entities needed, or is data naturally siloed?
4. **Consistency requirement:** strong (money, inventory) or eventual (likes, view counts) tolerable?
5. **Schema shape:** flat/relational, nested/hierarchical, or wide/sparse with a fixed access pattern?
6. **Scale ceiling:** will one machine's disk/IOPS ever be insufficient, forcing horizontal partitioning?

---

## 3. The Decision Tree (synthesizing Topics 005, 021, 028, 029, 030)

| If the answers point to... | Choose | Because |
|---|---|---|
| Joins + strong consistency + moderate scale | **Relational** (Postgres/MySQL) | ACID, first-class joins, well-understood stable schema |
| Massive write volume + known access pattern + eventual consistency OK | **Wide-Column** (Cassandra/DynamoDB) | Leaderless replication + LSM-Tree engine (Topic 029) |
| Nested/evolving schema + mostly-known queries + moderate write scale | **Document** (MongoDB) | Flexible schema-on-read + secondary indexing (Topic 030) |
| Simple ultra-fast key lookups (sessions, cache, counters) | **Key-Value** (Redis) | O(1) get/put, no query overhead |
| Relationship traversal IS the primary query | **Graph** (Neo4j) | Optimized for edge-walking, not table scans |

---

## 4. Polyglot Persistence — the Real-World Pattern

Real systems almost never use exactly one database. A strong interview answer names multiple stores, each solving a different sub-problem — not "I'll use X for everything." Example: an e-commerce system might use:
- **Postgres** for orders/inventory (ACID + joins)
- **Redis** for session/cart caching (fast, ephemeral)
- **Elasticsearch** for product search (full-text, Topic 093 preview)
- **S3/blob storage** for product images (Topic 096 preview)

Naming this explicitly — "different data has different needs, so I'll use X for this and Y for that, here's why" — separates a strong candidate from someone reciting one database's feature list.

**A common failure mode even after learning this principle:** defaulting back to one database for multiple different sub-problems under pressure (e.g., using one document store for both content storage AND full-text search) — knowing the principle and applying it live are different skills, and that gap is exactly what practice scenarios are for.

---

## 5. The Interview Justification Format (extends Topic 005's 3-part sentence)

> "For [this specific data/access pattern], I'll use [X] because [specific property]. I considered [Y] but rejected it because [what Y trades away]. This matters because [business impact]."

---

## 6. The Common Trap

Picking a database by familiarity or hype instead of matching it to the actual requirement:
- "I'll use MongoDB because it's flexible" — when the actual requirement is write throughput (Topic 028's Q1 trap).
- "I'll use Cassandra because it scales" — when the actual requirement is complex relational queries with joins.

Fix: name the specific property the system needs before naming a database (the rule since Topic 005).

---

## 7. Worked Example 1 — Ride-Sharing Trip History

Access pattern: "get all trips for a driver" / "get all trips for a rider" — known, fixed. Read:write: moderate reads, high writes (millions of trips/day). Relationships: minimal. Consistency: eventual is fine. Schema: flat, fixed fields.

**Walking the framework:** known+fixed access patterns, write-heavy, eventual consistency OK, flat schema → **Wide-Column**, with one table partitioned by `driver_id` and a second by `rider_id` (Topic 029's query-driven modeling).

---

## 8. Worked Example 2 — Real-Time Collaborative Document Editor (practice scenario)

Requirements: multiple users edit the same document simultaneously; store document content; track active editors; support search across a user's documents by title/content.

**Applying the framework:**
1. Access pattern: content lookup by document ID (fixed) + full-text search across content (not a fixed key-based pattern).
2. Read:write ratio: write-heavy — every keystroke/batched edit group across potentially thousands of concurrent editors, even if each individual write is tiny.
3. Relationships: minimal for content storage itself.
4. Consistency: eventual is acceptable for document content persistence (the real-time merge happens at the application layer, described below).
5. Schema: nested/flexible (document structure, comments, formatting metadata).
6. Scale: horizontal scale needed at consumer-product scale.

**The trap to avoid here:** treating "store content" and "search by title/content" as the same problem solvable by one database. They are two different sub-problems — this is polyglot persistence in practice, not just principle:
- **Document store (MongoDB)** — content/metadata storage; flexible schema fits.
- **Dedicated search engine (Elasticsearch)** — full-text, relevance-ranked search across content at scale; a document store's secondary indexes are not built for this.

**The bigger point the database-choice framework alone doesn't surface:** the hardest part of this specific system is NOT which database to use — it's that concurrent edits from multiple users must be merged without conflicts or lost keystrokes. That's a real-time sync problem (WebSockets, Topic 015) plus a conflict-resolution problem (Operational Transform / CRDTs — Topic 057, tagged SKIP in this track, but worth knowing the concept exists). No database choice solves this; the database just persists the eventually-agreed-upon state after the application/protocol layer resolves conflicts. A strong interview answer flags this explicitly rather than only reasoning about storage.

---

## 9. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Using one database to solve two different sub-problems (content storage + full-text search) | Apply polyglot persistence: a document store for content, a dedicated search engine (Elasticsearch) for full-text search — even when the principle was just learned, the instinct under pressure can default back to "one DB for everything" |
| Reasoning about database choice without flagging the system's actual hardest problem | For systems with a dominant non-storage challenge (e.g., concurrent edit conflict resolution), name that challenge explicitly — database choice is necessary but not sufficient |
| Calling a write pattern "moderate" without accounting for aggregate concurrent load (many small writes × many concurrent users) | Consider total aggregate write volume, not just per-user write frequency |

---

## 10. Revision Questions
See `Revision/Revision_031.md`.

## 11. Summary
- The decision framework: access pattern, read:write ratio, relationships, consistency requirement, schema shape, scale ceiling — walked in order.
- Decision tree maps these answers to Relational / Wide-Column / Document / Key-Value / Graph.
- Polyglot persistence: real systems use multiple databases, each for a different sub-problem — naming this explicitly is what separates strong answers from single-database defaults.
- The interview justification format: name the property, reject an alternative with its specific cost, state the business impact.
- Common trap: picking a DB by familiarity/hype instead of matching the actual requirement.
- Applying the framework under pressure can regress to "one DB for everything" even right after learning polyglot persistence — recognizing that gap is part of the skill.
- Some systems (real-time collaborative editing) have a hardest problem that isn't a database choice at all — flag it explicitly.
