# Topic 005 — How to Reason About Tradeoffs

**Module:** 0 — Orientation & Mental Models
**Status:** Completed
**Date:** 2026-07-08
**Confidence:** 3/5
**Difficulty:** Medium

---

## 1. Why This Topic Exists

Every system design question has multiple valid answers. What separates a strong candidate from a weak one isn't which technology they pick — it's *how they reason about the choice*. This topic builds the reusable thinking framework, tied directly to real database decisions.

---

## 2. Real Production Problem

A startup chose MongoDB for core payments data because "NoSQL is modern and scalable" — with no tradeoff reasoning behind it. 18 months later: $2M in inconsistent transaction records from partial writes and no atomic operations. The migration to PostgreSQL took 6 months and nearly killed the company. MongoDB isn't bad — it trades ACID guarantees for flexibility and write speed. For payments, that's the wrong trade.

---

## 3. Simple Intuition

Every tool is a trade, not a free upgrade. A tent is lighter than a house but doesn't survive a storm. Choosing infrastructure is choosing which storm you're willing to risk.

---

## 4. Core Concepts

### The 3 Questions Before Any Tech Choice
```
1. What problem am I actually solving?          (reads? writes? search? scale?)
2. What does this technology trade away?        (every tool gives something, takes something)
3. What happens when this choice is wrong at 10×?  (can I migrate? blast radius?)
```

### The Tradeoff Decision Tree — the core mental model

The single most useful question when a scary number appears:

> **"Is my problem READ pressure, or WRITE pressure?"**

```
                    Scary number appears
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
      Reads are the problem        Writes are the problem
              │                           │
              ▼                           ▼
        ADD A CACHE                SWITCH / SHARD THE DATABASE
   Same DB, same schema.        Data physically redistributed.
   Cheap, reversible,           Expensive, hard to reverse,
   ships fast.                  takes weeks/months.
```

**Why a cache fixes reads but not writes:** A read cache miss just costs time — fall back to the database and get the correct answer. A write has no such fallback: it must land durably in the one source of truth. If a write-back cache crashes before flushing, that data is gone forever — not slow, gone. So caching can reduce read volume on the database, but it can never reduce write volume; every write must still reach the database that persists it.

### Why Joins Get Expensive at Scale (not what most candidates think)

**Misconception:** "huge data volume" breaks SQL joins.
**Reality:** A single machine joins tables fine even at hundreds of GB with good indexes — a join there is a local, in-memory operation.

**The real trigger is sharding (spreading data across machines), not data size.**
```
Big data, ONE machine        → joins still work fine (indexing handles it)
Data SPREAD across machines  → joins become NETWORK calls (this is the real cost)
```
This is why NoSQL documents look "duplicated" — a document embeds related data directly instead of storing a foreign key and joining. This is **denormalization**: write once, read fast, accept duplication and potential staleness.

### The Tradeoff Vocabulary

| You want | You pay |
|----------|---------|
| Consistency | Latency + complexity |
| High availability | Risk of stale data |
| High write throughput | Weaker consistency or read complexity |
| Fast reads | Stale data (cache) or expensive indexing |
| Flexibility (schema-less) | Weaker query power, no joins |
| Low cost | Slower performance or operational burden |

### Database Decision Framework (see [TechChoices.md](../TechChoices.md) for the full table)

```
Flexible/nested schema, unknown shape     → MongoDB
Massive writes, KNOWN access pattern      → Cassandra / DynamoDB (leaderless writes)
Money, joins, ACID transactions           → PostgreSQL / MySQL (the default)
Hot reads, sessions, counters             → Redis (cache, not source of truth)
Full-text/fuzzy search                    → Elasticsearch (index, not source of truth)
PB-scale blobs                            → S3 / object storage
```

**Key correction from this lesson:** "flexible schema" (MongoDB's strength) and "known access pattern + high write throughput" (Cassandra's strength) are two DIFFERENT properties — don't conflate them. MongoDB shards still route writes through a per-shard primary (write bottleneck); Cassandra/DynamoDB use a leaderless model where any node accepts writes, which is why it's the default for messaging-scale write throughput (Discord uses Cassandra/ScyllaDB for messages).

---

## 5. Visualization

```
THE DECISION TREE
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

## 6. Animation Frames — Twitter vs WhatsApp Worked Examples

```
Frame 1: Twitter numbers (from Topic 003)
  Write QPS: 10,000/sec   Read QPS: 1,000,000/sec (100:1)

