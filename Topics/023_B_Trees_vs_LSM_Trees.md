# Topic 023 — B-Trees vs LSM-Trees

**Module:** 2 — Data Storage Foundations
**Status:** Completed
**Date:** 2026-07-20
**Confidence:** 3/5
**Difficulty:** Medium (Survey)

---

## 1. Why This Topic Exists

Direct continuation of Topic 022 — contrasts B-Trees (read-optimized, covered in depth already) with LSM-Trees, the write-optimized alternative used by Cassandra, RocksDB, LevelDB, and HBase.

---

## 2. Core Concepts

### The core problem B-Trees have

B-Trees are great for reads (O(log n), Topic 022), but every write requires finding the exact sorted position in the tree and modifying a page **in place** — a **random write** to disk. Inserting a new key could land anywhere in the tree's physical layout, and random writes are slow (Topic 020). At very high write volumes, this becomes the bottleneck.

**LSM-Trees (Log-Structured Merge-Trees)** trade this away: **optimize for writes by never doing random writes at all.**

### How an LSM-Tree handles writes

```
Write → append to WAL (Topic 020, same durability mechanism)
      → also written to an in-memory sorted buffer: the MEMTABLE

When the memtable fills up:
      → flush the ENTIRE memtable to disk as one new file:
        an SSTable (Sorted String Table) — sorted, and IMMUTABLE
        (never modified again once written)
```

**Every write becomes a sequential append** — to the WAL, and eventually to a new SSTable file. No random disk writes, ever. The sorting itself is deferred: the memtable keeps things sorted cheaply in memory, and when flushed, it's written out as one new sequential file — already sorted, since the memtable was sorted. No searching "where does this go among existing disk files" ever happens at write time — that's exactly why it stays sequential.

### The cost: reads get harder

