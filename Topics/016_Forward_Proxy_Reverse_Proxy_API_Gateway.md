# Topic 016 — Forward Proxy, Reverse Proxy, API Gateway

**Module:** 1 — Networking & Communication Foundations
**Status:** Completed
**Date:** 2026-07-13
**Confidence:** 4/5
**Difficulty:** Easy (Survey)

---

## 1. Why This Topic Exists

Sets up Load Balancers (Topic 017) directly — a load balancer IS a type of reverse proxy. Getting forward vs reverse proxy terminology precise here avoids confusion in every architecture diagram from this point forward.

---

## 2. Core Concepts

### The single distinguishing question

> **"Whose side is this thing standing on — the client's, or the server's?"**

### Forward Proxy — stands in front of the CLIENT
```
[Client] → [Forward Proxy] → [Internet/Server]
```
Makes requests on the client's behalf. The server sees the proxy's identity, not the real client's.

**Real uses:** corporate content filtering, VPNs (hide real IP from destination), client-side/ISP caching.

**Key property:** the **server doesn't know who the real client is.**

### Reverse Proxy — stands in front of the SERVER(S)
```
[Client] → [Reverse Proxy] → [Server 1, Server 2, Server 3...]
```
The client doesn't know (or care) which actual backend server handled their request.

**Real uses:** SSL/TLS termination (Topic 011), load balancing (Topic 017 — a load balancer IS a reverse proxy), hiding backend architecture, caching/compression at the edge.

**Key property:** the **client doesn't know which real server is behind it** — the opposite hiding direction from a forward proxy.

```
FORWARD PROXY  → hides the CLIENT's identity from the server
REVERSE PROXY  → hides the SERVER's identity from the client
```

### API Gateway — a reverse proxy specialized for APIs

Architecturally a reverse proxy, but purpose-built for microservices API traffic, adding cross-cutting concerns individual services shouldn't each reimplement:
```
[Client] → [API Gateway] → [Auth Service] [Orders Service] [Payments Service] ...
              ├── Authentication/Authorization (centralized, not per-service)
              ├── Rate limiting (protects all backend services centrally)
              ├── Request routing (path-based: /orders/* → Orders Service)
              ├── Request/response transformation
              └── Logging/analytics (single point of observability)
```

**Why it matters architecturally:** without a gateway, every microservice duplicates its own auth checks, rate limiting, and logging. The gateway centralizes these so services stay focused on business logic.

**A plain reverse proxy handles routing/SSL/load distribution but does NOT provide centralized auth, rate limiting, or per-route policy enforcement** — that's specifically what an API Gateway adds on top.

---

## Tech Decision Box

```
Forward Proxy → need to control/anonymize OUTGOING client traffic
                (corporate content filtering, VPNs, client-side caching)

Reverse Proxy → need to manage INCOMING traffic to your servers
                (SSL termination, load balancing, hiding backend topology)

API Gateway   → specifically for microservices APIs — need centralized
                auth/rate-limiting/routing across many backend services
```

**Interview sentence:** "I'll put an API Gateway in front of my microservices because I need centralized authentication and rate limiting rather than duplicating that logic in every service. A plain reverse proxy would only handle routing/SSL — it wouldn't give me the request transformation and per-route policies a gateway provides."

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Confusing which proxy hides which party | Forward hides the CLIENT from the server; Reverse hides the SERVER from the client |
| Thinking a reverse proxy and API Gateway are interchangeable | A reverse proxy handles routing/SSL/load distribution; an API Gateway adds centralized auth, rate limiting, and per-route policies on top |

---

## Real Interview Questions

1. "What's the difference between a forward proxy and a reverse proxy?" (universal)
2. "Is a load balancer a forward proxy or reverse proxy?" (universal — trap for the unprepared)
3. "Why would you add an API Gateway if you already have a reverse proxy doing load balancing?" (microservices-heavy companies)

---

## Revision Questions

1. What single question distinguishes forward proxy from reverse proxy?
2. Give a real-world use case for a forward proxy.
3. Why is a load balancer classified as a reverse proxy?
4. What does an API Gateway add that a plain reverse proxy does not?

---

## Cheat Sheet

```
THE DISTINGUISHING QUESTION
─────────────────────────────────────────────────
"Whose side is this thing standing on — client's or server's?"

FORWARD PROXY (client-side)          REVERSE PROXY (server-side)
──────────────────────────           ──────────────────────────
Hides CLIENT from server             Hides SERVER from client
Corporate filtering, VPNs,           SSL termination, load balancing,
client-side caching                  hiding backend topology, caching

API GATEWAY = reverse proxy + cross-cutting concerns for microservices
─────────────────────────────────────────────────
Centralized auth, rate limiting, routing, transformation, logging
— things a plain reverse proxy does NOT provide
```

---

## Summary

- **The one question that resolves proxy confusion:** whose side is it on?
- **Forward proxy** hides the client from the server — corporate filtering, VPNs.
- **Reverse proxy** hides the server from the client — SSL termination, load balancing, backend hiding.
- **A load balancer is a type of reverse proxy** (sets up Topic 017 directly).
- **An API Gateway is a specialized reverse proxy** that adds centralized cross-cutting concerns (auth, rate limiting, routing policies) that a plain reverse proxy doesn't provide.

> **You now can:** correctly classify any proxy-like component in a system diagram as forward, reverse, or gateway, and justify why an API Gateway is needed beyond a plain reverse proxy.
