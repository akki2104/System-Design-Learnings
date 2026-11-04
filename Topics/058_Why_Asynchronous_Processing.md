# Topic 058: Why Asynchronous Processing

**Module:** 6 — Messaging, Streaming & Async (taught early, as a prerequisite to Case Study #3 — Twitter / News Feed)
**Tier:** 🔴 MUST
**Completed:** 2026-11-04
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Every design so far has been **synchronous**: the caller sends a request and waits for the full answer. That works until one step in the chain is slow, unreliable, or has too much work. Anchor example: posting a tweet. If posting means "write the tweet, deliver it to all 10 million followers' timelines, then respond," the poster waits minutes for a 200 OK. Asynchronous processing means: write the tweet, **hand off** the delivery work, respond immediately, and let someone else finish later. This topic sets up the whole messaging module (queues 059, logs 060, delivery guarantees 061).

---

## 2. Sync vs Async

- **Synchronous:** the caller waits until the work is done. Simple; the result comes back immediately.
- **Asynchronous:** the caller hands off the work and moves on. A worker does it later; the result is delivered later, or never needed.

In HTTP terms the async answer is often **`202 Accepted`** — "I got it, I'll do it," not "it's done."

---

## 3. What Async Buys You

1. **Lower latency** — respond fast, do the slow part afterwards.
2. **Decoupling in time** — producer and consumer don't have to be up at the same moment.
3. **Load leveling** — a queue absorbs a spike; work arrives in a burst and is processed at the workers' steady pace.
4. **Failure isolation and retries** — if a downstream service is down, the work waits and is retried instead of failing the user's request.
5. **Independent scaling** — add workers without touching the producer.

---

## 4. What Async Costs You

1. **Eventual consistency** — "I posted but it's not in my feed yet" is the read-your-writes problem from Topic 040, now caused by design.
2. **More moving parts** — debugging "where did my message go" is harder than reading a stack trace.
3. **Delivery uncertainty** — messages can be duplicated or lost (Topic 061).
4. **No instant result or error** — the caller must poll a status or receive a callback.
5. **Unbounded backlog** — if workers are slower than producers, the queue grows without limit.

Async replication (Topic 039) is the same tradeoff: fast acknowledgement, small risk of loss, some staleness.

---

## 5. Worked Numeric Example — the Queue as a Buffer, and Its Limit

Producers send 1,000 msgs/sec; workers process 800 msgs/sec.
```
Backlog grows by 200/sec.
After 10 minutes: 200 × 600 = 120,000 messages waiting.
Delay for a newly added message ≈ backlog ÷ processing rate = 120,000 ÷ 800 = 150 s.
```
A queue absorbs a *temporary* spike. It cannot fix a *permanent* producer/consumer mismatch — that needs more workers, plus monitoring of queue depth and consumer lag.

**Checkpoint example (the learner's):** producers 500/s, workers 400/s for 5 minutes → surplus 100/s × 300 s = **30,000** backlog → a newly added message waits 30,000 ÷ 400 = **75 s**.

---

## 6. Decision Box: Sync or Async?

```
The user needs the result to continue (login check, payment authorization,
  stock check before an order)                                              → SYNC
It's a side effect the user doesn't wait for (email, fanout, analytics,
  thumbnails, notifications)                                                → ASYNC
It must be reliable even if a downstream service is down                    → ASYNC + retries
```
**The test:** *does the caller need this result to proceed?* Charging the card and checking stock are sync (the order can't continue, or must not be created, without the answer); the confirmation email and analytics update are async.

**Interview sentence:** "I'd accept the tweet synchronously — that's just a durable write — and fan it out asynchronously through a queue. The poster gets a fast response and followers see it a moment later."

---

## 7. Common Mistakes

| Mistake | Correction |
|---|---|
| Making everything async | If the caller needs the answer to proceed, async only adds complexity |
| Treating a queue as free capacity | It buffers only *temporary* excess; a permanent mismatch grows the backlog forever |
| Forgetting what the user sees | Async means eventual consistency — design for "not visible yet" |
| Ignoring failure semantics | Async work can be retried, so it can be duplicated — Topic 061 |
| Stopping a backlog calculation at the backlog size | Also state the resulting delay (backlog ÷ service rate) — that is what the user feels |

---

## 8. Real Interview Questions

1. "Which parts of placing an order are synchronous and which asynchronous, and why?"
2. "Why use a queue between the API and a worker?"
3. "What happens if producers consistently outpace consumers?"
4. "What does the user experience when a step becomes asynchronous?"
5. "How is async replication the same tradeoff as an async queue?"

---

## 9. Revision Questions
See `Revision/Revision_058.md`.

## 10. Summary
- Synchronous = caller waits for the full result; asynchronous = hand off the work and respond now (`202 Accepted`).
- Benefits: lower latency, time decoupling, load leveling, failure isolation + retries, independent scaling.
- Costs: eventual consistency, more moving parts, delivery uncertainty, no instant result, unbounded backlog if consumers are slower.
- A queue buffers temporary spikes only; backlog = (arrival − service rate) × time, delay = backlog ÷ service rate.
- Decide by one question: does the caller need this result to proceed? Yes → sync; side effect → async.
