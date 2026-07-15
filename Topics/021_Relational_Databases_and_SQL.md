# Topic 021 — Relational Databases & SQL

**Module:** 2 — Data Storage Foundations
**Status:** Completed
**Date:** 2026-07-13
**Confidence:** 4/5
**Difficulty:** Medium-Hard

---

## 1. Why This Topic Exists

Builds directly on Topic 020 (storage engine internals: pages, buffer pool, WAL) and connects back to Topic 005 (denormalization, join costs, "start with Postgres"). This is the foundation for Topic 024 (Transactions & ACID) and the normalized-vs-denormalized reasoning that recurs through every case study.

---

## 2. Core Concepts

### Data as related tables

A relational database organizes data into **tables** — rows (records) and columns (fields) — related through **keys**.

```
users table                          orders table
┌────┬──────────┐                    ┌────┬─────────┬────────┐
│ id │ name     │                    │ id │ user_id │ amount │
├────┼──────────┤                    ├────┼─────────┼────────┤
│ 1  │ Akash    │                    │ 101│ 1       │ 500    │
│ 2  │ Priya    │                    │ 102│ 1       │ 750    │
└────┴──────────┘                    │ 103│ 2       │ 300    │
                                      └────┴─────────┴────────┘
        ↑                                       │
        └──────────── foreign key ──────────────┘
        (orders.user_id references users.id)
```

- **Primary key** (`id`): uniquely identifies a row in its own table.
- **Foreign key** (`user_id` in orders): a reference to a primary key in another table — this is how relationships form.

### Normalization — the relational philosophy

Core discipline: **store each fact exactly once**, referencing it everywhere else via keys instead of duplicating it.

```
BAD (denormalized — duplication):
orders table
┌────┬───────────┬────────────┬────────┐
│ id │ user_name │ user_email │ amount │   ← if Akash changes his email,
├────┼───────────┼────────────┼────────┤     EVERY row he ordered must
│ 101│ Akash     │ old@x.com  │ 500    │     be updated
│ 102│ Akash     │ old@x.com  │ 750    │
└────┴───────────┴────────────┴────────┘

GOOD (normalized): orders stores user_id → look up name/email from users ONCE
```

**Why this matters:** normalization eliminates **update anomalies** — the same fact duplicated in many places, updated in some rows but not others, leaving data inconsistent with itself.

**The flip side (Topic 005 callback):** denormalization (deliberately duplicating data) is exactly how NoSQL/sharded systems avoid expensive cross-machine joins. Relational databases lean into normalization *because* they're typically NOT sharded — the whole dataset lives on one machine (or a few, tightly coupled), so joins stay cheap and normalization's consistency benefit is free.

```
Normalized (relational default)  → no duplication, but needs JOINS to reassemble data
Denormalized (NoSQL/sharded)     → duplication, but no expensive cross-machine joins
```

### The Normal Forms (1NF, 2NF, 3NF)

**1NF — Atomic values, no repeating groups.** Every column holds a single, indivisible value — no lists crammed into one field.
```
BAD (violates 1NF):
users
┌────┬───────┬─────────────────────────┐
│ id │ name  │ phones                  │
├────┼───────┼─────────────────────────┤
│ 1  │ Akash │ "9876543210,9123456789" │  ← multiple values in one column
└────┴───────┴─────────────────────────┘

GOOD (1NF): separate phones into their own table, one row per phone number
```

**2NF — No partial dependency** (only relevant when the primary key has multiple columns). Every non-key column must depend on the WHOLE composite key, not just part of it.
```
BAD (violates 2NF):
order_items
┌──────────┬────────────┬──────────────┐
│ order_id │ product_id │ product_name │  ← composite key: (order_id, product_id)
├──────────┼────────────┼──────────────┤
│ 101      │ 5          │ "Keyboard"   │  ← product_name depends ONLY on
│ 102      │ 5          │ "Keyboard"   │     product_id, not order_id too
└──────────┴────────────┴──────────────┘

GOOD (2NF): move product_name to a products table, keyed by product_id alone
```

**3NF — No transitive dependency.** A non-key column shouldn't depend on another non-key column — only on the primary key directly.
```
BAD (violates 3NF):
orders
┌──────────┬─────────────┬───────────────┬───────────────┐
│ order_id │ customer_id │ customer_zip  │ customer_city │
├──────────┼─────────────┼───────────────┼───────────────┤
│ 101      │ 7           │ 400001        │ Mumbai        │  ← customer_city depends on
└──────────┴─────────────┴───────────────┴───────────────┘    customer_zip, not order_id
                                                                  directly (transitive)

GOOD (3NF): customer_zip/customer_city belong in the customers table, referenced by customer_id
```

**BCNF** exists as a stricter refinement of 3NF, but is rarely tested at the system-design-interview level — know it exists, don't go deep.

**The practical takeaway:** design to **3NF by default** — it eliminates redundancy and update anomalies. Then, exactly as Topic 005 taught, deliberately **denormalize** when access patterns or sharding requirements demand it (e.g., embedding `product_name` directly in `order_items` to avoid a join, accepting duplication as a deliberate tradeoff).

```
3NF (default)            → no redundancy, but needs joins to reassemble data
Denormalize (Topic 005)  → duplication accepted, for read speed or to avoid cross-shard joins
```

### Joins — reassembling normalized data

```sql
SELECT orders.id, users.name, orders.amount
FROM orders
JOIN users ON orders.user_id = users.id;
```

