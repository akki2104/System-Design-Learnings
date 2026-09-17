# Revision — Topic 043: Choosing a Shard Key

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-09-17

---

## Q1. List the four criteria for a good shard key.

<details>
<summary>Answer</summary>

High cardinality, even access distribution, alignment with the dominant query pattern, and avoiding monotonically increasing keys specifically under range partitioning.

</details>

---

## Q2. Why can "high cardinality" and "aligns with the dominant query" pull in different directions? Give an example.

<details>
<summary>Answer</summary>

A field can have great cardinality and even distribution while still not being what the system's most frequent, most latency-critical query filters by. Example: in TinyURL, `user_id` has high cardinality and reasonably even access, but the dominant query is `short_code → long_url` — sharding by `user_id` would leave that redirect query unable to find the right shard without a separate lookup. `user_id` would serve a secondary query well ("list a user's URLs"), just not the dominant one.

</details>

---

## Q3. Walk through why `short_code` is a better shard key than `user_id` for TinyURL's mapping table.

<details>
<summary>Answer</summary>

`short_code` has high cardinality (millions/billions of distinct codes), reasonably even access under ordinary traffic (codes look random), and — critically — it's exactly what the dominant redirect query filters by, so every lookup hits one shard. `user_id` would make a secondary "list my URLs" query easy, but the redirect path (the highest-volume, most latency-critical query) would have to scatter-gather across every shard to find a given short_code, since ownership isn't known without a separate lookup.

</details>

---

## Q4. Does a well-chosen shard key protect against a single hot/viral key? What does?

<details>
<summary>Answer</summary>

No. Even a great shard key can't stop one specific viral value from concentrating massive read traffic on its one shard. Caching in front of the database absorbs that hot key's reads before they reach the shard at all — this is the same boundary Topic 042 drew for consistent hashing (it doesn't solve hot keys either).

</details>

---

## Q5. Is a monotonically increasing key always a bad shard key choice?

<details>
<summary>Answer</summary>

No — the trap is specifically the pairing of a monotonic key with RANGE partitioning (all new writes land on the "latest" shard). The same monotonic key under HASH partitioning doesn't have this problem, since hashing scrambles the ordering.

</details>

---

## 30-Second Elevator Pitch

> A shard key is the field fed into the partitioning strategy, and choosing it well is the real design decision — independent of how good the hashing mechanics are. Good shard keys have high cardinality, even access distribution, alignment with the dominant query, and avoid pairing a monotonic key with range partitioning. These criteria can conflict — a field like user_id can have great cardinality and distribution while still failing to match the dominant query, which is the wrong tradeoff to make. TinyURL should shard by short_code (matches the redirect path) not user_id (would wreck the redirect path to ease a secondary listing query, which should instead get its own index/table). No shard key protects against a single hot/viral value overloading its shard — that's what caching solves, a complementary layer, not a substitute.

---

## Weak Areas to Watch

- First pass at Q2 explained both criteria correctly in isolation but didn't supply the requested concrete conflicting example until asked directly — worth practicing generating the example unprompted next time, not just defining the terms.