Frame 2: Identify the pressure
  1,000,000 is the scary number → READ pressure

Frame 3: Apply the tree
  READ pressure → ADD A CACHE (Redis in front of DB)

Frame 4: WhatsApp numbers
  Write QPS: 500,000/sec   Read:Write ratio: 1:1

Frame 5: Identify the pressure
  500,000 writes is the scary number → WRITE pressure

Frame 6: Apply the tree
  WRITE pressure → SWITCH DATABASE (Cassandra — leaderless writes,
  known access pattern: fetch messages by conversation_id, ordered by time)
```

---

## 7. Architecture Diagram

```
READ-HEAVY FIX                          WRITE-HEAVY FIX
────────────────                        ────────────────
[Clients] → [Cache] → [DB]              [Clients] → [App] → [Sharded/Leaderless DB]
             ↑ absorbs                                        (writes distributed
             most reads                                        across many nodes)
```

---

## 8. Real Engineering Examples

- **Twitter:** Redis-backed timeline cache absorbs the 100:1 read:write ratio
- **Discord:** Cassandra (later ScyllaDB) for messages — leaderless writes at massive scale
- **Stripe:** PostgreSQL for payments — ACID transactions non-negotiable for money
- **Amazon product catalog:** MongoDB-style document stores — every category has different attributes (shirt: size/color; laptop: RAM/CPU)

---

## 9. Industry Examples

- **Payments (Stripe, Razorpay):** Always relational — reliability and ACID over flexibility
- **Messaging (Discord, WhatsApp):** Wide-column/leaderless stores — write throughput over ad-hoc query power
- **Catalogs (Amazon, eBay):** Document stores — schema flexibility over joins
- **Search (Uber Eats, GitHub):** Elasticsearch as a secondary index, never the source of truth

---

## 10. Tradeoffs

| Choice | Benefit | Cost |
|--------|---------|------|
| Add a cache for read pressure | Fast, cheap, reversible | Doesn't help writes at all; adds staleness risk |
| Shard/switch DB for write pressure | Scales write throughput | Expensive, hard to reverse, loses easy joins |
| MongoDB for flexible schema | No rigid schema, fast iteration | Weak multi-record transactions, no real joins |
| Cassandra for write throughput | Leaderless writes, massive scale | Requires known access patterns upfront; no ad-hoc queries |
| Denormalization (avoid cross-shard joins) | Fast reads, no network joins | Data duplication, staleness on updates |

---

## 11. Complexity Analysis

N/A — this topic is a decision framework, not an algorithm. The "complexity" is organizational: migrating databases is O(months), adding a cache is O(days).

---

## 12. Scaling Considerations

```
10×   → cache usually absorbs it if read-heavy
100×  → write-heavy systems start needing sharding/leaderless DBs
1000× → denormalization becomes mandatory; cross-shard joins are no longer viable
```

---

## 13. Failure Scenarios

- **Caching a write-heavy problem:** cache doesn't reduce DB write load; the real bottleneck remains, wasted engineering effort
- **Picking Cassandra for unknown/ad-hoc query needs:** query patterns must be known upfront; wrong choice here means expensive redesign later
- **Write-back cache crash:** unflushed writes lost permanently — this is why write-back caching is used cautiously, if at all, for critical data

---

## 14. Monitoring

- Cache hit rate (validates whether cache is actually absorbing read load)
- DB write latency/queue depth (signals write pressure building)
- Cross-shard query count (signals expensive network joins happening)

---

## 15. Observability

- Alert on cache hit rate dropping below target (read fix losing effectiveness)
- Alert on DB write queue depth exceeding threshold (write pressure building toward a database change)

---

## 16. Security

Denormalized/duplicated data (e.g., embedded restaurant name in an order document) must be kept in sync carefully — stale duplicated data can leak outdated information (e.g., old pricing) if not invalidated correctly.

---

## 17. Interview Discussion

This is the highest-leverage topic for the "Tradeoffs & Justification" rubric dimension (15–20% of score). The 3-part sentence format is the exact structure interviewers are listening for:

> "I'll use [X] because [specific problem property]. I considered [Y] but rejected it because [what Y trades away]."

Never say "I'll use technology X" without the reason and the rejected alternative — that's the #1 signal of Socratic vs buzzword thinking.

---

## 18. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Reacting to "a big number" without asking read vs write | Always classify pressure type first |
| Believing "huge data" breaks joins | It's sharding (data spread across machines), not data volume |
| Conflating "flexible schema" with "high write throughput" | These are different properties — MongoDB vs Cassandra solve different problems |
| Assuming a cache fixes write-heavy systems | Cache never reduces write volume — only read volume |
| Picking NoSQL for ad-hoc/unknown queries | NoSQL (esp. wide-column) needs known access patterns upfront |

---

## 19. Advanced Topics

- **Write-back vs write-through caching:** write-through writes to cache AND DB synchronously (safe, no reduction in DB write load); write-back writes to cache first and flushes later (risk of data loss on crash)
- **CQRS (Topic 065):** separates read and write models entirely — an even more aggressive version of "different fixes for different pressures"
- **Polyglot persistence:** using multiple databases in one system, each for its strength (e.g., Postgres for orders, Elasticsearch for search, Redis for sessions)

---

## 20. Real Interview Questions

1. "Why would you choose Cassandra over MongoDB for a chat application?" (Discord, Meta)
2. "Our read traffic just 10×'d — what's your first move?" (Universal)
3. "Our write traffic just 10×'d — what's your first move, and how is it different from the read case?" (Amazon, Uber)
4. "Why do NoSQL databases avoid joins?" (Google, general)
5. "When would you NOT use a cache to solve a scaling problem?" (Stripe, general)

---

## 21. Exercises

1. Amazon product catalog: pick a database and justify with the 3-part sentence.
2. A system shows DB write queue depth climbing — is this a cache problem or a database problem? What's your fix?
3. Explain to a junior engineer why "duplicated data" in MongoDB documents is a deliberate tradeoff, not sloppy design.
4. Name one system where you'd use BOTH a cache AND a sharded database — and explain which pressure each one solves.

---

## 22. Revision Questions

1. What's the single question that routes you to "add a cache" vs "switch the database"?
2. Why does a cache fix read pressure but never write pressure?
3. What's the REAL trigger for expensive joins — data size or something else?
4. Distinguish MongoDB's strength from Cassandra's strength — they are NOT the same "NoSQL flexibility."
5. State the 3-part interview sentence format for justifying a tech choice.
6. WhatsApp needs 500K writes/sec with a known access pattern. Which DB, and why not the alternative?

---

## 23. Cheat Sheet

```
THE TRADEOFF DECISION TREE
─────────────────────────────────────────────────
Scary number appears → READS or WRITES?
  READS  → ADD A CACHE       (cheap, reversible, same DB)
  WRITES → SWITCH/SHARD DB   (expensive, hard to reverse)

