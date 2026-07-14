# Topic 020: Storage Engine Fundamentals

**Module:** 2 — Data Storage Foundations
**Completed:** 2026-07-13
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Modules 0–1 covered getting a request to a server. Module 2 begins asking: once a server needs to persist or retrieve data, what happens underneath any database (SQL or NoSQL)? Every database sits on a storage engine — the layer that writes bytes to disk and reads them back efficiently. This is the foundation for reasoning about DB internals in a Deep Dive instead of treating "the database" as a black box.

---

## 2. RAM vs Disk — The Fundamental Constraint

RAM is fast but volatile (wiped on crash/power loss). Disk is persistent but much slower:
- RAM access: ~100 nanoseconds
- SSD random read: ~100–150 microseconds (~1,000× slower than RAM)
- HDD seek: ~10 milliseconds (~100,000× slower than RAM)

Everything in storage engine design follows from this one gap: **minimize disk seeks, maximize sequential I/O.**

---

## 3. Pages — The Unit of Disk I/O

Disks read/write in fixed-size blocks called **pages** (commonly 4KB, 8KB, or 16KB). Even a query needing 10 bytes pulls in the entire containing page.

This is why random access is expensive and sequential access is cheap: scattered reads across a file mean many separate page fetches (many seeks); sequential reads reuse physically adjacent pages, enabling prefetching. Every table and index is organized as a structure of pages on disk (concrete in Topic 022's B-Trees).

---

## 4. The Buffer Pool (Page Cache)

Databases cache recently-used pages in RAM — the **buffer pool** (Postgres/MySQL terminology). Reads check the buffer pool first, falling back to disk on a miss. Writes are applied to the in-memory copy first (marking the page "dirty") rather than flushed to disk immediately — flushing on every write would mean a random disk write per transaction.

**The durability problem this creates:** if the server crashes before a dirty page is flushed, that write is lost — even if the client already got a success response.

---

## 5. WAL — Write-Ahead Log

The mechanism that solves durability without forcing a slow random disk write on every transaction.

**Mechanism:** before modifying the in-memory data page, the DB first appends a log record describing the change to a separate append-only file — the **WAL** — and fsyncs *that* append immediately. The actual data page stays dirty in memory and gets flushed to disk later, in the background (**checkpointing**).

**Why it works:** WAL writes are sequential I/O (fast), regardless of how random the underlying data changes are. The random data-page writes get deferred and batched — if a crash happens before they're flushed, the DB replays the WAL on restart to reconstruct unflushed changes.

**Write path:**
```
Client write → WAL entry appended + fsynced (sequential, fast) →
  client gets "success" (durability guaranteed by the WAL)
→ data page updated in memory (dirty)
→ [later, background] checkpoint flushes dirty pages to disk
```

**Read path:**
```
Client read → check buffer pool → hit: return from memory
                                 → miss: read page from disk into buffer pool → return
```

---

## 6. Why the DB Still Flushes Data Pages to Disk (Not Just WAL Forever)

If the DB relied purely on WAL replay with no data-page flushing, reconstructing "current state" would require replaying the **entire history of every write ever made**, from the beginning of time — potentially billions of entries after a year of operation. That's not viable.

**Data pages flushed to disk = a materialized checkpoint of state.** Checkpointing lets the DB discard old WAL entries — crash recovery only needs to replay the log **from the last checkpoint onward**, not from the beginning.

- **WAL** → durability + bounded recovery time
- **Data pages on disk (via checkpointing)** → fast reads without replay, and a floor under how much WAL history must be retained
- **Buffer pool (RAM)** → fast reads for recently-used pages — a separate mechanism from checkpointing, easy to conflate

---

## 7. Why This Matters Beyond Durability

- This is what makes the **D in ACID** (Topic 024) work efficiently — durability without a random-write penalty per transaction.
- The same WAL is **streamed to replicas** in replication (Topic 039 preview) — a replica just applies the same log of changes the primary applied.
- The core insight — "sequential writes are fast, random writes are slow, so defer/batch the random part" — is the same idea behind **log-structured storage / LSM-Trees** (Topic 023 preview) and systems like Kafka.

---

## 8. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Believing the DB could rely on WAL replay forever instead of ever flushing data pages | An unbounded WAL would require replaying the entire write history to reconstruct state — checkpointing bounds recovery time by materializing current state periodically |
| Conflating the buffer pool (RAM cache for fast reads) with checkpointing (disk flush that bounds WAL size) | They're separate mechanisms solving separate problems — one is about read speed, the other about recovery time |

---

## 9. Revision Questions
See `Revision/Revision_020.md`.

## 10. Summary
- Disk I/O (not CPU) is usually the bottleneck; storage engines are designed to minimize seeks and maximize sequential I/O.
- Pages are the fixed-size unit of disk I/O — even tiny queries pull a full page.
- The buffer pool caches pages in RAM for fast reads; writes are applied to dirty in-memory pages first.
- WAL guarantees durability cheaply: sequential log append + fsync, with data-page writes deferred and batched.
- Data pages are still flushed to disk (checkpointing) so recovery only replays the log since the last checkpoint, not the entire history.
- This same principle (defer/batch random writes behind a sequential log) underlies LSM-Trees, replication via WAL shipping, and Kafka.
