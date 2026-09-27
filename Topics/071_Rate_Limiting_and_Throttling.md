# Topic 071: Rate Limiting & Throttling

**Module:** Building Blocks (taught JIT, pre-Case-Study-#2 batch)
**Tier:** 🔴 MUST
**Completed:** 2026-09-27
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Every system estimated in Topic 003 has a capacity ceiling — a QPS beyond which it degrades or falls over. Rate limiting is the mechanism that enforces "no client gets to exceed their fair share," protecting the system from a single abusive/buggy client, protecting downstream dependencies from overload, and enabling tiered pricing (free vs paid API limits). It's one of the most universally asked case studies precisely because the algorithms have real, nameable tradeoffs — not just "add a counter." Direct prerequisite for Case Study #2 (Rate Limiter).

---

## 2. Where to Enforce It

- **Client-side:** advisory only — a well-behaved client can self-throttle, but nothing stops a malicious or buggy one from ignoring it. Never trust this alone.
- **API Gateway / reverse proxy (Topic 016):** the standard place — centralizes rate limiting once, at the edge, so individual services don't each need their own logic.
- **Per-service:** more granular (different limits per endpoint), but requires coordinating state across every service instance.

---

## 3. The Core Tension: In-Memory vs Shared State

If each app server behind a load balancer tracks its own request counter in local memory, the *effective* limit is silently multiplied by the number of servers — a "100 requests/minute" limit becomes 500/minute across 5 servers (or **1000/minute across 10 servers**), since each server only sees and counts its own slice of traffic; the servers have no way of knowing how much the others have already allowed. **The counter must be shared/centralized** across all servers for the limit to mean what it says. This is exactly why Redis is the near-universal implementation choice — fast, atomic increment operations, and TTL-based window expiry, all things already covered in Topics 032-037.

---

## 4. The Five Algorithms

**Fixed Window Counter** — divide time into fixed windows (e.g., per-minute), count requests per window, reset at the boundary. Simple, cheap. **The boundary-burst flaw:** a client can send the full limit at the very end of window 1 and the full limit again at the very start of window 2 — 2x the intended limit within a couple of seconds, since the algorithm only ever looks at one window at a time.

**Sliding Window Log** — keep a timestamped log of every request in the trailing window; count log entries to decide if under limit. Perfectly accurate, no boundary flaw — but memory-expensive, since every individual request timestamp must be stored per client.

**Sliding Window Counter** — a hybrid: weight the *previous* fixed window's count proportionally by how much it overlaps the current sliding window, and add the current window's count. Approximate, but far cheaper than logging every timestamp, and largely smooths the boundary-burst problem without the log's memory cost.

**Token Bucket** — a bucket holds up to N tokens, refilled at a fixed rate. Each request consumes a token; empty bucket → reject/delay. **Allows bursts** up to the bucket's capacity while enforcing a steady-state average rate over time (e.g., burst up to 20 instantly, average no more than 5/sec long-term). What most real-world APIs (Stripe, AWS) actually use.

**Leaky Bucket** — requests enter a queue and are processed ("leak out") at a fixed constant rate; a full queue drops new requests. **Smooths everything into a constant output rate** — does not allow bursts through at all, even legitimate ones.

**The key distinguishing tradeoff:** token bucket tolerates bursts (better UX for legitimately spiky-but-valid usage, e.g. a client retrying a batch of failed payments); leaky bucket enforces a hard constant rate (better when the downstream system genuinely cannot handle any burst, period). Neither is "better" — they solve for different goals.

---

## 5. Atomicity Is Non-Negotiable

Checking the current count against the limit and then incrementing it must happen as **one atomic operation**. If it's two separate steps (read count, then increment), two concurrent requests can both read count=9 (limit=10), both see "under limit," both increment to 10, and both get allowed — when the correct sequential outcome should have rejected one right at the boundary. The same read-then-write race pattern as Topic 026's locking discussion. Redis's `INCR` is a single atomic operation for exactly this reason; more complex check-and-act logic (e.g., token bucket's refill math) typically needs a Lua script executed atomically server-side — full detail in R01.

---

## 6. What the Client Sees When Limited

`HTTP 429 Too Many Requests`, plus standard headers so well-behaved clients can back off intelligently rather than guessing: `Retry-After`, `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (ties directly to Topic 069's retry/backoff preview — a client that ignores these headers and retries immediately makes the overload worse, not better).

---

## 7. Granularity: What Gets Limited

Per-user, per-API-key, per-IP, or global. Per-IP is the weakest signal — easily defeated by IP rotation, or overly punishing to many legitimate users sharing one IP behind NAT. Per-user/per-API-key is precise but requires authentication to already be in place before rate limiting can apply.

---

## 8. Decision Box

```
Need simplicity, boundary-burst risk acceptable      → Fixed Window
Need perfect accuracy, memory cost acceptable         → Sliding Window Log
Need good accuracy, low memory cost (most common)     → Sliding Window Counter
Need to ALLOW legitimate bursts, steady average rate  → Token Bucket
Need to ENFORCE a hard constant rate, no bursts ever   → Leaky Bucket
```
**Interview sentence:** "I'd use token bucket — it allows short legitimate bursts (a user retrying after a network blip shouldn't get instantly blocked) while still enforcing a steady-state average rate, and I'd implement it in Redis with a Lua script so the check-and-decrement is atomic across all app servers."

---

## 9. Common Mistakes

| Mistake | Correction |
|---|---|
| Implementing the counter in local app-server memory in a multi-server deployment | The effective limit gets multiplied by the number of servers (e.g., 10 servers × 100/min = 1000/min) — the counter must live in shared state (Redis) |
| Confusing token bucket with leaky bucket | Token bucket allows bursts up to capacity; leaky bucket smooths everything to a constant output rate — opposite tradeoffs for opposite goals |
| Not naming fixed window's boundary-burst flaw | 2x the limit can pass through in a short window straddling the boundary — a specific, nameable flaw, not just "it's simple" |
| Treating check-then-increment as safe without atomicity | A classic read-then-write race — two concurrent requests can both pass a check that should have rejected one of them |

---

## 10. Revision Questions
See `Revision/Revision_071.md`.

## 11. Summary
- Rate limiting protects a system from exceeding its estimated capacity ceiling (Topic 003) — enforced at the client (advisory only), the API Gateway (standard), or per-service.
- In a multi-server deployment, per-server in-memory counters silently multiply the effective limit — the counter must be shared/centralized (Redis).
- Five algorithms: Fixed Window (simple, boundary-burst flaw), Sliding Window Log (accurate, memory-expensive), Sliding Window Counter (good accuracy, low memory — the common middle ground), Token Bucket (allows bursts, steady average), Leaky Bucket (enforces a hard constant rate, no bursts).
- Token bucket vs leaky bucket is the classic interview distinction: burst-tolerant vs burst-eliminating.
- The check-and-increment must be atomic (Topic 026's race-condition pattern) — Redis's `INCR`, or a Lua script for compound logic.
- Exceeded limits return 429 with `Retry-After`/`X-RateLimit-*` headers so well-behaved clients back off correctly (ties to Topic 069).