WHY CACHE FIXES READS, NOT WRITES
─────────────────────────────────────────────────
Read cache miss  = fall back to DB, just slower (no data loss)
Write cache loss = data gone forever (no fallback) — DB must absorb all writes

WHY JOINS GET EXPENSIVE
─────────────────────────────────────────────────
Big data, ONE machine       → joins fine (indexing handles it)
Data SPLIT across machines  → joins = network calls (the real cost)
Fix: denormalize to avoid cross-shard joins

DB CHOICE QUICK MAP
─────────────────────────────────────────────────
Flexible/nested schema, unknown shape   → MongoDB
Massive writes, KNOWN access pattern    → Cassandra / DynamoDB
Money, joins, ACID                      → PostgreSQL / MySQL (default)
Hot reads, sessions, counters           → Redis (cache, not truth)
Full-text/fuzzy search                  → Elasticsearch (index, not truth)
PB-scale blobs                          → S3 / object storage

THE 3-PART INTERVIEW SENTENCE
─────────────────────────────────────────────────
"I'll use [X] because [specific problem property].
 I considered [Y] but rejected it because [what Y trades away]."
```

---

## 24. Summary

- **Every tech choice trades something for something.** Never present one option as "the answer."
- **The core routing question:** read pressure → cache; write pressure → switch/shard the database.
- **Cache fixes reads because misses just cost time.** Writes have no such fallback — they must persist durably.
- **Joins get expensive from sharding (data spread across machines), not from data volume alone.**
- **MongoDB flexibility ≠ Cassandra write throughput.** Different properties, different problems solved.
- **Always speak in the 3-part sentence:** tech + reason + rejected alternative + why rejected.
- **Full DB decision table lives in [TechChoices.md](../TechChoices.md)** — review before every case study.

> **You now can:** classify any scaling problem as read-pressure or write-pressure, apply the correct fix, and justify a database choice with the interview-ready 3-part reasoning sentence.
