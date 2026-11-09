# Topic 059: Message Queues (the RabbitMQ model)

**Module:** 6 — Messaging, Streaming & Async (taught in the batch of prerequisites for Case Study #3 — Twitter / News Feed)
**Tier:** 🔴 MUST
**Completed:** 2026-11-09
**Confidence:** 5/5

---

## 1. Why This Topic Exists

Topic 058 said: hand the work off and respond. But *hand it off to what?* If the producer calls the worker directly, that is a synchronous call again, and it fails when the worker is down or slow. A **message queue** is the thing in the middle: a durable buffer that holds work until a consumer is ready for it. It is the mechanism behind async processing, and the source of the duplicates that Topic 056's idempotency exists to absorb.

---

## 2. Vocabulary

- **Producer:** code that creates a message ("tweet 9 was posted").
- **Message:** the payload plus metadata.
- **Queue:** the buffer that holds messages in order.
- **Broker:** the server that runs the queues (RabbitMQ, Amazon SQS, ActiveMQ).
- **Consumer:** the worker that takes a message and processes it.

---

## 3. The Work-Queue Model: Competing Consumers

Many consumers read from the **same queue**, and **each message goes to exactly one of them**. That is how you scale: if work piles up, add workers, and the queue spreads the work automatically.

This differs from **publish/subscribe**, where *every* subscriber gets its own copy. Brokers like RabbitMQ support both through an **exchange**:

```
Producer → [ Exchange ] ──routes──► Queue A ──► workers for the fanout service
                         ├────────► Queue B ──► workers for the search-indexing service
                         └────────► Queue C ──► workers for the notification service
```
The producer publishes once; the exchange decides which queues get a copy; each queue's workers then compete among themselves. Result: every service hears about the event, and each service's workers share the load.

---

## 4. Acknowledgements — the Mechanism That Matters Most

```
1. Broker delivers the message to a consumer. It is NOT deleted yet — it is "in flight" (unacked).
2. Consumer processes it.
3. Consumer sends an ACK  →  broker deletes the message.

If the consumer crashes or disconnects before the ACK:
   → the broker notices and REDELIVERS the message to another consumer.
```
Redelivery is how work survives a worker crash. SQS does the same with a **visibility timeout**: the message is hidden for N seconds, and if it isn't deleted by then, it reappears.

**When you ack decides your guarantee** (ties to Topic 056):

| Ack when… | If the worker crashes mid-way | Guarantee |
|---|---|---|
| **Before** processing | The work is lost (the broker already deleted it) | at-most-once |
| **After** processing | The message is redelivered, so the work may run twice | at-least-once |

Almost everyone acks **after** processing, so **consumers must be idempotent** (Topic 056): a crash after the work but before the ack means the same message arrives again.

**Rejecting:** a consumer can send a NACK ("I can't handle this"); the broker then requeues it or routes it to a dead-letter queue.

---

## 5. Durability — the Whole Reliability Chain

```
Producer ──(publisher confirm)──► Broker stores it ──(durable queue, persisted to disk)──►
Consumer ──(ack after processing)──► Broker deletes it
```
- **Publisher confirms:** the broker tells the *producer* "I stored it." Without them the producer cannot know the message arrived.
- **Durable queue + persistent messages:** written to disk, so a broker restart doesn't lose them.
- **Replicated queues:** messages kept on more than one broker, so losing one machine doesn't lose the queue.

---

## 6. Failure Handling

- **Retries with a delay.** Don't hammer a failing dependency in a tight loop.
- **Poison messages.** A message that fails *every* time (bad data) would be redelivered forever, wasting workers and, if order matters, blocking the queue. After N failed attempts, route it to a **dead-letter queue (DLQ)**; the main queue keeps moving. **Alert on DLQ depth** so someone actually looks at it.
- **Message TTL:** drop messages that are too old to be useful.

---

## 7. Ordering

A single queue with a single consumer gives first-in-first-out. With several competing consumers, or with retries, **order is not guaranteed**: a redelivered message can finish after later ones. If you need order per entity (say per user), route all of that entity's messages to the same consumer. Kafka's partitioning (Topic 060) is the standard answer.

---

## 8. Backpressure and Capacity

From Topic 058: if arrival rate exceeds the workers' combined rate, the backlog grows. Fixes, in order: add consumers, slow producers (rate limiting, Topic 071), or shed low-priority work. Always monitor **queue depth** and **message age**.

---

## 9. Queue vs the Alternatives

```
Message queue (RabbitMQ / SQS) : message deleted after ack, per-message acknowledgement, routing.
                                 No replay of old messages.
Kafka (Topic 060)              : a retained log; many readers, replay possible, very high throughput.
Redis List as a queue          : a worker that pops a message and then dies has lost it — no ack.
```
- **RabbitMQ:** rich routing, per-message acks; good throughput (roughly tens of thousands of messages per second per node).
- **SQS:** managed, scales on its own; at-least-once with a visibility timeout.

---

## 10. Worked Example — Twitter Fanout

```
User posts → API writes the tweet to the DB → publishes "tweet_created(9)" to a queue → responds.
A pool of fanout workers consumes: for each follower, ZADD the tweet ID into their timeline
   (idempotent, so a redelivery is harmless — Topic 056)
Ack after all followers are done. Crash → redelivery. A bad message → DLQ.

Capacity: 2,000 tweets/sec at peak; one worker handles 100/sec.
   2,000 ÷ 100 = 20 workers minimum → ~26 with 30% headroom.
   With only 15 workers: 2,000 − 1,500 = 500/sec backlog growth.
```

---

## 11. Decision Box

```
Work can be done later, must survive a worker crash          → queue, ack after processing
Each task handled by one worker among many                    → work queue (competing consumers)
Several services must each react to the same event            → exchange + a queue per service
Need to replay history / many independent readers             → a log (Kafka, 060), not a queue
Failing message must not block everything                     → retries + dead-letter queue
Need strict order per user                                    → same consumer per key / partitioning
```
**Interview sentence:** "I'd put a durable queue between the API and the fanout workers. Workers ack after processing, so a crash causes a redelivery, and the processing is idempotent so redelivery is harmless. Poison messages go to a dead-letter queue."

---

## 12. Common Mistakes

| Mistake | Correction |
|---|---|
| Acking before processing | A crash loses the work (at-most-once) |
| Non-idempotent consumers | Redelivery means duplicates; make processing idempotent (056) |
| No dead-letter queue | One poison message is redelivered forever |
| Assuming ordering with multiple consumers | Retries and parallelism break order |
| Treating the queue as unlimited | A permanent producer/consumer gap grows the backlog without bound |
| Using a Redis List when work can't be lost | A pop followed by a crash loses the message; use a broker with acks |

---

## 13. Real Interview Questions

1. "Why put a queue between two services?"
2. "A consumer crashes mid-processing — what happens to the message?"
3. "Ack before or after processing, and what does each give you?"
4. "How do you handle a message that always fails?"
5. "How do you keep order with multiple consumers?"
6. "How many workers do you need for X msgs/sec?"

---

## 14. Revision Questions
See `Revision/Revision_059.md`.

## 15. Summary
- A queue is a durable buffer between producer and consumer; the broker runs it; competing consumers share one queue (each message to one worker); an exchange can also copy a message to several queues (pub/sub).
- Acks drive reliability: ack after processing gives at-least-once (redelivery on a crash) and requires idempotent consumers; ack before gives at-most-once.
- Durability is a chain: publisher confirms, durable/persistent storage, replication, then consumer acks.
- Poison messages go to a dead-letter queue after N retries; alert on DLQ depth. Ordering isn't guaranteed with parallel consumers or retries.
- Size workers from arrival rate ÷ per-worker rate plus headroom; watch queue depth and message age.
- Queues delete on ack and don't replay; Kafka (060) retains a log.
