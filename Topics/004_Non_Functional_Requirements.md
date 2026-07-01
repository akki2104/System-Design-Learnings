# Topic 004 — Non-Functional Requirements: Scalability, Availability, Reliability, Maintainability, Cost

**Module:** 0 — Orientation & Mental Models
**Status:** Completed
**Date:** 2026-06-30
**Confidence:** 3/5
**Difficulty:** Medium

---

## 1. Why This Topic Exists

NFRs have appeared as vocabulary since Topic 001. This topic answers: what are the actual NFRs that matter in system design, why does each one force specific architectural decisions, and how do you reason about them under interviewer pressure?

---

## 2. Real Production Problem

In 2013, Amazon calculated that every 100ms of additional latency cost them 1% in sales. Their NFR for product page load was < 100ms. That single number drove decisions about CDNs, caching layers, read replicas, and pre-computed recommendations — all justified by one measurable NFR. One number. Many architectural decisions.

---

## 3. Simple Intuition

NFRs are the building code for software. A house that "has walls and a roof" (functional) isn't enough — it also needs to "withstand 100km/h winds" (reliability), "be accessible to all occupants" (availability), and "cost under ₹50L to build" (cost). The building code constrains the architect before the first brick is laid.

---

## 4. Core Concepts — The 5 NFRs

### NFR 1: Availability

**Definition:** The percentage of time the system is operational and responding.

Measured in **nines:**
```
99%     (2 nines) → 3.65 days downtime/year
99.9%   (3 nines) → 8.76 hours downtime/year
99.99%  (4 nines) → 52.6 min downtime/year   ← consumer product standard
99.999% (5 nines) → 5.26 min downtime/year   ← payments, telecom
```

**Series availability (chain of components):**
```
A_total = A1 × A2 × A3
3 components at 99.9% → 0.999³ = 99.7%
Every dependency added degrades overall availability.
```

**Parallel availability (redundant components):**
```
A_total = 1 - (1-A1)(1-A2)
Two 99.9% servers in parallel → 1 - (0.001 × 0.001) = 99.9999%
Redundancy dramatically improves availability.
```

**Interview move:** Convert "high availability" to nines + state the implication.
> "I'll target 99.99% — ~52 min downtime/year. That requires no single point of failure: redundant servers, multi-AZ deployment, automatic failover."

---

### NFR 2: Reliability

**Definition:** When the system responds, it gives correct results.

≠ Availability. A system can be available (responding) but unreliable (wrong answers).

| | Available | Not Available |
|---|---|---|
| **Reliable** | ✅ Goal | Offline |
| **Not Reliable** | ⚠️ Dangerous | Complete failure |

Available + unreliable is the worst case — users trust the system but get wrong answers.

**Real example:** Knight Capital (2012) — system was available, but a deployment bug made it execute wrong trades. Lost $440M in 45 minutes before anyone noticed.

**Sub-concepts:**
```
Fault Tolerance      → keeps working despite component failures
Redundancy           → duplicate components so one failure ≠ outage
Graceful Degradation → under failure, do less but don't crash entirely
```

**Graceful degradation rule:**
```
Critical path          → always on, never shed
Personalization/ML     → first to go
Real-time extras       → degrade to cached/stale
Non-essential reads    → return empty, queue for later
```

---

### NFR 3: Scalability

**Definition:** The system handles growing load without degrading performance.

```
Vertical scaling   → bigger machine (simple, has a ceiling, single point of failure)
Horizontal scaling → more machines (complex, no ceiling)
```

**The real bottleneck order in a typical web app:**
```
10×  traffic → server CPU/memory       → add servers + load balancer (easy)
50×  traffic → database connections    → read replicas + cache
100× traffic → database write path     → sharding (hard)
500× traffic → network / storage I/O   → CDN, object storage, async queues
```

The database bottlenecks before the server in almost every web application — servers are stateless and horizontally scalable; databases are stateful and hard to scale.

**Interview move:** When an interviewer says "traffic 10×s" — name what breaks first and why.

---

### NFR 4: Maintainability

**Definition:** The system can be understood, changed, and operated by humans over time.

Three sub-dimensions:
```
Operability    → easy to run, monitor, debug at 3AM
Simplicity     → low accidental complexity; new engineer can understand it
Evolvability   → easy to add features without rewriting everything
```

**Interview signal:** Candidates who add monitoring, logging, and runbooks to their design score higher on Operational Maturity — even at the 2-year level.

---

### NFR 5: Cost

**Definition:** The system fits a budget; it uses resources efficiently.

```
Compute cost  → number and size of servers
Storage cost  → S3 ~$0.02/GB/month; hot vs cold tiering
Network cost  → egress is expensive; CDN reduces it
Operational   → how many engineers to keep it running
```

**Interview move:** After describing the architecture, add:
> "The most expensive component here is [X]. We can reduce it by moving data older than 30 days to cold storage — cuts storage cost ~80%."

---

## 5. Visualization

