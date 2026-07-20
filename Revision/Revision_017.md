# Revision — Topic 017: Load Balancers

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-14

---

## Q1. What's the core difference between L4 and L7 load balancing?

<details>
<summary>Answer</summary>

L4 routes based on IP + port only (the socket 5-tuple) — fast, protocol-agnostic, no visibility into content. L7 understands HTTP — can route on URL path, headers, cookies, and can do TLS termination — at the cost of more per-request processing overhead.

</details>

---

## Q2. Why does IP Hash's session affinity have the same fragility as cookie-based sticky sessions, even though it needs no shared session store?

<details>
<summary>Answer</summary>

IP Hash is a routing-layer trick — it consistently sends a client to the same backend via hashing, but the session data still lives only in that one server's memory. If that server dies, the session is lost, same as cookie-based sticky sessions. The only real fix is moving session state to a shared store (Redis) so any server can serve any request.

</details>

---

## Q3. A backend process is alive but its DB connection pool is exhausted, so every real request times out. Would an active or passive health check catch this?

<details>
<summary>Answer</summary>

Passive. An active health check pings a lightweight `/health` endpoint that can succeed even while the process is functionally broken for real traffic. A passive health check observes actual request outcomes (timeouts/errors) and catches this class of failure that active checks miss.

</details>

---

## Q4. Why does sticky-session load balancing undercut the point of horizontal scaling?

<details>
<summary>Answer</summary>

Horizontal scaling means spreading load across many servers so no single one is overwhelmed. Sticky sessions pin a given client permanently to one specific server — that client's traffic is never actually load-balanced across the pool, and that one server becomes a de facto single point of failure for that client's session.

</details>

---

## Q5. The load balancer itself is now a single point of failure. Name two ways to make it redundant.

<details>
<summary>Answer</summary>

1. **DNS-based failover** — multiple LB IPs behind a domain, DNS returns a healthy one.
2. **Active-passive LB pair via virtual IP (VIP)** — a standby LB takes over if the active one dies (keepalived/VRRP).

(Cloud-managed LBs like AWS ALB/NLB handle this redundancy automatically.)

</details>

---

## 30-Second Elevator Pitch

> A load balancer is a specialized reverse proxy that distributes traffic across a backend pool, solving both horizontal scaling and redundancy. L4 load balancers route on IP/port only — fast and protocol-agnostic; L7 understands HTTP and can route by path/header/cookie and terminate TLS, at higher processing cost. Algorithms like round robin, weighted round robin, least connections, and IP hash fit different traffic shapes. Health checks come in two flavors: active (synthetic pings) and passive (observing real traffic) — passive catches failures active pings miss, like an exhausted DB connection pool. Sticky sessions and IP-hash affinity are stopgaps that pin a client to one server, undermining both statelessness and horizontal scaling; the resilient fix is a shared session store like Redis. And the load balancer itself needs redundancy — DNS failover, active-passive VIP pairs, or a managed cloud LB.

---

## Weak Areas to Watch

- IP Hash and Redis solve the SAME problem via DIFFERENT mechanisms — not one a "better version" of the other
- Active health checks can miss functional failures (exhausted resources) that passive checks catch
