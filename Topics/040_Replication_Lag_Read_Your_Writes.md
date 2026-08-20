# Topic 040: Replication Lag & Read-Your-Writes

**Module:** 4 — Scaling & Distributing Data
**Tier:** 🔴 MUST
**Completed:** 2026-08-20
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Topic 039 established that asynchronous replication is the common real-world choice, and that it lets followers fall behind the leader. This topic covers exactly what that lag causes in practice, and the specific, named guarantees systems build to hide it from users — this is the single most commonly probed follow-up after any "we use read replicas" answer.

---

## 2. What Replication Lag Actually Is

```
Leader writes at T=0
Follower applies that write at T=0 + lag

Lag can be milliseconds under normal load, or seconds/minutes if:
  - Network latency between leader and follower is high
  - The follower is under heavy read load and can't keep up
  - A large transaction/batch of writes is being replicated
  - The follower's own hardware (disk I/O, CPU) is a bottleneck
```
Lag isn't a fixed number — it varies with load, and it's exactly why "eventually consistent" doesn't mean "consistent after a known, bounded delay."

---

## 3. The Classic Anomaly: Read-Your-Writes Violation

```
User posts a comment → write goes to the LEADER, succeeds
Page reloads → read is load-balanced to a FOLLOWER that hasn't caught up yet
User sees: their own comment is MISSING
```
This is the exact scenario TopicPriority.md flags as "the classic follow-up: user posts then can't see it" — and it's a strong interview signal to name precisely, not just gesture at "eventual consistency."

---

## 4. Three Specific Guarantees Built to Hide Lag (preview of Topic 048)

These aren't abstract — each is a targeted fix for a specific anomaly:

```
READ-YOUR-WRITES (RYW) CONSISTENCY
  Guarantee: a user always sees their OWN writes immediately, even if reads
  generally go to a lagging replica.
  Fix options:
    - Route reads to the LEADER for a short window after that user's own write
    - Track a "last write" timestamp per user; only read from a replica that
      has caught up past that timestamp
    - Route a user's reads to whichever specific replica received their write

MONOTONIC READS
  Guarantee: once a user has seen a value, they never see an OLDER value later.
  Without this: read #1 hits a caught-up replica (sees the new comment), read
  #2 (moments later) hits a MORE-lagging replica (comment disappears again) —
  looks like time ran backwards.
  Fix: route a given user's reads to the SAME replica consistently (sticky
  session routing), so they can only ever move forward in that replica's
  own timeline.

CONSISTENT PREFIX READS
  Guarantee: if writes happen in a causally meaningful order, readers see
  them in that same order.
  Without this (relevant in sharded/multi-leader setups where different
  writes can take different paths): a reader could see the ANSWER to a
  question before seeing the question itself, if the two writes replicate
  out of order to where that reader is reading from.
```
**The pattern:** each guarantee targets one specific, nameable failure mode. "Eventual consistency" is not one problem — it's a family of distinct anomalies, and naming which one you're solving for is what separates a strong answer from a hand-wave.

---

## 5. Cosmetic Staleness vs Correctness Staleness — the Distinction That Actually Matters

**Cosmetic staleness:** a stale read is *displayed* to a user. Nothing happens as a result except "it looks wrong for a moment." The instant the replica catches up, the problem is gone with zero lasting effect. (Example: a tweet not appearing on refresh — annoying, self-resolving.)

**Correctness staleness:** a stale read is *acted upon* — by a human making a real-world decision based on it, or by the system itself automatically. That action has a side effect that **outlives the lag** — it doesn't get undone just because the replica eventually catches up.

### Worked Example — Payment Status, Step by Step
```
T=0    User pays. Leader writes: payment_status = SUCCESS
T=0.1s Client's "check payment status" call gets routed to a lagging replica
       Replica still shows: payment_status = PENDING (hasn't replicated yet)
T=0.1s System/user sees "PENDING", concludes the payment might have failed
T=0.2s → User manually re-enters card details and pays AGAIN
         OR → an automated retry job fires a second charge
T=2s   Replica finally catches up, now correctly shows SUCCESS
       ...but it's too late — TWO charges already went through
```
The replica catching up at T=2s does **not** undo the second charge — the damage is already committed to the world, not just to a screen. That's the tell that separates correctness staleness from cosmetic staleness.

### More Examples, Same Shape (stale read → real action taken → consequence that persists)

