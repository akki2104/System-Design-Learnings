# Topic 026: Concurrency Control: Locks, 2PL, Deadlocks

**Module:** 2 — Data Storage Foundations
**Completed:** 2026-07-25
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topic 025 described *what* isolation levels guarantee — no dirty reads, no lost updates, etc. This topic covers *how* a database actually enforces those guarantees when multiple transactions run concurrently: locking. This is the mechanism behind last topic's guarantees.

---

## 2. The Basic Idea: Locks

A lock is a flag a transaction acquires on a piece of data (row, page, table) before touching it, forcing other transactions to wait. Two lock types:

- **Shared Lock (S-lock)** — for reading. Multiple transactions can hold a shared lock on the same row simultaneously.
- **Exclusive Lock (X-lock)** — for writing. Only one transaction can hold an exclusive lock on a row at a time, and no other transaction can hold ANY lock (shared or exclusive) on that row while it's held.

**Compatibility:**
| | S-lock held | X-lock held |
|---|---|---|
| Request S-lock | Allowed | Must wait |
| Request X-lock | Must wait | Must wait |

---

## 3. Preventing Lost Update — the Correct Locking Sequence

**A subtle trap:** acquiring a shared lock to read, then attempting to upgrade to exclusive to write, does NOT reliably prevent Lost Update. Shared locks are compatible with each other — both Transaction A and Transaction B could hold a shared lock on the same row simultaneously and both read the stale value. When both then try to upgrade to exclusive, neither can proceed (each blocked by the other's shared lock) — a **lock-upgrade deadlock**, not a clean resolution.

**The correct fix:** acquire the **exclusive lock at read time**, not a shared lock — exactly what `SELECT ... FOR UPDATE` does in SQL:
```
A: SELECT balance FROM accounts WHERE id=1 FOR UPDATE  → A gets X-lock, reads 100
B: SELECT balance FROM accounts WHERE id=1 FOR UPDATE  → B must WAIT (A holds X-lock)
A: writes balance=0, COMMITS (releases X-lock)
B: (now proceeds) reads balance=0 (the NEW value, not the stale 100) → writes based on correct data
```
The key: B's **read** is blocked, not just B's write — this is what makes the read-modify-write sequence atomic.

---

## 4. Two-Phase Locking (2PL)

Acquiring locks isn't enough on its own — a **protocol** for when locks are acquired/released is needed, or results can still be inconsistent. Two-Phase Locking is the standard protocol:

- **Growing phase** — a transaction can acquire locks, but cannot release any yet.
- **Shrinking phase** — once a transaction releases its FIRST lock, it can never acquire another new lock — only release the rest.

```
Acquire, acquire, acquire, ... [peak] ... release, release, release
```

This guarantees **serializability** — the outcome of concurrently running transactions under strict 2PL is equivalent to some serial (one-at-a-time) execution order. This is literally how a database implements the Serializable isolation level from Topic 025.

**Strict 2PL (the practical variant):** hold ALL exclusive locks until the transaction actually commits or aborts — not just until the shrinking phase begins. This avoids **cascading rollback**: if a transaction released a lock early and another transaction read that (still-uncommitted) value, then the first transaction later rolls back, the second transaction now holds a value that never really existed and must also roll back — potentially cascading through every transaction that read the dirty value in between.

---

## 5. Deadlocks

Locking introduces a new failure mode: two (or more) transactions each hold a lock the other needs, and neither can proceed.
```
Transaction A: holds lock on Row 1, wants lock on Row 2
Transaction B: holds lock on Row 2, wants lock on Row 1
→ both wait forever
```

**Detection and resolution:**
- Databases maintain a **wait-for graph** — who is waiting on whom. A cycle means a deadlock.
- On detecting a cycle, the DB picks a **victim** (usually the transaction that's done the least work / cheapest to roll back) and forcibly aborts it, releasing its locks so the others can proceed. The aborted transaction's application code is expected to retry.

**Prevention (the alternative to detection):** enforce a strict lock-acquisition **ordering** — e.g., always acquire locks in ascending row-ID order across all transactions. If both A and B must lock Row 1 before Row 2, the deadlock above becomes structurally impossible — whichever transaction arrives first holds Row 1, and the other simply waits, never circularly.

**The tradeoff between the two approaches:** deadlocks are NOT caused by choosing detection over prevention — they occur (or don't) based purely on the actual timing/ordering of lock requests, independent of strategy.
- **Detection** — reactive. Transactions are free to acquire locks in any order (more flexible, generally more concurrency in the common case), but pays a cost when a deadlock does form: periodic cycle-scanning, then aborting a victim and forcing an application-level retry (wasted work).
- **Prevention** — proactive. Deadlocks become structurally impossible, but every transaction is constrained to a fixed global lock-acquisition order — which can force waiting even in cases where a deadlock never would have occurred, and can be awkward to enforce in application code that doesn't naturally know a global row ordering.

---

## 6. The Cost of Locking

Same tradeoff pattern as the rest of this program: stricter/more locking = safer, but less concurrency (more transactions blocked waiting) = lower throughput. This is exactly why not every isolation level relies on heavy 2PL — Topic 027 (MVCC) shows how Postgres/MySQL achieve strong isolation without blocking readers against writers at all.

---

## 7. Where This Goes Next

- **Topic 027 (MVCC)** — an alternative to lock-heavy concurrency control: keep multiple versions of a row so readers see a consistent old version while a writer creates a new one, avoiding most of the blocking cost described here.

---

## 8. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Preventing Lost Update via "shared lock to read, then upgrade to exclusive to write" | Both transactions can hold compatible shared locks simultaneously and both read stale data — then deadlock on the upgrade attempt. Correct fix: acquire the EXCLUSIVE lock at read time (`SELECT ... FOR UPDATE`), blocking the second transaction's READ, not just its write. |
| Believing deadlock detection strategy causes more deadlocks than prevention | Deadlocks occur based on actual lock-request timing/ordering, independent of strategy. Detection is reactive (more flexible, pays an abort/retry cost when a cycle forms); prevention is proactive (zero deadlock risk, but more restrictive/conservative waiting). |

---

## 9. Revision Questions
See `Revision/Revision_026.md`.

## 10. Summary
- Shared locks (reading, compatible with each other) and exclusive locks (writing, fully exclusive) are the basic building blocks of concurrency control.
- Preventing Lost Update requires taking an exclusive lock AT READ TIME (`SELECT ... FOR UPDATE`), not a shared-then-upgrade sequence, which risks a lock-upgrade deadlock.
- Two-Phase Locking (growing phase, then shrinking phase) guarantees serializability; Strict 2PL holds all locks until commit/abort to avoid cascading rollback.
- Deadlocks are resolved via detection (wait-for graph + victim abort/retry) or avoided via prevention (strict lock-acquisition ordering) — a flexibility-vs-guaranteed-safety tradeoff, not a "which causes more deadlocks" question.
- More locking = safer but less concurrent; MVCC (Topic 027) is the alternative that avoids most of this blocking cost.
