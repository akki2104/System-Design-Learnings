# Revision — Topic 040: Replication Lag & Read-Your-Writes

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-08-20

---

## Q1. Walk through the classic read-your-writes violation scenario end to end.

<details>
<summary>Answer</summary>

A user posts a comment; the write goes to the leader and succeeds. The page reloads and the read is load-balanced to a follower that hasn't yet replicated that write. The user sees their own comment missing, even though it was successfully saved.

</details>

---

## Q2. Name the three lag-related consistency guarantees and the specific anomaly each one fixes.

<details>
<summary>Answer</summary>

Read-your-writes: guarantees a user always sees their own writes immediately. Monotonic reads: guarantees a user never sees an older value after having seen a newer one (prevents "time running backwards" from hitting differently-lagged replicas). Consistent prefix reads: guarantees causally-ordered writes are seen in that same order by readers, relevant across shards/multi-leader setups.

</details>

---

## Q3. Why is "the replica is 2 seconds behind" not a reliable number to design around?

<details>
<summary>Answer</summary>

Replication lag is a variable delay, not a fixed one — it changes with network conditions, follower load, and transaction/batch size. There's no guaranteed bound unless a system is explicitly engineered to enforce one, so designing around an assumed fixed lag number is unsafe.

</details>

---

## Q4. Why would you apply a read-your-writes fix only to certain reads, not globally?

<details>
<summary>Answer</summary>

Applying strong consistency (e.g., always reading from the leader) to every read defeats the purpose of having read replicas at all — it pushes all read load back onto the leader. The fix should target only the specific reads that actually need the guarantee (e.g., a user checking their own just-submitted order), while reads that tolerate staleness (e.g., a public view/review count) keep using replicas normally.

</details>

---

## Q5. Give a concrete example where replication lag causes a real correctness problem, not just a visible UX glitch.

<details>
<summary>Answer</summary>

The test: was an action TAKEN on the stale read, with a consequence that outlives the lag? Examples: (1) A payment status check hits a lagging replica showing "pending" when it's actually "success," so the user or an automated job retries and causes a duplicate charge. (2) User B blocks Account A; a lagging replica still shows "not blocked" when Account A messages B, so the message is delivered — B has now actually received a message from someone they blocked, which can't be undone once the replica catches up. (3) Inventory oversell: a lagging replica still shows stock available after the last unit sold, letting a second customer also purchase it. In each case, the damage (money moved, a message delivered, a promise made) persists even after the replica catches up — unlike a tweet not appearing on refresh, which self-resolves with no lasting effect.

</details>

---

## 30-Second Elevator Pitch

> Replication lag is the variable delay between a write landing on the leader and becoming visible on a follower — never fixed, always load-dependent. The classic anomaly is a user's own write appearing to vanish when their next read hits a lagging replica. Three named guarantees fix three distinct anomalies: read-your-writes (see your own writes), monotonic reads (never go backwards), consistent prefix reads (causally-ordered writes stay ordered) — apply each only to the specific reads that need it, not globally. The sharper distinction is cosmetic staleness (a stale value shown, self-resolving, harmless) versus correctness staleness (a stale value acted upon — a duplicate charge, an oversold item, a message delivered to someone who blocked the sender — causing a consequence that outlives the lag itself).

---

## Weak Areas to Watch

- First attempt at "a correctness bug beyond UX" reached for another display-only glitch (a tweet not appearing) instead of a scenario where the stale read was actually acted upon with a lasting consequence. Corrected cleanly on a second attempt (blocked-user message delivery) once the cosmetic-vs-correctness distinction was explained in depth — worth a quick re-check next revision pass to confirm it's now independently reachable, not just recognizable when re-explained.