**Important correction:** SSTables ARE individually sorted (that's what the "S" in SSTable means) — binary search works fine *within* one SSTable. The actual cost is that data is spread across **multiple separate sorted structures** (the memtable plus several SSTables written at different times), so a read may need to check several of them, not just one.

```
1. Check the memtable (in-memory, fast)
2. Check the newest SSTable (binary search within it — sorted)
3. Key not found? Check the next SSTable (binary search within it too)
4. ...potentially check several SSTables, until the key is found
```

**Why can't you just check the newest SSTable and stop?** Because the key you're looking for might have been written *before* the newest SSTable was created — it may live in an older SSTable that hasn't been touched since. The newest SSTable only contains what was flushed most recently.

**Complexity:** roughly `O(k × log n)` where k = number of SSTables checked — multiplying binary search across files, not losing binary search entirely. This is the direct tradeoff mirror of Topic 022's clustered-vs-secondary cost: **LSM-Trees pay at read time what B-Trees pay at write time.**

### Compaction — controlling the read-side cost

Over time, SSTables pile up (more files = more to check on read) and old/deleted/updated values sit around wastefully across multiple files. **Compaction** periodically merges multiple SSTables into fewer, larger ones — discarding obsolete/overwritten/deleted entries in the process.

```
SSTable1 + SSTable2 + SSTable3 → compaction → one merged SSTable
                                   (fewer sorted files to search on read,
                                    stale/duplicate data cleaned up)
```

This is a background cost (disk I/O + CPU), but it's what keeps read performance from degrading indefinitely as more writes accumulate.

### Bloom filters — the practical fix for "many SSTables to check"

Real LSM systems avoid checking every SSTable on every read using a **Bloom filter** — a compact, probabilistic structure per SSTable that can quickly answer "is this key **definitely not** in this file?" This lets reads skip SSTables that certainly don't contain the key, without an actual disk read. (Full topic at 089 — for now, know it exists and why.)

---

## Tech Decision Box: B-Tree vs LSM-Tree

```
B-Tree (Postgres, MySQL/InnoDB)   → read-heavy or balanced workloads, need
                                     fast point lookups and range queries,
                                     moderate write volume

LSM-Tree (Cassandra, RocksDB,     → write-heavy workloads, need to absorb
          LevelDB, HBase)           massive write throughput, can tolerate
                                     slightly higher read latency and
                                     background compaction overhead
```

**Connects directly to Topic 005's decision tree:** write pressure → LSM-Tree (Cassandra); read pressure or balanced → B-Tree (Postgres). Same reasoning, now with the internal mechanism to back it up.

**Interview sentence:** "I'll use an LSM-Tree-backed store here because writes need to scale past what random-write B-Tree updates can sustain. I considered Postgres, but rejected it because every write would require an in-place random disk write, and at this volume that becomes the bottleneck — LSM-Trees convert all writes to sequential appends instead."

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Thinking SSTables are unsorted, so LSM reads can't binary search | SSTables ARE sorted (the "S" stands for it) — binary search works fine within one; the cost is checking MULTIPLE sorted structures, not losing sorting |
| Claiming "LSM-Trees are strictly better because writes are faster" | It's a tradeoff based on read:write ratio — LSM wins for write-heavy workloads, B-Tree wins for read-heavy/balanced ones |
| Forgetting why you can't just check the newest SSTable | The key might have been written before the newest SSTable existed — it could be in an older one |
| Describing B-Tree writes without explaining WHY LSM avoids the same cost | LSM writes are always sequential appends (WAL + memtable); sorting is deferred, never done "in place" against existing disk files |

---

## Real Interview Questions

1. "Why do Cassandra and similar databases use LSM-Trees instead of B-Trees?" (Discord, Netflix, write-heavy systems)
2. "Why does an LSM-Tree read need to check multiple files?" (universal)
3. "What is compaction, and what would happen without it?" (universal)
4. "Is an LSM-Tree strictly better than a B-Tree? Why or why not?" (trap question, universal)

---

## Revision Questions

1. Why does a B-Tree write require a random disk write, while an LSM-Tree write never does?
2. Walk through what a read has to do in an LSM-Tree that it doesn't have to do in a B-Tree.
3. Why can't a read just check the newest SSTable and stop?
4. What does compaction do, and why is it necessary?
5. Why is "LSM-Trees are strictly better because writes are faster" a flawed statement?

---

## Cheat Sheet

```
B-TREE vs LSM-TREE
─────────────────────────────────────────────────
B-Tree:    in-place random writes (slow writes), ONE sorted structure (fast reads)
LSM-Tree:  sequential appends only (fast writes), MULTIPLE sorted structures (slower reads)

LSM WRITE PATH
─────────────────────────────────────────────────
Write → WAL (append) + memtable (in-memory, sorted)
Memtable full → flush as new immutable SSTable (sorted, sequential write)

LSM READ PATH
─────────────────────────────────────────────────
Check memtable → check newest SSTable (binary search WITHIN it, it IS sorted)
→ not found → check next SSTable → ... (key may be in an OLDER SSTable)
Cost: O(k × log n) — k = number of SSTables checked, not "no sorting"

COMPACTION
─────────────────────────────────────────────────
Merges multiple SSTables → fewer files to check + removes stale/duplicate/
deleted entries. Background cost that bounds read-side degradation.

BLOOM FILTERS (Topic 089 preview)
─────────────────────────────────────────────────
Per-SSTable structure: quickly says "definitely NOT here" → skip that
SSTable without a disk read

DECISION (extends Topic 005)
─────────────────────────────────────────────────
Write-heavy → LSM-Tree (Cassandra, RocksDB, LevelDB, HBase)
Read-heavy/balanced → B-Tree (Postgres, MySQL/InnoDB)
NOT strictly better either way — tradeoff based on read:write ratio
```

---

## Summary

- **B-Trees optimize reads** (one sorted structure, O(log n)) **at the cost of random writes.**
- **LSM-Trees optimize writes** (sequential appends only, via WAL + memtable + immutable SSTables) **at the cost of reads checking multiple sorted structures.**
- **SSTables ARE sorted** — binary search works within each one; the LSM cost is checking several of them, not losing sorting.
- **Compaction merges SSTables**, reducing file count and removing stale data, bounding read-side degradation over time.
- **Neither structure is strictly better** — the choice depends on the read:write ratio (Topic 005's decision tree), applied here with the internal mechanism to justify it.

> **You now can:** explain why LSM-Trees trade read speed for write speed, correctly describe the LSM read/write paths including compaction, and justify choosing between B-Tree and LSM-Tree databases based on workload shape.
