# Revision — Topic 058: Why Asynchronous Processing

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-11-04

---

## Q1. What single question decides whether a step should be sync or async?

<details>
<summary>Answer</summary>

Does the caller need this result to proceed? If yes (card charge, stock check), it must be synchronous. If it's a side effect the user doesn't wait for (confirmation email, analytics, fanout), it can be asynchronous.

</details>

---

## Q2. List four benefits of asynchronous processing.

<details>
<summary>Answer</summary>

Lower latency (respond fast), decoupling in time, load leveling (a queue absorbs spikes), failure isolation with retries, and independent scaling of workers — any four.

</details>

---

## Q3. Producers send 500 msgs/sec and workers process 400 msgs/sec. After 5 minutes, what is the backlog and how long would a new message wait?

<details>
<summary>Answer</summary>

Surplus = 500 − 400 = 100/sec; × 300 s = 30,000 messages. A newly added message waits backlog ÷ service rate = 30,000 ÷ 400 = 75 seconds.

</details>

---

## Q4. Name three things that get harder when a step moves from sync to async.

<details>
<summary>Answer</summary>

Consistency (the user may not see the effect yet — read-your-writes, Topic 040), delivery uncertainty (duplicates/loss — Topic 061), and no instant result or error (needs polling or callbacks). Also: more moving parts to debug, and an unbounded backlog if consumers fall behind.

</details>

---

## Q5. Can a queue fix a permanent producer/consumer mismatch? Why or why not?

<details>
<summary>Answer</summary>

No. A queue only buffers a temporary excess. If arrival rate stays above processing rate, the backlog and delay grow without limit — the fix is more workers, plus monitoring queue depth and consumer lag.

</details>

---

## 30-Second Elevator Pitch

> Asynchronous processing means handing work off and responding immediately (often `202 Accepted`) instead of making the caller wait. It buys lower latency, decoupling in time, load leveling, failure isolation with retries and independent scaling — and costs eventual consistency, more moving parts, delivery uncertainty, no instant result, and a backlog that grows without limit if consumers are slower than producers. A queue absorbs temporary spikes only: backlog = (arrival − service rate) × time, delay = backlog ÷ service rate. Decide with one question — does the caller need this result to proceed? Yes means sync (charge the card, check stock); a side effect means async (email, analytics, fanout).

---

## Weak Areas to Watch

- Finish the arithmetic: state the resulting delay (backlog ÷ service rate), not just the backlog size.
