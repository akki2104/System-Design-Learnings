# Topic 001 — Introduction to System Design
**Module:** 0 — Orientation & Mental Models
**Status:** Completed | **Date:** 2026-06-28
**Revision file:** [Revision_001.md](../Revision/Revision_001.md)

---

## 1. Why This Topic Exists

Before touching a single concept, you need to know what game you're playing. System design is not a trivia contest — it's a structured way of solving an open-ended engineering problem under time pressure. Knowing what the game is changes how you play it.

---

## 2. Real Production Problem

When Instagram was acquired by Facebook in 2012, it had 13 employees serving 30 million users on a simple Django + PostgreSQL stack. When a feature breaks at 10,000 users but nobody designed for 10,000,000 — servers melt, revenue is lost, and the post-mortem asks: *"Why didn't we design for this?"*

System design is the discipline of **thinking through these failure modes before they happen** — while still shipping something useful.

---

## 3. Simple Intuition

Think of it like **city planning vs. building a single house**.

A developer who only codes is building houses — one at a time, well. A system designer is thinking about roads, water pipes, power grids, emergency services — how does the whole city function when 10 million people live in it? What happens when one water pipe bursts?

You're moving from *building a house* to *designing a city*.

---

## 4. Core Concepts

**1. A system is just components talking to each other over a network.**
User's phone → server → database. The complexity comes from: what happens when there are millions of phones, the server crashes, and the database gets slow?

**2. HLD vs LLD — two levels of design.**
- **High-Level Design (HLD):** Bird's-eye view. Which components? How do they connect? What DB? How does data flow?
- **Low-Level Design (LLD):** Ground-level. How is the code structured? Which classes, patterns, interfaces?

Both are tested in interviews — usually in separate rounds.

**3. Every design decision is a tradeoff.**
There is no "best" database. There is no "best" architecture. There is only *best for these constraints*. Speed vs. cost. Simplicity vs. flexibility. Consistency vs. availability.

**4. NFRs often matter more than features.**
Non-Functional Requirements — how fast, how reliable, how scalable, how secure — usually determine the *architecture*. Features tell you *what to build*. NFRs tell you *how to build it*.

**5. Interviews test your thinking, not your memory.**
The interviewer doesn't want you to recite the "correct" design. They want to see how you clarify, estimate, decide, defend tradeoffs, and adapt.

---

## 5. Visualization — The Simplest Possible System

```
          Request
  User ──────────────► Server ──────────────► Database
  (Browser/           (Business              (Stores
   Mobile App)         Logic)                 State)

          Response
  User ◄────────────── Server ◄────────────── Database
```

Every complex system in this program is an elaboration of this.
The question that drives all of system design: **"What breaks first?"**

---

## 6. Animation — How Complexity Enters

**Frame 1 — Day 1: One user, one server**
```
  User ──► Server ──► DB
```
Works perfectly. Ship it.

**Frame 2 — 1,000 users: Server gets slow**
```
  Users (1000) ──► Server (struggling) ──► DB
                        ↑
                  CPU/RAM maxing out
```
Fix: vertical scaling (bigger server) or horizontal (more servers).

**Frame 3 — 10,000 users: DB becomes the bottleneck**
```
  Users ──► Server ──► DB (slow reads/writes)
                         ↑
                   Disk I/O saturated
```
Fix: caching, read replicas, better indexing.

**Frame 4 — 1,000,000 users: Full rethink**
```
  Users ──► Load Balancer ──► Server 1 ─┐
                          ──► Server 2 ─┼──► DB Cluster ──► Cache
                          ──► Server 3 ─┘
```
Now we need: load balancers, DB clusters, cache layers, CDN for static content.

This evolution — Frame 1 → Frame 4 — is the story the entire program tells.

---

## 7. Architecture Diagrams

### HLD vs LLD — Where Each Lives in an Interview

```
┌─────────────────────────────────────────────────┐
│                  HLD Interview                  │
│  "Design Twitter's timeline"                    │
│   → What components? DB choice? Cache? Queue?  │
│   → 45–60 minutes, whiteboard-level             │
│   → Breadth + 1–2 key deep-dives               │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│                  LLD Interview                  │
│  "Design classes for a Parking Lot system"      │
│   → What classes? Which patterns? Thread-safe? │
│   → 45–60 minutes, code or class-diagram level  │
│   → Depth + code correctness                    │
└─────────────────────────────────────────────────┘
```

---

## 8. Real Engineering Example

Instagram started simple (Django + PostgreSQL + a few servers). At 1 billion users it became hundreds of microservices, sharding, distributed cache (Memcached → Redis), CDN for photos, dedicated feed infrastructure.

**The original design was correct for its constraints. The evolved design was correct for its constraints.** Neither is "better" in the abstract. System design is always design for *current + near-future* constraints, not a theoretical maximum.

---

## 9. Industry Examples

| Company | SD Round Focus |
|---|---|
| Google | Technical depth, data structures behind the design, correctness at scale |
| Meta | Design products Meta builds (feeds, messaging, CDN); product + API design |
| Amazon | Operational rigor, ownership, failure handling, cost reasoning |
| Uber | Geo-specific problems (driver matching, ETA, maps), real-time scale |
| Stripe | Payment correctness, idempotency, exactly-once, failure recovery |

Same fundamentals — different emphasis. Master the core, flex the emphasis.

---

## 10. Tradeoffs

