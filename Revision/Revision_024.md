# Revision — Topic 024: Transactions & ACID

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "ACID prevents partial-failure bugs like a crash mid-bank-transfer. Atomicity means all-or-nothing, enabled by the WAL/undo log tracking uncommitted changes for rollback. Consistency means the database moves from one valid state to another, respecting schema constraints — this is a DIFFERENT concept from CAP's Consistency, which is about cross-node agreement in a distributed system. Isolation means concurrent transactions don't interfere (exact rules are Topic 025). Durability means committed changes survive a crash — and this isn't a new mechanism, it's literally the WAL fsync-before-acknowledging-commit behavior from Topic 020. Not every operation needs a transaction — skip it where partial failure genuinely doesn't matter, like analytics logging."

---

## Active-Recall Q&A

<details>
<summary>Q1: What mechanism allows a database to roll back an incomplete transaction after a crash?</summary>

The WAL (Write-Ahead Log, Topic 020). It records the intended change before applying it to data pages. If a transaction never reaches a COMMIT record in the log, crash recovery treats it as incomplete and discards/undoes those changes — as if the transaction never happened.

</details>

<details>
<summary>Q2: Why is it a mistake to equate ACID's Consistency with CAP's Consistency?</summary>

They're different concepts sharing the same word:
- **ACID Consistency:** respecting schema constraints (foreign keys, uniqueness, CHECK) WITHIN a single database.
- **CAP Consistency:** all nodes agreeing on the same data in a DISTRIBUTED system (Topic 046).

Conflating them is one of the most common interview mistakes.

</details>

<details>
<summary>Q3: What is Durability, mechanistically — what earlier topic already taught you this?</summary>

Durability means once a transaction commits, its changes survive a crash, even a microsecond later. This is exactly Topic 020's WAL mechanism: COMMIT triggers a WAL fsync (fast, sequential, durable write) BEFORE the client gets a success response. If the server crashes right after, WAL replay on restart reconstructs the committed change. It's not a separate concept — WAL IS how durability is implemented.

</details>

<details>
<summary>Q4: When would you deliberately skip wrapping an operation in a transaction?</summary>

When partial failure genuinely doesn't cause real-world harm — e.g., analytics/logging/event data, where losing or duplicating an occasional log line doesn't matter, and the throughput/complexity cost of a strict transaction isn't justified.

</details>

<details>
<summary>Q5: In the bank transfer example, which ACID property specifically prevents money from vanishing mid-transfer?</summary>

**Atomicity.** If the crash happens between debiting Account A and crediting Account B, atomicity ensures the entire transaction rolls back — the debit never "sticks" without the corresponding credit. All-or-nothing.

</details>

---

## Key Diagram

```
ACID MECHANISM MAP
─────────────────────────────────────────
Atomicity   ← WAL/undo log (rollback incomplete txns)
Consistency ← constraint checks (Topic 021 schema enforcement)
Isolation   ← concurrency control (Topics 026/027 — locks/MVCC)
Durability  ← WAL fsync before ack (Topic 020 — SAME mechanism, not new)

ACID-C vs CAP-C — DO NOT CONFLATE
─────────────────────────────────────────
ACID Consistency → schema constraints, ONE database
CAP Consistency  → cross-node agreement, DISTRIBUTED system
```

---

## My Weak Areas (from lesson 2026-07-20)

- None significant — clean 4/4 pass, including the ACID-vs-CAP consistency trap and the "skip transactions for analytics" reasoning

---

## Past Mistakes

None logged for this topic — clean pass across all checkpoint questions.
