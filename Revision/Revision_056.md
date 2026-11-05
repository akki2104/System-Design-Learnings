# Revision — Topic 056: Idempotency & Exactly-Once Semantics

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-11-05

---

## Q1. What is the lost-response problem, and why does it force retries to be made safe?

<details>
<summary>Answer</summary>

The server may have completed the work, but the response was lost, so the client can't tell "request lost" from "response lost." Retrying risks a duplicate (double charge); not retrying risks the work never happening. Since retrying is the only safe-feeling choice, the system must make retries harmless — idempotency.

</details>

---

## Q2. Which are naturally idempotent: `balance = 100`, `balance += 10`, `DELETE /orders/7`, `POST /orders`, `ZADD timeline 123 tweet9`, `LPUSH timeline tweet9`?

<details>
<summary>Answer</summary>

Idempotent: `balance = 100`, `DELETE /orders/7`, `ZADD timeline 123 tweet9` (a sorted set holds each member once). Not idempotent: `balance += 10`, `POST /orders` (creates a new order each time), `LPUSH` (appends a duplicate).

</details>

---

## Q3. Describe the idempotency-key flow, including what the server must store and what happens if the client generates a new key on retry.

<details>
<summary>Answer</summary>

The client generates one key per intent and sends it with the request. The server atomically claims the key; if unseen, it does the work and stores the result under the key; if already done, it returns the stored response without redoing the work. It must store the key and the response (ideally also a request hash and an in-progress status). If the client generates a new key on retry, the server treats it as a new request and charges again.

</details>

---

## Q4. A server charges the card, then writes the idempotency record in a separate step, and crashes between them. What goes wrong and what is the fix?

<details>
<summary>Answer</summary>

On retry there is no record, so the server charges again. Fix: commit the effect and the idempotency record in one database transaction (all or nothing). If the effect is an external call that can't join the transaction, also forward the idempotency key to that provider.

</details>

---

## Q5. Why is "exactly-once delivery" a claim to be skeptical of, and what do real systems provide instead?

<details>
<summary>Answer</summary>

Failures over an unreliable network mean a message can't be guaranteed to be delivered exactly one time. Real systems provide effectively-once: at-least-once delivery plus idempotent processing (deduplication). When a system advertises exactly-once (e.g. Kafka), name what is exactly-once and within which boundary.

</details>

---

## 30-Second Elevator Pitch

> A client can't tell a lost request from a lost response, so retries must be safe. An operation is idempotent if repeating it leaves the same final state; GET, PUT and DELETE are, POST isn't — so POST gets an idempotency key: the client generates one key per intent and reuses it on every retry, the server atomically claims it, does the work, stores the response, and returns the stored response on a duplicate (handling in-progress duplicates, binding the key to the request body, and expiring keys after a retention window). For money, the effect and the idempotency record are committed in one transaction, and the key is forwarded to external providers. Exactly-once delivery isn't achievable over a network; real systems deliver at-least-once and process idempotently, which is effectively-once.

---

## Weak Areas to Watch

- None this session — clean pass. Q3 (crash window) was correct but loosely worded; state the mechanism crisply: no record exists, so the retry charges again.