```
SYSTEM UNDER THE 5 NFRs
─────────────────────────────────────────────────────────
           ┌──────────────────────────────────┐
           │         Your System              │
           │                                  │
Availability → Is it UP?                      │
Reliability  → Are answers CORRECT?          │
Scalability  → Does it handle MORE load?     │
Maintain.    → Can humans operate it?        │
Cost         → Does it fit the budget?       │
           └──────────────────────────────────┘

All 5 pull in different directions. You pick which to prioritize.
```

---

## 6. Animation Frames — Availability Math

```
Frame 1: Single component
  [Service] 99.9% available
  Downtime: 8.76 hours/year

Frame 2: Three in series
  [A] → [B] → [C]
  0.999 × 0.999 × 0.999 = 0.997 = 99.7%
  Downtime: ~26 hours/year ← WORSE

Frame 3: Two in parallel
  [A] ↘
       → [User]
  [B] ↗
  1 - (0.001 × 0.001) = 99.9999%
  Downtime: ~31 seconds/year ← MUCH BETTER
```

---

## 7. Architecture Diagram

```
NFR → Design Decision mapping:

Availability 99.99%  → Multi-AZ, redundant servers, health checks, failover
Reliability          → Idempotency, retries with backoff, exactly-once semantics
Scalability 10M DAU  → Horizontal server scaling, read replicas, caching, sharding
Maintainability      → Structured logging, distributed tracing, dashboards, runbooks
Cost optimized       → Cold storage tiering, spot instances, CDN for bandwidth
```

---

## 8. Real Engineering Examples

- **Google Search:** < 200ms at p99; 99.99%+ availability; serves stale cached results rather than errors
- **Stripe:** Reliability-first; idempotency keys on every payment; exactly-once semantics
- **Netflix:** Graceful degradation — disables recommendations during partial outages, keeps streaming alive
- **Amazon:** Every 100ms latency = 1% revenue; strict p99 latency NFRs drive all architecture decisions

---

## 9. Industry Examples

- **Payment systems (UPI, Stripe):** Reliability > Availability — wrong transaction is worse than no transaction
- **Social feeds (Twitter, Instagram):** Availability > Consistency — stale feed better than no feed
- **Healthcare:** Consistency > all — wrong patient data causes harm
- **Real-time gaming:** Latency < 50ms — anything more is unplayable
- **Internal analytics:** Cost > all — no SLA; just needs to be cheap and correct eventually

---

## 10. Tradeoffs

| Conflict | What gives |
|---------|-----------|
| Availability vs Consistency | Serving always-available results often means accepting stale data |
| Scalability vs Cost | More servers = higher availability = higher bill |
| Reliability vs Maintainability | Strong consistency protocols are complex and hard to operate |
| Latency vs Cost | Caching everything is fast but expensive |

**The tradeoff framework:**
1. Name the conflict
2. Challenge whether the stricter NFR is truly needed at that level
3. Propose the specific tradeoff with business justification

---

## 11. Complexity Analysis

NFRs don't have time complexity — they impose constraints that shape architectural complexity. The more NFRs you lock in at high thresholds, the more engineering complexity you accept.

---

## 12. Scaling Considerations

```
Availability:  each additional nine costs ~10× more engineering effort
Reliability:   distributed transactions and idempotency add significant complexity
Scalability:   each 10× scale step reveals a new bottleneck layer
Cost:          often improves with scale (amortized infra), but storage grows linearly
```

---

## 13. Failure Scenarios

- **Over-specifying NFRs:** Demanding 99.999% + strong consistency + low cost = impossible triangle. Team wastes months on infeasible targets.
- **Vague NFRs:** "Fast" and "reliable" with no numbers = no accountability, no architecture anchor
- **Wrong NFR priority:** Optimizing for latency when the real risk is data loss = wrong design entirely

---

## 14. Monitoring

Each NFR has a corresponding metric to watch:
```
Availability    → uptime %, error rate
Reliability     → error rate, data correctness checks, duplicate transaction rate
Scalability     → QPS, CPU/memory utilization, DB connection pool usage
Maintainability → MTTR (mean time to recovery), deploy frequency
Cost            → $/request, $/GB stored, monthly cloud bill
```

---

## 15. Observability

- Availability: uptime dashboard, SLO burn rate alerts
- Reliability: anomaly detection on output correctness, idempotency violation alerts
- Scalability: capacity planning dashboards, autoscaling triggers
- Cost: per-service cost attribution, budget alerts

---

## 16. Security

NFRs and security interact:
- High availability designs (redundant nodes) expand attack surface
- Strong consistency requirements may conflict with encryption-at-rest performance
- Cost optimization (shared infra) can conflict with data isolation requirements

---

## 17. Interview Discussion

NFRs appear in Step 1 (Clarify & Scope) of the 7-step framework. Interviewers evaluate:
- Do you distinguish availability from reliability?
- Do you state measurable thresholds with percentiles?
- Do you give business justifications?
- Do you name the NFR tradeoffs your design makes?
- Do you know which NFR is most critical for this system type?

**The format that gets full marks:**
`[Quality] + [Measurable threshold + percentile] + [Business justification]`

---