| Approach | Benefit | Cost |
|---|---|---|
| Over-engineer early | Ready for scale | Slow to build, complex to debug |
| Under-engineer early | Ship fast, stay simple | Technical debt, painful migrations |
| Design for current constraints | Pragmatic | Must revisit as scale changes |

**Interview answer:** Design for the given constraints, acknowledge what changes at 10×, be explicit about tradeoffs.

---

## 11. Complexity Analysis

- **Complexity ≈ number of components × number of interactions**
- Every new component adds failure modes
- Every component interaction is a potential network call that can fail, be slow, or deliver wrong data
- **Simplicity is a feature.** Add components only when necessary.

---

## 12. Scaling Considerations

| Layer | Scaling Lever |
|---|---|
| Compute | Horizontal (more servers) or vertical (bigger server) |
| Data | Replication (read scale) or Sharding (write scale) |
| Network | Load balancing, CDN (geographic scale) |
| Application | Caching, async queues (throughput scale) |

---

## 13. Failure Scenarios

Every component will fail. Disks crash. Networks partition. Servers run out of memory.

The goal of system design is not to prevent failure — it's to ensure failure in one component does not cascade into total system failure. This discipline is called **fault tolerance**.

---

## 14. Monitoring

For any system, operations asks first:
- Is it **up**? (health check / heartbeat)
- Is it **slow**? (latency metrics)
- Is it **correct**? (error rate)

Even in an interview: *"I'd add a health endpoint and alert on error rate > 1%"* signals operational maturity.

---

## 15. Observability

Three pillars:
- **Metrics** — numbers over time (QPS, latency p99, error rate)
- **Logs** — events with context (request ID, user, action, error)
- **Traces** — the journey of one request across all components

Each alone is insufficient. You need all three.

---

## 16. Security

The four base questions for any system:
1. Who can call my API? (**authentication**)
2. What can they do? (**authorization**)
3. Is data encrypted in transit? (**HTTPS/TLS** — always yes)
4. Is sensitive data encrypted at rest? (**yes, for PII/payments**)

Even one sentence in an interview shows maturity.

---

## 17. Interview Discussion

No interviewer asks "explain system design." But the concepts here are the **implicit contract** of every SD interview:
- You will clarify requirements before designing
- You will reason about NFRs, not just features
- You will justify every technology choice
- You will discuss tradeoffs, not recite solutions

---

## 18. Common Mistakes

1. **Jumping straight to a solution** without clarifying requirements
2. **Treating SD as trivia** — naming tech without explaining why
3. **Ignoring NFRs** — designing functionally but ignoring scale/availability/latency
4. **Over-engineering** — adding 12 microservices to a 10,000-user app
5. **Going silent** — SD is a conversation; narrate your thinking constantly

---

## 19. Advanced Topics (for later)

- **Evolutionary architecture** — designing systems that can change without a rewrite
- **Domain-Driven Design (DDD)** — using business domain to drive component boundaries
- **Cell-based architecture** — slicing systems into independent cells for blast-radius containment

---

## 20. Real Interview Questions

1. "Walk me through how you'd approach a system design problem you've never seen before."
2. "What's the difference between scalability and availability?"
3. "What's the difference between HLD and LLD? Which do you find harder?"
4. "A system works for 1,000 users and falls over at 100,000. How do you diagnose it?"
5. "Why is simplicity important in system design?"

---

## 21. Exercises

1. **Mental model:** Pick any app you use daily. Sketch just the boxes: client, server(s), database(s), any obvious cache. Don't worry if it's wrong — practice thinking in components.
2. **NFR identification:** For a ride-booking app, list 5 NFRs. Start each with "The system must…"
3. **Tradeoff thinking:** A startup asks: "SQL or NoSQL?" Write 3 tradeoffs for each side before deciding.

---

## 22. Revision Questions → [Revision_001.md](../Revision/Revision_001.md)

1. What is the difference between HLD and LLD?
2. Name the five core ideas of system design.
3. What is an NFR? Give two examples for a note-taking app.
4. What is the difference between an FR and an NFR?
5. Name the four scaling layers and one lever each.
6. What are the three pillars of observability?
7. An interviewer asks you to design Instagram and you immediately say "microservices + Kafka." What did you skip?

---

## 23. Cheat Sheet → [CheatSheets.md](../CheatSheets.md)

```
SYSTEM DESIGN — TOPIC 001
══════════════════════════════════════════════════════════
WHAT IT IS: Designing systems that work at scale, reliably.
Interviews test: thinking + tradeoffs + adaptability.

HLD = components, connections, data flow (bird's eye)
LLD = classes, patterns, interfaces (ground level)

FR  = WHAT the system does  ("users can save a note")
NFR = HOW WELL it does it   ("notes must save in < 200ms")

NFR → drives architecture → solution chosen to satisfy NFR

COMMON MISTAKES:
1. Jump to solution before clarifying requirements
2. Recite tech without justifying why
3. Ignore NFRs
4. Over-engineer
5. Go silent

RULE: Never draw a single box until you've asked 3 clarifying questions.
══════════════════════════════════════════════════════════
```

---

## 24. Summary

- System design = building systems that are scalable, reliable, maintainable at real-world scale.
- **HLD** = architecture; **LLD** = code design. Both tested, usually in separate rounds.
- **Every decision is a tradeoff.** Interviewers reward reasoning, not memorized answers.
- **NFRs drive architecture** more than features do. Always identify them first.
- The Frame 1 → Frame 4 evolution is the story this program tells.
- **You can now:** describe what system design is, name its two levels, distinguish FR from NFR, explain why tradeoffs matter, and state the five core ideas anchoring everything ahead.
