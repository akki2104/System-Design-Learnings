# Revision — Topic 098: Unique ID Generation (Snowflake, ULID)

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-09-21

---

## Q1. Why can't you just use auto-increment IDs once you've sharded your database?

<details>
<summary>Answer</summary>

Auto-increment relies on a single counter that every insert must coordinate with to get the next value. Once data is sharded across multiple machines, that single counter becomes exactly the kind of centralized bottleneck sharding was meant to eliminate — every shard's writes would have to funnel through one coordination point. Decentralized alternatives (Snowflake, ULID) or a reduced-coordination compromise (range/block allocation) are needed instead.

</details>

---

## Q2. Break down the structure of a Snowflake ID and explain why timestamp occupies the most significant bits.

<details>
<summary>Answer</summary>

64 bits: [41-bit timestamp (ms since a custom epoch)][10-bit machine/worker ID][12-bit sequence number]. Timestamp is most significant because that's what determines whether numeric ID order approximates GLOBAL chronological order across the whole system. If machine ID were most significant instead, numeric sort would group IDs by which machine generated them first, with timestamp only breaking ties within one machine's own stream — an ID created earlier in real time on a higher-numbered machine could sort as numerically larger than a later ID from a lower-numbered machine, breaking global ordering.

</details>

---

## Q3. How is a UUID (v4) generated, and why is its collision probability considered negligible?

<details>
<summary>Answer</summary>

128 bits total, with 6 bits reserved for version/variant markers, leaving 122 bits of true randomness from a cryptographically secure random number generator — no clock, no machine ID, no coordination. Collision probability follows birthday-paradox math over a 2^122 space: a 50% chance of any collision requires generating roughly 2^61 (≈2.1 × 10^18) UUIDs, far beyond what any real system generates over its lifetime — so it's treated as zero in practice, though not mathematically impossible.

</details>

---

## Q4. What's the real cost of using random UUIDs as primary keys, beyond their size?

<details>
<summary>Answer</summary>

Random UUIDs used as a primary key/index mean every insert lands at a random point in the B-tree rather than at the end, causing constant page splits and poor locality — a real, measurable write-performance cost at scale (ties to Topics 022/023), not just a style preference.

</details>

---

## Q5. What happens to Snowflake ID generation if a machine's clock jumps backward (e.g., an NTP step correction), and how do real implementations guard against it?

<details>
<summary>Answer</summary>

Since Snowflake reads "current time in ms" as its most-significant bits, a backward clock jump risks the generator reusing a timestamp it already used — potentially minting a duplicate ID (if the sequence number also coincides) or at minimum producing an ID numerically smaller than one generated earlier in real time, breaking sortability. Real implementations track the last timestamp used and, if a new reading comes back lower, either wait until the clock catches back up past that value or error out, rather than silently generating a bad ID.

</details>

---

## 30-Second Elevator Pitch

> Sharding breaks the single auto-increment counter, so decentralized ID generation is needed instead. Random UUID v4 uses 122 bits of randomness — collision-negligible by birthday-paradox math, but not sortable and costly for B-tree inserts. Snowflake packs a 64-bit ID as [timestamp][machine ID][sequence], with timestamp in the most significant bits specifically so numeric sort approximates GLOBAL chronological order (machine-ID-first would only sort within one machine's own stream) — the tradeoff is a one-time worker-ID coordination step and a dependency on the system clock, which NTP corrections can occasionally jump backward. ULID achieves similar timestamp-first sortability with zero coordination at all, at the cost of being twice the size of Snowflake. Range/block allocation is a practical middle ground that reduces, rather than eliminates, central-counter coordination.

---

## Weak Areas to Watch

- None this session — clean pass on all three checkpoint questions, plus a sharp clarifying question on why bit-field ordering (timestamp-first vs machine-ID-first) determines global vs per-machine sortability.
