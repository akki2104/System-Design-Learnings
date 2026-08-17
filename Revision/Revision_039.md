# Revision — Topic 039: Replication

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-17

---

## Q1. Name three motivations for replication beyond "in case a machine dies."

<details>
<summary>Answer</summary>

Read scaling (spreading read traffic across multiple copies), geographic locality (keeping a copy close to users in each region for lower latency), and disaster recovery (a replica in another datacenter survives a regional outage).

</details>

---

## Q2. Explain the difference between synchronous, asynchronous, and semi-synchronous replication, and the tradeoff each makes.

<details>
<summary>Answer</summary>

Synchronous: the leader waits for follower(s) to confirm before acking the client — no data loss if the leader dies right after, but adds a full round-trip to write latency and a slow follower can stall writes. Asynchronous: the leader acks immediately and replicates in the background — fast writes, but an acknowledged write is lost if the leader dies before it reaches any follower. Semi-synchronous: the leader waits for confirmation from at least one follower, then replicates to the rest asynchronously — bounds the data-loss window while keeping latency to one round-trip, the common real-world compromise.

</details>

---

## Q3. Under asynchronous replication, describe exactly how an acknowledged write can be lost.

<details>
<summary>Answer</summary>

The leader applies the write locally and acks the client immediately, before the write has been sent to (or received/applied by) any follower. If the leader then dies before that replication happens, no follower ever received the write — the client was told it succeeded, but the only copy that ever existed is gone with the leader.

</details>

---

## Q4. What are followers actually used for in a real system, beyond being a failover target?

<details>
<summary>Answer</summary>

Read replicas absorb read traffic so the leader isn't overloaded (often the primary reason replication is added, before any failure ever happens), and isolated workloads — analytics/reporting queries run against a replica so they never compete with production traffic on the leader.

</details>

---

## Q5. Give one scenario where multi-leader replication is the right choice over leader-follower.

<details>
<summary>Answer</summary>

A system serving users across multiple geographic regions, where each region needs low-latency LOCAL writes. With a single leader, users far from that leader's region would experience high write latency; multi-leader gives each region its own local leader, syncing with the others asynchronously (accepting the cost of resolving occasional write conflicts).

</details>

---

## 30-Second Elevator Pitch

> Replication exists for availability, read scaling (often the primary real-world motivation), geographic locality, and disaster recovery. Leader-follower (one write path, default for relational DBs), multi-leader (multiple regional write paths, needs conflict resolution), and leaderless (any node writes, reconciled via quorums) are the three topologies. Synchronous replication trades write latency for zero data loss; asynchronous trades data-loss risk for fast writes — an acknowledged write is lost if the leader dies before any follower received it; semi-synchronous is the practical middle ground. Followers actively absorb read traffic and isolate analytics load, not just sit idle as failover insurance. Async replication's follower lag sets up Topic 040.

---

## Weak Areas to Watch

- None logged — clean 5/5 pass, all mechanisms (sync/async/semi-sync tradeoffs, the exact async data-loss mechanism, multi-leader's latency motivation) correctly reasoned, despite some answers being a bit loosely phrased in the moment.
