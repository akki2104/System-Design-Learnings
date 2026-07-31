# Revision — Topic 033: Caching Patterns

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-31

---

## Q1. Walk through the exact read and write sequence for cache-aside.

<details>
<summary>Answer</summary>

Read: check cache — hit returns immediately; miss queries the DB, then populates the cache with the result before returning. Write: write to the DB, THEN delete the cache entry for that key — do not write the new value into the cache. If the write skipped the cache entirely, the next read would be a HIT returning the stale value until TTL expiry.

</details>

---

## Q2. Why does cache-aside delete the cache entry on write instead of writing the new value into the cache directly?

<details>
<summary>Answer</summary>

Two concurrent writers could race: Transaction A writes DB then cache, Transaction B writes DB then cache — if A's cache-write is delayed and lands after B's, the cache ends up holding A's older value even though B's is now correct in the DB. Deleting instead of writing avoids this race entirely — the next read simply repopulates from whatever the DB currently (correctly) holds.

</details>

---

## Q3. What does write-through guarantee that cache-aside doesn't, and what does it cost?

<details>
<summary>Answer</summary>

Write-through guarantees there is never a stale-read window after a write — cache and DB are updated synchronously, together. The cost: every write now pays the latency of two writes (cache + DB) instead of one, and data that's written but never read still consumes cache space.

</details>

---

## Q4. What is the defining risk of write-behind, and why does that risk exist mechanically?

<details>
<summary>Answer</summary>

Data loss: writes only touch the cache and are acknowledged immediately; the DB is updated asynchronously/batched later. If the cache crashes before flushing, those writes never made it to the DB and are gone permanently. The upside (fast writes, reduced/batched DB load) and this downside are the same mechanism — the flush is deferred, which is exactly what makes it both fast and risky.

</details>

---

## Q5. Given a product catalog and a real-time view counter in the same system, which caching pattern would you pick for each, and why?

<details>
<summary>Answer</summary>

Product catalog → cache-aside: read-heavy, DB stays authoritative, a brief miss-then-repopulate window is fine. View counter → write-behind: extremely write-heavy (every view increments it) and loss-tolerant (losing the last few seconds of counts on a rare crash is acceptable) — the write-latency and DB-load reduction are worth the small, bounded loss risk.

</details>

---

## 30-Second Elevator Pitch

> Cache-aside is the default: on a miss the app reads the DB and populates the cache; on a write the app writes the DB and deletes the cache entry — never overwrites it, because concurrent writers could race and leave a stale value cached. Write-through updates cache and DB together on every write, guaranteeing freshness at the cost of doubled write latency. Write-behind writes only to the cache and flushes to the DB asynchronously/batched — fastest writes, reduced DB load, but unflushed writes are lost if the cache crashes before flushing. Real systems mix patterns per data type: cache-aside for the catalog, write-through for user settings that need instant read-after-write, write-behind for loss-tolerant, write-heavy data like view counters.

---

## Weak Areas to Watch

- First-pass answer on the cache-aside write step tends to say "write to DB" and stop there — always state the invalidation step explicitly, since skipping it means cache-aside doesn't actually solve write-side staleness at all.
- Framing write-behind candidates by raw "read/write heavy" rather than the two properties that actually matter: write volume (is the DB the bottleneck?) and loss tolerance (is losing the last few seconds acceptable?).
