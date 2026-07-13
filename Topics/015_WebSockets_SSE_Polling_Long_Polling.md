# Topic 015 — WebSockets, SSE, Polling, Long Polling

**Module:** 1 — Networking & Communication Foundations
**Status:** Completed
**Date:** 2026-07-13
**Confidence:** 4/5
**Difficulty:** Medium

---

## 1. Why This Topic Exists

HTTP's client-server model (Topic 006) is fundamentally client-initiated — servers can only respond to requests, never push data unprompted. Every "real-time" technique is a workaround for this constraint, each trading off simplicity, latency, and resource cost differently.

---

## 2. Core Concepts — The Real-Time Delivery Spectrum

### 1. Short Polling

Client repeatedly asks on a timer:
```
Client: "Any new messages?" → Server: "No"  (repeat every N seconds)
```

**The core tension:** you CAN poll more frequently to reduce latency, but cost scales with `clients × frequency` — polling 30× more often multiplies request volume ~30×, almost all wasted "no" responses. At scale (thousands of clients) this becomes economically unbearable, and latency can never drop below network round-trip time anyway. **You're always paying for freshness with wasted requests, at a ratio that worsens as client count grows.**

### 2. Long Polling

Server holds the request open until it has data (or hits a timeout):
```
Client: "Any new messages?" → Server: [...waits...] → "Yes! Here it is."
Client immediately re-asks → Server: [...waits again...]
```

**The improvement:** eliminates wasted "no" responses; latency becomes near-instant once data exists (server responds the moment it has something).

**Still unsolved:**
- Every response still forces the client to immediately fire a brand-new HTTP request, paying connection/header overhead again each cycle.
- Server holds many idle connections open simultaneously at scale — a real resource cost.

### 3. SSE (Server-Sent Events)

One persistent HTTP connection; server pushes events down it as they happen:
```
Client opens ONE connection → stays open indefinitely
Server pushes event 1, 2, 3... all over the SAME connection
```

**Key properties:**
- **One-directional only** — server → client. Client cannot send data back over this connection; needs a separate normal HTTP request for that.
- Built on HTTP (`text/event-stream`) — works through most proxies/firewalls without special handling.
- **Automatic reconnection is part of the spec** — browser's `EventSource` API reconnects automatically, can resume from last event ID.
- Text-based only — not efficient for binary data.

### 4. WebSockets

Starts as HTTP, then **upgrades** to a different protocol over the same TCP connection:
```
Client: HTTP request with "Upgrade: websocket" header
Server: "101 Switching Protocols" — connection is now a WebSocket

From here: EITHER side can send data to the other, at ANY time. Full duplex.
```

**Key properties:**
- **Bidirectional** — both client and server push data anytime.
- Persistent, low-overhead once upgraded — no repeated HTTP headers per message.
- Can carry binary OR text data.
- **The tradeoff:** server holds a persistent, stateful connection per client. Load balancers need sticky routing (same client → same server instance); can't treat requests as independent/stateless like normal HTTP.

**Why the 101 status matters for firewalls/proxies:** intermediaries need to explicitly understand and support the protocol upgrade. Older/stricter middleboxes that only understand plain HTTP semantics may block, buffer incorrectly, or terminate a connection that silently becomes non-HTTP. This is why `wss://` typically runs over port 443 (Topic 007) — it looks like normal HTTPS traffic to less sophisticated network equipment, avoiding this problem.

---

## Tech Decision Box: The Full Spectrum

```
Short Polling   → simplest, works everywhere, ok when latency doesn't matter much
                   (e.g., checking a background job's status every 10s)

Long Polling    → moderate real-time need, mostly one-directional, want broad
                   compatibility with old infra/proxies

SSE             → server needs to PUSH continuously, client rarely/never needs
                   to send data back, want auto-reconnect for free
                   (e.g., live score updates, stock tickers, notification feeds)

WebSockets      → TRUE two-way, low-latency, high-frequency communication both
                   directions (e.g., chat apps, multiplayer games, collaborative
                   editing like Google Docs, live trading dashboards)
```

