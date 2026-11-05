# Topic 056: Idempotency & Exactly-Once Semantics

**Module:** 5 — Distributed Systems (taught early, in the Module 6 batch, as a prerequisite for Case Study #3 — Twitter / News Feed — and for the later Payment Gateway case study)
**Tier:** 🔴 MUST
**Completed:** 2026-11-05
**Confidence:** 5/5

---

## 1. Why This Topic Exists — the Lost-Response Problem

A client sends `POST /charge $50`. The server charges the card successfully, but the **response is lost** (a timeout or dropped connection). The client now knows only that it got no answer. It has two choices, and both can be wrong:

- **Retry:** if the first attempt had worked, the customer is charged twice.
- **Don't retry:** if the first attempt had failed, the customer is never charged.

From the client's side, "my request never arrived" and "my request arrived but the reply got lost" look **identical**. Retrying is the only safe-feeling move, so the system has to make retries harmless. That property is **idempotency**. It is the safety net for the asynchronous work introduced in Topic 058 and the answer to the duplicates that at-least-once delivery creates (Topic 061).

---

## 2. Three Outcomes of Sending a Message (plain-language preview of Topic 061)

- **At-most-once:** send once, never retry. It might be lost, but it never duplicates.
- **At-least-once:** retry until acknowledged. It never gets lost, but it might arrive more than once.
- **Exactly-once:** processed one time, no more, no less. Everyone wants it; it is the hard one.

Most real systems choose **at-least-once** and then handle the duplicates.

---

## 3. What "Idempotent" Means

An operation is **idempotent** if doing it N times leaves the **same final state** as doing it once (Topic 012: it is about *state*, not the response body).

```
IDEMPOTENT                      NOT idempotent
─────────────────────────────   ─────────────────────────────
balance = 100                   balance = balance + 10
DELETE /orders/7                POST /orders          (creates a new one each time)
PUT /users/5 {name: "Asha"}     "charge $50"
ZADD timeline 123 tweet9        LPUSH timeline tweet9 (appends a duplicate)
status = "shipped"              send an email
```
HTTP `GET`, `PUT` and `DELETE` are meant to be idempotent; `POST` is not and needs help.

**Design habit:** where possible, choose the naturally idempotent form of an operation. Adding a tweet to a timeline with `ZADD` (a sorted set stores each member once) is safe to repeat; appending with `LPUSH` creates duplicates.

---

## 4. The Idempotency Key Pattern (the core of the topic)

For operations that aren't naturally idempotent, give each **intent** a name.

```
1. The CLIENT generates a unique key (a UUID) once, when the user clicks "Pay".
2. It sends it with the request:  POST /charge   Idempotency-Key: K1
3. The SERVER, atomically:
     key K1 never seen   → claim it, do the work, STORE the result under K1
     key K1 already done → do NOT redo the work; return the STORED result
```
The lost-response story now ends well: the first request charges and stores the result, the response is lost, the client retries with `K1`, the server recognizes `K1`, skips the charge, and returns the saved result.

**Details interviewers probe:**
1. **Reuse the same key on every retry.** A new key per retry defeats the purpose — the server sees a brand-new request and charges again.
2. **The check-and-claim must be atomic.** Two retries arriving at once must not both pass the "not seen" check. Use a database unique constraint on the key, or Redis `SET NX` (the same atomicity idea as Topic 071).
3. **Handle "in progress."** A duplicate arriving while the first is still running should get a conflict or wait — not run twice.
4. **Store the response too**, so a duplicate gets the same answer.
5. **Tie the key to the request body** (store a hash). Same key with a *different* body is a client bug — reject it.
6. **Expire keys after a retention window** (e.g. 24 hours). A retry arriving after expiry would execute again, so the window must outlast realistic retry times.

---

## 5. The Crash Window (a classic trap)

Suppose the server does **(1) charge the card, then (2) write the idempotency record**, and crashes between the two. On retry there is no record, so it charges again.

**Fix:** for money, write the **effect and the idempotency record in one database transaction**, so either both happen or neither. Redis is fine as a fast pre-check, but for payments the authoritative record belongs in the same transactional database as the effect (previewed in R01 §4.8(i)).

When the effect is an **external call** (you call a payment provider), you can't put it inside your own transaction. So **pass the idempotency key downstream** to the provider as well — idempotency has to hold end to end.

---

## 6. "Exactly-Once" Is Mostly a Myth

Over an unreliable network you cannot guarantee a message is delivered exactly one time. What good systems provide is **effectively-once**:

```
at-least-once delivery  +  idempotent processing  =  effectively exactly-once
```
Some systems advertise "exactly-once" (Kafka does, within its own transactional boundaries — Topics 060/061). Treat the phrase with care, and in an interview say *what* is exactly-once and *where*.

---

## 7. Other Ways to Get Idempotency

- **Natural business key + unique constraint:** one payment per `order_id`; a second insert fails harmlessly.
- **Conditional / versioned writes:** "update only if version = 7" (compare-and-set, Topics 025/027).
- **Consumer-side dedup:** a message consumer records processed event IDs and skips repeats — the standard answer for queue consumers.
- **Upserts** instead of inserts.

---

## 8. Worked Example — Payments

```
Table idempotency(key PRIMARY KEY, request_hash, status, response, created_at)

Request 1 (Idempotency-Key K1):
  BEGIN
    INSERT idempotency(K1, hash, 'in_progress')    ← unique constraint = atomic claim
    INSERT ledger entry (the charge)                ← the effect
    UPDATE idempotency SET status='done', response=...
  COMMIT                                            ← effect + record, all or nothing
  response lost on the network

Retry (K1):  INSERT conflicts → read stored row → return the stored response. No second charge.
Concurrent duplicate while Request 1 is running:  conflict + status 'in_progress' → return 409.
```

**Twitter tie-in:** a fanout worker may be retried after a crash. Writing the tweet ID into each follower's timeline with `ZADD` is naturally idempotent, so a retried fanout cannot create duplicate timeline entries.

---

## 9. Decision Box

```
Operation is naturally idempotent (set, PUT, DELETE, ZADD)      → just allow retries
Operation isn't (create, charge, send, increment) and may be
  retried by a client, queue, or webhook                         → idempotency key + dedup
Money / must never double-apply                                  → record in the SAME DB transaction
                                                                   as the effect; pass the key downstream
Low-stakes duplicate protection                                  → Redis SET NX EX is fine
```
**Interview sentence:** "Delivery is at-least-once, so I make processing idempotent. The client sends an idempotency key, I atomically claim it in the same transaction as the charge, and a retry returns the stored result instead of charging again."

---

## 10. Common Mistakes

| Mistake | Correction |
|---|---|
| New idempotency key on every retry | One key per *intent*, reused on every retry |
| Writing the effect first, the record second | A crash between them duplicates on retry; commit both in one transaction |
| "GET is idempotent because it returns the same response" | Idempotency is about final *state*, not the response body |
| Believing exactly-once delivery exists | It is at-least-once plus idempotent processing |
| Expiring keys too early | A retry after expiry runs again |
| Ignoring same-key-different-body | Reject it; it is a bug |
| Forgetting external calls | A call to another service can't join your transaction — forward the key |

---

## 11. Real Interview Questions

1. "A payment request times out. How do you make a retry safe?"
2. "Walk me through an idempotency-key implementation, including concurrent duplicates."
3. "Why can't you write the effect and then the idempotency record separately?"
4. "Is exactly-once delivery possible? What do real systems do?"
5. "Which operations are naturally idempotent, and how would you make a POST idempotent?"

---

## 12. Revision Questions
See `Revision/Revision_056.md`.

## 13. Summary
- The lost-response problem: a client can't tell "request lost" from "response lost," so retries must be made safe.
- Idempotent = same final state after N executions; `GET`/`PUT`/`DELETE` are, `POST` isn't. Prefer naturally idempotent operations (`ZADD` over `LPUSH`).
- Idempotency key: one key per intent, reused on every retry; atomic claim; store the response; handle in-progress; bind to the request body; expire after a retention window.
- For money, commit the effect and the idempotency record in one transaction; forward the key to external providers.
- Exactly-once delivery isn't achievable over a network; real systems give effectively-once = at-least-once delivery + idempotent processing.
