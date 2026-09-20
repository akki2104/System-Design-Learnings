# Topic 098: Unique ID Generation (Snowflake, ULID)

**Module:** Building Blocks (taught early, as the final pre-TinyURL prerequisite)
**Tier:** 🔴 MUST
**Completed:** 2026-09-21
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Topics 041-044 built out sharding, consistent hashing, and rebalancing — a dataset spread across many machines. But every one of those machines needs to independently create new record IDs, and those IDs must never collide across shards. On a single unsharded machine this was trivial: an auto-increment column (`SERIAL`, `AUTO_INCREMENT`) just counts up, one machine, one counter, done. The moment you shard, that trivial solution breaks — a single shared counter across all shards becomes exactly the kind of centralized bottleneck sharding was built to eliminate (Topic 041). This is the last prerequisite before Case Study #1 (TinyURL), where generating a unique short code is the core problem of the whole system.

---

## 2. The Core Tension

Four properties you'd like an ID to have, and they pull against each other:
```
UNIQUE        — no two IDs collide, ever, across all machines
DECENTRALIZED — any machine can generate one without asking a coordinator per-ID
SORTABLE      — newer IDs are lexicographically/numerically larger (useful for
                indexing, range scans, "give me the 20 most recent")
COMPACT       — small enough to be cheap to store/index/transmit
```
Auto-increment gets sortable + compact, but fails decentralized. Random UUIDs get unique + decentralized, but fail sortable and compact. The interesting solutions here try to get all four (or close to it) without a central bottleneck.

---

## 3. Approach 1: Auto-Increment / DB Sequence (the baseline)

A single counter, incremented atomically by the DB on each insert. Sortable, compact, trivially unique on one machine.

**Why it breaks under sharding:** every insert, on every shard, has to coordinate with the one counter — reintroducing exactly the single-machine bottleneck sharding was meant to eliminate. A common practical compromise: **range/block allocation** ("ticket servers," used by Flickr/Instagram-style systems) — a central service hands out entire *ranges* of IDs at once (e.g., "server A, you now own IDs 1000-1999"), so coordination happens once per 1000 IDs instead of once per ID. Reduces coordination frequency by orders of magnitude while keeping strict global ordering — the coordinator still exists, just called far less often.

---

## 4. Approach 2: Random UUID (v4)

**Generation:** 128 bits total, but not all random — 6 bits are reserved to mark version/variant, leaving **122 bits of actual randomness** from a cryptographically secure random number generator. No clock, no machine ID, no coordination at all.

**Why collision is effectively zero:** birthday-paradox math over a 2^122 ≈ 5.3 × 10³⁶ space. A 50% chance of *any* collision requires generating roughly √(2^122) ≈ 2^61 ≈ 2.1 × 10¹⁸ UUIDs — over a billion-billion. No real system approaches that volume over its entire lifetime, so collision is treated as zero in practice (not mathematically impossible, just astronomically unlikely).

**The real cost — ties directly to Topics 022/023:** a random UUID used as a primary key/index means every insert lands at a random point in the B-tree, not at the end, causing constant page splits and poor locality — a real, measurable write-performance cost at scale, not a style choice. UUIDs are also large (128 bits / 36 characters as a string) versus a 64-bit integer.

---

## 5. Approach 3: Twitter Snowflake

A 64-bit ID assembled from three parts, most-significant-bit first:
```
[ 41 bits: timestamp ]  [ 10 bits: machine/worker ID ]  [ 12 bits: sequence ]
 (ms since custom epoch)   (up to 1024 machines)          (up to 4096 IDs/ms/machine)
```
**Why this works without per-ID coordination:** each machine has its own fixed worker ID (assigned once, at startup — not per ID) and generates IDs from its own clock + a local counter. No two machines can produce the same ID (different worker-ID bits); no single machine can produce a duplicate within the same millisecond (the sequence bits).

**Why timestamp specifically occupies the MOST significant bits (not machine ID):** this determines whether numeric ID order approximates *global* chronological order across the whole system, or only chronological order *within a single machine's own stream*. Worked comparison:

```
LAYOUT A — timestamp first: [timestamp][machine][seq]
  Machine 1 @ T=1000 → numeric value dominated by "1000..."
  Machine 2 @ T=2000 → numeric value dominated by "2000..."
  2000 > 1000 → later-created ID is numerically larger. Matches real time order. ✅

LAYOUT B — machine ID first: [machine][timestamp][seq]
  Machine 2 @ T=500  (EARLIER in real time) → dominated by "2..."
  Machine 1 @ T=1000 (LATER in real time)   → dominated by "1..."
  "2..." > "1..." → the EARLIER-created ID is numerically LARGER. ❌ Contradicts real
  time order — numeric sort now just groups IDs by which machine made them, with
  timestamp only breaking ties WITHIN one machine's own IDs, not across machines.
```
Timestamp-first is what makes numeric ID order approximate global creation-time order regardless of which machine generated each ID — the actual property Snowflake exists to provide. Machine-ID-first would only give sortability within a single machine's own stream, which is a much weaker and less useful guarantee (most real query patterns — "20 most recent items across the whole system" — need global recency, not per-machine recency).

**The catch:** "no coordination" really means "no *per-ID* coordination" — assigning each machine a unique worker ID still requires one-time coordination (a registry or startup-time allocation), a much smaller and rarer cost than a central counter hit on every insert.

