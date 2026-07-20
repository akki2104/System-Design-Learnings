# Revision — Topic 021: Relational Databases & SQL

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "Relational databases organize data as related tables connected via primary/foreign keys. Normalization (1NF/2NF/3NF) eliminates redundancy by storing each fact once — 1NF requires atomic values, 2NF removes partial dependency on a composite key, 3NF removes transitive dependency between non-key columns. Default to 3NF; deliberately denormalize (Topic 005) when sharding makes joins expensive or read patterns benefit from pre-joined data. Joins are cheap on one machine (in-memory matching) but expensive once sharded (network hop per join). SQL is declarative — the query planner decides execution, shaped by storage engine internals (Topic 020). Schema enforcement gives automatic data integrity, the opposite of MongoDB's flexibility."

---

## Active-Recall Q&A

<details>
<summary>Q1: What problem does normalization solve, and what's the tradeoff it makes?</summary>

Normalization eliminates **update anomalies** — the same fact duplicated across many rows/tables, updated in some places but not others, leaving data inconsistent with itself. The tradeoff: normalized data requires **joins** to reassemble related information, since it's split across multiple tables rather than duplicated inline.

</details>

<details>
<summary>Q2: Give an example violating 2NF and explain the fix.</summary>

Table `order_items(order_id, product_id, product_name)` with composite key `(order_id, product_id)`. `product_name` depends only on `product_id`, not the full composite key — a partial dependency, violating 2NF.

**Fix:** extract a `products` table keyed by `product_id` alone, storing `product_name` there.

</details>

<details>
<summary>Q3: Give an example violating 3NF and explain the fix.</summary>

Table `orders(order_id, customer_id, customer_zip, customer_city)` with `order_id` as the sole key. `customer_city` depends on `customer_zip`, which depends on `customer_id` — not directly on `order_id`. This is a transitive dependency (non-key → non-key), violating 3NF.

**Fix:** extract `customer_zip`/`customer_city` into a `customers` table, referenced via `customer_id`.

</details>

<details>
<summary>Q4: Why are joins cheap on a single machine but expensive when sharded?</summary>

On one machine, a join is a local, in-memory/disk-page matching operation — no network involved. Once data is sharded across machines, a join requires a network call to fetch matching rows from another machine — that's the real cost (Topic 005's core insight: sharding causes expensive joins, not data volume).

</details>

<details>
<summary>Q5: What's the practical rule for when to normalize vs. denormalize?</summary>

**Default to 3NF** — eliminates redundancy and update anomalies. **Deliberately denormalize** specifically when sharding would make joins expensive (network hops), or when a read-heavy access pattern benefits from pre-joined/embedded data (accepting duplication as a deliberate tradeoff, per Topic 005).

</details>

---

## Key Diagram

```
NORMAL FORMS
─────────────────────────────────────────
1NF → atomic values, no repeating groups in one column
2NF → no partial dependency (non-key depends on WHOLE composite key)
3NF → no transitive dependency (non-key doesn't depend on another non-key)

NORMALIZE (default, 3NF)  ←→  DENORMALIZE (Topic 005 tradeoff)
no redundancy, needs joins    duplication, avoids cross-shard joins
```

---

## My Weak Areas (from lesson 2026-07-15)

- Initially only knew normalization as an intuition, not the formal 1NF/2NF/3NF rules — this was caught and corrected mid-lesson (learner proactively asked whether the topic was complete)
- Q4's practical rule was initially phrased slightly circularly ("normalize to remove redundancy") rather than stating the denormalization trigger condition directly — sharpened during discussion

---

## Past Mistakes

See [InterviewMistakes.md](../InterviewMistakes.md) — entries dated 2026-07-15.
