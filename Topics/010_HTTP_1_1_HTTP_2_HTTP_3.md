# Topic 010 — HTTP/1.1, HTTP/2, HTTP/3 (QUIC)

**Module:** 1 — Networking & Communication Foundations
**Status:** Completed
**Date:** 2026-07-09
**Confidence:** 4/5
**Difficulty:** Medium-Hard

---

## 1. Why This Topic Exists

HTTP is the application protocol riding on top of TCP/UDP (Topic 008). Understanding how each HTTP version evolved to fix the previous version's bottleneck is one of the most commonly probed areas in networking-focused interview rounds.

---

## 2. Core Concepts

### HTTP/1.1 — Application-Level Head-of-Line Blocking

Browsers cap connections per domain (~6). A page with 30 resources queues the remaining 24 behind the first 6:
```
HTTP/1.1: 6 connections max per domain
30 resources → 6 load immediately, 24 queue waiting for a free connection
```
**Old workaround:** domain sharding (splitting resources across subdomains to get more connections) — a hack, not a fix; adds DNS lookups and handshake overhead.

### HTTP/2 — Multiplexing (fixes app-level blocking)

**One TCP connection, many requests interleaved simultaneously** via binary framing:
```
HTTP/1.1: [Req1]→[Res1] [Req2]→[Res2]   (queued, one at a time per connection)
HTTP/2:   [Req1][Req2][Req3] → interleaved as small binary frames, in parallel
```
**Bonus fix — HPACK header compression:** avoids resending full headers (cookies, user-agent) on every request; compresses and caches repeated headers per connection.

### The Trap — TCP-Level Head-of-Line Blocking (HTTP/2's hidden flaw)

TCP guarantees strictly ordered delivery for the **entire connection**, not per-stream. HTTP/2 multiplexes independent streams (image, CSS, JS) onto that one TCP connection — but TCP doesn't know about "streams," it just sees one ordered byte sequence.

```
If a packet carrying Stream B's data is LOST:
   TCP holds EVERYTHING after it — including already-arrived Stream A
   and Stream C bytes — until the lost packet is retransmitted.

Result: unrelated streams get blocked behind one lost packet.
```

```
HTTP/1.1  → head-of-line blocking at the APPLICATION layer (connection queue)
HTTP/2    → fixed that, but still blocked at the TRANSPORT layer (TCP ordering)
```

This is baked into TCP itself — cannot be fixed by changing HTTP's framing again.

### HTTP/3 — QUIC (fixes transport-level blocking)

HTTP/3 replaces TCP entirely with **QUIC**, built on UDP:
```
QUIC gives each STREAM its own independent delivery guarantee —
NOT one shared ordered byte stream for the whole connection.

If Stream B's packet is lost: Stream A and Stream C are UNAFFECTED.
Only Stream B pauses until its own retransmission arrives.
```
QUIC reimplements TCP's reliability (ACKs, retransmission) **per-stream** on top of UDP, and bundles the handshake + TLS negotiation together — enabling 0-RTT connection resumption on repeat connections (skipping the round-trip cost from Topic 008's 3-way handshake).

```
HTTP/1.1 → TCP, one request per connection, app-level HOL blocking
HTTP/2   → TCP, multiplexed streams, transport-level HOL blocking remains
HTTP/3   → QUIC (UDP-based), independent per-stream delivery, no HOL blocking,
           faster connection setup (0-RTT)
```

---

## Tech Decision Box: Which HTTP Version to Design Around

```
HTTP/1.1 → legacy support, simple APIs with few resources, wide compatibility
HTTP/2   → default for modern web apps with many resources (images, JS, CSS)
           — multiplexing + header compression are big wins
HTTP/3   → high-latency/lossy networks (mobile, satellite), video streaming,
           anywhere connection setup speed or packet loss resilience matters most
```

**Interview sentence:** "I'll design assuming HTTP/2 as the baseline since it removes the 6-connection bottleneck. For mobile-heavy or high-latency use cases, I'd push for HTTP/3 specifically because it eliminates TCP head-of-line blocking."

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Thinking HTTP/2's multiplexing fully solved head-of-line blocking | It fixed app-level blocking but reintroduced transport-level blocking via shared TCP ordering |
| Assuming HTTP/3 is "just HTTP/2 but faster" | HTTP/3 replaced the entire transport layer (QUIC/UDP) specifically to solve TCP's connection-wide ordering guarantee |
| Believing domain sharding is a real fix | It's a workaround for HTTP/1.1's connection cap, obsoleted by HTTP/2's multiplexing |

---

## Real Interview Questions

1. "Why did HTTP/2 introduce multiplexing, and what problem does it solve?"
2. "HTTP/2 still has head-of-line blocking — where, and why?"
3. "Why couldn't HTTP/3 just be 'HTTP/2 improved' on top of TCP?"
4. "When would you NOT need HTTP/3 — where does HTTP/2 remain sufficient?"

---

## Revision Questions

1. What is application-level head-of-line blocking, and which HTTP version suffers from it?
2. How does HTTP/2 multiplexing work?
3. Why does HTTP/2 still suffer from head-of-line blocking, just at a different layer?
4. Why did HTTP/3 need a new transport protocol instead of improving HTTP/2 further?
5. What does QUIC do differently from TCP regarding stream independence?

---

## Cheat Sheet

```
HTTP/1.1 vs HTTP/2 vs HTTP/3
─────────────────────────────────────────────────
HTTP/1.1 → TCP, ~6 connections/domain limit, app-level head-of-line blocking
HTTP/2   → TCP, ONE connection, multiplexed binary streams, HPACK header
           compression — fixes app-level blocking, but TCP-level blocking remains
HTTP/3   → QUIC (UDP-based), independent per-stream delivery, no HOL blocking,
           faster setup (0-RTT), handshake+TLS bundled together

HEAD-OF-LINE BLOCKING — TWO DIFFERENT LAYERS
─────────────────────────────────────────────────
App-level (HTTP/1.1)   → limited connections force requests to queue
Transport-level (HTTP/2 on TCP) → one lost packet blocks ALL multiplexed
                                   streams sharing that TCP connection

WHY HTTP/3 NEEDED A NEW TRANSPORT (not just new HTTP framing)
─────────────────────────────────────────────────
TCP's ordering guarantee applies to the WHOLE connection — this can't be
fixed by changing HTTP's layer again. QUIC replaces TCP itself, giving
each stream independent delivery + reliability.

WHEN TO DESIGN AROUND WHICH
─────────────────────────────────────────────────
HTTP/1.1 → legacy support, simple low-resource-count APIs
HTTP/2   → default for modern web apps (many resources)
HTTP/3   → high-latency/lossy networks, mobile, video streaming
```

---

## Summary

- **HTTP/1.1's bottleneck:** app-level head-of-line blocking from the ~6-connections-per-domain cap.
- **HTTP/2's fix:** multiplexing over one TCP connection — but this exposes TCP's own connection-wide ordering guarantee as a new bottleneck.
- **HTTP/2's hidden flaw:** transport-level head-of-line blocking — one lost packet blocks all multiplexed streams sharing that TCP connection.
- **HTTP/3's fix:** replace TCP with QUIC (UDP-based), giving each stream independent delivery so a lost packet only blocks its own stream.
- **The layering lesson:** some bottlenecks can't be fixed by changing the layer above — you have to go down a layer (application → transport).

> **You now can:** explain why each HTTP version exists, trace head-of-line blocking across both the application and transport layers, and justify which HTTP version fits a given system's network conditions.
