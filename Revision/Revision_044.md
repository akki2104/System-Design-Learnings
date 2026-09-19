# Revision — Topic 044: Rebalancing & Resharding

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-09-19

---

## Q1. What's the difference between fixed-partition rebalancing and dynamic partitioning?

<details>
<summary>Answer</summary>

Fixed partitions: pre-split data into far more logical partitions than current physical nodes; rebalancing reassigns whole partitions between nodes (simple copy-verify-cutover). Dynamic partitioning: partitions automatically split when they grow past a size threshold and merge when they shrink, adapting to real data volume but with more complex split/merge logic.

</details>

---

## Q2. Why must the current owner stay authoritative during a background copy, rather than cutting over immediately?

<details>
<summary>Answer</summary>

The new owner is only partially populated mid-copy. Cutting over before the copy finishes means reads for not-yet-migrated data would fail or return nothing. The current owner must remain authoritative until the new owner has fully caught up, including any writes that landed during the copy window.

</details>

---

## Q3. When rebalancing after a node failure, where does the copied data actually come from — and why can't it come from the failed node?

<details>
<summary>Answer</summary>

It comes from surviving replicas — other nodes that already held a copy of the failed node's partitions (because replication factor was > 1, Topic 041). A failed/unreachable node cannot serve as a copy source at all. This only works because the data already existed elsewhere; if a partition had replication factor 1, losing its one node would mean that data is genuinely gone or unavailable until that node recovers.

</details>

---

## Q4. Why is fully automatic, immediate rebalancing often a bad default in production?

<details>
<summary>Answer</summary>

A transient issue (a brief GC pause, a flaky network blip) can make a node miss heartbeats for a few seconds without it actually being dead. If rebalancing reacts immediately, it triggers a large, unnecessary data migration — consuming CPU/disk/network on healthy nodes and degrading their real traffic — for a problem that would have resolved itself. The fix is gating rebalancing behind a grace period (sustained unreachability) or explicit operator confirmation, so only genuine failures trigger the real, necessary migration.

</details>

---

## 30-Second Elevator Pitch

> Consistent hashing decides which keys need to move; rebalancing is the operational mechanics of actually moving them live, without downtime or data loss. Systems either pre-split into many fixed partitions (simple whole-partition moves) or dynamically split/merge partitions as data grows. During migration, the current owner stays authoritative until the new owner fully catches up, then cutover happens atomically. When rebalancing follows a node failure, the copy source is always a surviving replica — never the failed node itself, since it's unreachable — which is a direct, concrete reason per-partition replication exists at all: it's what makes the data survivable. Rebalancing should be gated by a grace period or operator action, not fully automatic, since a short transient blip shouldn't trigger a large unnecessary migration.

---

## Weak Areas to Watch

- None this session — clean pass, plus a genuinely sharp follow-up question that caught an imprecision in the mentor's own explanation (the copy source during a failure-triggered rebalance).