**The clock dependency:** computer clocks drift, and NTP (Network Time Protocol) periodically corrects them against an authoritative time server. Usually this is a smooth nudge, but sometimes it's a **step** correction — the clock can jump backward by a noticeable amount all at once (e.g., after a large drift is detected, or a VM pause/resume). Since Snowflake reads "current time in ms" as its most-significant bits, a backward clock jump between two ID-generation calls risks reusing an already-used timestamp — a possible duplicate ID (if the sequence also lines up) or at minimum an ID that's numerically smaller than one generated earlier in real time, breaking sortability. Real implementations track the last timestamp used and, if a new reading is lower, either wait for the clock to catch back up or error out, rather than silently generating a bad ID.

---

## 6. Approach 4: ULID (Universally Unique Lexicographically Sortable ID)

A 128-bit ID, UUID-compatible, structured for sortability where random UUIDs aren't:
```
[ 48 bits: timestamp (ms) ]  [ 80 bits: randomness ]
```
Like Snowflake, the timestamp occupies the most significant bits, so lexicographic string sort matches chronological order. Unlike Snowflake, there's no machine-ID field — uniqueness comes entirely from the large random component, so **zero coordination of any kind is needed**, not even one-time worker-ID assignment. Tradeoff: larger than Snowflake (128 vs 64 bits), since it needs enough randomness bits to make collisions negligible without a dedicated machine-ID field to help.

---

## 7. Decision Box

```
Single DB, no sharding, need strict order + simplicity     → Auto-increment/sequence
Sharded, need decentralized + compact + time-sortable       → Snowflake
  (accept one-time worker-ID coordination at startup)
Sharded, need decentralized + zero coordination ever,
  UUID-compatible tooling, size less critical                → ULID
Want to reduce (not eliminate) central-counter coordination
  without going fully decentralized                          → Range/block allocation
```
**Interview sentence:** "For a sharded system needing globally unique, roughly time-ordered IDs, I'd use Snowflake — each shard generates IDs independently with no per-ID coordination, and the timestamp-first layout keeps IDs index-friendly and globally time-sortable, unlike a random UUID which would fragment the B-tree on every insert."

---

## 8. Worked Example — TinyURL's Short Code

TinyURL needs a unique identifier for every submitted URL, encoded into a short, URL-safe string (Case Study #1, next). Three realistic options:
- **Auto-increment + base62 encoding:** a central counter's integer converted to base62 (0-9, a-z, A-Z) for compactness — simple and short, but reintroduces the central-counter bottleneck at scale (fixable with range allocation).
- **Snowflake ID + base62 encoding:** decentralized generation across app servers, then compress the resulting 64-bit number into a short base62 string — no central bottleneck, but longer than a pure sequential counter at low volumes.
- **Random string + collision check:** generate a random short string directly, check the DB for a collision, retry if taken — zero coordination, but requires a DB round-trip specifically to check for collisions, and collision probability grows as the namespace fills up.

---

## 9. Common Mistakes

| Mistake | Correction |
|---|---|
| Using a random UUID as a primary key without considering write performance | Random UUIDs fragment B-tree inserts (Topic 022/023) — a real, measurable cost at write-heavy scale, not just a style choice |
| Claiming Snowflake needs zero coordination | It needs one-time worker-ID assignment per machine at startup — "no coordination" means no *per-ID* coordination, a much smaller cost than a central counter |
| Assuming IDs must be strictly consecutive integers | They only need to be unique and roughly time-ordered — Snowflake/ULID IDs have gaps and aren't dense sequential integers, and that's fine |
| Using plain auto-increment across a sharded system | Recreates the exact centralized bottleneck sharding was meant to eliminate — needs Snowflake, ULID, or range allocation instead |
| Assuming machine-ID-first bit layout would sort just as well as timestamp-first | Machine-ID-first only preserves ordering *within* a single machine's own stream — across machines, numeric sort order is dominated by which machine generated the ID, not when, breaking global chronological ordering |

---

## 10. Real Interview Questions

1. "Why can't you just use auto-increment IDs once you've sharded your database?" (tests the core motivation for this topic)
2. "Walk me through the structure of a Snowflake ID and why each field is sized the way it is." (tests the 41/10/12-bit breakdown and reasoning)
3. "Why does Snowflake put the timestamp in the most significant bits instead of the machine ID?" (tests the global-vs-per-machine sortability distinction)
4. "How is a UUID generated, and why is the collision probability considered negligible?" (tests the 122-bits-of-randomness + birthday-paradox reasoning)
5. "What happens to ID generation if a machine's clock jumps backward?" (tests the NTP-step-correction failure mode and its mitigation)

---

## 11. Revision Questions
See `Revision/Revision_098.md`.

## 12. Summary
- Sharding (041-044) breaks the single-counter auto-increment model — a shared counter across shards recreates the exact bottleneck sharding was meant to eliminate.
- Four competing properties: unique, decentralized, sortable, compact — no single approach maximizes all four.
- Random UUID (v4): 122 bits of randomness, collision-negligible via birthday-paradox math, but not sortable and fragments B-tree inserts.
- Snowflake: 64-bit [timestamp | machine ID | sequence], decentralized after one-time worker-ID assignment, globally time-sortable specifically because timestamp occupies the most significant bits — machine-ID-first would only sort within one machine's own stream.
- Snowflake depends on the system clock; an NTP step correction that moves the clock backward can risk duplicate or out-of-order IDs unless explicitly guarded against.
- ULID: 128-bit, timestamp-first + pure randomness (no machine-ID field), UUID-compatible, needs zero coordination of any kind — larger than Snowflake as the cost.
- Range/block allocation is a practical middle ground: reduces (doesn't eliminate) central-counter coordination frequency.