- **Inventory oversell:** User A buys the last unit — leader writes `stock=0`. User B's checkout reads `stock` from a lagging replica still showing `stock=1`, and lets them buy it too. Two paid orders now exist for one physical item.
- **Idempotency-key bypass:** A client's request times out after the write actually succeeded, so it retries with the same idempotency key. The "have I seen this key before?" check reads from a lagging replica that hasn't replicated the original write yet, says "no," and processes the request a second time — a duplicate order or charge. (Exactly why idempotency checks, Topic 056, often deliberately read from the leader rather than a replica.)
- **Access-revocation / block-list bypass:** User B blocks Account A. Leader writes `is_banned/blocked = {account_a_id}` into B's block list. Account A immediately messages B; the "is A blocked by B?" check for that message hits a lagging replica that hasn't caught up, returns "not blocked," and the message is delivered to B. B has now actually received a message from someone they explicitly blocked — a real privacy/trust violation that already happened by the time the replica catches up; it can't be un-received.
- **Rate-limit bypass:** A rate limiter increments a request counter on the leader. A near-simultaneous check reads the counter from a lagging replica still showing the old (lower) count, and allows more requests through right when the limit should have kicked in — a real abuse window, not a cosmetic one.

---

## Tech Decision Box: Which Guarantee Do You Actually Need?

```
Need RYW when: a user acts and then immediately checks the result of their
  own action (post a comment, submit a form, make a payment)

Need Monotonic Reads when: a user polls/refreshes repeatedly and must never
  see data go "backwards" (a live feed, a status page)

Need Consistent Prefix Reads when: causally related writes (a question then
  an answer, a message then a reply) must be seen in the right order,
  especially across shards/multi-leader setups

Don't pay for any of these when: reads are anonymous/aggregate and no
  single user's own recent action is being checked (e.g., a public view
  counter — a few seconds of staleness for anyone is fine)
```
**Interview sentence:** "I'd guarantee read-your-writes on the 'my order was placed' confirmation page — route that specific read to the leader briefly after a purchase — but I wouldn't pay that cost on the general product-browsing pages, where any customer seeing a few-second-old view count is harmless."

---

## Common Mistakes

| Mistake | Correction |
|---|---|
| Treating "eventual consistency" as one single problem | It's a family of distinct, nameable anomalies (read-your-writes, monotonic reads, consistent prefix reads) — naming the specific one relevant to a scenario is the actual skill |
| Assuming replication lag is a fixed, predictable number | Lag varies with load, network conditions, and transaction size — there's no guaranteed bound unless a system is explicitly designed to enforce one |
| Applying a strong-consistency fix (e.g., always read from the leader) globally | This defeats the purpose of read replicas for every read, not just the ones that need it — target the specific reads that need the guarantee, not all reads |
| Confusing replication lag with clock skew | Replication lag is about how far behind a *copy of the data* is; clock skew (Topic 049 preview) is about disagreement between machines' *clocks* — different mechanisms, often confused because both cause "wrong-looking" timestamps |
| Giving a "correctness bug beyond UX" example that's actually still just a display glitch | The test is whether an action was TAKEN on the stale read with a consequence that outlives the lag (a duplicate charge, an oversold item, a delivered message to a blocker) — not just "the user saw something wrong for a moment" (e.g., a tweet not appearing on refresh is cosmetic, not a correctness bug) |

---

## Real Interview Questions

1. "A user posts a comment and it disappears on refresh — what's happening, and how would you fix it?" (the canonical read-your-writes scenario)
2. "What's the difference between read-your-writes consistency and monotonic reads?" (tests precise distinction between anomalies)
3. "Would you apply a read-your-writes fix to every read in your system, or just some?" (tests targeted-vs-global thinking)
4. "Why can't you just say a replica is '2 seconds behind' and rely on that number?" (tests understanding that lag is variable, not fixed)
5. "How could replication lag actually cause a financial/correctness bug, not just a UX glitch?" (tests the cosmetic-vs-correctness distinction with a concrete scenario — payment retry, oversell, block-list bypass, rate-limit bypass)

---

## 6. Revision Questions
See `Revision/Revision_040.md`.

## 7. Summary
- Replication lag is the variable delay between a write landing on the leader and becoming visible on a follower — never a fixed, guaranteed number.
- The classic anomaly: a user's own write appears to vanish when their next read hits a lagging replica.
- Three specific, named guarantees fix three specific anomalies: read-your-writes (see your own writes), monotonic reads (never go backwards), consistent prefix reads (causally-ordered writes stay ordered).
- These fixes should be applied to the specific reads that need them, not globally.
- **Cosmetic staleness** (a stale value shown, self-resolves, no lasting effect) is different from **correctness staleness** (a stale value acted upon, causing a consequence — a duplicate charge, an oversold item, a message delivered to a blocker — that persists after the lag resolves).

> **You now can:** explain the read-your-writes anomaly precisely, name and distinguish the three lag-related consistency guarantees, and correctly distinguish a cosmetic staleness glitch from a genuine correctness bug caused by replication lag.
