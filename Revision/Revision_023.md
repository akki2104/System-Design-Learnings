# Revision — Topic 023: B-Trees vs LSM-Trees

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "B-Trees optimize for reads — one sorted structure, O(log n) lookups — but every write requires an in-place random disk write to insert at the correct sorted position. LSM-Trees flip this: writes go to a WAL and an in-memory memtable, then flush as sequential, sorted, immutable SSTables — no random writes ever. The cost moves to reads: a key might be in the memtable, the newest SSTable, or an older one, so a read may check several sorted structures (SSTables ARE sorted — binary search works within each one; the cost is checking multiple of them). Compaction merges SSTables to bound this cost and remove stale data. Neither is strictly better — it's a tradeoff based on read:write ratio, same decision tree as Topic 005."

---

## Active-Recall Q&A

<details>
<summary>Q1: Why does a B-Tree write require a random disk write, while an LSM-Tree write never does?</summary>

A B-Tree write must insert the key at its correct sorted position in the tree — that position could be anywhere in the tree's physical layout, so it's a random write. An LSM-Tree write is always a sequential append (to the WAL, then to the in-memory memtable). Sorting is deferred: the memtable stays sorted cheaply in memory, and when flushed, it's written out as one new sequential file, already sorted. No searching "where does this go among existing disk files" ever happens at write time.

</details>

<details>
<summary>Q2: Walk through what a read has to do in an LSM-Tree that it doesn't have to do in a B-Tree.</summary>

Check the memtable first (fast, in-memory). Then check the newest SSTable — binary search WITHIN it, since SSTables are sorted. If not found, check the next-older SSTable, and so on, until the key is found or all SSTables are exhausted. A B-Tree read only ever checks ONE structure. Cost is roughly O(k × log n), k = number of SSTables checked.

</details>

<details>
<summary>Q3: Why can't a read just check the newest SSTable and stop?</summary>

The key you're looking for might have been written BEFORE the newest SSTable was created — it could live in an older SSTable that hasn't been touched since its flush. The newest SSTable only contains what was flushed most recently, not everything ever written.

</details>

<details>
<summary>Q4: What does compaction do, and why is it necessary?</summary>

Compaction merges multiple SSTables into fewer, larger ones, discarding obsolete/overwritten/deleted entries in the process. Without it, SSTables would pile up indefinitely, making every read check more and more files — compaction bounds this read-side degradation and reclaims space from stale data.

</details>

<details>
<summary>Q5: Why is "LSM-Trees are strictly better because writes are faster" a flawed statement?</summary>

It ignores the read-side cost. LSM-Trees trade write speed for read complexity (checking multiple SSTables). The correct framing is a tradeoff based on read:write ratio — LSM-Trees win for write-heavy workloads (Cassandra, RocksDB), B-Trees win for read-heavy or balanced workloads (Postgres, MySQL). Neither is "strictly better" in general.

</details>

---

## Key Diagram

```
B-TREE vs LSM-TREE
─────────────────────────────────────────
B-Tree:    in-place random writes (slow)  | ONE sorted structure (fast reads)
LSM-Tree:  sequential appends only (fast) | MULTIPLE sorted structures (slower reads)

LSM WRITE PATH
─────────────────────────────────────────
Write → WAL + memtable (sorted, in-memory)
Memtable full → flush as new immutable SSTable (sorted, sequential)

LSM READ PATH
─────────────────────────────────────────
memtable → newest SSTable (binary search) → older SSTable → ... 
(key may be in an OLDER file — can't stop at just the newest)
```

---

## My Weak Areas (from lesson 2026-07-20)

- **Core misconception:** believed SSTables were unsorted, so LSM reads couldn't use binary search — corrected: SSTables ARE sorted (the "S" in the name); the cost is checking MULTIPLE sorted structures, not losing sorting within one
- Initial complexity claim was imprecise ("O(n)" instead of "O(k × log n)")
- Q1 initially only explained the B-Tree side, not why LSM specifically avoids random writes (deferred sorting via memtable)

---

## Past Mistakes

See [InterviewMistakes.md](../InterviewMistakes.md) — entries dated 2026-07-20.
