# Revision — Topic 025: Isolation Levels & Anomalies

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-20

---

## Q1. Transaction A reads balance=100, Transaction B updates and commits it to 50, then A re-reads and sees 50. Name this anomaly and the minimum isolation level that prevents it.

<details>
<summary>Answer</summary>

**Non-Repeatable Read.** Minimum level to prevent it: **Repeatable Read** (the transaction sees a consistent snapshot of rows it has already read, for its whole duration).

</details>

---

## Q2. Why isn't Serializable the default isolation level in virtually every production database?

<details>
<summary>Answer</summary>

Serializable prevents all anomalies but at the highest cost — implemented via heavier locking (more blocking) or transaction aborts/retries under contention (in optimistic implementations), both of which reduce throughput. Most systems default to a weaker, pragmatic level (Postgres: Read Committed; MySQL/InnoDB: Repeatable Read) and only reach for Serializable when the specific business logic can't tolerate anomalies at a weaker level.

</details>

---

## Q3. Two concurrent bank transfers both read the same starting balance under Read Committed and both proceed to withdraw. What's this anomaly called, and why doesn't Read Committed prevent it?

<details>
<summary>Answer</summary>

**Lost Update.** Read Committed only guarantees you never read *uncommitted* data — it does not make a read-then-write sequence atomic. Both transactions can validly read the same committed balance before either writes; when both later write and commit, one write silently overwrites the other's update.

</details>

---

## Q4. What's the difference between a Non-Repeatable Read and a Phantom Read?

<details>
<summary>Answer</summary>

Non-Repeatable Read: a specific row's value changes between two reads within the same transaction. Phantom Read: the set of rows matching a query's condition changes between two identical queries (rows appear or disappear), not just a value within one row.

</details>

---

## Q5. What are two ways to prevent the Lost Update anomaly?

<details>
<summary>Answer</summary>

1. **Serializable isolation** — transactions behave as if executed one after another.
2. **Explicit row locking** (`SELECT ... FOR UPDATE`) — forces a second transaction to wait until the first fully commits before it can even read the row, making the read-modify-write sequence atomic.

</details>

---

## 30-Second Elevator Pitch

> Isolation is a spectrum, not a binary guarantee. Four standard levels — Read Uncommitted, Read Committed, Repeatable Read, Serializable — each prevent progressively more of three anomalies (Dirty Read, Non-Repeatable Read, Phantom Read) at progressively higher cost. A fourth anomaly, Lost Update, arises specifically because Read Committed protects individual reads and writes but not the read-then-write sequence as a unit — two transactions can both read the same committed value before either writes, and one update silently disappears. Postgres defaults to Read Committed, MySQL to Repeatable Read; Serializable is the strongest but costs the most throughput, so the right answer is always "pick the level your business logic's tolerance for anomalies actually requires," not "always use the strictest."

---

## Weak Areas to Watch

- Name the ANOMALY, not the isolation level that permits it (Non-Repeatable Read ≠ "a Read Committed type anomaly")
- Read Committed does not make read-then-write atomic — this is exactly why Lost Update can still occur