## 18. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Confusing availability and reliability | Availability = UP. Reliability = CORRECT. |
| Vague NFRs ("fast", "reliable") | Always add threshold + percentile + justification |
| Missing percentile on latency | Always say "< Xms at p99" not just "< Xms" |
| Saying all 3 conflicting NFRs are achievable | Name the impossible triangle; propose the tradeoff |
| Ignoring cost | Name the most expensive component + one reduction strategy |
| Ignoring maintainability | Add monitoring and logging to every design |

---

## 19. Advanced Topics

- **Error budgets:** SRE concept — if your SLO is 99.9%, you have 8.76 hours/year to spend on planned downtime + incidents. Once the budget is exhausted, all changes freeze.
- **SLI / SLO / SLA distinction:** SLI = measurement, SLO = internal target, SLA = contractual commitment with penalties
- **The CAP theorem:** formally proves you can't have consistency + availability + partition tolerance simultaneously (Topic 046)
- **PACELC:** extends CAP to include latency vs consistency tradeoff even when there's no partition (Topic 047)

---

## 20. Real Interview Questions

1. "What NFRs would you define for a payment gateway?" (Stripe, Razorpay)
2. "How would you achieve 99.999% availability?" (Amazon, Google)
3. "Our system needs strong consistency and 99.99% availability — what's your approach?" (Stripe, distributed systems roles)
4. "What breaks first when your system scales 10×?" (Uber, DoorDash)
5. "How would you gracefully degrade this design during a partial outage?" (Netflix, Amazon)
6. "How would you reduce the cost of this design by 50% without sacrificing core functionality?" (Amazon — Leadership Principles)
7. "What's the difference between an SLI, SLO, and SLA?" (Google SRE-adjacent roles)

---

## 21. Exercises

1. State three NFRs for WhatsApp — with threshold, percentile, justification.
2. Calculate: 4 services in a chain, each 99.95% available. What's overall availability?
3. For a stock trading system, rank the 5 NFRs in order of priority. Justify each.
4. A PM demands < 50ms latency + 99.999% availability + lowest possible cost. What's your response?
5. Design the monitoring setup for a ride-sharing app — what metrics map to each NFR?

---

## 22. Revision Questions

1. Availability vs Reliability — one sentence each.
2. Three components at 99.9% in series — overall availability?
3. Why does the database bottleneck before the server?
4. Three sub-dimensions of Maintainability.
5. State a latency NFR for Instagram Stories (threshold + percentile + justification).
6. PM wants 99.999% + strong consistency + minimum cost. What do you say?
7. Name the NFR priority for: payment system, social feed, healthcare records.
8. What's the interview move after computing NFRs — what sentence do you add about cost?

---

## 23. Cheat Sheet

```
THE 5 NFRs
─────────────────────────────────────────────────────
Availability    → system UP; measured in nines
                  Series: A = A1 × A2 × A3 (degrades with each dependency)
                  Parallel: A = 1-(1-A1)(1-A2) (improves with redundancy)

Reliability     → correct results when UP (≠ Availability)
                  Available + wrong = most dangerous failure mode
                  Sub: fault tolerance, redundancy, graceful degradation

Scalability     → handles growing load; DB bottlenecks before server
                  Bottleneck order: server → DB connections → DB writes → I/O

Maintainability → operable, simple, evolvable
                  Signal: add monitoring + logging to every design

Cost            → compute + storage + network + ops
                  Move: name most expensive component + how to reduce it

NFR FORMAT (use every time)
─────────────────────────────────────────────────────
[Quality] + [Threshold + p99/p95] + [Business justification]
"< 2s at p99 — users abandon search after 3s; direct revenue impact"
"99.99% — meal-hour outages = lost orders = direct revenue impact"

NINES TABLE
─────────────────────────────────────────────────────
99%     → 3.65 days/year downtime
99.9%   → 8.76 hours/year
99.99%  → 52.6 min/year   ← consumer standard
99.999% → 5.26 min/year   ← payments/telecom

NFR PRIORITY BY SYSTEM
─────────────────────────────────────────────────────
Payments     → Reliability first
Social feed  → Availability first
Healthcare   → Consistency first
Real-time    → Latency first
Internal     → Cost first

TRADEOFF FRAMEWORK
─────────────────────────────────────────────────────
1. Name the conflict
2. Challenge whether stricter NFR is truly needed
3. Propose specific tradeoff + business justification
```

---

## 24. Summary

- **5 NFRs drive every architecture decision:** Availability, Reliability, Scalability, Maintainability, Cost.
- **Availability ≠ Reliability.** Available + wrong = dangerous. Reliable + down = at least honest.
- **Availability compounds:** series degrades it; parallel improves it. Every dependency is a risk.
- **Database always bottlenecks first** in web apps — that's why replication, sharding, and caching exist.
- **NFRs conflict.** Name the tradeoff, challenge the requirement, propose a justified resolution.
- **State every NFR with threshold + percentile + justification.** Vague NFRs = no architectural anchor.
- **Cost and Maintainability** are increasingly tested — add both to every design.

> **You now can:** state well-formed NFRs for any system, reason about their conflicts, and propose justified tradeoffs under interviewer pressure.
