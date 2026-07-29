# Topic 030: Document Stores (MongoDB)

**Module:** 2 — Data Storage Foundations
**Tier:** 🟡 SKIM (condensed — folds into Topic 028)
**Completed:** 2026-07-26

---

## Condensed Notes

Builds on Topic 028's schema-on-read tradeoff. Two things specific to Document stores:

**1. Nesting.** Documents support deep nested structure (arrays, sub-documents) — Wide-Column's columns stay flat key-value pairs. Document stores fit naturally hierarchical entities (product with variants, user profile with nested preferences) that would otherwise need multiple join-heavy relational tables or multiple denormalized Wide-Column tables.

**2. Flexible secondary indexing.** Unlike Wide-Column's rigid one-table-per-access-pattern model (Topic 029), MongoDB lets you add a secondary index on any field and query it reasonably efficiently — no need to pre-build a separate table per query shape. Better fit when query patterns are only *mostly* known upfront, at the cost of lower write-throughput ceiling than a leaderless Wide-Column store.

**Sharding** still uses a shard key (same concept as Wide-Column's partition key) — same hot-shard risk from a bad key choice.

**Quick comparison:**

| | Relational | Document | Wide-Column |
|---|---|---|---|
| Structure | Flat, normalized, fixed schema | Nested, flexible schema | Flat, sparse, fixed access pattern |
| Query flexibility | High (joins) | Medium (secondary indexes) | Low (partition-key-first only) |
| Write throughput ceiling | Lowest (single primary) | Medium | Highest (leaderless) |

No revision file for this skim-tier topic — see CheatSheets.md entry.
