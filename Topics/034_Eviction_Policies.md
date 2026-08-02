# Topic 034: Eviction Policies

**Module:** 3 — Caching
**Tier:** 🟡 SKIM
**Completed:** 2026-08-02
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topic 032 established that a cache is finite — it can only hold the hot set, not everything. That raises the natural next question: when the cache is full and a new item needs space, what gets removed? That's an eviction policy. Small, mechanical topic — but "what happens when the cache fills up" is a near-certain follow-up after any caching discussion, so it needs a fluent, one-line answer with the right mechanism behind it.

---

## 2. LRU — Least Recently Used (the default)

Evict the item that hasn't been *accessed* in the longest time. Directly exploits temporal locality (Topic 032): something untouched for a while is statistically unlikely to be needed again soon.

**Implementation — why it's O(1):**
```
HashMap: key → pointer to its node        (O(1) existence check + O(1) locate)
Doubly-Linked List: maintains recency order (O(1) move-to-front on access,
                                              O(1) remove-from-back on eviction)
```
A hashmap alone cannot do this — it has no ordering, so finding "which key is least recently used" would require an O(n) scan (or storing timestamps and still scanning for the minimum). The linked list is what makes reordering and eviction both O(1) via pure pointer rewiring. The two structures divide the labor: hashmap finds, linked list orders/evicts. This is why LRU is implemented as *both*, never either alone.

This is the default eviction policy in Redis and most caching libraries.

---

## 3. LFU — Least Frequently Used

Evict the item with the lowest access *count*, not recency. Sounds smarter than LRU at first glance, but has a real blind spot: **stale popularity**. An item that was extremely popular in the past keeps a high count and refuses to get evicted even though it's gone cold, while a currently-active-but-newer item with a lower total count gets evicted instead — the exact wrong call.

**Worked example:** A product goes viral during a flash sale — 10,000 accesses in an hour — then the sale ends and nobody looks at it again. A different, moderately popular product gets 50 accesses today and is still being actively browsed. Cache is full. LFU compares 10,000 vs 50 and evicts the *actively browsed* item, keeping the dead flash-sale item. LRU would have already caught this — the flash-sale item hasn't been touched recently, so it sits at the back of the recency list and gets evicted correctly.

LFU needs extra bookkeeping (a counter per item, often with decay to fix the above problem) — more implementation complexity for a benefit that's often marginal over LRU in practice.

---

## 4. FIFO — First In, First Out

Evict whatever was cached *first*, regardless of access frequency or recency. Simplest to implement (just a queue), but ignores access patterns entirely — can evict a very hot item just because it happened to be cached early. Rarely the right choice when LRU is available at similar implementation cost.

---

## 5. TTL-Based Expiry — a Different Axis Entirely

TTL (Topic 032) auto-expires entries after N seconds, regardless of memory pressure. This solves **staleness** (is this data too old to trust?), not **memory pressure** (what do I remove when full?) — two different questions, often needed together.

**Important default behavior:** a standard TTL is a fixed clock set at insertion (e.g. `SET key val EX 300`) — it counts down regardless of how often the item is read in between. It does **not** automatically refresh on access. "TTL keeps extending as long as requests keep coming" is a specific, opt-in design (a sliding/refresh-on-read TTL, e.g. Redis's `GETEX`) — not the default. Under the default, a hot item still expires at the TTL mark even while being accessed constantly, unless the system was explicitly built to refresh on read.

A cache can use TTL and an eviction policy together: TTL bounds how stale data can get; the eviction policy (LRU/LFU/FIFO) decides what to remove early if the cache fills up before TTL expiry. They fire independently and can each act first depending on timing.

---

## 6. Worked Example — TTL vs LRU, Which Governs What

Cache has TTL = 5 min and LRU eviction. An item was cached 2 minutes ago and has been accessed constantly since. The cache fills up (because many *other, different* keys are being cached — filling up is a global capacity event across all keys, not something this one item's repeated reads cause; reading the same key again doesn't add a new entry).

- **At the fill-up moment (before the 5-min mark):** LRU governs. This item is the most recently used, so it is NOT the eviction victim — a colder item gets evicted instead.
- **At the 5-min mark:** TTL governs, independent of recency. Under the standard (non-refreshing) default, this item expires and is removed regardless of how hot it's been — unless the cache was specifically built with refresh-on-read TTL.

The two mechanisms answer different questions and can each take effect at different times for the same key.

---

## Decision Box

```
Default: LRU — O(1), exploits temporal locality, battle-tested (Redis default)
Use LFU when: frequency is genuinely a better popularity signal than recency AND
              you accept the bookkeeping cost (e.g. globally popular CDN content
              whose rank doesn't shift hour-to-hour)
Use FIFO when: no strong opinion needed, just want bounded memory cheaply —
               rare once LRU is available
TTL: use ALONGSIDE any eviction policy — solves staleness, not memory pressure;
     "is this too old?" vs "what do I remove first when full?"
```

**Interview sentence:** "I'd default to LRU eviction since it's O(1) and matches the temporal-locality assumption the cache already relies on to work at all. I'd combine it with a TTL so stale data doesn't linger indefinitely even if it stays hot enough to avoid LRU eviction — and I'd be explicit that a standard TTL doesn't reset on access unless I deliberately build that in."

---

## Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Believing a hashmap alone can implement LRU | A hashmap has no ordering — finding the least-recently-used key without a linked list requires an O(n) scan; the doubly-linked list is what makes reorder + evict O(1) |
| Assuming LFU is strictly better than LRU because "frequency sounds smarter" | LFU can trap stale-but-formerly-popular items (old high counts) and costs more to implement; LRU is usually the better default |
| Treating TTL as if it refreshes automatically on every access | Standard TTL is a fixed clock from insertion — it does not reset on read unless explicitly implemented as a sliding/refresh-on-read TTL |
| Treating TTL and eviction policy as the same mechanism | TTL bounds staleness (time-based); eviction policy decides what to remove under memory pressure (space-based) — a cache typically needs both, and either can fire first depending on timing |
| Confusing "cache fills up" with "this specific item recurring" | Fill-up is a global capacity constraint from many distinct keys accumulating over time; re-reading the same cached key doesn't consume additional space |

---

## 7. Revision Questions
See `Revision/Revision_034.md`.

## 8. Summary
- Eviction policy decides what to remove under memory pressure when the cache is full; TTL decides what's too stale to trust — different questions, often used together.
- **LRU** (default): evicts by recency, O(1) via hashmap (find) + doubly-linked list (reorder/evict). Exploits temporal locality directly.
- **LFU**: evicts by frequency count; blind spot is stale popularity (a formerly-hot item with a high old count outlasts a currently-active item with a lower count).
- **FIFO**: evicts by insertion order; ignores access patterns entirely, rarely preferred over LRU.
- Standard TTL is a fixed clock from insertion and does not refresh on access by default — that requires an explicit sliding/refresh-on-read design.
- "Cache fills up" is a global-capacity event across all distinct keys, not something one item's repeated access causes.
