# Revision — Topic 026: Concurrency Control: Locks, 2PL, Deadlocks

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-25

---

## Q1. Why doesn't "acquire a shared lock to read, then upgrade to exclusive to write" reliably prevent Lost Update?

<details>
<summary>Answer</summary>

Shared locks are compatible with each other — both transactions could hold a shared lock on the same row simultaneously and both read the same stale value. When both then try to upgrade to exclusive, neither can proceed (each blocked by the other's shared lock) — a lock-upgrade deadlock, not a clean resolution.

</details>

---

## Q2. What's the correct locking sequence to prevent Lost Update, and what SQL construct implements it?

<details>
<summary>Answer</summary>

Acquire the EXCLUSIVE lock at read time, not a shared lock — implemented via `SELECT ... FOR UPDATE`. This blocks a second transaction's READ (not just its write) until the first transaction commits, so the second transaction always reads the up-to-date (post-commit) value.

</details>

---

## Q3. What are the two phases of Two-Phase Locking, and what guarantee does the protocol provide?

<details>
<summary>Answer</summary>

Growing phase (acquire locks, release none) and Shrinking phase (once you release your first lock, you can never acquire a new one). This guarantees serializability — the outcome is equivalent to some serial (one-at-a-time) execution order.

</details>

---

## Q4. Why does Strict 2PL hold all exclusive locks until commit/abort instead of releasing them as soon as the shrinking phase begins?

<details>
<summary>Answer</summary>

To avoid cascading rollback: if a lock were released early and another transaction read that (still uncommitted) value, then the first transaction later rolled back, the second transaction would be holding data that never really existed and would also have to roll back — potentially cascading through every transaction that read the dirty value.

</details>

---

## Q5. Deadlock detection and deadlock prevention solve the same problem two different ways. What's the actual tradeoff?

<details>
<summary>Answer</summary>

Neither strategy causes more or fewer deadlocks to occur — that depends purely on actual lock-request timing/ordering. Detection (wait-for graph + victim abort) is reactive: more flexible lock ordering, generally more concurrency, but pays a cost (cycle-scanning, aborting a victim, forcing a retry) when a deadlock does form. Prevention (strict lock-acquisition ordering) is proactive: deadlocks become structurally impossible, but every transaction is constrained to a fixed global order, which can force unnecessary waiting and can be awkward to enforce in real application code.

</details>

---

## 30-Second Elevator Pitch

> Locking is how databases actually enforce the isolation guarantees from Topic 025. Shared locks allow concurrent readers; exclusive locks are fully exclusive. Preventing Lost Update requires taking the exclusive lock at read time (SELECT ... FOR UPDATE) — a shared-then-upgrade sequence risks a lock-upgrade deadlock instead. Two-Phase Locking (acquire-only growing phase, then release-only shrinking phase) guarantees serializability; Strict 2PL holds locks until commit/abort specifically to avoid cascading rollback. Locking introduces deadlocks — resolved either reactively via a wait-for graph and victim abort/retry, or proactively via a strict lock-acquisition ordering that makes deadlocks structurally impossible at the cost of flexibility. More locking means stronger correctness but less concurrency — which is exactly why MVCC (Topic 027) exists as a mostly-lock-free alternative.

---

## Weak Areas to Watch

- Lost Update prevention needs an X-lock AT READ TIME, not shared-then-upgrade (which deadlocks instead)
- Detection vs prevention is a flexibility-vs-guaranteed-safety tradeoff — neither strategy itself "causes" deadlocks
