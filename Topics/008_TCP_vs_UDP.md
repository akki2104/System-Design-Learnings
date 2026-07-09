# Topic 008 — TCP vs UDP

**Module:** 1 — Networking & Communication Foundations
**Status:** Completed
**Date:** 2026-07-09
**Confidence:** 4/5
**Difficulty:** Medium

---

## 1. Why This Topic Exists

TCP and UDP are the two transport protocols underlying nearly every design decision from here on — WebSockets, load balancers, video streaming, gaming, HTTP itself. Choosing between them (or building on top of one) is a recurring interview question.

---

## 2. Core Concepts

### Why guaranteed delivery costs speed

```
1. Handshake before any data flows   (3-way handshake — connection setup)
2. Sequence numbers on every packet  (detect missing/out-of-order)
3. Acknowledgments (ACKs)            (receiver confirms "got it")
4. Retransmission on timeout         (resend if no ACK — costs a round trip)
5. Buffering to reorder packets      (hold packet #5 until #4 arrives)
```
Every guarantee adds latency. **Reliability costs time** — the fundamental tradeoff.

### TCP vs UDP

```
TCP (Transmission Control Protocol)      UDP (User Datagram Protocol)
──────────────────────────────────       ──────────────────────────────
Connection-oriented (handshake first)    Connectionless (just send)
Guaranteed delivery + ordering           No guarantees — loss/duplication/
                                          reordering all possible
Flow control + congestion control        No flow control
Slower (overhead of guarantees)          Faster (no overhead)
Used for: file transfer, APIs, web,      Used for: video/voice calls, gaming,
          databases, trading systems               DNS, live streaming
```

### The 3-Way Handshake (TCP's setup cost)
```
Frame 1: Client → Server   SYN        ("I want to connect")
Frame 2: Server → Client   SYN-ACK    ("OK, I'm ready too")
Frame 3: Client → Server   ACK        ("Confirmed, let's go")

Only AFTER this does real data start flowing — costs 1 full round trip.
```
Cross-region, that's ~150ms wasted before byte #1 is sent — a key reason HTTP/3's QUIC moved to a UDP-based approach (Topic 010).

### The Real Decision Question

Not "is speed important?" (the trap) — but:

> **"Does losing/reordering ONE piece of data cause real harm, or does the next update make the old one obsolete anyway?"**

```
Video/audio call  → old frame becomes irrelevant fast → UDP fine
DNS query         → tiny, stateless, cheap to retry    → UDP fine
Stock prices      → old/missing price = wrong decision  → TCP required
File transfer     → partial/corrupt file is useless     → TCP required
```

**Why DNS uses UDP despite seeming risky:** a DNS query is small, one-shot, and stateless — no ongoing session. TCP's handshake (1 round trip) would cost more than the actual query+response (also ~1 round trip with UDP, none with the handshake). If a packet is lost, the fix is trivially cheap: the client just retries the whole tiny, idempotent query — no need to build retransmission into the protocol itself.

---

## Tech Decision Box: TCP vs UDP vs QUIC

```
TCP    → file transfer, REST APIs, database connections, email — correctness > speed
UDP    → video/voice calls, live streaming, gaming, DNS — speed > perfect delivery
QUIC   → HTTP/3's answer: UDP-based but adds its OWN reliability layer, skipping
          TCP's handshake cost while still guaranteeing delivery (Topic 010)
```

**Interview sentence:** "I'll use UDP here because [data is loss-tolerant / time-sensitive]. I considered TCP but rejected it because the handshake and retransmission overhead would hurt latency more than occasional packet loss hurts the experience."

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Picking UDP just because "speed matters" | Ask: does losing/reordering data cause real harm, or does it become obsolete anyway? |
| Assuming DNS-over-UDP is risky | DNS queries are tiny, stateless, and cheap to retry — handshake overhead would dominate |
| Forgetting TCP's setup cost | 3-way handshake = 1 full round trip before any real data flows |

---

## Real Interview Questions

1. "Why does DNS use UDP but HTTP traditionally uses TCP?" (universal)
2. "We're building a real-time multiplayer game — TCP or UDP?" (gaming companies)
3. "A stock trading system needs real-time updates — is UDP appropriate?" (trading/fintech — the trap question)
4. "Why did HTTP/3 move to UDP-based QUIC instead of TCP?" (Google, networking-focused roles)

---

## Revision Questions

1. Why does guaranteeing delivery inherently cost speed? List the mechanisms.
2. Describe the 3-way handshake in order.
3. What's the real question to ask when choosing TCP vs UDP (not "is speed important")?
4. Why does DNS use UDP despite the risk of losing a query?
5. Trap: a trading system needs real-time price updates — TCP or UDP, and why?

---

## Cheat Sheet

```
TCP vs UDP
─────────────────────────────────────────────────
TCP: connection-oriented, guaranteed+ordered delivery, handshake first, slower
UDP: connectionless, no guarantees, no handshake, faster

3-WAY HANDSHAKE (TCP setup cost)
─────────────────────────────────────────────────
SYN → SYN-ACK → ACK   (1 full round trip before real data flows)

WHEN TO USE WHICH
─────────────────────────────────────────────────
TCP → data must be correct & complete: files, APIs, DB, trading systems
UDP → stale data is harmless, speed matters more: video/voice, gaming, DNS

THE REAL QUESTION (not "is speed important")
─────────────────────────────────────────────────
"Does losing/reordering ONE update cause real harm, or does the
 next update make the old one obsolete anyway?"
Harm → TCP.  Obsolete anyway → UDP.

QUIC (HTTP/3, Topic 010 preview)
─────────────────────────────────────────────────
UDP-based but adds its own reliability layer — skips TCP's handshake
cost while still guaranteeing delivery
```

---

## Summary

- **Reliability costs time** — every TCP guarantee (handshake, ACKs, retransmission, ordering) adds latency.
- **The real decision question:** does losing/reordering data cause real harm, or does it become obsolete anyway?
- **DNS uses UDP** because queries are tiny, stateless, and cheap to retry — not because loss doesn't matter.
- **The trap:** "speed matters" does NOT automatically mean UDP — trading systems need speed AND correctness, so TCP wins.
- **QUIC (HTTP/3)** is UDP-based but adds its own reliability layer, avoiding TCP's handshake cost while keeping guarantees.

> **You now can:** choose between TCP and UDP for any system by reasoning about the actual cost of lost/reordered data, not just "speed vs reliability" at a surface level.
