# Topic 022 — Indexing Deep Dive

**Module:** 2 — Data Storage Foundations
**Status:** Completed
**Date:** 2026-07-13
**Confidence:** 4-5/5
**Difficulty:** Medium-Hard

---

## 1. Why This Topic Exists

Fuses Topic 020 (storage engine, pages) and Topic 021 (relational model, query planner) together — indexing is *why* the query planner can be fast at all. Without this, "the query planner decides execution strategy" is an abstract statement; indexing makes it concrete.

---

## 2. Core Concepts

### The core problem — full table scans

Without an index, the database has no shortcut — it reads every page of the table (Topic 020) and checks every row against the filter. **O(n)** — cost grows linearly with table size.

```
1,000 rows      → scan 1,000 rows    → fast enough, unnoticed
10,000,000 rows → scan 10,000,000    → seconds or minutes, unacceptable
```

### What an index is

A separate data structure mapping column values to row locations, kept **sorted**:

```
Table (physical row order):                    Index on email (sorted):
┌────┬───────────────┐                         ┌───────────────┬────────┐
│ id │ email         │                         │ email         │ row_id │
├────┼───────────────┤                         ├───────────────┼────────┤
│ 3  │ zack@x.com    │                         │ akash@x.com   │ 7      │
│ 7  │ akash@x.com   │                         │ bob@x.com     │ 12     │
│ 12 │ bob@x.com     │                         │ zack@x.com    │ 3      │
└────┴───────────────┘                         └───────────────┴────────┘
```

The index is itself stored as **pages on disk** (Topic 020), organized as a **B-Tree**, giving **O(log n)** lookups instead of O(n).

**Why O(log n) specifically — the mechanism:** sorting lets you eliminate half the remaining search space at each comparison (same principle as binary search in an array). A B-Tree extends this across disk pages, not just an in-memory array. Because of this, the tree's **height** stays tiny even as data grows massively — for 10 million rows, height ≈ log₂(10M) ≈ 24. That means only ~24 page reads are needed to find any row, **regardless of table size** — this is why O(log n) barely grows even at huge scale.

```
Full table scan: O(n)         — check every row
B-Tree index lookup: O(log n)  — 10M rows → ~24 page reads, not 10 million
```

### Why B-Trees specifically

Sorted and balanced, supporting both:
- **Exact match** (`WHERE email = 'x'`) — walk down the tree
- **Range queries** (`WHERE age > 25`) — walk to the start of the range, read sequentially

This dual capability (point lookups AND ranges/ordering) is why B-Trees are the default index structure in Postgres/MySQL. (Topic 023 contrasts this with LSM-Trees, optimized differently for write-heavy workloads.)

### Clustered vs Non-Clustered (Secondary) Index

```
Clustered index    → the table's ACTUAL ROW DATA is physically stored in
                      index order (usually the primary key). Only ONE per
                      table — data can only be physically sorted one way.

Non-clustered (secondary) index → a SEPARATE structure pointing to where the
                      real row lives (via row ID or primary key). Many per table.
```

```
Clustered (on id):                    Secondary (on email):
┌────┬───────────────┐                ┌───────────────┬────┐
│ id │ email         │  ← actual      │ email         │ id │  ← points back
│ 1  │ ...           │    row data    │ akash@x.com   │ 7  │    to the row
│ 2  │ ...           │    lives here  │ bob@x.com     │ 12 │    via id
└────┴───────────────┘                └───────────────┴────┘
```

**Key implication — the "bookmark lookup" cost:** a secondary-index query costs **two hops**: (1) find the row's location in the secondary index, (2) fetch the actual row via that location from the clustered index/table. A clustered index lookup costs only **one hop** — the data IS the index, so there's nothing further to fetch.

### Composite indexes and the leftmost-prefix rule

`INDEX ON (last_name, first_name)` — order matters enormously:

```
HELPS:
  WHERE last_name = 'Yadav'                          ✓ uses index
  WHERE last_name = 'Yadav' AND first_name = 'Akash'  ✓ uses index fully

Does NOT help:
  WHERE first_name = 'Akash'                          ✗ not the leftmost column
```

Like a phone book sorted by last name then first name — you can jump straight to "Yadav," but not to "Akash" without scanning every last name first. A composite index only helps queries filtering on a **prefix** of its columns, in order.

### Covering indexes — avoiding the row lookup entirely

If a secondary index contains **every column** a query needs, the database skips the bookmark lookup entirely — an **index-only scan**, the fastest possible query.

```
Query: SELECT email, name FROM users WHERE email = 'x'
Index on (email) INCLUDING (name) → covers the whole query, never touches the table
```

### The cost side — indexes are not free

Every index must update on **every INSERT, UPDATE, or DELETE** to the indexed column(s) — connects directly to Topic 005's read/write tradeoff framework:

