# Revision 001 — Introduction to System Design
**Completed:** 2026-06-28 | **Next due:** 2026-06-29 (+1 day)

---

## 30-Second Elevator Explanation (say this from memory)

> "System design is the discipline of designing software systems that are scalable, reliable, and maintainable at real-world scale. It has two levels: HLD — the bird's-eye view of components and data flow — and LLD — the code-level view of classes and patterns. Every design decision is a tradeoff. NFRs like availability, latency, and scalability drive the architecture. In interviews, they're testing your thinking process and how you reason about tradeoffs — not whether you memorize the right answer."

---

## Active Recall Q&A

<details>
<summary>Q1: What is the difference between HLD and LLD?</summary>

**HLD (High-Level Design):** The bird's-eye view — which components exist, how they connect, what databases and caches are used, how data flows. Whiteboard-level.

**LLD (Low-Level Design):** The ground-level view — class structure, design patterns, interfaces, thread safety. Code or class-diagram level.

Both are tested in interviews — usually in separate rounds.
</details>

<details>
<summary>Q2: What is a Non-Functional Requirement? Give two examples for a note-taking app.</summary>

An NFR describes **how well** the system does something — not what it does.
It's stated as a constraint or quality, not a technology.

Examples:
- "The app must have 99.9% uptime — users should never open it and find it down." (Availability)
- "A note saved on mobile must appear on desktop within 5 seconds." (Consistency + Latency)

**Common trap:** Saying "use SQL" or "add a load balancer" — those are SOLUTIONS, not NFRs.
</details>

<details>
<summary>Q3: What is the difference between a Functional Requirement and an NFR?</summary>

**FR = WHAT the system does.** Can be satisfied by writing a function.
Example: "Users can save a note."

**NFR = HOW WELL it does it.** Requires architectural decisions to satisfy.
Example: "Notes must save in under 200ms, 99.99% of the time."
</details>

<details>
<summary>Q4: Name the four scaling layers and one lever for each.</summary>

| Layer | Lever |
|---|---|
| Compute | More servers (horizontal) or bigger server (vertical) |
| Data | Replication (reads) or Sharding (writes) |
| Network | Load balancer / CDN |
| Application | Cache / async queue |
</details>

<details>
<summary>Q5: What are the three pillars of observability?</summary>

1. **Metrics** — numbers over time (QPS, latency p99, error rate)
2. **Logs** — events with context (request ID, user, action, error)
3. **Traces** — the journey of one request across all components
</details>

<details>
<summary>Q6: An interviewer asks you to design Instagram and you immediately say "I'll use microservices + Kafka." What did you skip and why does it cost you?</summary>

You skipped:
1. **Requirements clarification** — you don't know the scale, features in scope, or NFRs
2. **Estimation** — no idea of QPS, storage, or user count
3. **Justification** — no reason given for microservices vs monolith, or Kafka vs simple queue

Cost: interviewer marks you low on "Requirements & Scoping" and suspects you're pattern-matching buzzwords. You risk designing the wrong thing for 40 minutes.

**Rule:** Never draw a single box until you've asked at least 3 clarifying questions.
</details>

---

## Personal Weak Areas (this learner)
- **Confused NFRs with solutions on first attempt** — jumped to "use SQL / use load balancer" instead of stating the quality constraint first. Fix: always start NFRs with "The system must…" or "Users must never…"
- **Confused FR with NFR on second attempt** — "users can sign in from anywhere" is a feature (FR), not an NFR. Fix: ask "does this describe WHAT it does, or HOW WELL it does it?"

---

## Key Diagram to Redraw

```
FR vs NFR:

  FR  → "Users can save a note"            (WHAT it does)
  NFR → "Notes must save in under 200ms"   (HOW WELL)
              ↓
        drives architecture choices
              ↓
        Solution: "Add a write cache"
```

---

## Interview Trap to Remember

> Jumping to technology (microservices, Kafka, Redis) before clarifying requirements = **automatic low score on scoping dimension**. The fix: ask 3 clarifying questions before drawing anything.