**Interview sentence:** "I'll use WebSockets here because the client needs to send data back to the server in real time, not just receive it — SSE would only solve half the problem since it's one-directional. I considered long polling but rejected it because of the added latency and per-request overhead at this message frequency."

---

## Real Engineering Examples

- **Slack/Discord:** WebSockets for live message delivery and presence — genuinely bidirectional, high-frequency
- **Stock tickers / live scores:** SSE — server pushes constantly, client never talks back on that channel
- **Older/simpler notification systems:** long polling as a reliable, broadly-compatible fallback

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Thinking "poll more frequently" fully solves latency | Cost scales with clients × frequency — becomes unbearable at scale; latency floor is still network RTT |
| Describing long polling's improvement only as "holds the connection" | The improvement is eliminating wasted "no" responses + near-instant latency; the mechanism is holding the connection |
| Assuming WebSocket firewall issues are about "no indefinite connections allowed" | The real issue: intermediaries must explicitly understand the protocol Upgrade; unsupporting proxies may block/mishandle it |
| Picking SSE for bidirectional needs | SSE is one-directional (server→client only) — anything needing client→server real-time data needs WebSockets |

---

## Real Interview Questions

1. "Why can't you just poll very frequently to get low latency?" (universal)
2. "What's the difference between long polling and SSE?" (universal)
3. "Why does a WebSocket connection complicate load balancer design?" (Slack, Discord, real-time systems)
4. "Design a live collaborative editor — which real-time technique, and why?" (Google, collaborative tools)

---

## Revision Questions

1. Explain the fundamental latency-vs-waste tension in short polling.
2. What's the one key improvement long polling makes, and what limitation remains?
3. Why is SSE described as one-directional, and what would you need to add for two-way communication?
4. What HTTP status code confirms a WebSocket upgrade, and why does this matter for firewalls/proxies?
5. Pick the right technique for: a live collaborative code editor with real-time cursor positions.

---

## Cheat Sheet

```
THE REAL-TIME SPECTRUM
─────────────────────────────────────────────────
Short Polling → repeat request on timer; wasted "no" responses; cost = clients × frequency
Long Polling  → server holds request open until data/timeout; near-instant once data exists;
                still re-requests every cycle + many idle open connections at scale
SSE           → ONE persistent connection, server pushes continuously; ONE-DIRECTIONAL
                (server→client only); auto-reconnect built into spec; text only
WebSockets    → HTTP upgrades (101 Switching Protocols) to full-duplex persistent connection;
                BOTH directions, binary or text; needs sticky LB routing at scale

DECISION MAP
─────────────────────────────────────────────────
Latency doesn't matter much        → Short Polling
Moderate need, broad compatibility → Long Polling
Server pushes, client never replies → SSE
True two-way, high-frequency       → WebSockets

THE 101 STATUS
─────────────────────────────────────────────────
Confirms protocol upgrade from HTTP → WebSocket.
Middleboxes that don't understand Upgrade may block/mishandle it —
why wss:// typically runs on port 443 (looks like normal HTTPS)
```

---

## Summary

- HTTP is client-initiated; every real-time technique works around this constraint differently.
- **Short polling:** simplest, but cost scales with clients × frequency — can't cheaply fix latency.
- **Long polling:** removes wasted "no" responses, near-instant latency, but still re-requests each cycle and holds many idle connections.
- **SSE:** one persistent, one-directional (server→client) stream with built-in auto-reconnect.
- **WebSockets:** true bidirectional, persistent, low-overhead — but stateful, complicating load balancing at scale.
- **Pick based on directionality and frequency needs**, not just "do I need real-time."

> **You now can:** choose the correct real-time delivery technique for any system by reasoning about directionality, frequency, and connection-cost tradeoffs — not just "is this real-time or not."
