# Topic 037: Cache Consistency & Invalidation

**Module:** 3 — Caching
**Tier:** 🔴 MUST
**Completed:** 2026-08-06
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topic 033 said cache-aside deletes the cache entry on write, not overwrites it. That's true — but it hides a real subtlety: *when exactly* you delete relative to the DB write, and what happens when *more than one* cache is holding a copy, can reopen the exact race conditions caching was supposed to avoid. This is the part interviewers specifically call "the hard part" of caching, and it's where most candidates' answers fall apart under a follow-up push.

---

## 2. Order Matters: Delete-Then-Update vs Update-Then-Delete

**The wrong order — delete cache, then update DB:**
```
T1 (write):  DELETE cache key ──────────────────────▶ UPDATE DB
T2 (read):        │  MISS → query DB (gets OLD value, T1 hasn't written yet)
                  │         → repopulate cache with OLD value
                  ▼
Result: cache now holds the OLD value — and stays that way until TTL,
        even though the DB has since been updated correctly.
```
This is actively **worse** than not invalidating at all — you've manufactured a guaranteed staleness window instead of just accepting the small risk of overwriting.

**The order Topic 033 actually specified — update DB, then delete cache:**
```
T1 (write):  UPDATE DB ──────────────────────▶ DELETE cache key
T2 (read):   │ MISS → query DB (started before T1's update, gets OLD value)
             │        → repopulates cache with OLD value, AFTER T1's delete
             ▼
Result: still possible, but the race window is much narrower — T2's DB
        read has to land in the split second between T1's DB write and
        T1's cache delete, not the whole duration of the DB write itself.
```
**The honest answer for an interview:** "update DB then delete" is *better*, not *bulletproof*. A residual race still exists — narrower, but real under high concurrency on a hot key.

---

## 3. Mitigations for the Residual Race

```
Delayed double-delete: delete the cache key again after a short delay
  (e.g. 500ms-1s) — catches a straggling read that repopulated stale
  data in that narrow window. Cheap, commonly used, not theoretically
  perfect (delay is a guess), but closes the practical gap.

Short TTL as a safety net: even if a stale value sneaks in, it self-heals
  within the TTL window — belt-and-suspenders alongside explicit deletes.

Versioned / CAS writes: store a version/timestamp alongside the cached
  value; only overwrite the cache if the incoming write's version is
  NEWER than what's already there. A straggling stale write then simply
  loses the compare-and-swap instead of clobbering a fresher value —
  this is the most robust fix, at the cost of implementation complexity.

Write-around: for data where staleness right after a write is expected
  anyway and it's rarely re-read immediately, just don't touch the cache
  on the write path at all — write straight to the DB, let the NEXT read
  populate the cache normally via the standard cache-aside miss path.
  Sidesteps the whole race by never racing in the first place.
```

---

## 4. The Part Everyone Forgets: Multiple Cache Copies

A single `DEL key` only clears *one* cache. Real systems commonly have several independent copies of the same cached data:
```
App Server 1's local (L1) cache ─┐
App Server 2's local (L1) cache ─┼─▶  Shared distributed (L2) cache (Redis)
App Server 3's local (L1) cache ─┘
CDN edge caches (geographically distributed, Topic 018)
```
**Why deleting the shared/distributed cache doesn't touch the L1 caches:** an app server's local cache isn't a *node* of the distributed cache's own cluster — it's a completely separate, in-process cache the app itself maintains (e.g., a local hashmap), which never talks to the distributed cache's internal replication in the first place. A `DEL` on Redis (L2) simply never reaches L1 — they are two structurally different caches, not two replicas of the same one. The same logic applies to CDN edges: if the cached key is present at an edge node, that edge will keep serving it regardless of what happened to the origin's cache.

This is exactly the multi-tier caching setup Topic 035 mentioned as an avalanche mitigation — the same structure that helps availability is what makes invalidation harder.

