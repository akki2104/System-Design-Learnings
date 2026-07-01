# Revision — Topic 004: Non-Functional Requirements

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

> "Every system design starts with 5 NFRs: Availability (is it UP?), Reliability (are answers correct?), Scalability (handles growing load?), Maintainability (humans can operate it?), and Cost (fits budget?). Each NFR is stated as [quality] + [measurable threshold + percentile] + [business justification]. They conflict — you can't maximize all five simultaneously. The tradeoff framework: name the conflict, challenge whether the strict NFR is truly needed, propose the specific resolution."

---

## Active-Recall Q&A

<details>
<summary>Q1: Availability vs Reliability — one sentence each.</summary>

**Availability:** The percentage of time the system is UP and responding.

**Reliability:** When the system responds, it gives CORRECT results.

Key distinction: A system can be available (responding) but unreliable (wrong answers). Available + wrong = the most dangerous failure mode — users trust it while it silently fails.

</details>

<details>
<summary>Q2: Three components at 99.9% in series. What's overall availability?</summary>

```
A_total = 0.999 × 0.999 × 0.999 = 0.997 = 99.7%
```

3 nines in series → drops to 99.7%. That's ~26 hours downtime/year vs ~9 hours for a single component. Every dependency you add degrades overall availability.

</details>

<details>
<summary>Q3: Why does the database bottleneck before the server in a typical web app?</summary>

Web servers are **stateless** — you can add more behind a load balancer instantly. Horizontal scaling is easy.

Databases are **stateful** — they have:
- Fixed connection pool limits (100–500 connections)
- Disk I/O limits
- A single write path that can't be horizontally scaled without sharding

Result: servers scale in minutes; database scaling requires replication (reads), caching, and eventually sharding (writes).

</details>

<details>
<summary>Q4: Three sub-dimensions of Maintainability.</summary>

1. **Operability** — easy to run, monitor, and debug at 3AM
2. **Simplicity** — low accidental complexity; new engineers can understand it
3. **Evolvability** — easy to add features without rewriting everything

</details>

<details>
<summary>Q5: State a latency NFR for Instagram Stories (threshold + percentile + justification).</summary>

Example: "Stories must load within 2 seconds at p99 — users abandon video content after 3 seconds of buffering, and slow Stories directly reduces daily active engagement."

Key elements:
- Threshold: 2 seconds
- Percentile: p99 (99% of requests)
- Justification: business impact (abandonment → engagement drop)

</details>

<details>
<summary>Q6: PM wants 99.999% availability + strong consistency + minimum cost. What do you say?</summary>

"All three simultaneously isn't achievable. High availability requires redundant servers (cost++). Strong consistency requires synchronous replication across nodes (cost++ and latency++). Together they multiply cost.

My recommendation: challenge whether strong consistency is truly needed everywhere. For Instagram Stories, eventual consistency is fine — a story appearing 2 seconds late is acceptable. Reserve strong consistency for writes that can't be wrong (payments, inventory). That lets us hit 99.99% availability at reasonable cost, with strong consistency only where it truly matters."

Framework: Name conflict → Challenge stricter requirement → Propose specific tradeoff.

</details>

<details>
<summary>Q7: NFR priority by system type.</summary>

| System | Top NFR | Why |
|--------|---------|-----|
| Payments (Stripe/UPI) | Reliability | Wrong money = unrecoverable trust damage |
| Social feed (Twitter/Insta) | Availability | Stale feed > no feed |
| Healthcare | Consistency | Wrong data = patient harm |
| Real-time gaming | Latency | > 50ms lag = unplayable |
| Internal analytics | Cost | No SLA; just needs to be cheap |

</details>

<details>
<summary>Q8: What's the cost interview move?</summary>

After describing your architecture, add:

> "The most expensive component here is [X]. We can reduce it by [Y] — for example, moving data older than 30 days to cold storage cuts storage cost by ~80%."

This puts you in the top 20% of candidates at the 2-year level.

</details>

---

## Key Diagrams

```
AVAILABILITY MATH
────────────────────────────────────────────
Series (chain):   A = A1 × A2 × A3  → degrades
Parallel (copies): A = 1-(1-A1)(1-A2) → improves

NINES REFERENCE
────────────────────────────────────────────
99%     → 3.65 days/year
99.9%   → 8.76 hours/year
99.99%  → 52.6 min/year  ← standard
99.999% → 5.26 min/year  ← payments

NFR FORMAT
────────────────────────────────────────────
[Quality] + [Threshold + percentile] + [Business justification]
```

---

## Personal Weak Areas (from lesson 2026-06-30)

- **Missing p99/percentile qualifier on latency NFRs** — always add "at p99" or "at p95" to latency statements
- **Initial NFRs were vague** (fixed during lesson) — "low latency" without threshold is not an NFR
- **Warm-up NFR for scalability underestimated Google's scale** — calibrate DAU estimates to the product's real scale

---

## Past Mistakes

See [InterviewMistakes.md](../InterviewMistakes.md) — entries dated 2026-06-30.
