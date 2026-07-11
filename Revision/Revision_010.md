# Revision — Topic 010: HTTP/1.1, HTTP/2, HTTP/3

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "HTTP/1.1 has app-level head-of-line blocking — the ~6-connections-per-domain cap forces requests to queue. HTTP/2 fixes this by multiplexing many requests over ONE TCP connection using binary framing, plus HPACK header compression. But HTTP/2 exposes a new problem: TCP guarantees ordered delivery for the WHOLE connection, so one lost packet blocks ALL multiplexed streams — transport-level head-of-line blocking. This can't be fixed by changing HTTP again, so HTTP/3 replaces TCP entirely with QUIC, built on UDP, giving each stream independent delivery. A lost packet in HTTP/3 only blocks its own stream."

---

## Active-Recall Q&A

<details>
<summary>Q1: What is application-level head-of-line blocking, and which HTTP version suffers from it?</summary>

**HTTP/1.1.** Browsers cap connections per domain (~6). A page needing 30 resources loads 6 immediately; the other 24 queue, waiting for a connection to free up. The old workaround was domain sharding (splitting resources across subdomains) — a hack, not a real fix.

</details>

<details>
<summary>Q2: How does HTTP/2 multiplexing work?</summary>

HTTP/2 sends many requests over **one single TCP connection**, breaking messages into small binary frames that interleave. Instead of [Req1→Res1][Req2→Res2] sequentially, requests/responses are sent and received in parallel, interleaved as frames on the same connection. No more 6-connection cap needed.

</details>

<details>
<summary>Q3: Why does HTTP/2 still suffer from head-of-line blocking, just at a different layer?</summary>

TCP guarantees strictly ordered delivery for the **entire connection**, not per-stream. HTTP/2 multiplexes independent streams (image, CSS, JS) onto one TCP connection, but TCP just sees one ordered byte sequence — it doesn't know about "streams."

If a packet carrying Stream B's data is lost, TCP holds EVERYTHING after it — including already-arrived Stream A and Stream C data — until the lost packet is retransmitted. Unrelated streams get blocked behind one lost packet. This is transport-level head-of-line blocking.

</details>

<details>
<summary>Q4: Why did HTTP/3 need a new transport protocol instead of improving HTTP/2 further?</summary>

TCP's connection-wide ordering guarantee is baked into TCP itself — it can't be fixed by changing HTTP's framing again (that's an application-layer change; the problem lives at the transport layer). So HTTP/3 replaces TCP entirely with QUIC, built on UDP, which gives each stream its own independent delivery guarantee.

</details>

<details>
<summary>Q5: What does QUIC do differently from TCP regarding stream independence?</summary>

QUIC reimplements reliability (ACKs, retransmission) **per-stream**, instead of one shared ordered byte stream for the whole connection. If Stream B's packet is lost, only Stream B pauses — Streams A and C are unaffected. QUIC also bundles the handshake and TLS negotiation together, enabling 0-RTT connection resumption on repeat connections.

</details>

---

## Key Diagram

```
HTTP EVOLUTION — WHERE HEAD-OF-LINE BLOCKING MOVES
─────────────────────────────────────────────────
HTTP/1.1 → App-level blocking (6-connection cap forces queueing)
    │ fix: multiplexing
    ▼
HTTP/2   → Transport-level blocking (TCP's shared ordering blocks all streams)
    │ fix: replace transport entirely
    ▼
HTTP/3   → No blocking (QUIC gives each stream independent delivery)
```

---

## My Weak Areas (from lesson 2026-07-09)

- Needed the transport-level head-of-line blocking mechanism explained fully (didn't have it) — now understood
- Mastery check answer needed slight polish on "TCP guarantees ordering for the CONNECTION, not per-stream" phrasing

---

## Past Mistakes

None logged — no full mistake, just needed the transport-level explanation walked through.