"For every order, find its matching user by user_id, and give me the combined row." As learned in Topic 005 — this is fast and cheap when both tables live on the same machine (the engine just matches rows in memory/disk pages), and only becomes expensive once you shard data across machines (network hops per join).

**The four join types:**
```
INNER JOIN → only rows that match in BOTH tables
LEFT JOIN  → all rows from the left table, matched data from the right (or NULL if no match)
RIGHT JOIN → mirror of LEFT JOIN
FULL JOIN  → all rows from both tables, matched where possible
```

### SQL — the language

```sql
SELECT name, amount FROM orders WHERE amount > 500;     -- read, filtered
INSERT INTO orders (user_id, amount) VALUES (1, 999);   -- create
UPDATE orders SET amount = 600 WHERE id = 101;           -- modify
DELETE FROM orders WHERE id = 103;                       -- remove

SELECT user_id, SUM(amount) FROM orders GROUP BY user_id; -- aggregate
```

SQL is **declarative** — you say *what* you want, not *how* to get it. The database's **query planner** decides the actual execution strategy (which indexes to use, join order, etc.) — this is why Topic 020's storage engine internals (pages, buffer pool) matter: the planner's decisions are shaped by what's cheap to read from disk/cache.

### Schema enforcement

Relational databases enforce a **fixed schema**: every row in a table must have the same columns, with declared types and constraints.

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  age INT CHECK (age >= 0)
);
```

This is the direct opposite of MongoDB's flexible schema (Topic 005) — the relational DB **rejects** a row that violates the schema (missing email, duplicate email, negative age) at write time. Real, automatic data integrity enforcement — not something application code has to check.

---

## Tech Decision Box (extends Topic 005's DB decision table)

```
Relational (Postgres/MySQL) → data has clear relationships, integrity matters
                                (foreign keys, uniqueness, NOT NULL), moderate
                                scale that fits on one primary + replicas
                                — still the DEFAULT starting point

Choose relational specifically when:
  - You need enforced referential integrity (orders MUST reference a real user)
  - Your access patterns involve JOINING related entities often
  - Strong consistency/ACID matters more than infinite horizontal write scale
```

**Interview sentence:** "I'll model this relationally because orders must always reference a valid user — the foreign key constraint enforces that automatically, and I need to join orders with user data frequently. I considered a document store for flexibility, but rejected it since referential integrity is a hard requirement here, not a nice-to-have."

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Only knowing normalization as an intuition, not the formal 1NF/2NF/3NF forms | Know the concrete rule for each: atomic values (1NF), no partial dependency on composite keys (2NF), no transitive dependency (3NF) |
| Thinking normalization and denormalization are opposing "correct vs incorrect" choices | Both are valid — 3NF is the default; denormalization is a deliberate tradeoff for read speed or sharding (Topic 005) |
| Assuming joins are always expensive | Joins are cheap on a single machine (in-memory/disk page matching); only expensive once data is sharded across machines |

---

## Real Interview Questions

1. "Why would you normalize your schema, and when would you deliberately break normalization?" (universal)
2. "Explain 1NF, 2NF, 3NF with an example." (data-heavy roles, sometimes asked directly)
3. "When would you choose a relational database over a document store?" (universal)

---

## Revision Questions

1. What problem does normalization solve, and what's the tradeoff it makes?
2. Give an example violating 2NF and explain the fix.
3. Give an example violating 3NF and explain the fix.
4. Why are joins cheap on a single machine but expensive when sharded?
5. What's the practical rule for when to normalize vs. denormalize?

---

## Cheat Sheet

```
RELATIONAL MODEL
─────────────────────────────────────────────────
Tables + rows + columns, related via primary/foreign keys

NORMALIZATION — store each fact ONCE
─────────────────────────────────────────────────
1NF → atomic values, no repeating groups in one column
2NF → no partial dependency on a composite key
3NF → no transitive dependency (non-key depending on non-key)
Default to 3NF; denormalize deliberately (Topic 005) for read speed/sharding

JOINS
─────────────────────────────────────────────────
Cheap on ONE machine (in-memory/page matching)
Expensive once SHARDED (network hop per join)
INNER / LEFT / RIGHT / FULL

SQL = declarative — query planner decides execution (uses Topic 020's
      buffer pool / page internals to decide what's cheap)

SCHEMA ENFORCEMENT
─────────────────────────────────────────────────
Fixed schema + types + constraints, enforced at write time —
opposite of MongoDB's flexible schema (Topic 005)

WHEN TO USE RELATIONAL (extends Topic 005)
─────────────────────────────────────────────────
Clear relationships + integrity matters + frequent joins + ACID > infinite
horizontal write scale → still the DEFAULT starting point
```

---

## Summary

- **Relational databases organize data as related tables**, connected via primary/foreign keys.
- **Normalization (1NF/2NF/3NF) eliminates redundancy and update anomalies** — store each fact once.
- **Denormalization (Topic 005) is the deliberate opposite tradeoff** — accepted for read speed or to avoid cross-shard joins.
- **Joins are cheap on one machine, expensive once sharded** — the same insight from Topic 005's tradeoff framework.
- **SQL is declarative**; the query planner's decisions are shaped by storage engine internals (Topic 020).
- **Schema enforcement gives automatic data integrity** — the opposite of NoSQL's flexibility.

> **You now can:** design a normalized relational schema to 3NF, explain when and why to deliberately denormalize, and justify choosing a relational database over a document store with the interview-ready reasoning format.
