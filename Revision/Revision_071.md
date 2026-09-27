# Revision — Topic 071: Rate Limiting & Throttling

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-09-27

---

## Q1. A payment API needs to allow a burst up to 20 requests instantly but average no more than 5/sec long-term. Which algorithm fits, and why not the alternative?

<details>
<summary>Answer</summary>

Token bucket — the bucket holds up to 20 tokens (burst capacity) refilled at 5/sec (the average rate), so a client can spend accumulated tokens instantly but can't sustain more than the refill rate over time. Leaky bucket is the wrong fit because it smooths everything to a constant output rate and doesn't allow any burst through, even a legitimate one like retrying a batch of failed payments.

</details>

---

## Q2. Why does Redis's atomic INCR matter for rate limiting, tying back to Topic 026?

<details>
<summary>Answer</summary>

Checking the current count against the limit and then incrementing it must be one atomic operation. If read and increment are two separate steps, two concurrent requests can both read the same pre-increment count, both see "under limit," and both get allowed — the same read-then-write race pattern as Topic 026's locking discussion. Redis's INCR performs the read-and-update as a single atomic operation, closing that race.

</details>

---

## Q3. 10 app servers each keep their own in-memory counter capped at 100 requests/minute per server. What's precisely wrong with this?

<details>
<summary>Answer</summary>

The effective limit becomes N × 100 = 1000 requests/minute across the cluster, not 100 — because the servers don't share state, none of them knows how many requests the others have already allowed. The counter must live in shared state (e.g., Redis) so the limit means what it says regardless of how many servers are serving traffic.

</details>

---

## Q4. What's the specific flaw in the Fixed Window Counter algorithm?

<details>
<summary>Answer</summary>

The boundary-burst flaw: a client can send the full limit right at the end of one window and the full limit again right at the start of the next window — up to 2x the intended limit within a short span straddling the boundary — because the algorithm only ever evaluates one window at a time with no memory of the previous one.

</details>

---

## Q5. What headers should a 429 response include, and why do they matter?

<details>
<summary>Answer</summary>

`Retry-After`, `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`. They let a well-behaved client back off intelligently (wait the right amount of time, understand its remaining quota) instead of guessing or retrying immediately — a client that ignores them and retries right away makes the overload worse, not better (ties to Topic 069's retry/backoff).

</details>

---

## 30-Second Elevator Pitch

> Rate limiting enforces a system's estimated capacity ceiling against any single client, best centralized at the API Gateway with shared state (Redis) — per-server in-memory counters silently multiply the effective limit by the server count. Five algorithms trade off accuracy, memory, and burst tolerance: Fixed Window (simple, boundary-burst flaw), Sliding Window Log (accurate, memory-heavy), Sliding Window Counter (the practical middle ground), Token Bucket (allows bursts, steady average — most real APIs), and Leaky Bucket (hard constant rate, no bursts ever). The check-and-increment must be atomic, mirroring Topic 026's race-condition lesson. Exceeded limits return 429 with headers that let clients back off correctly rather than hammering harder.

---

## Weak Areas to Watch

- None this session — clean pass on all three checkpoint questions.
