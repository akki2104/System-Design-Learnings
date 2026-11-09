# Revision — Topic 059: Message Queues

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-11-09

---

## Q1. A worker finishes the work but crashes before sending the ack. What does the broker do, and what does that require of the worker?

<details>
<summary>Answer</summary>

The message was never acknowledged, so the broker keeps it and redelivers it to another worker. The worker's processing must therefore be idempotent (Topic 056) so the retry doesn't create duplicate effects.

</details>

---

## Q2. What is the difference between acking before and after processing?

<details>
<summary>Answer</summary>

Ack before processing: the broker deletes the message immediately, so a crash loses the work — at-most-once. Ack after processing: the message is deleted only once the worker confirms it is done, so a crash causes redelivery — at-least-once (and duplicates are possible).

</details>

---

## Q3. Peak arrival is 2,000 msgs/s and each worker handles 100 msgs/s. How many workers at minimum, and what happens with 15?

<details>
<summary>Answer</summary>

2,000 ÷ 100 = 20 workers minimum (about 26 with 30% headroom). With 15 workers capacity is 1,500/s, so the backlog grows by 500 msgs/sec.

</details>

---

## Q4. A message fails every time it is processed. What happens without protection, and what is the fix?

<details>
<summary>Answer</summary>

It is redelivered forever, wasting worker capacity (and blocking the queue if order matters). The fix is a retry limit followed by routing the message to a dead-letter queue, with an alert on DLQ depth.

</details>

---

## Q5. Why is the exchange useful, and how does a work queue differ from publish/subscribe?

<details>
<summary>Answer</summary>

In a work queue each message goes to exactly one of many competing consumers (load sharing). In publish/subscribe every subscriber gets its own copy. An exchange routes one published message into one or more queues, so each service gets a copy (its own queue) while that service's workers share the load.

</details>

---

## 30-Second Elevator Pitch

> A message queue is a durable buffer between a producer and its consumers, run by a broker such as RabbitMQ or SQS. Competing consumers share one queue so workers scale out; an exchange can copy a message into several queues for pub/sub. Acknowledgements drive reliability: ack after processing means a crashed worker's message is redelivered (at-least-once), so consumers must be idempotent; ack before processing risks losing work (at-most-once). Durability is a chain of publisher confirms, persistent/replicated storage and consumer acks. Poison messages go to a dead-letter queue after N retries, ordering isn't guaranteed with parallel consumers, and worker count comes from arrival rate divided by per-worker rate plus headroom. Queues delete on ack and don't replay — Kafka's retained log (Topic 060) does.

---

## Weak Areas to Watch

- None this session — clean pass on all four checkpoint questions.
