# Topic 043: Choosing a Shard Key

**Module:** 4 — Scaling & Distributing Data
**Tier:** 🔴 MUST
**Completed:** 2026-09-17
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Topic 041 gave you the partitioning strategies (range/hash/directory) and Topic 042 gave you consistent hashing to minimize reshuffling. Neither answers the actual make-or-break design decision: **which field do you shard by?** That choice — the shard key — is what actually determines whether hot partitions happen, completely independent of how good your hashing mechanics are. A perfect consistent-hashing implementation with a bad shard key still produces hot shards.

---

## 2. What a Shard Key Is

```
The field (or fields) of a record fed into the partitioning strategy
to decide which shard that record lives on.

users table, sharded by shard_key = user_id:
  hash(user_id) → determines the shard → ALL of that user's rows live there
```

---

## 3. The Criteria for a Good Shard Key

```
1. HIGH CARDINALITY
   Many distinct values — enough room to actually spread data across
   many shards. A key like "country" (~195 values) or "status" (3-4
   values) caps how finely you can ever partition, no matter how many
   shards you add.

2. EVEN ACCESS DISTRIBUTION (not just even VALUE distribution)
   Even if values are evenly spread, real-world access can still be
   skewed — a "celebrity" value can dominate traffic regardless of how
   many total distinct values exist. Cardinality solves the WRITE/STORAGE
   spread; it does NOT by itself guarantee even READ traffic.

3. ALIGNS WITH THE DOMINANT QUERY PATTERN
   The single most important, most frequent, most latency-sensitive
   query should be answerable by hitting exactly ONE shard. If the
   shard key doesn't match what the dominant query filters by, that
   query becomes a scatter-gather across every shard (Topic 041's cost).

4. AVOID MONOTONIC KEYS UNDER RANGE PARTITIONING
   Topic 041's classic trap: a sequential/timestamp key under RANGE
   partitioning concentrates all new writes on the "latest" shard.
   (Hashing the same monotonic key removes this specific danger —
   the trap is the PAIRING of monotonic-key + range-partitioning,
   not the monotonic key alone.)
```

---

## 4. The Real Skill: These Criteria Can Conflict

A key that spreads load evenly might not match the dominant query (forcing scatter-gather). A key that perfectly matches the dominant query might have poor cardinality or uneven access. **Naming this tension explicitly, and resolving it by prioritizing the dominant/latency-critical query, is the actual interview skill** — not reciting the four criteria in isolation.

**Concrete example of the conflict:** in TinyURL's mapping table, `user_id` has high cardinality and reasonably even access distribution — but it fails to align with the dominant query (`short_code → long_url`). It would, however, serve a *secondary* query well ("show all short URLs created by this user") — just not the one that matters most. This is exactly the shape of the tension: a key can satisfy cardinality/distribution while still being the wrong choice, because it optimizes for the wrong query.

---

## 5. Worked Example — TinyURL's Mapping Table

```
Table: short_code → long_url
Dominant query (by far): "given this short_code, what's the long_url?"
  — this is the REDIRECT path. Every single click hits it. It must be fast.

Candidate shard key: short_code
  ✓ High cardinality — millions/billions of distinct codes
  ✓ Even access (mostly) — codes look random (base62-encoded counter or
    hash-based), so ordinary traffic spreads naturally across shards
  ✓ PERFECT query alignment — the only real query IS "look up by short_code,"
    so sharding by short_code means every redirect hits exactly ONE shard,
    never a scatter-gather

→ short_code is an excellent shard key for this table.
```

---

## 6. The Contrast That Makes the Point

```
What if we sharded by user_id instead (to make "show all URLs I created"
easy)?

  Redirect query ("given short_code, get long_url") no longer knows
  which shard to check — you don't know WHO created a given short_code
  without a separate lookup — so the single most latency-critical,
  highest-volume query in the entire system now has to scatter-gather
  across every shard.

  The secondary "my URLs" listing query becomes easy, but at the cost
  of wrecking the primary one.
```
**The lesson:** shard by what the *dominant, latency-critical* query needs — even if it makes a secondary, lower-volume query harder. That secondary query (a user's URL list) can be served by a separate index or table (Topic 031's polyglot-persistence instinct: a different access pattern gets a different store/index, not a compromise on the primary shard key).

---