```
More indexes → faster reads (the point)
             → slower writes (every index needs updating too)
             → more storage (each index is its own on-disk structure)
```

**When indexes actively hurt:**
- **Write-heavy tables** with many indexes — every write pays the update cost for all of them (Topic 003's read:write ratio directly informs how many indexes are worth it)
- **Low-cardinality columns** — indexing a boolean `is_active` barely helps; roughly half the table matches either value, not selective enough to beat a scan

---

## Tech Decision Box

```
Add an index when:
  - Column is frequently used in WHERE, JOIN, or ORDER BY
  - Column has high cardinality (many distinct values — email, user_id)
  - Read:write ratio favors reads (Topic 003) — read speedup outweighs write cost

Avoid/reconsider an index when:
  - Table is write-heavy and read-rarely (extreme case: 500:1 write:read ratio
    → minimal or zero secondary indexes, just the mandatory clustered/primary key)
  - Column has low cardinality (boolean flags, few distinct values)
  - Already too many indexes on one table (write cost compounds)

Composite index column order → most selective / most frequently filtered
                                 column first, following leftmost-prefix rule
```

**Interview sentence:** "I'll index `user_id` on the orders table since it's high-cardinality and used in every lookup and join. I won't index the `status` column alone since it's low-cardinality — I'd only include it as a secondary column in a composite index if a specific query pattern needs it."

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Thinking indexes are free speedups | Every index costs write time (must update on every write) and storage |
| Adding an index on every column "just in case" | Reserve indexes for high-cardinality, frequently-queried columns; skip on write-heavy, read-rare tables |
| Assuming composite index `(A, B)` helps queries filtering only on B | Leftmost-prefix rule — only helps queries filtering on A, or A+B, not B alone |
| Forgetting the bookmark-lookup cost of secondary indexes | Secondary index = 2 hops (index → row location → actual row); clustered = 1 hop |

---

## Real Interview Questions

1. "Why is a B-Tree the default index structure in relational databases?" (universal)
2. "What's the difference between a clustered and non-clustered index?" (universal)
3. "You have a composite index on (a, b) — does it help a query filtering only on b?" (universal, common trap)
4. "A write-heavy table has 10 indexes — what's wrong with this, and what would you recommend?" (Amazon, operational-maturity-focused rounds)

---

## Revision Questions

1. Why does B-Tree lookup cost stay near-constant (O(log n)) even as table size grows massively?
2. Walk through the steps of a secondary-index query vs. a clustered-index query — why does one cost more?
3. Composite index on (country, city) — which of two queries (filter by city alone vs. country+city) can use it, and why?
4. A table has 50,000 writes/sec and 100 reads/sec. What's your indexing recommendation, and why?
5. What is a covering index, and what specific cost does it eliminate?

---

## Cheat Sheet

```
FULL SCAN vs INDEX
─────────────────────────────────────────────────
Full table scan: O(n)          — check every row
B-Tree index lookup: O(log n)   — height stays tiny even at huge scale
                                   (10M rows → ~24 page reads)

CLUSTERED vs SECONDARY (NON-CLUSTERED)
─────────────────────────────────────────────────
Clustered   → row data physically stored in index order; ONE per table; 1 hop
Secondary   → separate structure pointing to row location; MANY per table; 2 hops
              (the "bookmark lookup" cost)

COMPOSITE INDEX — LEFTMOST PREFIX RULE
─────────────────────────────────────────────────
INDEX (A, B) helps: WHERE A=x | WHERE A=x AND B=y
INDEX (A, B) does NOT help: WHERE B=y alone

COVERING INDEX
─────────────────────────────────────────────────
Index contains ALL columns a query needs → index-only scan,
no bookmark lookup to the actual table at all

THE COST SIDE
─────────────────────────────────────────────────
Every index → slower writes (must update on every INSERT/UPDATE/DELETE)
            → more storage
Add when: high cardinality + frequently queried + read-favoring ratio
Avoid when: write-heavy table, low-cardinality column, already over-indexed
```

---

## Summary

- **Indexes turn O(n) full table scans into O(log n) lookups** via sorted B-Tree structures stored as pages (Topic 020).
- **Clustered index** = the table's physical row order (1 hop); **secondary index** = a separate pointer structure (2 hops — the "bookmark lookup").
- **Composite indexes follow the leftmost-prefix rule** — only help queries filtering on a prefix of the indexed columns, in order.
- **Covering indexes eliminate the bookmark lookup entirely** by containing every column a query needs.
- **Indexes are a read/write tradeoff** (Topic 005): faster reads, slower writes, more storage — the read:write ratio (Topic 003) should drive how aggressively you index.

> **You now can:** explain why B-Tree indexes are fast, distinguish clustered from secondary indexes and their cost difference, apply the leftmost-prefix rule to composite indexes, and make a correctly-reasoned indexing recommendation based on a table's read:write ratio.
