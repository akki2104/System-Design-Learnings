# Revision — Topic 008: TCP vs UDP

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "TCP guarantees ordered, reliable delivery via a handshake, sequence numbers, ACKs, and retransmission — but all of that costs latency. UDP skips all of it for speed, with no guarantees. The real decision question isn't 'is speed important' — it's 'does losing or reordering one piece of data cause real harm, or does the next update make it obsolete anyway?' Video calls and DNS: obsolete anyway, UDP is fine. Stock prices and file transfers: real harm, TCP is required. QUIC (HTTP/3) is UDP-based but adds its own reliability layer to get the best of both."

---

## Active-Recall Q&A

<details>
<summary>Q1: Why does guaranteeing delivery inherently cost speed?</summary>

Five mechanisms add latency:
1. Handshake before data flows (connection setup)
2. Sequence numbers on every packet (detect missing/out-of-order)
3. Acknowledgments (ACKs) — receiver confirms receipt
4. Retransmission on timeout — resend costs a round trip
5. Buffering to reorder packets — hold newer packets until older ones arrive

</details>

<details>
<summary>Q2: Describe the 3-way handshake in order.</summary>

1. Client → Server: SYN ("I want to connect")
2. Server → Client: SYN-ACK ("OK, I'm ready too")
3. Client → Server: ACK ("Confirmed, let's go")

Only after this does real data start flowing — costs 1 full round trip (e.g., ~150ms cross-region before byte #1).

</details>

<details>
<summary>Q3: What's the real question to ask when choosing TCP vs UDP?</summary>

NOT "is speed important?" — that's the trap.

The real question: **"Does losing/reordering ONE piece of data cause real harm, or does the next update make the old one obsolete anyway?"**

Harm (wrong decision, corrupted file) → TCP.
Obsolete anyway (old video frame irrelevant) → UDP.

</details>

<details>
<summary>Q4: Why does DNS use UDP despite the risk of losing a query?</summary>

DNS queries are tiny, one-shot, and stateless — no ongoing session. TCP's handshake (1 round trip) would cost more than the entire query+response exchange. If a packet is lost, the fix is trivially cheap: the client just retries the whole tiny, idempotent query. No need to build retransmission into the protocol when the application-level retry is nearly free.

</details>

<details>
<summary>Q5: Trap — a stock trading system needs real-time price updates. TCP or UDP, and why?</summary>

**TCP.** This is the classic trap: "real-time" and "speed matters" superficially suggest UDP, but the real question is whether losing/reordering data causes harm. In video calls, an old frame becomes irrelevant fast — no harm. In stock prices, a missing or out-of-order price update means a trader or algorithm could act on stale/wrong data — real financial harm. TCP's guarantees are worth the latency cost here.

</details>

---

## Key Diagram

```
TCP vs UDP — THE DECISION
─────────────────────────────────────────
Scary requirement: "needs to be fast/real-time"
         │
   Does losing/reordering ONE update cause REAL HARM?
         │
    ┌────┴────┐
   YES        NO (next update makes old one obsolete anyway)
    │          │
   TCP        UDP
```

---

## My Weak Areas (from lesson 2026-07-09)

- None major — mastery check passed cleanly, including the trading-system trap question

---

## Past Mistakes

None logged for this topic — clean pass.
