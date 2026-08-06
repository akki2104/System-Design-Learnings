# Revision — Topic 037: Cache Consistency & Invalidation

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-06

---

## Q1. Why is deleting the cache before updating the DB worse than not invalidating at all?

<details>
<summary>Answer</summary>

A concurrent read can miss right after the delete, query the DB before the write lands, and repopulate the cache with the OLD value. The subsequent DB update then happens, but the cache is now stuck holding the pre-update value until TTL — a guaranteed staleness window that wouldn't exist if invalidation had been skipped entirely.

</details>

---

## Q2. Even with the correct "update DB, then delete cache" order, describe the residual race condition that can still occur.

<details>
<summary>Answer</summary>

A read that started querying the DB *before* the write's DB update completed can still be holding the old value when it goes to populate the cache — and if that populate happens *after* the write's delete, the cache ends up stale again. The window is much narrower (has to straddle the write precisely) but not zero.

</details>

---

## Q3. Name three mitigations for that residual race, and the tradeoff of each.

<details>
<summary>Answer</summary>

Delayed double-delete: delete the key again after a short delay (e.g. 500ms-1s) to catch a straggler — cheap, but the delay is a guess, not a guarantee. Short TTL: bounds how long any stale value that sneaks through can survive — simple safety net, doesn't prevent the race, just limits its damage. Versioned/CAS writes: only overwrite the cache if the incoming write is newer than what's cached — the most robust fix, at the cost of implementation complexity (tracking versions everywhere).

</details>

---

## Q4. Why doesn't deleting a shared distributed cache entry invalidate an app server's local (L1) cache or a CDN edge cache holding the same key?

<details>
<summary>Answer</summary>

An app server's L1 cache is not a node of the distributed cache's own cluster — it's a completely separate, in-process cache the app maintains itself, which never receives the distributed cache's delete because it was never connected to that replication in the first place. They are two structurally different caches, not two replicas of one. The same applies to CDN edges: a cached key at an edge node is independent of the origin's cache and keeps serving until its own TTL or an explicit purge.

</details>

---

## Q5. Give one example of data where TTL-only eventual consistency is fine, and one where it isn't.

<details>
<summary>Answer</summary>

Fine: a "number of views" counter — a few seconds of staleness harms nobody and self-corrects. Not fine: an account's "is this user banned" flag gating access — a stale read for even 30 seconds means a banned user keeps acting during that window, which is a real correctness/security gap, not a cosmetic one.

</details>

---

## 30-Second Elevator Pitch

> Cache-aside's "delete on write" rule from Topic 033 isn't bulletproof. Deleting the cache before updating the DB guarantees a staleness window; updating the DB first and deleting after (the correct order) narrows the race but doesn't close it — a read that started before the write can still repopulate the cache with a stale value after the delete. Mitigations trade cost for tightness: delayed double-delete, a short TTL safety net, versioned/CAS writes (most robust), or write-around (skip caching on write entirely). Multiple independent cache copies — per-server local caches, the shared distributed cache, CDN edges — each need their own invalidation, because an L1 cache isn't a node of the distributed cache's cluster, it's a structurally separate cache; this needs a broadcast (pub/sub or a purge API), not a single delete. Ultimately this is a small instance of the general distributed-consistency problem, not a caching-specific quirk.

---

## Weak Areas to Watch

- First-pass explanation of why L1/CDN survive a distributed-cache delete drifted toward "cluster node replication lag" — the real reason is structural separation (L1 was never part of that cluster's replication at all), not lag within one system. Self-corrected on a direct nudge.
