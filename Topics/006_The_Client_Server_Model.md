# Topic 006 — The Client–Server Model

**Module:** 1 — Networking & Communication Foundations
**Status:** Completed
**Date:** 2026-07-09
**Confidence:** 4/5
**Difficulty:** Easy (Survey)

---

## 1. Why This Topic Exists

Every system design conversation uses "client" and "server" — but the model is about ROLE, not a specific technology (browser/backend). Getting this precise matters once chains of client-server relationships appear (App Server is client to DB, server to Browser).

---

## 2. Core Concepts

```
SERVER = anything that provides a resource/service and LISTENS for requests
CLIENT = anything that INITIATES a request to consume that resource
```

The same process can be both, depending on who it's talking to:
```
[Browser] ──request──> [App Server] ──query──> [Database]
  CLIENT                CLIENT  ↑        ↑ SERVER
                         SERVER ┘ (to browser)
```

**The model is asymmetric:** clients initiate, servers respond. Servers must always be listening (bound to a port); clients connect only when they need something.

**Failure propagation:** if the Database goes down, the App Server (acting as client) gets no response — a timeout/connection error. That failure then propagates up: the App Server can't serve the Browser, so the Browser sees a timeout/500 error. This is exactly why replication and failover (Topic 039) exist — a standby replica the App Server falls back to.

---

## Tech Decision Box: Client-Server vs Peer-to-Peer

```
Client-Server (the default)  → centralized control, easy to secure/update,
                                but server is a bottleneck/single point of failure
Peer-to-Peer (BitTorrent,
some blockchain systems)     → no single bottleneck, scales with more peers,
                                but hard to secure/coordinate, no central authority
```

**Interview move:** almost every interview design is client-server. Mention P2P exists but default to client-server unless the question explicitly calls for decentralization.

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Thinking "client = frontend, server = backend" literally | Client/server are ROLES — any process can be both depending on context |
| Assuming DB failure means "App Server has nothing to do" | App Server still tries the query; it just gets no response — failure propagates upward |

---

## Cheat Sheet

```
CLIENT = initiates requests | SERVER = listens & responds
Same machine can be BOTH — client to one system, server to another
Model is ASYMMETRIC: servers always listening; clients connect on demand
Failure propagates UP the chain: DB down → App Server times out → Browser errors

Client-Server (default) → centralized, easy to secure, but server = bottleneck
Peer-to-Peer (rare in interviews) → no bottleneck, hard to secure/coordinate
```

---

## Revision Questions

1. What makes something a "server" vs a "client" — is it the software type or something else?
2. Can one process be both a client and a server? Give an example.
3. In a Browser → App Server → Database chain, what happens to the Browser if the Database goes down?
4. When would you use Peer-to-Peer instead of Client-Server in an interview?

---

## Summary

- Client/server are **roles**, not fixed properties of a technology
- The model is **asymmetric** — servers listen, clients connect on demand
- Failures **propagate up** the client-server chain
- Client-Server is the default architecture for interviews; P2P is rare and situational

> **You now can:** correctly describe any system as a chain of client-server relationships and reason about failure propagation through that chain.
