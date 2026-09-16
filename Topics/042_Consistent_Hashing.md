# Topic 042: Consistent Hashing

**Module:** 4 — Scaling & Distributing Data
**Tier:** 🔴 MUST
**Completed:** 2026-09-16
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Topic 041 ended on a specific, named problem: naive `hash(key) % N` partitioning reshuffles almost every key's shard assignment the moment N changes — adding or removing one shard forces a massive, disruptive data migration across the whole cluster. Consistent hashing is the specific technique that fixes exactly this, and it's asked about by name often enough (and is itself the basis of a dedicated case study, #7 Key-Value Store) that it earns its own topic.

---

## 2. The Problem, Precisely

```
4 shards, hash(key) % 4:
  key "alice" → hash=17 → 17 % 4 = 1 → Shard 1
  key "bob"   → hash=22 → 22 % 4 = 2 → Shard 2

Add a 5th shard, hash(key) % 5:
  key "alice" → hash=17 → 17 % 5 = 2 → Shard 2  (moved!)
  key "bob"   → hash=22 → 22 % 5 = 2 → Shard 2  (moved!)

Nearly EVERY key's target shard changes when N changes from 4 to 5 —
not just the ~1/5 share that logically needs to move to the new shard.
```
This is expensive and disruptive at real scale: every existing shard has to ship most of its data to other shards just because one new machine joined.

---

## 3. The Consistent Hashing Idea: a Hash Ring

```
Both KEYS and NODES (shards) are hashed onto the SAME circular space
(e.g., 0 to 2^32 - 1), visualized as a ring:

              Node A (hash=10)
             /              \
   Node D (hash=280)      Node B (hash=90)
             \              /
              Node C (hash=190)

RULE: a key belongs to the FIRST node found walking clockwise from
      the key's own hash position on the ring.

key "alice" hashes to 150 → walk clockwise → first node hit is Node C (190)
  → alice belongs to Node C
```

---

## 4. Why This Fixes the Reshuffling Problem

```
Adding Node E at hash=200 (between C at 190 and D at 280):

ONLY the keys that fall between C (190) and E (200) move — from
whichever node used to own that small slice (previously D, since D
was the next node clockwise past C) — to the new Node E.

Every other key on the ring is COMPLETELY UNAFFECTED. Keys between
A and B still go to B. Keys between B and C still go to C. Nothing
about their assignment changed.
```
```
Removing Node C:
  Only the keys that WERE assigned to C (between B and C) now fall
  through to the next node clockwise (D) — everyone else is untouched.
```
**The core property:** adding or removing one node only moves the keys that were "local" to that node's slice of the ring — roughly `1/N` of all keys, not nearly all of them. This is the entire reason consistent hashing exists.

**Worked numeric check (illustrating why an example must actually cross a boundary):** with N=3, a key at hash=13 lands on `13 % 3 = 1`. Going to N=4, the same key lands on `13 % 4 = 1` too — this particular key happens to stay put across that specific resize, even though the *general* claim (most keys move under naive `% N`) still holds for other keys. A cleaner illustrative example: hash=15 → `15 % 3 = 0`, `15 % 4 = 3` — this one does move. The lesson: pick a concrete example and actually verify the arithmetic before using it to make a point, especially live in an interview.

---

## 5. The Real-World Wrinkle: Uneven Load Without Virtual Nodes

If each physical node gets exactly one point on the ring, node placement is essentially random — some nodes end up owning huge arcs of the ring (lots of keys) while others own tiny slivers (almost no keys), purely by the luck of where their hash landed. This is a real, practical problem, not a theoretical one.

```
FIX: VIRTUAL NODES

Instead of placing each physical node at ONE point on the ring, place
it at MANY points (e.g., 100-200 virtual points per physical node),
each INDEPENDENTLY HASHED (e.g., hashing "NodeA-1", "NodeA-2", ...,
"NodeA-150" as separate labels, each landing at its own random ring
position).

Node A → virtual points at hash 10, 340, 890, 1200, ... (150 points)
Node B → virtual points at hash 45, 200, 670, 1500, ... (150 points)

Now each physical node "owns" many small, scattered arcs instead of
one big contiguous one — the LAW OF LARGE NUMBERS smooths out the
uneven-ownership problem: with enough independently-random points,
each node's total share converges toward a fair 1/N even though any
individual arc is still random. The balance comes from HAVING MANY
random points, not from deliberately spacing them evenly — evenly
spacing them would require knowing all future node positions in
advance, which defeats the purpose.

BONUS: when a node is ADDED or REMOVED, its virtual points' load is
now spread across MANY other nodes instead of dumping entirely onto
whichever single node was "next" on the ring — this smooths out the
rebalancing load too, not just the steady-state distribution.
```
This is exactly what Topic 036 previewed with Redis Cluster's 16,384 fixed hash slots — a related but distinct approach (fixed slot count assigned to shards, rather than a continuous ring with virtual points) solving the same underlying problem: even distribution without full reshuffling on membership change.

---

## 6. What Consistent Hashing Does NOT Solve

It's purely a **key-placement** technique — it doesn't solve:
- **Hot keys** (Topic 043 preview) — a single extremely popular key still overloads whichever one node/slice owns it, regardless of how evenly the ring is balanced overall.
- **Data migration itself** — when ownership changes, the actual bytes still have to be physically copied to the new owner; consistent hashing only minimizes *how much* needs to move, not the mechanics of moving it (Topic 044, Rebalancing).
- **Replication** — consistent hashing decides which node owns a key; a separate decision (Topic 039) decides how many copies of that key exist for availability. Real systems (Cassandra, DynamoDB) commonly replicate each key to the N nodes immediately following it clockwise on the ring, combining both ideas.

---

## Tech Decision Box: Consistent Hashing vs Fixed Hash Slots vs Naive `% N`

```
Use NAIVE hash(key) % N when:
  - The number of nodes will never change (rare in practice) — otherwise avoid

Use CONSISTENT HASHING (ring + virtual nodes) when:
  - Node count changes over time and minimizing data movement on
    add/remove matters — Cassandra, DynamoDB's core approach

Use FIXED HASH SLOTS (Redis Cluster style, Topic 036) when:
  - You want simpler, more predictable operational reasoning (a fixed,
    small number of slots explicitly reassigned during resharding)
    rather than a continuous ring — a related but more constrained variant
```
**Interview sentence:** "I'd use consistent hashing with virtual nodes for the shard-assignment layer — it means adding or removing a node only reshuffles roughly 1/N of the keys instead of nearly all of them, and virtual nodes keep the load balanced across physical nodes even though ring placement is otherwise random."

---

## Common Mistakes

| Mistake | Correction |
|---|---|
| Describing consistent hashing without mentioning virtual nodes | Without virtual nodes, physical-node placement on the ring is effectively random and produces uneven load — virtual nodes are what makes the even-distribution property actually hold in practice, not an optional extra |
| Believing consistent hashing eliminates data movement entirely | It minimizes movement to roughly the keys "local" to the node that joined/left (~1/N of all keys), not zero — some data always has to move when membership changes |
| Assuming consistent hashing solves hot keys | It solves uneven *node* load from membership changes, not a single overwhelmingly popular *key* landing on one node — that's a shard-key-choice problem (Topic 043) |
| Conflating consistent hashing with replication | Consistent hashing decides which node owns a key; replication (a separate decision) decides how many copies exist for availability — real systems combine both (e.g., replicate to the next N nodes clockwise) |
| Describing virtual node placement as deliberately "equidistant"/evenly spaced | Virtual nodes are placed via many INDEPENDENTLY HASHED (effectively random) points, not engineered even spacing — the balancing effect comes from the law of large numbers over many random points, not from deliberate uniform placement (which would require knowing all future node positions in advance) |
| Using an unverified numeric example to illustrate why `%N` breaks | Always check the arithmetic — a poorly chosen example (like a key that happens to land on the same node number before and after a resize) can accidentally undercut the point being made, even though the general principle still holds |

---

## Real Interview Questions

1. "Why does naive `hash(key) % N` partitioning break down when you add a node, and how does consistent hashing fix it?" (the core mechanism)
2. "What problem do virtual nodes solve in consistent hashing, and why is plain consistent hashing insufficient without them?"
3. "Does consistent hashing solve the hot-key problem?" (tests the boundary of what it actually solves)
4. "How would you combine consistent hashing with replication in a system like Cassandra?" (tests understanding they're separate, composable decisions)
5. "Adding a node to a consistent-hashing ring — roughly how much data has to move, and why?" (tests the ~1/N quantitative intuition)

---

## 7. Revision Questions
See `Revision/Revision_042.md`.

## 8. Summary
- Naive `hash(key) % N` partitioning reshuffles nearly all keys whenever N changes — expensive and disruptive at scale.
- Consistent hashing places both keys and nodes on a hash ring; a key belongs to the first node found walking clockwise from its position.
- Adding or removing one node only moves the keys "local" to that node's slice (~1/N of all keys) — everything else is untouched, which is the entire point of the technique.
- Virtual nodes (each physical node placed at many INDEPENDENTLY HASHED, effectively random ring points — not evenly spaced) are necessary in practice to smooth out otherwise-uneven load via the law of large numbers, and to spread rebalancing load across many nodes rather than dumping it on one.
- Consistent hashing does NOT solve hot keys, the mechanics of actually moving data, or replication — those are separate, composable concerns (Topics 043, 044, 039).

> **You now can:** explain precisely why naive modulo hashing breaks on resize (with a correctly verified example), describe the hash-ring mechanism and why it minimizes data movement, explain why virtual nodes are placed via independent hashing rather than deliberate spacing, and correctly state what consistent hashing does and doesn't solve.
