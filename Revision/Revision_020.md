# Revision — Topic 020: Storage Engine Fundamentals

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-14

---

## Q1. What's the fundamental constraint that drives storage engine design?

<details>
<summary>Answer</summary>

The huge speed gap between RAM (~100ns) and disk (SSD random read ~100-150μs, ~1000× slower; HDD seek ~10ms, ~100,000× slower). Storage engines are designed to minimize disk seeks and maximize sequential I/O.

</details>

---

## Q2. Why does a query needing only 20 bytes still cause a full page read from disk?

<details>
<summary>Answer</summary>

Disks read/write in fixed-size blocks called pages (e.g., 4KB/8KB/16KB) — not individual bytes. Even a tiny query pulls in the entire containing page.

</details>

---

## Q3. How does WAL provide durability without forcing a slow random disk write on every transaction?

<details>
<summary>Answer</summary>

Before modifying the in-memory data page, the DB appends a log record to a separate append-only WAL file and fsyncs that append immediately — a fast, sequential I/O operation. The actual (random) data-page write is deferred and flushed later in the background (checkpointing). If a crash happens before the data page is flushed, the DB replays the WAL to reconstruct it.

</details>

---

## Q4. Why doesn't the database just rely on WAL replay forever instead of ever flushing data pages to disk?

<details>
<summary>Answer</summary>

An ever-growing WAL would mean reconstructing current state requires replaying the entire write history from the beginning of time — potentially billions of entries after a year, far too slow. Flushing data pages (checkpointing) creates a materialized snapshot of current state, so recovery only needs to replay the log from the last checkpoint onward, bounding recovery time.

</details>

---

## Q5. What's the difference in data-loss outcome between a crash before vs after a WAL entry is fsynced?

<details>
<summary>Answer</summary>

Crash before fsync: the write is lost — nothing durable was recorded. Crash after fsync (but before the data page is flushed): no data is lost — the DB replays the fsynced WAL entry on restart to reconstruct the change.

</details>

---

## 30-Second Elevator Pitch

> Storage engines are built around one constraint: disk is orders of magnitude slower than RAM, so minimize seeks and maximize sequential I/O. Disks read/write in fixed-size pages, not individual bytes — even tiny queries pull a full page. The buffer pool caches pages in RAM for fast reads; writes hit the in-memory page first (dirty), not disk. Durability comes from the Write-Ahead Log: append a change record to a sequential log and fsync that, instead of syncing the random data-page write immediately — fast, and crash-safe via replay. Data pages are still flushed to disk periodically (checkpointing) so recovery only replays the log since the last checkpoint, not the full history. This "sequential log defers random writes" pattern reappears in LSM-Trees, replication (WAL shipping), and systems like Kafka.

---

## Weak Areas to Watch

- Buffer pool (RAM, read speed) and checkpointing (disk flush, bounds WAL replay time) are separate mechanisms — don't conflate them
- WAL replay is bounded by checkpoints, not infinite
