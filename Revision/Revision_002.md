# Revision 002 — The System Design Interview Framework
**Completed:** 2026-06-28 | **Next due:** 2026-06-29 (+1 day)

---

## 30-Second Elevator Explanation
> "The system design interview framework is a 7-step skeleton: Clarify & Scope, Estimate, Define APIs, Data Model, High-Level Design, Deep Dive, Bottlenecks & Wrap-Up. You always do them in this order. Step 1 is worth 10–15% of the score — never skip it. Ask FRs first, then NFRs, minimum 3 questions. Never ask the interviewer to make architectural decisions — those are yours. When requirements change mid-session, your first sentence is: 'Okay, that changes things — let me think about where that impacts the design.'"

---

## Active Recall Q&A

<details>
<summary>Q1: Name the 7 steps in order.</summary>

1. Clarify & Scope
2. Estimate
3. Define APIs
4. Data Model
5. High-Level Design
6. Deep Dive
7. Bottlenecks & Wrap-Up
</details>

<details>
<summary>Q2: What's the time budget for a 45-minute interview?</summary>

5 / 5 / 5 / 5 / 15 / 10 / 5 minutes (Steps 1–7).
Order is fixed. Time is flexible based on problem.
</details>

<details>
<summary>Q3: What's the difference between what you ask in Step 1 FR vs NFR?</summary>

**FR questions:** What the system does — features, scope, channels, triggers.
Example: "Is this push only, or also SMS and email?"

**NFR questions:** How well it must do it — scale, latency, availability, reliability.
Example: "How many DAU? What's the acceptable notification latency?"

**FR first, then NFR. Always.**
</details>

<details>
<summary>Q4: An interviewer changes requirements mid-session. What's your first sentence?</summary>

"Okay, that changes things — let me think about where that impacts the design."

Then: identify what specifically changed → re-enter the framework at the right step.
Never panic. Never go silent. Never jump straight to a technical answer.
</details>

<details>
<summary>Q5: Name two questions you should NEVER ask the interviewer.</summary>

1. "Should we use microservices or a monolith?"
2. "What should we choose from CAP?"

These are YOUR architectural decisions to make and defend.
Asking the interviewer signals you don't know how to reason independently.
</details>

<details>
<summary>Q6: For a notification system, what is the single most design-changing FR question?</summary>

"What channels are we supporting — push notification, SMS, email, in-app, or some combination?"

Why: each channel requires a different third-party integration (APNs, FCM, Twilio, SendGrid).
Adding all four = multi-channel dispatcher with per-channel retry and rate limiting.
One channel = simple 2-service architecture.
</details>

<details>
<summary>Q7: What two things should you always call out when defining APIs in Step 3?</summary>

1. **Pagination** — for any endpoint that returns a list
2. **Idempotency** — for any write endpoint (so retries don't cause duplicates)
</details>

---

## Personal Weak Areas
- **Step 1:** Initially asked "what should we choose from CAP?" — that's an architectural decision, not a clarifying question. Interviewer should never decide CAP for you.
- **Step 1:** Missed delivery channel question initially — the most design-changing FR for notification systems.
- **Step 1:** Missed reliability NFR ("at-least-once delivery or dropping okay?").
- **Requirement change meta-skill:** Didn't know the correct first sentence. Now locked in.

---

## Key Diagram
```
1. CLARIFY → 2. ESTIMATE → 3. APIs → 4. DATA →
5. HLD → 6. DEEP DIVE → 7. WRAP-UP

Step 7 always: Scale it → Fail it → Secure it → Monitor it → Summarize
```
