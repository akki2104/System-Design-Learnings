# Revision — Topic 005: How to Reason About Tradeoffs

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "Every tech choice trades something for something — never present one option as the only answer. The core routing question when a scary number appears: is it read pressure or write pressure? Read pressure gets fixed with a cache — cheap, reversible, same database. Write pressure needs a database switch or shard — expensive, hard to reverse. A cache never fixes writes because a write has no fallback: it must land durably in the source of truth. Joins get expensive from sharding data across machines, not from data volume. Always state tech decisions as: 'I'll use X because Y. I considered Z but rejected it because W.'"

---

## Active-Recall Q&A

<details>
<summary>Q1: What's the single question that routes you to "add a cache" vs "switch the database"?</summary>

**"Is my problem READ pressure or WRITE pressure?"**

Read pressure → add a cache (cheap, reversible, same DB).
Write pressure → switch/shard the database (expensive, hard to reverse).

</details>

<details>
<summary>Q2: Why does a cache fix read pressure but never write pressure?</summary>

A read cache miss just costs time — fall back to the database and get the correct answer, no data lost.

A write has no such fallback. It must land durably in the one source of truth. If a write-back cache crashes before flushing, that data is gone forever. So every write must still reach the database — caching can never reduce write volume, only read volume.

</details>

<details>
<summary>Q3: What's the REAL trigger for expensive joins — data size or something else?</summary>

**Sharding (spreading data across machines), not data volume.**

A single machine joins tables fine even at hundreds of GB with good indexes — it's a local, in-memory operation. Once data is split across machines, a join requires a network call to another machine for every row — that's the real cost. This is why NoSQL documents "duplicate" data (denormalization) — to avoid cross-shard network joins.

</details>

<details>
<summary>Q4: Distinguish MongoDB's strength from Cassandra's strength.</summary>

**MongoDB:** flexible/nested schema — for data whose SHAPE is unknown or varies (product catalogs with different attributes per category).

**Cassandra/DynamoDB:** massive write throughput with a KNOWN access pattern — leaderless writes, any node accepts a write. For messaging/feeds where you know exactly how data will be queried (by conversation_id, ordered by time).

These are NOT the same "NoSQL flexibility" — conflating them is a common mistake. MongoDB shards still route writes through a per-shard primary (a write bottleneck); Cassandra doesn't.

</details>

<details>
<summary>Q5: State the 3-part interview sentence format for justifying a tech choice.</summary>

"I'll use [X] because [specific problem property]. I considered [Y] but rejected it because [what Y trades away]."

</details>

<details>
<summary>Q6: WhatsApp needs 500K writes/sec with a known access pattern. Which DB, and why not the alternative?</summary>

**Cassandra** — because writes need to scale past what one primary can take, and the access pattern is known and fixed (fetch messages by conversation_id, ordered by time). Cassandra's leaderless model lets any node accept writes.

**Rejected: MongoDB** — its shards still route writes through a per-shard primary, capping write throughput the same way sharded SQL would. MongoDB's strength (flexible schema) isn't the property that matters here — write throughput is.

</details>

---

## Key Diagram

```
THE TRADEOFF DECISION TREE
─────────────────────────────────────────
        Scary number
             │
      READ or WRITE?
      /            \
  READ            WRITE
    │                │
  CACHE          SHARD/SWITCH DB
 (cheap,          (expensive,
  reversible)      hard to reverse)
```

---

## My Weak Areas (from lesson 2026-07-08)

- **Initially conflated "flexible schema" (MongoDB) with "high write throughput" (Cassandra)** — these are different axes; a system can need one, both, or neither
- **Initial join-overhead reasoning attributed cost to data VOLUME instead of data DISTRIBUTION (sharding)** — corrected during lesson
- **Q1 (cache-crash reasoning) needed sharpening** — the precise principle is "reads have a fallback, writes don't," not just "cache might crash"

---

## Past Mistakes

See [InterviewMistakes.md](../InterviewMistakes.md) — entries dated 2026-07-08.
