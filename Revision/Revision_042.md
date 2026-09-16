# Revision — Topic 042: Consistent Hashing

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-09-16

---

## Q1. Explain why naive `hash(key) % N` breaks down when N changes, with a concrete (verified) example.

<details>
<summary>Answer</summary>

Changing N changes the divisor in `hash(key) % N` for every key, so most keys land on a different shard number even though only one machine was added or removed — not just the fair 1/N share that logically needs to move. Example: hash=15, N=3 → 15%3=0; N=4 → 15%4=3 — this key moves. (Careful: not every example moves — hash=13 gives 13%3=1 and 13%4=1, staying put for that specific key even though the general principle holds across the keyspace as a whole. Always verify the arithmetic before using an example to make the point.)

</details>

---

## Q2. Describe how the hash ring works — how is a key assigned to a node?

<details>
<summary>Answer</summary>

Both keys and nodes are hashed onto the same circular hash space (e.g., 0 to 2^32-1). A key belongs to the first node encountered walking clockwise from the key's own hash position on the ring.

</details>

---

## Q3. When a node is added to the ring, which keys move, and which don't?

<details>
<summary>Answer</summary>

Only the keys that fall in the arc between the previous node (counter-clockwise neighbor) and the newly added node move — and they move from whichever node used to own that slice (the node that was previously next clockwise) to the new node. Every other key's assignment is completely unaffected.

</details>

---

## Q4. Why are virtual nodes necessary, and what specific problem do they solve?

<details>
<summary>Answer</summary>

With one ring point per physical node, placement is effectively random, so some nodes end up owning large arcs (lots of keys) while others own tiny slivers — a real load-imbalance problem. Virtual nodes place each physical node at many (100-200+) independently-hashed, effectively random points on the ring instead of one; the law of large numbers then smooths the total share each physical node ends up with toward a fair 1/N, even though any individual arc is still random. This also spreads rebalancing load across many nodes on add/remove, rather than dumping it all on one neighbor.

</details>

---

## Q5. Name two things consistent hashing does NOT solve on its own.

<details>
<summary>Answer</summary>

Hot keys (a single very popular key still overloads whichever one node/slice owns it, regardless of ring balance) and replication (consistent hashing only decides which node owns a key — how many copies exist for availability is a separate decision, though real systems often combine both by replicating to the next N nodes clockwise). The actual mechanics of physically moving data during rebalancing is also a separate concern (Topic 044).

</details>

---

## 30-Second Elevator Pitch

> Naive `hash(key) % N` reshuffles nearly all keys whenever N changes, because the divisor itself changes for every key. Consistent hashing fixes this by placing both keys and nodes on a hash ring — a key belongs to the first node clockwise from its position — so adding or removing one node only moves the ~1/N of keys local to that node's slice, leaving everything else untouched. In practice this needs virtual nodes: each physical node gets many independently-hashed, effectively random ring points (not evenly spaced — that would require knowing future node positions), and the law of large numbers averages out load across enough random points. Consistent hashing is purely a key-placement technique: it doesn't solve hot keys (Topic 043), the mechanics of actually moving data (Topic 044), or replication (Topic 039) — those are separate, composable concerns.

---

## Weak Areas to Watch

- First attempt at a numeric example for Q1 contained an arithmetic slip (claimed 13%4=2 when it's actually 1) — confirmed as a typo, not a conceptual gap, but worth double-checking quick modular arithmetic live rather than assuming it's right.
- First description of virtual node placement said "equidistant" — corrected immediately to "independently hashed / effectively random, balanced via the law of large numbers" once the "equidistant defeats the purpose" contradiction was pointed out.
