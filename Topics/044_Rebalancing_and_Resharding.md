# Topic 044: Rebalancing & Resharding

**Module:** 4 — Scaling & Distributing Data
**Tier:** 🟡 SKIM
**Completed:** 2026-09-19
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Topic 042 (Consistent Hashing) ended by explicitly punting on this: consistent hashing tells you *which* keys now belong to a new node, but it doesn't move the actual bytes. Rebalancing is the operational mechanics of executing that move — safely, on a live system, with zero data loss and minimal downtime.

---

## 2. Two Ways Systems Actually Organize Rebalancing

**Fixed number of partitions (the common real-world approach):** pre-split data into far more logical partitions than current physical nodes (e.g., 1,000 partitions across 10 nodes = 100 partitions/node). Rebalancing = reassigning *whole partitions* to different nodes, not recomputing per-key boundaries. Moving a whole partition is operationally simple: copy it, verify, cut over. This is how MongoDB (chunks) and Kafka (partitions) actually operate.

**Dynamic partitioning:** partitions split automatically when they grow past a size threshold (like a B-tree node splitting) and merge when they shrink. More adaptive to real data volume, but the split/merge logic itself is more complex than a fixed, pre-decided partition count.

---

## 3. The Live-Migration Mechanics

```
1. New node joins → gets assigned some partitions/key-range (from consistent hashing, 042)
2. Background copy: bulk-copy that data from the current owner(s) → new owner (can take time at scale)
3. During the copy: the CURRENT owner keeps serving reads/writes for that data — it's still authoritative
4. Catch-up: replicate any writes that landed during the copy window, so the new owner converges
5. Cutover: once the new owner is fully caught up, atomically switch routing — it becomes authoritative
6. Cleanup: delete the now-redundant data from the old location
```
**The key operational principle:** the current owner stays authoritative until the new owner has fully caught up. Cutting over mid-copy means the new owner is only partially populated — reads for not-yet-migrated data would fail or return nothing.

---

## 4. Where the Copy Actually Comes From (a precision worth getting right)

When rebalancing is triggered by a genuinely failed node (not just adding capacity), it's tempting to describe the migration as "copying the dead node's data to a new node" — but a dead/unreachable node cannot serve as a copy source at all. In a properly replicated system (Topic 039/041 — every partition already has replication factor > 1, e.g. 3 copies across 3 nodes), the failed node was never the *only* copy of its data. The actual flow is:

```
Surviving replica(s) → new replacement node

NOT: dead node → new node (impossible — it's unreachable)
```

The rebalancer copies from the OTHER nodes that already hold a copy of each of the failed node's partitions, restoring the replication factor back to its configured level. This is a direct, concrete illustration of why per-shard/per-partition replication's primary job (Topic 041) is making a node's data *survive* that node dying: the only reason there's anything to rebalance instead of permanent data loss is that the data already existed elsewhere. If a partition had replication factor 1 (sharded but never replicated), losing its one node would mean that slice of data is genuinely gone or unavailable until that specific node recovers — there would be nothing for any other node to copy from.

---

## 5. A Critical Operational Guardrail

Rebalancing should generally be **operator-triggered or gated by a grace period, not fully automatic-and-immediate**. If a node has a brief hiccup (a GC pause, a flaky network blip) lasting seconds, an automatic rebalancer that reacts instantly can kick off a massive, unnecessary data migration.

**Worked example:** a 10-node cluster, each node holding ~500GB. Node 7 misses a few heartbeats due to a 15-second GC pause. If rebalancing is fully automatic, the cluster immediately treats Node 7 as gone and starts copying its ~500GB (from surviving replicas, per §4) across the other 9 nodes — consuming real CPU/disk/network on those *healthy* nodes and degrading their normal traffic-serving during the copy. Then Node 7 comes back online 15 seconds later, completely fine — but its data has already been reassigned elsewhere, so either another rebalance is needed to move work back, or the now-redundant copies must be reconciled/discarded. A 15-second blip triggers minutes of expensive, unnecessary cluster-wide churn.

**Fix:** require sustained unreachability (e.g., 5 minutes) before treating a node as actually gone. A short blip never crosses that threshold — nothing moves. Only a genuinely dead node triggers the real, necessary rebalance.

---

## 6. Decision Box

```
Fixed partitions (over-partition upfront) → simpler ops, whole-partition moves,
  but must guess partition count upfront (too few = coarse rebalancing later)
Dynamic partitioning → adapts to real data growth automatically, but the
  split/merge logic itself is more operationally complex
Always: keep the current owner authoritative until the new owner fully catches
  up, copy from SURVIVING REPLICAS (never the failed node itself), and gate
  large rebalances behind a grace period or explicit operator action —
  not automatic reaction to transient node issues
```

---

## 7. Common Mistakes

| Mistake | Correction |
|---|---|
| Treating rebalancing as instant/free once consistent hashing decides ownership | The actual bytes still have to be copied — this can take real time at scale, during which the old owner must remain authoritative |
| Making rebalancing fully automatic | A transient/slow node can trigger unnecessary, expensive data movement — most systems require a grace period or explicit operator action for large rebalances |
| Cutting over before the new owner has caught up on in-flight writes | Any writes that landed during the copy window must be replicated to the new owner before cutover, or they're silently lost |
| Describing a failed-node rebalance as copying data FROM the dead node | An unreachable node cannot serve as a copy source — the data comes from surviving replicas that already held a copy (this only works because of replication factor > 1, Topic 041) |

---

## 8. Revision Questions
See `Revision/Revision_044.md`.

## 9. Summary
- Consistent hashing (042) decides *which* keys move; rebalancing is the operational mechanics of actually moving the bytes, live, without downtime or data loss.
- Two organizing approaches: fixed over-partitioning (simple whole-partition moves — MongoDB, Kafka) vs. dynamic split/merge partitioning (adapts to data volume, more complex logic).
- Live migration: current owner stays authoritative during the background copy, catches up any writes made during the copy window, then cutover happens atomically.
- When rebalancing after a node failure, the copy source is always a **surviving replica**, never the failed node itself — a direct, concrete reason per-partition replication (Topic 041) exists: it's what makes data survivable at all.
- Rebalancing should be gated by a grace period or operator action, not fully automatic — a transient blip shouldn't trigger a massive, unnecessary data migration.
