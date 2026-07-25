# Revision — Topic 027: MVCC (Multi-Version Concurrency Control)

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-25

---

## Q1. Why does MVCC allow "readers never block writers, writers never block readers"?

<details>
<summary>Answer</summary>

Readers and writers act on physically different row versions at the same moment — a reader keeps reading the old version (untouched by the write in progress) while the writer creates and finalizes a new version elsewhere. There's no shared point of contention, so no lock is needed between them.

</details>

---

## Q2. What are xmin and xmax, and how does a transaction use them to decide which row version to see?

<details>
<summary>Answer</summary>

xmin = the transaction ID that created this row version. xmax = the transaction ID that superseded/deleted it (empty if still current). A transaction checks whether the xmin transaction committed before its own snapshot, and whether any xmax transaction committed before its snapshot, to decide if a given version is visible to it.

</details>

---

## Q3. Why does Postgres's Repeatable Read prevent phantom reads in practice, even though the SQL standard doesn't require it?

<details>
<summary>Answer</summary>

Read Committed takes a fresh snapshot per STATEMENT; Repeatable Read takes ONE snapshot for the entire transaction's lifetime. Since the snapshot never advances mid-transaction under Repeatable Read, no new rows can ever appear to match a repeated query — phantoms are structurally impossible.

</details>

---

## Q4. Does MVCC alone guarantee Lost Update can never happen? Explain the Read Committed vs Repeatable Read difference.

<details>
<summary>Answer</summary>

No. MVCC solves visibility, not write conflicts. Under Read Committed, a naive app-level read-then-write pattern (SELECT balance, compute new value in app code, UPDATE with that value) can silently lose an update — Postgres re-applies the UPDATE against the newest committed row (EvalPlanQual) but doesn't know the app's math was based on stale data. Under Repeatable Read (or Serializable), Postgres detects that the row was modified by a transaction that committed after the current transaction's snapshot, and raises "could not serialize access due to concurrent update" — forcing the application to catch the error and retry instead of silently overwriting.

</details>

---

## Q5. Why is "just use MVCC" not a complete interview answer for preventing Lost Update?

<details>
<summary>Answer</summary>

Because whether Lost Update is caught depends entirely on the isolation level, not on MVCC alone. Read Committed (even with MVCC) can silently allow it in naive app code; Repeatable Read/Serializable actively detect the write-write conflict and force a retry. You must name the specific isolation level to give a complete answer.

</details>

---

## 30-Second Elevator Pitch

> MVCC keeps multiple versions of each row instead of locking, so readers and writers act on physically different versions and never block each other — no shared point of contention. Each row carries xmin/xmax metadata, and a transaction's snapshot decides which version it's allowed to see. This is how Postgres actually implements isolation levels: Read Committed takes a fresh snapshot per statement, Repeatable Read takes one snapshot per transaction — which is why Repeatable Read prevents phantoms in practice even though the SQL standard doesn't strictly require it. Crucially, MVCC alone does NOT prevent Lost Update — that depends on isolation level. Read Committed can silently lose an update in naive read-then-write app code; Repeatable Read/Serializable detect the write-write conflict and force an application-level retry via a serialization error. The operational cost of MVCC is VACUUM — reclaiming old row versions no longer visible to any transaction; falling behind causes table bloat. MVCC handles reader-writer conflicts; locking still handles writer-writer conflicts — they work together.

---

## Weak Areas to Watch

- MVCC solves visibility, not write conflicts — Lost Update prevention depends on isolation level, not MVCC alone
- Read Committed = snapshot per statement; Repeatable Read = snapshot per transaction (this difference IS why phantoms are blocked)
