# Topic 039: Replication

**Module:** 4 — Scaling & Distributing Data
**Tier:** 🔴 MUST
**Completed:** 2026-08-17
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Topic 038 drew the line: horizontally scaling the *stateful* database tier is the hard problem. Replication — keeping copies of the same data on multiple machines — is the first and most fundamental tool for doing that. It's the mechanism behind read scaling, availability, and disaster recovery all at once, and "leader-follower" is vocabulary that shows up in nearly every data-heavy system design.

---

## 2. Why Replicate at All

```
1. AVAILABILITY  — one machine dies, a replica still has the data
2. READ SCALING  — spread read traffic across multiple copies
3. GEOGRAPHIC LOCALITY — keep a copy close to users in each region (lower latency)
4. DISASTER RECOVERY — a replica in another datacenter survives a regional outage
```
Every one of these is a real production motivation — replication isn't just "for redundancy," it's frequently adopted purely for read-scaling even on a system that would otherwise never lose a machine.

---

## 3. The Three Replication Topologies

```
LEADER-FOLLOWER (a.k.a. primary-replica, master-slave)
  All writes go to ONE leader. Leader propagates changes to N followers.
  Followers serve reads (and can promote to leader if the leader dies).

  Client ──write──▶ Leader ──replicate──▶ Follower 1
                        │  ──replicate──▶ Follower 2
                        │  ──replicate──▶ Follower 3
  Client ──read───▶ any Follower (or the Leader)

MULTI-LEADER (multi-master)
  More than one node accepts writes; each propagates to the others.
  Used across multiple datacenters — each region has a local leader for
  low-latency local writes, syncing with the others asynchronously.

LEADERLESS (Topic 005/028 preview — Cassandra/DynamoDB style)
  ANY node accepts a write; the client (or a coordinator) writes to
  multiple nodes directly and reads from multiple nodes, reconciling
  differences (quorums, Topic 050 preview).
```

---

## 4. Leader-Follower: Sync vs Async Replication

This is the tradeoff that actually gets probed in interviews:
```
SYNCHRONOUS replication
  Leader waits for follower(s) to confirm the write before acking the client.
  + No data loss if the leader dies right after acking (follower has it too)
  - Write latency = leader's write + network round-trip to follower
  - If the follower is slow/down, writes STALL (or the leader must decide
    to proceed anyway, breaking the guarantee)

ASYNCHRONOUS replication
  Leader acks the client immediately, replicates to followers in the background.
  + Fast writes — no waiting on followers
  - If the leader dies before replication completes, that write is LOST,
    even though the client was told it succeeded

SEMI-SYNCHRONOUS (the common real-world compromise)
  Leader waits for confirmation from AT LEAST ONE follower (not all), acks,
  then replicates to the rest asynchronously. Bounds data loss to "at most
  what wasn't yet sent to any follower" while keeping latency bounded to
  one round-trip, not N.
```
**The direct tradeoff:** synchronous trades latency for durability; asynchronous trades durability for latency. Semi-sync is the practical middle ground most real systems (e.g., MySQL semi-sync, many Postgres setups) actually run.

---

## 5. What Followers Are Actually For

A common shallow answer treats followers as "just backups." In practice:
- **Read replicas** absorb read traffic so the leader isn't overloaded — this is often the *primary* reason a team adds replication, well before any failure ever happens.
- **Failover targets** — if the leader dies, a follower is promoted (manually or via automated leader election, Topic 051 preview).
- **Isolated workloads** — analytics/reporting queries can run against a replica so they never compete with production traffic on the leader.

---

## 6. The Cost This Sets Up (preview)

Because followers apply writes asynchronously (in the common case), they can fall behind the leader — this is **replication lag**, covered fully next in Topic 040. Every read-replica architecture inherits this tradeoff: better read scalability, at the cost of followers potentially serving slightly stale data.

---

## Tech Decision Box: Leader-Follower vs Multi-Leader vs Leaderless

```
Use LEADER-FOLLOWER when:
  - Single-region (or single-primary-region) system
  - Strong consistency for writes matters, reads can tolerate slight staleness
  - Default choice for most relational databases (Postgres, MySQL)

Use MULTI-LEADER when:
  - Multiple regions each need low-latency LOCAL writes
  - Some write conflicts are acceptable/resolvable (Topic 057 preview, CRDTs)
  - Example: a collaborative app with regional offices each writing locally

Use LEADERLESS when:
  - Massive write throughput with tunable consistency is the priority
  - Cassandra/DynamoDB (Topic 029) — no single write bottleneck at all
```
**Interview sentence:** "I'd use leader-follower replication for the primary orders database — writes need strong consistency, and read replicas can absorb the reporting/analytics load. I would NOT use leaderless here, since orders need a clear, unambiguous write path, not tunable eventual consistency."

---

## Common Mistakes

| Mistake | Correction |
|---|---|
| Treating replicas as "just backups" | Read-scaling is frequently the primary motivation for replication, adopted long before any failure ever occurs |
| Assuming synchronous replication has no cost | It adds a full network round-trip to every write's latency, and a slow/down follower can stall writes entirely |
| Assuming asynchronous replication never loses data | If the leader dies before an async write reaches any follower, that acknowledged write is gone — this is exactly why semi-sync exists as a middle ground |
| Confusing leader-follower with leaderless replication | Leader-follower has one write path (the leader); leaderless lets any node accept writes with reconciliation via quorums — very different consistency and availability tradeoffs |

---

## Real Interview Questions

1. "Why would you add read replicas to a database that's never had an availability problem?" (tests whether read-scaling is understood as a primary motivation, not just failover insurance)
2. "What's the tradeoff between synchronous and asynchronous replication?" (latency vs durability, core interview question)
3. "If your leader dies mid-write under asynchronous replication, what happens to that write?" (tests understanding of the actual data-loss mechanism)
4. "When would you reach for multi-leader replication instead of leader-follower?" (tests multi-region reasoning)
5. "How is leaderless replication (Cassandra-style) fundamentally different from leader-follower?" (ties back to Topic 005/028/029)

---

## 7. Revision Questions
See `Revision/Revision_039.md`.

## 8. Summary
- Replication exists for availability, read scaling (frequently the primary real-world motivation), geographic locality, and disaster recovery.
- Three topologies: leader-follower (one write path, default for relational DBs), multi-leader (multiple regional write paths, needs conflict resolution), leaderless (any node writes, reconciled via quorums — Cassandra/DynamoDB).
- Synchronous replication trades write latency for zero data loss; asynchronous trades data-loss risk for fast writes; semi-synchronous is the common practical middle ground.
- Followers are actively used for read-scaling and isolating analytics load, not just sitting idle as failover insurance.
- Async replication's follower lag is the direct setup for Topic 040 (Replication Lag).

> **You now can:** name multiple motivations for replication, explain the sync/async/semi-sync tradeoff precisely including the exact mechanism by which async can lose data, and choose the right replication topology for a given scenario.
