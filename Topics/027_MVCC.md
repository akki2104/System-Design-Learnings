# Topic 027: MVCC (Multi-Version Concurrency Control)

**Module:** 2 — Data Storage Foundations
**Completed:** 2026-07-25
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Topic 026 showed how locking enforces isolation — but at a real cost: readers and writers block each other, hurting concurrency. MVCC is the mechanism Postgres and MySQL actually use to avoid most of that blocking. This is a favorite deep-dive question: "how does Postgres handle concurrent reads and writes without constant locking?"

---

## 2. The Core Idea

Instead of making a reader wait for a writer to finish (locking), MVCC keeps **multiple versions of each row**. A writer never overwrites data in place — it creates a new version. Readers see a consistent snapshot of the data as it existed at a specific point in time, regardless of writes happening concurrently.

**Classic selling point: readers never block writers, writers never block readers.** The mechanism behind this: readers and writers literally act on physically different row versions at the same moment — a reader keeps reading the old version (untouched by the write in progress) while the writer creates and finalizes a new version elsewhere. There's no shared point of contention, so no lock is needed between them. (Writers can still block *other writers* on the same row — MVCC doesn't eliminate that; covered in Section 5.)

---

## 3. The Mechanism (Postgres-style)

Every row carries hidden metadata:
- **xmin** — the ID of the transaction that created this row version.
- **xmax** — the ID of the transaction that superseded/deleted this row version (empty if still current).

When a transaction begins, it's assigned a transaction ID and takes a **snapshot** — a record of which other transactions had already committed at that moment. When reading any row, it checks: was this version's `xmin` transaction committed before my snapshot? Was it superseded by an `xmax` transaction committed before my snapshot? Based on this, the transaction decides whether it can "see" this particular row version.

**When a writer "updates" a row:** Postgres doesn't modify it in place. It marks the old version's `xmax` as the updating transaction's ID (superseded), and inserts a brand-new row version with `xmin` set to the updating transaction. The old version isn't deleted yet — any transaction whose snapshot predates the update still correctly sees the old version.

---

## 4. How Postgres Actually Implements Isolation Levels (resolves Topic 025's asterisk)

Topic 025 noted: "Postgres's actual Repeatable Read implementation also prevents phantoms in practice, even though the SQL standard doesn't require it." Here's the mechanism:

- **Read Committed** (Postgres default): each individual **statement** within a transaction takes a fresh snapshot. A later statement in the same transaction can see rows committed after the transaction started.
- **Repeatable Read**: the **entire transaction** uses one snapshot taken at its start — every statement sees exactly the same consistent view. Since the snapshot never advances mid-transaction, no new rows can ever appear to match a repeated query — phantoms are structurally impossible here, not just non-repeatable reads.

---

## 5. MVCC and Lost Update — the Critical Nuance

**MVCC by itself does not guarantee Lost Update can never happen** — it only solves *visibility* (which version you see), not *write conflicts*. Whether a concurrent write conflict is caught depends entirely on isolation level:

**Under Read Committed:** if Transaction B tries to `UPDATE` a row Transaction A already modified and committed, Postgres doesn't error — it silently re-fetches the new committed row version and re-applies B's `WHERE` clause/logic against it (internal mechanism: **EvalPlanQual**). But if the **application code** does the classic pattern — `SELECT balance` in one round trip, compute `new_balance = balance - 100` in app code, then `UPDATE ... SET balance = new_balance` in a second round trip — the database has no way to know the app's math was based on a stale read. B's `UPDATE` silently overwrites A's committed change, because from the DB's perspective B is just issuing an unconditional `SET balance = <B's number>`. **Lost Update still happens silently here.**

**Under Repeatable Read (or Serializable):** the transaction uses one fixed snapshot for its whole lifetime. If B tries to `UPDATE` a row modified by a transaction that committed *after* B's snapshot was taken, Postgres detects this as a write-write conflict and refuses to proceed silently:
```
ERROR: could not serialize access due to concurrent update
```
B's transaction aborts; the application must catch this and retry (re-read, recompute, re-attempt).

**Takeaway:** "just use MVCC" isn't a complete interview answer — you have to name *which isolation level*, because that determines whether Lost Update is silently allowed (Read Committed + naive app code) or actively caught and forced to retry (Repeatable Read/Serializable).

---

## 6. The Cost: VACUUM

Old row versions aren't deleted immediately — some in-progress transaction's snapshot might still need them. Postgres runs a background process called **VACUUM** to identify row versions no longer visible to any active transaction and reclaim their space. If VACUUM falls behind (e.g., a long-running transaction holds an old snapshot open for hours), old versions pile up — called **table bloat**, a real operational cost of MVCC with no equivalent under pure locking.

---

## 7. MVCC Doesn't Replace Locking Entirely

MVCC eliminates reader-writer blocking, but writer-writer conflicts on the same row still need protection — either explicit row locks (`SELECT ... FOR UPDATE`, Topic 026) or the commit-time conflict detection described above. MVCC and locking aren't competing alternatives — production databases use both together: MVCC for reads, locking/conflict-detection for concurrent writes.

---

## 8. Where This Goes

- Closes the transactions/concurrency arc (Topics 020, 024, 025, 026, 027) — together these explain how a database stays durable, atomic, isolated, and concurrent all at once.
- Ties forward to Topic 039 (Replication) — the WAL from Topic 020 is what actually ships these row versions to replicas.

---

## 9. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Assuming MVCC alone guarantees Lost Update can never happen | MVCC solves visibility, not write conflicts. Read Committed + naive read-then-write app code can still silently lose an update; only Repeatable Read/Serializable actively detects the conflict and errors |
| Treating "MVCC" as a complete answer without naming the isolation level | Whether Lost Update is caught depends entirely on isolation level — always name which one |

---

## 10. Revision Questions
See `Revision/Revision_027.md`.

## 11. Summary
- MVCC keeps multiple row versions instead of locking, so readers and writers act on physically different versions and never block each other.
- Each row has xmin/xmax metadata; transactions use a snapshot to decide which version is visible to them.
- Postgres's Read Committed takes a fresh snapshot per statement; Repeatable Read takes one snapshot per transaction — this is why Postgres's Repeatable Read prevents phantoms in practice.
- MVCC does NOT automatically prevent Lost Update — Read Committed + naive app code can still lose updates silently; Repeatable Read/Serializable detect the write-write conflict and force a retry via a serialization error.
- VACUUM reclaims old row versions no longer visible to any transaction; falling behind causes table bloat — the real operational cost of MVCC.
- MVCC handles reader-writer conflicts; locking (Topic 026) still handles writer-writer conflicts — the two work together, not as alternatives.
