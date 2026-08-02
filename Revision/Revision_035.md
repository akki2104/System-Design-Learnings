# Revision — Topic 035: Cache Problems (Stampede, Penetration, Avalanche)

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-02

---

## Q1. Distinguish cache penetration, stampede, and avalanche by their TRIGGER (not just "DB gets hammered").

<details>
<summary>Answer</summary>

Penetration: repeated queries for a key that exists in neither cache nor DB — never has anything to cache. Stampede: one very hot key expires and thousands of concurrent requests all miss and regenerate it at once. Avalanche: many different keys expire around the same time (mass simultaneous TTL expiry), OR the cache cluster itself goes down entirely.

</details>

---

## Q2. Why is a Bloom filter specifically a penetration fix, and not a general cache-problem fix?

<details>
<summary>Answer</summary>

A Bloom filter has a one-sided error: it can say "might exist" (false positive possible) or "definitely does not exist" (never a false negative). This makes it safe as a hard gate in front of the DB for keys that never existed at all — exactly penetration's problem. It's irrelevant to stampede/avalanche because those involve keys that DO exist and are simply expiring under load — the filter would correctly say "might exist" and let the request through, doing nothing to prevent the DB overload.

</details>

---

## Q3. How does a mutex/lock fix cache stampede, and what's its downside compared to logical expiration?

<details>
<summary>Answer</summary>

Only the first request that misses acquires the lock and queries the DB; every other concurrent request waits for the lock holder to repopulate the cache, then reads the fresh value — converting N concurrent DB queries into 1. The downside: every waiting request is blocked synchronously until the lock holder finishes, which can itself become a bottleneck under very high concurrency. Logical expiration avoids this entirely — it serves the (slightly stale) cached value immediately to every requester while one background task refreshes, so no request ever waits.

</details>

---

## Q4. A product catalog is bulk-warmed into the cache with the same TTL for every key. What failure mode does this risk, and what's the fix?

<details>
<summary>Answer</summary>

Cache avalanche via mass simultaneous TTL expiry — all keys expire around the same instant, and the DB gets hit with the load of the entire warmed set at once. Fix: jitter every TTL (e.g. `TTL = base + random(0, N)`) so expirations spread across a window instead of firing simultaneously.

</details>

---

## Q5. Name the two distinct causes of cache avalanche and the mitigation specific to each.

<details>
<summary>Answer</summary>

(1) Mass simultaneous TTL expiry → fix: jittered TTLs. (2) Cache cluster outage (the distributed cache itself goes down) → fix: high availability/replication for the cache cluster, plus a circuit breaker in front of the DB and multi-tier caching so a cache outage means degraded caching, not zero caching. Jitter alone does not address cause 2.

</details>

---

## 30-Second Elevator Pitch

> Three cache failure modes get conflated but have distinct triggers: penetration is a key that never existed anywhere (fix: cache negative results + a Bloom filter's one-sided "definitely not" gate), stampede is one hot key expiring under concurrent load (fix: a mutex that serializes the regeneration, or better, logical/soft expiration that serves stale immediately while refreshing in the background with zero waiting), and avalanche is either mass simultaneous TTL expiry across many keys (fix: jitter every TTL) or a total cache cluster outage (fix: HA/replication, a circuit breaker in front of the DB, and multi-tier caching). The interview skill is matching the specific fix to the specific trigger rather than reaching for one fix reflexively for all three.

---

## Weak Areas to Watch

- None this session — clean pass on all three checkpoint questions, including the precise one-sided-error reasoning for Bloom filters.
