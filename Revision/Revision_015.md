# Revision — Topic 015: WebSockets, SSE, Polling, Long Polling

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "HTTP is client-initiated — servers can't push data unprompted. Short polling asks repeatedly on a timer, wasting requests; cost scales with clients × frequency, so you can't cheaply fix latency by polling faster. Long polling holds the request open until data exists, eliminating waste and getting near-instant latency, but still re-requests every cycle and holds many idle connections. SSE keeps ONE persistent connection where the server pushes continuously — but it's one-directional only. WebSockets upgrade HTTP to a full-duplex persistent connection — true two-way, high-frequency communication, at the cost of stateful connections that complicate load balancing."

---

## Active-Recall Q&A

<details>
<summary>Q1: Explain the fundamental latency-vs-waste tension in short polling.</summary>

You CAN poll more frequently to reduce latency, but cost scales with `clients × frequency`. Polling 30× more often multiplies request volume ~30×, almost all wasted "no" responses. At scale (thousands of clients) this becomes economically unbearable, and latency can never drop below network round-trip time anyway. You're always paying for freshness with wasted requests, at a ratio that worsens as client count grows.

</details>

<details>
<summary>Q2: What's the one key improvement long polling makes, and what limitation remains?</summary>

**Improvement:** eliminates wasted "no" responses and gets latency near-instant once data exists — the server responds the moment it has something, not on the next poll tick.

**Remaining limitations:** every response still forces the client to fire a brand-new HTTP request (re-paying connection/header overhead each cycle); the server holds many idle open connections simultaneously at scale.

</details>

<details>
<summary>Q3: Why is SSE described as one-directional, and what would you need to add for two-way communication?</summary>

SSE only lets the SERVER push data to the client over the persistent connection. The client cannot send data back over that same connection — it would need a separate, normal HTTP request for that. For true two-way real-time communication, you'd need WebSockets instead.

</details>

<details>
<summary>Q4: What HTTP status code confirms a WebSocket upgrade, and why does this matter for firewalls/proxies?</summary>

**101 Switching Protocols.**

It matters because intermediaries (proxies/firewalls) need to explicitly understand and support the protocol upgrade. Older/stricter middleboxes that only understand plain HTTP semantics may block, buffer incorrectly, or terminate a connection that silently becomes non-HTTP. This is why `wss://` typically runs over port 443 — it looks like normal HTTPS traffic to less sophisticated network equipment.

</details>

<details>
<summary>Q5: Pick the right technique for a live collaborative code editor with real-time cursor positions.</summary>

**WebSockets.** The client needs to send its own cursor position and typing activity back to the server in real time, not just receive updates — SSE would only solve half the problem since it's one-directional. Long polling would add too much latency and per-request overhead at this message frequency (cursor moves are high-frequency, bidirectional events).

</details>

---

## Key Diagram

```
THE REAL-TIME SPECTRUM
─────────────────────────────────────────
Short Polling → repeat on timer; wasted "no"s; cost = clients × frequency
Long Polling  → hold request open; near-instant once data exists; still
                re-requests each cycle + many idle connections at scale
SSE           → ONE persistent connection; server pushes; ONE-DIRECTIONAL
WebSockets    → HTTP upgrades (101) to full-duplex; BOTH directions
```

---

## My Weak Areas (from lesson 2026-07-13)

- **Q1:** Framed the tension as pure overhead/load rather than the precise "cost scales with clients × frequency, and latency floor is network RTT" mechanism
- **Q2:** Captured the mechanism (holds connection) and one limitation (idle connections) but missed the "re-request every cycle" cost as a separate unsolved limitation
- **Q4:** Correct status code, but attributed the firewall issue to "no indefinite connections allowed" rather than the precise reason: middleboxes not understanding the protocol Upgrade

---

## Past Mistakes

See [InterviewMistakes.md](../InterviewMistakes.md) — entries dated 2026-07-13.