**Mitigation — invalidation broadcast:** publish an "invalidate key X" event (Redis Pub/Sub, a Kafka topic, or a CDN's purge API) that every cache layer/node subscribes to and reacts to independently. This is why CDNs expose a purge API as a first-class operation, not an afterthought — a single delete on the origin cache is not enough to actually clear a cached value system-wide.

---

## 5. The Bigger Picture (preview)

"How do multiple copies of the same data agree on what's current" is, structurally, the exact question distributed consistency models exist to answer (Topic 048 preview) and is bounded by the same reality as replication lag (Topic 040 preview). Cache invalidation across multiple nodes is a small, concrete instance of the general distributed-consistency problem — not a separate, simpler thing.

---

## Tech Decision Box: How Much Consistency Rigor Does This Data Need?

```
Accept eventual staleness (TTL + best-effort delete-on-write) when:
  - Display/browsing data (Topic 032's distinction) — product descriptions,
    view counts, "last seen" timestamps
  - A brief stale window causes no real harm

Pay for stronger guarantees (versioned/CAS writes + invalidation broadcast
+ short TTL safety net) when:
  - A stale read after a write causes a real problem: pricing shown at
    checkout, feature flags gating a rollout, an account's banned/suspended
    status gating access, inventory counts
  - Multiple cache layers/nodes exist and must agree quickly
```
**Interview sentence:** "For the product description, I'd accept the narrow residual race and a short TTL as a safety net — the cost of getting it wrong is negligible. For an account-banned flag gating access, I'd use versioned writes and an invalidation broadcast to every app server's local cache, because a stale read there isn't 'slightly outdated' — it's a real security gap where a banned user keeps acting for the full TTL window."

---

## Common Mistakes

| Mistake | Correction |
|---|---|
| Deleting the cache before updating the DB | Creates a *guaranteed* staleness window (a concurrent read populates the cache with the pre-update value) — strictly worse than not invalidating at all |
| Assuming "update DB then delete cache" fully eliminates race conditions | It narrows the race window dramatically but doesn't close it to zero — a straggling read that started before the write can still repopulate a stale value after the delete |
| Explaining why an L1 local cache survives a distributed-cache delete via "cluster node replication lag" | An app server's L1 cache isn't a node of the distributed cache's cluster at all — it's a separate, independent in-process cache that never received the delete in the first place, structurally distinct from Redis's own internal replica sync |
| Forgetting that L1 local caches and CDN edges need their own invalidation propagation | Deleting the shared/distributed cache does nothing to other independent copies — each needs to hear about the invalidation, typically via a pub/sub broadcast |
| Treating cache invalidation as "solved" after Topic 033's delete-not-overwrite rule | That rule fixes the *concurrent-writer* race from Topic 033; it does not fix the *write-vs-concurrent-reader* race covered here, which is a separate, harder problem |

---

## Real Interview Questions

1. "What goes wrong if you delete the cache before writing to the DB, versus after?" (tests the ordering reasoning directly)
2. "Even doing it in the 'correct' order, is there still a race condition? Describe it." (tests whether the candidate oversells cache-aside as bulletproof)
3. "How would you invalidate a cached value across multiple app servers, each running its own local cache?" (tests the multi-layer/broadcast reasoning)
4. "What's the risk of relying purely on TTL for a feature-flag cache?" (tests judgment on when eventual staleness is actually unacceptable)
5. "How does cache invalidation relate to the broader problem of distributed consistency?" (tests whether the candidate sees this as one instance of a general problem, not a caching-specific quirk)

---

## 6. Revision Questions
See `Revision/Revision_037.md`.

## 7. Summary
- Order matters: delete-then-update guarantees a staleness window; update-then-delete (Topic 033's rule) narrows the race but doesn't eliminate it.
- The residual race: a read that started before a write can still repopulate the cache with a stale value after the write's delete has already happened.
- Mitigations — delayed double-delete, short TTL as a safety net, versioned/CAS writes (most robust), or write-around (sidesteps the race by never caching on write) — each trades implementation cost against how tightly closed the race needs to be.
- Multiple independent cache copies (per-server local caches, distributed cache, CDN edges) each need their own invalidation — a single delete on one doesn't touch the others, because an L1 local cache isn't a *node* of the distributed cache's cluster, it's a structurally separate cache. This needs a broadcast mechanism.
- Cache invalidation across nodes is a concrete instance of the general distributed-consistency problem, not a caching-specific quirk.

> **You now can:** explain why invalidation order matters and what breaks if it's reversed, describe the residual race even in the "correct" order, name mitigations with their tradeoffs, and correctly explain why L1/CDN caches need their own invalidation broadcast rather than confusing it with cluster replication lag.
