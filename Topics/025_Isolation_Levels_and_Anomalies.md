# Topic 025: Isolation Levels & Anomalies

**Module:** 2 — Data Storage Foundations
**Completed:** 2026-07-20
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topic 024 introduced Isolation as one of the four ACID properties — "concurrent transactions don't see each other's uncommitted changes." That's a simplification. Isolation isn't binary; it's a spectrum of levels, each preventing different anomalies at a different performance cost. This is heavily tested in system design interviews because it forces real tradeoff reasoning rather than reciting a feature.

---

## 2. The Core Tension: Isolation vs Performance

Perfect isolation — as if every transaction ran one at a time, in sequence — is the safest guarantee but kills concurrency. Weak isolation allows massive concurrency but risks reading corrupted/inconsistent data. Databases offer a menu of isolation levels, and picking one is a genuine tradeoff call (same spirit as Topic 005's framework).

---

## 3. The Anomalies — What Can Go Wrong Under Weak Isolation

- **Dirty Read** — Transaction A reads data Transaction B has written but not yet committed. If B rolls back, A acted on data that never actually existed.
- **Non-Repeatable Read** — Transaction A reads the same row twice, but gets a different value the second time because Transaction B updated and *committed* a change to that row in between A's two reads.
- **Phantom Read** — Transaction A runs the same query twice, but gets a different *set of rows* the second time, because Transaction B inserted or deleted rows matching the query's condition in between.
- **Lost Update** — Transaction A reads a value, Transaction B reads the same value (before A commits), both compute a new value based on their own read, and both write + commit — one of the two updates is silently overwritten and lost, even though neither transaction ever read uncommitted data.

**Key distinctions:**
- Non-repeatable read = one specific **row's value** changes underneath you.
- Phantom read = the **set of rows** matching a condition changes (rows appear/disappear).
- Lost Update = the read-then-write **sequence** isn't atomic — Read Committed protects individual reads/writes, but not the whole read-modify-write operation as a unit.

---

## 4. The Four Standard Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Lost Update |
|---|---|---|---|---|
| **Read Uncommitted** | Possible | Possible | Possible | Possible |
| **Read Committed** | Prevented | Possible | Possible | Possible |
| **Repeatable Read** | Prevented | Prevented | Possible (per strict SQL standard) | Possible (needs explicit locking or Serializable) |
| **Serializable** | Prevented | Prevented | Prevented | Prevented |

- **Read Uncommitted** — essentially no protection. Rarely used.
- **Read Committed** — can only read committed data (no dirty reads), but a re-read of the same row can change. **Postgres's default.**
- **Repeatable Read** — transaction sees a consistent snapshot of rows it has already read, for its entire duration. Phantoms possible per the strict standard (though Postgres's actual MVCC-based implementation happens to also prevent phantoms in practice — concrete in Topic 027). **MySQL/InnoDB's default.**
- **Serializable** — transactions behave as if executed one after another sequentially, even though they ran concurrently. Prevents all anomalies, at the highest cost (heavier locking, or transaction aborts/retries under contention).

---

## 5. Why Not Always Use Serializable?

Because the cost is real: heavier locking means more blocking between transactions, or (in optimistic implementations) more aborts/retries under contention — both reduce throughput. The right interview answer is never "always use the strictest level" — it's **"choose the isolation level based on which anomalies this specific business logic can tolerate."**

---

## 6. Concrete Examples

**Bank transfer (Lost Update walkthrough):**
```
T=0: Balance = 100
Transaction A: reads balance = 100 (committed, valid read under Read Committed)
Transaction B: reads balance = 100 (still committed — A hasn't written yet)
Transaction A: withdraws 100 → writes balance = 0, commits
Transaction B: withdraws 100 (based on ITS OWN read of 100) → writes balance = 0, commits
```
Both reads were valid under Read Committed — neither touched uncommitted data. But A's withdrawal is silently lost; $200 left the account, but the balance only reflects one $100 withdrawal. Fix: Serializable isolation, or explicit `SELECT ... FOR UPDATE` row locking (forces B to wait until A's transaction fully commits before B can even read) — the actual mechanism covered in Topic 026.

**Analytics query ("how many orders in the last hour?"):** a textbook phantom read scenario — if new orders keep committing while the count query runs, the result can vary by exact timing even under Repeatable Read (per the strict standard).

---

## 7. Where This Goes Next

- **Topic 026 (Concurrency Control: Locks, 2PL, Deadlocks)** — the actual locking mechanisms databases use to implement these isolation levels.
- **Topic 027 (MVCC)** — how Postgres/MySQL achieve Repeatable Read/snapshot isolation efficiently without blocking readers against writers.

---

## 8. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Naming an anomaly by the isolation level that allows it (e.g., "a Read Committed type of anomaly") instead of its actual name | The anomaly has its own name (Non-Repeatable Read, Phantom Read, Lost Update) — distinct from which isolation level permits it |
| Assuming Read Committed prevents double-spend-style races because it blocks dirty reads | Read Committed only guards individual reads/writes against uncommitted data — it does NOT make a read-then-write sequence atomic. Two transactions can both read the same committed value before either writes, causing a Lost Update |

---

## 9. Revision Questions
See `Revision/Revision_025.md`.

## 10. Summary
- Isolation is a spectrum (Read Uncommitted → Read Committed → Repeatable Read → Serializable), not a single guarantee.
- Three core anomalies: Dirty Read, Non-Repeatable Read, Phantom Read — plus Lost Update, which specifically arises from Read Committed's read-then-write sequence not being atomic.
- Postgres defaults to Read Committed; MySQL/InnoDB defaults to Repeatable Read.
- Serializable prevents everything but costs the most throughput — never the automatic "safe default" answer in an interview.
- Preventing Lost Update needs Serializable isolation or explicit row locking (`SELECT ... FOR UPDATE`) — the mechanism Topic 026 covers.