## 7. Even a Great Shard Key Doesn't Stop Hot Keys

`short_code` as a shard key is excellent for *ordinary* traffic — but if one specific URL goes viral, that single key still concentrates massive read load onto whichever one shard owns it. **No shard key choice fixes this** — this is exactly the boundary Topic 042 drew ("consistent hashing doesn't solve hot keys"). The actual fix is **caching** (Module 3) sitting in front of the shard, absorbing that one hot key's reads before they ever reach the database shard at all. Shard-key choice and caching are complementary layers, not substitutes for each other.

---

## Tech Decision Box: The Shard-Key Selection Framework

```
1. What is the DOMINANT, most latency-critical query? Identify it first.
2. Does the field that query filters by have HIGH CARDINALITY?
3. Will real-world ACCESS to that field's values be reasonably even
   (accepting that outlier hot keys still need caching as backstop)?
4. If using RANGE partitioning, is the key monotonic? If so, either
   switch to hash partitioning or pick a different key.

Prioritize #1 above all — a shard key that serves a secondary query well
but forces the primary query into scatter-gather is the wrong choice,
even if it "feels" more natural for the data model.
```
**Interview sentence:** "I'd shard the URL-mapping table by `short_code` — it's the field the dominant redirect query filters by, so every lookup hits exactly one shard, and it has enough cardinality and natural randomness to spread load evenly. I would NOT shard by `user_id` here, even though it would make a 'list my URLs' feature easier, because that would force the much more frequent, latency-critical redirect path to scatter-gather across every shard."

---

## Common Mistakes

| Mistake | Correction |
|---|---|
| Choosing a shard key purely for cardinality/distribution, ignoring query patterns | A well-distributed key that doesn't match the dominant query forces that query into an expensive scatter-gather across every shard |
| Choosing a shard key purely for query convenience, ignoring cardinality/distribution | A key that matches a query but has low cardinality or skewed access produces hot shards regardless of good intentions |
| Assuming a well-chosen shard key eliminates hot-key problems | Shard-key choice fixes uneven load from the overall access pattern; a single viral/hot value can still overload its one shard — that's what caching is for, not a better shard key |
| Treating "monotonic key = always bad" as an absolute rule | The trap is specifically monotonic key + RANGE partitioning; the same key under HASH partitioning doesn't have this problem |
| Optimizing the shard key for a secondary query at the expense of the dominant one | Serve secondary access patterns via a separate index/table (polyglot persistence, Topic 031) rather than compromising the shard key that serves the highest-volume, most latency-critical query |

---

## Real Interview Questions

1. "What makes a good shard key? Walk me through your criteria." (tests fluency with all four criteria, not just cardinality)
2. "You're sharding a URL-mapping table — would you shard by short_code or by user_id? Why?" (the exact worked example — tests query-pattern prioritization)
3. "Your shard key has great cardinality and even distribution, but one specific value suddenly goes viral — does your shard key choice help here?" (tests the hot-key boundary)
4. "Is a monotonically increasing ID always a bad shard key?" (tests the range-vs-hash nuance, not a blanket rule)
5. "How would you handle a secondary query pattern that doesn't align with your chosen shard key?" (tests reaching for a separate index/table rather than compromising the primary shard key)

---

## 8. Revision Questions
See `Revision/Revision_043.md`.

## 9. Summary
- A shard key is the field fed into the partitioning strategy — choosing it well is the actual design decision, independent of how good the underlying hashing/ring mechanics are.
- Four criteria: high cardinality, even access distribution, alignment with the dominant query, and avoiding monotonic keys specifically under range partitioning.
- These criteria can conflict — e.g., `user_id` in TinyURL has good cardinality/distribution but fails to align with the dominant redirect query. The resolution is to prioritize the field the dominant, latency-critical query needs, and serve secondary access patterns through a separate index/table instead.
- Worked example: TinyURL should shard by `short_code` (matches the redirect path) not `user_id` (would wreck the redirect path to ease a secondary listing query).
- No shard key choice protects against a single hot/viral value overloading its shard — that's what caching solves, a complementary layer, not a substitute.

> **You now can:** name and apply the four shard-key criteria, resolve the tension between them with a concrete conflicting example, correctly reason through TinyURL's shard-key choice, and clearly state the boundary between shard-key design and hot-key mitigation via caching.
