# Topic 017: Load Balancers

**Module:** 1 — Networking & Communication Foundations
**Completed:** 2026-07-13
**Confidence:** 4/5

---

## 1. Why Load Balancers Exist

A single server caps throughput at one machine's capacity and is a single point of failure. A load balancer sits in front of a pool of backend servers and distributes incoming requests across them — solving horizontal scaling (Topic 001) and redundancy (Topic 004) at once.

A load balancer is a specialized reverse proxy (Topic 016) whose primary job is distributing load across multiple backends, rather than just fronting one.

---

## 2. L4 vs L7 Load Balancing

**L4 (Transport layer)** — routes based on IP + port only (the socket 5-tuple, Topic 007). Never inspects payload content.
- Pro: very fast, protocol-agnostic (any TCP/UDP traffic)
- Con: no content-aware routing decisions

**L7 (Application layer)** — understands HTTP; routes on URL path, headers, cookies:
```
/api/*     → API server pool
/static/*  → static asset pool
/video/*   → video streaming pool
```
- Pro: content-aware routing, can do TLS termination (Topic 011) at the LB
- Con: more per-request processing overhead, HTTP/HTTPS only

**Default interview answer:** L7 for web applications (content routing + TLS termination); L4 when raw throughput matters more than routing intelligence (DB connection poolers, very high-throughput non-HTTP TCP traffic).

---

## 3. Load Balancing Algorithms

| Algorithm | Mechanism | Best for |
|---|---|---|
| Round Robin | Sequential rotation across servers | Equal-capacity servers, stateless requests |
| Weighted Round Robin | Proportional to server capacity | Heterogeneous server pool |
| Least Connections | Route to server with fewest active connections | Long-lived connections (WebSockets, streaming) |
| IP Hash | Hash client IP → consistent backend | Session affinity without a shared session store |

---

## 4. Health Checks

- **Active health checks:** LB periodically pings a `/health` endpoint; unresponsive backends are pulled from rotation.
- **Passive health checks:** LB observes real traffic — a backend erroring/timing out on actual requests gets marked unhealthy without a separate ping. This is how a resource-exhaustion failure (e.g., DB connection pool exhausted, so requests time out even though the process is alive) gets caught — active pings to `/health` can pass while real traffic still fails, so passive observation is what actually detects this class of failure.

This is the mechanism that makes the "failover" and "graceful degradation" NFRs (Topic 004) work automatically instead of requiring a human to notice.

---

## 5. Sticky Sessions (Session Affinity) vs Shared Session Store

**Sticky sessions:** if a server holds session state in memory, the same client must always hit that server (via a cookie) or their session disappears. **IP Hash** achieves the same effect purely through routing (hash client IP → same backend) — no shared store, just pinned routing.

Both directly conflict with REST's statelessness (Topic 012) and with the whole point of horizontal scaling — the client isn't actually load-balanced across the pool, they're pinned to one server. If that server dies, their session state is lost.

**The real fix: move session state to a shared external store (Redis).** Then any server can handle any request — the LB is free to use plain round robin / least connections.

| | IP Hash | Redis shared store |
|---|---|---|
| Extra infra | None | Needs Redis deployment |
| Extra latency | None | +1 hop (server → Redis) |
| Resilience on backend death | Session lost (pinned to that server) | Seamless — any server reads from Redis |
| Load distribution | Can be uneven (NAT/corporate proxies hash to the same server) | Even |

IP Hash is a cheap stopgap that inherits sticky sessions' core fragility. Redis-backed shared sessions is the stronger answer whenever real availability matters.

---

## 6. What Happens When the Load Balancer Itself Fails?

- **DNS-based failover:** multiple LB IPs behind a domain; DNS returns a healthy one (Topic 009).
- **Active-passive LB pair:** standby LB takes over via a virtual IP (VIP) if the active one dies (keepalived, VRRP).
- **Cloud-managed LBs** (AWS ALB/NLB, GCP Load Balancer): redundancy handled by the provider — why almost nobody self-hosts a single LB instance in production.

---

## 7. Decision Box

| Use L4 when | Use L7 when |
|---|---|
| Raw TCP/UDP throughput matters most | Content-based routing needed |
| Protocol-agnostic traffic (DB connections, non-HTTP) | TLS termination at the LB |
| Simpler/faster processing priority | Web/HTTP applications (common case) |

**Common tools:** HAProxy, Nginx (software L7), AWS ALB (L7) / NLB (L4), GCP Load Balancer, Envoy (service mesh, Topic 100 preview).

---

## 8. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Treating IP Hash and Redis-backed sessions as the same solution | IP Hash = routing-layer trick (still fragile, pinned to one server). Redis = removes the need for affinity entirely (any server can serve any request) |
| Assuming active health checks catch all failures | Active checks (`/health` ping) can pass while the process is alive but functionally broken (e.g., exhausted DB pool) — passive checks (observing real traffic) catch this class of failure |

---

## 9. Revision Questions
See `Revision/Revision_017.md`.

## 10. Summary
- A load balancer is a specialized reverse proxy distributing load across a backend pool — solves both scaling and redundancy.
- L4 = fast, protocol-agnostic, no content awareness. L7 = content-aware routing + TLS termination, more overhead.
- Algorithms: Round Robin, Weighted Round Robin, Least Connections, IP Hash — each fits a different traffic shape.
- Health checks: active (synthetic pings) vs passive (real-traffic observation) — passive catches failures active pings miss.
- Sticky sessions/IP Hash are stopgaps; Redis-backed shared session state is the resilient, evenly-balanced fix, and preserves true statelessness.
- The LB itself needs redundancy — DNS failover, active-passive VIP pairs, or cloud-managed LBs.
