# Topic 024 — Transactions & ACID

**Module:** 2 — Data Storage Foundations
**Status:** Completed
**Date:** 2026-07-20
**Confidence:** 4-5/5
**Difficulty:** Medium-Hard

---

## 1. Why This Topic Exists

Unifies Topic 020 (WAL, durability) and Topic 021 (relational schema constraints) into one concept. ACID is the set of four guarantees preventing partial-failure bugs — like the classic bank transfer scenario.

---

## 2. Core Concepts

### The core problem: partial failure

A bank transfer moves $500 from Account A to Account B — two operations: debit A, then credit B. Without protection, a crash exactly between these two operations leaves $500 debited from A but never credited to B — money vanished.

```
BEGIN TRANSACTION
  UPDATE accounts SET balance = balance - 500 WHERE id = 'A';
  UPDATE accounts SET balance = balance + 500 WHERE id = 'B';
COMMIT;
```

**ACID** — Atomicity, Consistency, Isolation, Durability — is the set of guarantees preventing exactly this class of failure.

### Atomicity — all or nothing

A transaction either fully completes or fully rolls back — no partial state is ever visible to anyone.

**Mechanism:** the database tracks uncommitted changes via the WAL (Topic 020). The WAL records the intended change before it's applied; if a transaction never reaches a COMMIT record in the log, crash recovery treats it as incomplete and discards/undoes those changes rather than replaying them — as if neither line ever ran.

### Consistency — valid state to valid state

Every transaction moves the database from one valid state to another valid state — all constraints (foreign keys, unique, NOT NULL, CHECK — Topic 021's schema enforcement) hold both before and after.

```
CHECK (balance >= 0)  ← a transaction that would violate this constraint
                         gets REJECTED, not partially applied
```

**Critical interview trap — ACID's "C" vs CAP's "C" are DIFFERENT concepts:**
```
ACID Consistency → respecting schema constraints WITHIN a single database
CAP Consistency  → all nodes agreeing on data in a DISTRIBUTED system (Topic 046)
```
These share a word but refer to entirely different guarantees. Conflating them is one of the most common mistakes candidates make.

### Isolation — concurrent transactions don't interfere

If two transactions run at the same time, each should behave as if it were the only one running — no seeing the other's half-finished work.

```
Transaction 1: reading account A's balance
Transaction 2: simultaneously updating account A's balance
             → Isolation defines what Transaction 1 is allowed to see
```

The exact rules (what interference is allowed vs forbidden) are covered fully in **Topic 025 (Isolation Levels & Anomalies)**. For now: isolation is the property; the specific levels (Read Committed, Serializable, etc.) are next.

### Durability — committed means committed, permanently

Once a transaction returns "success," its changes survive a crash, even one microsecond later.

**This is not a new mechanism — it's exactly Topic 020's WAL.** COMMIT triggers a WAL fsync — a fast, sequential, durable write — *before* the client gets a success response. If the server crashes right after, WAL replay on restart reconstructs the committed change.

```
ACID mechanism recap:
Atomicity   → WAL/undo log enables rollback of incomplete transactions
Consistency → constraint checks (Topic 021) enforced before commit
Isolation   → concurrency control (locks/MVCC — Topics 026/027)
Durability  → WAL fsync before acknowledging commit (Topic 020)
```

---

## Tech Decision Box: when do you need full ACID?

```
Need full ACID transactions when:
  - Money/inventory/anything where partial writes cause real-world harm
    (bank transfers, e-commerce checkout: payment + inventory decrement)
  - Multiple related writes must succeed or fail together

Can relax ACID when:
  - Analytics/logging/event data — losing or duplicating one log line
    rarely matters, and strict transactions cost throughput and add complexity
  - Systems already designed around eventual consistency (Topic 048 preview)
```

**Interview sentence:** "I'll wrap the payment and inventory decrement in a single transaction because a partial failure here means charging a customer without reserving their item — an unacceptable, hard-to-reverse state. For the analytics event log tracking this purchase, I won't use a transaction — losing an occasional event doesn't justify the throughput cost."

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Conflating ACID's Consistency with CAP's Consistency | ACID-C = schema constraints in one DB; CAP-C = cross-node agreement in a distributed system — different concepts sharing a word |
| Wrapping every database operation in a transaction "to be safe" | Transactions cost time/complexity; skip them where partial failure genuinely doesn't matter (analytics, logging) |
| Thinking durability needs a separate mechanism from WAL | Durability IS the WAL fsync-before-acknowledging-commit behavior from Topic 020 — not something new |

---

## Real Interview Questions

1. "Explain ACID with a real-world example." (universal)
2. "What's the difference between ACID Consistency and CAP Consistency?" (universal, common trap)
3. "Would you use a transaction for logging analytics events? Why or why not?" (Amazon, operational-maturity rounds)
4. "How does a database implement durability without a random-write penalty per transaction?" (ties back to Topic 020)

---

## Revision Questions

1. What mechanism allows a database to roll back an incomplete transaction after a crash?
2. Why is it a mistake to equate ACID's Consistency with CAP's Consistency?
3. What is Durability, mechanistically — what earlier topic already taught you this?
4. When would you deliberately skip wrapping an operation in a transaction?
5. In the bank transfer example, which ACID property specifically prevents money from vanishing mid-transfer?

---

## Cheat Sheet

```
ACID
─────────────────────────────────────────────────
Atomicity   → all-or-nothing; WAL/undo log enables rollback of incomplete txns
Consistency → valid state → valid state; schema constraints hold (Topic 021)
              ≠ CAP's Consistency (cross-node agreement, Topic 046) — different concept!
Isolation   → concurrent txns don't interfere (levels detailed in Topic 025)
Durability  → committed = survives crash; THIS IS WAL fsync-before-ack (Topic 020)

WHEN TO USE FULL TRANSACTIONS
─────────────────────────────────────────────────
Money/inventory/anything where partial writes cause real harm → YES
Analytics/logging/event data → often NO, cost > benefit

MECHANISM MAP
─────────────────────────────────────────────────
Atomicity   ← WAL/undo log
Consistency ← constraint checks (Topic 021)
Isolation   ← concurrency control (Topics 026/027)
Durability  ← WAL fsync (Topic 020)
```

---

## Summary

- **ACID prevents partial-failure bugs** — the classic case being a crash mid-transaction (e.g., a bank transfer).
- **Atomicity**: all-or-nothing, enabled by the WAL/undo log tracking uncommitted changes.
- **Consistency**: valid-state-to-valid-state via schema constraints — NOT the same concept as CAP's Consistency.
- **Isolation**: concurrent transactions don't interfere — the specific rules are Topic 025.
- **Durability**: committed survives a crash — this is literally the WAL mechanism from Topic 020, not a new concept.
- **Not every operation needs a transaction** — weigh the real-world cost of partial failure against the throughput/complexity cost of wrapping everything.

> **You now can:** explain all four ACID properties with their actual implementation mechanisms, correctly distinguish ACID Consistency from CAP Consistency, and reason about when full transactional guarantees are worth their cost.
