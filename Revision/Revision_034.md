# Revision — Topic 034: Eviction Policies

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-02

---

## Q1. Why does LRU need BOTH a hashmap and a doubly-linked list — why can't a hashmap alone do it?

<details>
<summary>Answer</summary>

A hashmap gives O(1) existence check and O(1) locate, but it has no ordering — finding "which key is least recently used" from a hashmap alone would require an O(n) scan (even with per-entry timestamps, finding the minimum still costs a scan). The doubly-linked list maintains recency order so an access can move a node to the front in O(1) and eviction can remove from the back in O(1). Hashmap finds; linked list orders and evicts — neither alone can do both.

</details>

---

## Q2. Give a concrete scenario where LFU evicts the wrong item compared to LRU.

<details>
<summary>Answer</summary>

A product goes viral during a flash sale (10,000 accesses in an hour), then goes cold. A different product gets 50 accesses today and is still actively browsed. Cache is full. LFU compares raw counts (10,000 vs 50) and evicts the actively-browsed item, keeping the now-dead flash-sale item — the wrong call. LRU would have already flagged the flash-sale item as least-recently-used and evicted it correctly, since it hasn't been touched recently regardless of its historical count.

</details>

---

## Q3. Does a standard TTL refresh every time an item is read? Why does this matter?

<details>
<summary>Answer</summary>

No — a standard TTL is a fixed clock set at insertion time; it counts down regardless of read frequency and does not auto-refresh on access. A hot item can still expire at the TTL mark even while being read constantly. Refresh-on-read ("sliding TTL") is a specific, opt-in design (e.g. Redis's `GETEX`), not the default. This matters because assuming TTL auto-extends leads to wrongly predicting data will "live forever" just because it's popular — it won't, unless sliding TTL was explicitly built in.

</details>

---

## Q4. A cache has TTL=5min and LRU eviction. An item cached 2 minutes ago has been accessed constantly. The cache fills up (before the 5-min mark) due to many other keys being added. What happens to this item, and at what point does each mechanism take over?

<details>
<summary>Answer</summary>

At the fill-up moment: LRU governs — this item is the most recently used, so it is NOT evicted; a colder item is evicted instead. At the 5-minute mark: TTL governs, independent of recency — under the standard non-refreshing default, this item expires and is removed regardless of how hot it's been, unless the cache was built with refresh-on-read TTL. The two mechanisms answer different questions (memory pressure vs staleness) and can each act at different times for the same key.

</details>

---

## Q5. Why is "the cache filled up" not caused by one item being read over and over?

<details>
<summary>Answer</summary>

Filling up is a global capacity event driven by many DIFFERENT keys accumulating over time (other users' data, other products, etc.) — re-reading the same already-cached key doesn't consume any additional space; it's still one slot. Eviction is only triggered when a NEW, distinct key needs space and none is free.

</details>

---

## 30-Second Elevator Pitch

> An eviction policy decides what to remove when the cache is full (a memory-pressure question), while TTL decides what's too stale to trust (a time question) — different axes, often combined. LRU is the default: O(1) via a hashmap (find) plus a doubly-linked list (reorder/evict in O(1)), directly exploiting temporal locality. LFU tracks frequency instead of recency but has a blind spot — stale historical popularity can outlast genuinely active items unless counts decay. FIFO ignores access patterns entirely and is rarely preferred once LRU is available. A standard TTL is a fixed clock from insertion that does not refresh on access by default. "Cache fills up" refers to global capacity across many distinct keys, not repeated access to one key.

---

## Weak Areas to Watch

- Assuming a hashmap alone is sufficient for LRU (missing that ordering requires the linked list)
- Assuming TTL automatically refreshes/extends on every read (it doesn't, by default)
- Distinguishing "cache fills up" (global, many-key event) from "this item is read repeatedly" (doesn't consume new space)
