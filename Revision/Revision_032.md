# Revision — Topic 032: Caching Fundamentals

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-31

---

## Q1. What is hit ratio, and why is it the single most important number for a cache?

<details>
<summary>Answer</summary>

Hit ratio = hits / (hits + misses). It directly measures how much read load the cache is absorbing away from the database — a 95% hit ratio means the DB only sees 5% of read traffic, which is the entire point of introducing a cache.

</details>

---

## Q2. Why is a cache hit faster than even a well-indexed DB query, given that the DB's buffer pool already keeps hot pages in RAM?

<details>
<summary>Answer</summary>

RAM-vs-disk speed is only one factor and doesn't even apply once data is buffer-pool-resident. The real gap: a DB query still pays for query parsing/planning, lock acquisition, MVCC visibility checks (xmin/xmax), and WAL bookkeeping — even when the row is already in RAM. A dedicated cache is typically just a hashmap lookup by key, skipping all of that overhead.

</details>

---

## Q3. Give three criteria for a GOOD caching candidate and three for a BAD one.

<details>
<summary>Answer</summary>

Good: read-heavy/write-light, expensive to compute, tolerates some staleness.
Bad: write-heavy, requires strong consistency/read-your-own-write, huge and rarely accessed (low hit-ratio payoff).

</details>

---

## Q4. Why is a shopping cart (viewed after nearly every add/remove) a bad caching candidate — name both reasons?

<details>
<summary>Answer</summary>

1. Read:write ratio is close to 1:1, failing "read-heavy, write-light."
2. It's a read-your-own-write case — the user must see their own edit immediately, failing the "tolerates staleness / no strong consistency needed" criterion. It fails two separate bad-candidate criteria at once, not just one.

</details>

---

## Q5. A product's price is cached with TTL=60s. It changes in the DB at t=0. A customer views the product page at t=30s, then proceeds to checkout at t=45s. What should happen at each point, and why?

<details>
<summary>Answer</summary>

At t=30s (browsing/display read): showing the stale cached price is generally acceptable — a bounded staleness window is a normal display-cache tradeoff. At t=45s (checkout/transactional read): the stale cached price must NOT be used to charge the customer — that read has to bypass the cache or revalidate against the source of truth. Same underlying data, different acceptability depending on whether the read is for display or for a transaction.

</details>

---

## 30-Second Elevator Pitch

> A cache is a fast, small copy of hot data sitting in front of a slower source of truth, justified by the 80/20 rule and measured by hit ratio. A cache hit beats even a well-indexed DB query because it skips per-query DB overhead — parsing, locking, MVCC checks, WAL bookkeeping — not just because RAM is faster than disk. The real cost of caching is invalidation: staleness is the price of speed, managed via TTL or explicit invalidation. Good candidates are read-heavy and staleness-tolerant; bad candidates are write-heavy or require read-your-own-write consistency — a shopping cart fails both at once. The same data can be acceptable-stale for display but unacceptable-stale for a transaction. Cache stampede is the signature failure mode when a hot key expires under load.

---

## Weak Areas to Watch

- Justifying "don't cache X" with vague UX language instead of naming the specific framework criterion failed (read:write ratio vs read-your-own-write)
- Splitting "acceptable staleness" by use case (display vs transactional read) rather than treating it as one fixed property of the data
