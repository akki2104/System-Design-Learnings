# Topic 002 — The System Design Interview Framework
**Module:** 0 — Orientation & Mental Models
**Status:** Completed | **Date:** 2026-06-28
**Revision file:** [Revision_002.md](../Revision/Revision_002.md)

---

## 1. Why This Topic Exists
Without a framework, candidates go silent, jump to solutions, or spend 40 minutes on the wrong part. This 7-step skeleton works for any problem — freeing your brain to think instead of remembering what to do next.

---

## 2. Real Production Problem
A senior engineer asked to redesign a payment system opens a doc and starts writing code. A strong engineer asks: *"What's broken? How many transactions/day? What's our SLA?"* The framework is how experienced engineers approach any unfamiliar system — in production and in interviews.

---

## 3. Simple Intuition
Like a doctor's consultation. Bad doctor: patient says "I have pain" → immediately prescribes surgery. Good doctor: asks where, how long, what makes it worse, runs tests, then recommends. **Diagnose before you prescribe.**

---

## 4. The 7-Step Framework

```
┌────┬────────────────────────┬──────────────────┐
│ #  │ Step                   │ Time (45-min)    │
├────┼────────────────────────┼──────────────────┤
│ 1  │ CLARIFY & SCOPE        │ ~5 min           │
│ 2  │ ESTIMATE               │ ~5 min           │
│ 3  │ DEFINE APIs            │ ~5 min           │
│ 4  │ DATA MODEL             │ ~5 min           │
│ 5  │ HIGH-LEVEL DESIGN      │ ~15 min          │
│ 6  │ DEEP DIVE              │ ~10 min          │
│ 7  │ BOTTLENECKS & WRAP-UP  │ ~5 min           │
└────┴────────────────────────┴──────────────────┘
Total: 45 min | Order is fixed | Time is flexible
```

---

## 5. Each Step in Detail

### Step 1 — Clarify & Scope (~5 min)
Two sub-parts — FRs first, then NFRs:

**Functional Requirements (what it does):**
- Which features are in scope? What's explicitly OUT of scope?
- Delivery channels? (push/SMS/email/in-app)
- What triggers the feature?
- Edge cases: offline users, multiple devices, user preferences?

**Non-Functional Requirements (how well):**
- Scale: how many DAU / requests per day?
- Latency: what's acceptable?
- Availability: 99.9% or 99.99%?
- Reliability: at-least-once delivery, or dropping is okay?
- Consistency: strong or eventual?

> **Rule:** Minimum 3 clarifying questions. Never draw a box until Step 1 is done.

### Step 2 — Estimate (~5 min)
```
QPS = (DAU × requests/user/day) / 86,400
Peak QPS ≈ 2–5× average
Storage/day = writes/day × avg object size × replication factor
Bandwidth = QPS × avg payload size
```
State every assumption out loud. Interviewers give credit for correct reasoning even if numbers are off.

### Step 3 — Define APIs (~5 min)
Agree on contracts before drawing boxes. Always note:
- **Pagination** for list endpoints
- **Idempotency** for write endpoints

```
Example (notification system):
sendNotification(user_id, type, payload, channel)
  → {notification_id, status: "queued"}

getNotifications(user_id, cursor, limit)
  → {notifications: [...], next_cursor}
```

### Step 4 — Data Model (~5 min)
Identify entities, relationships, schema. Make the SQL vs NoSQL decision here with justification. Choose the shard/partition key.

```
Example:
User         { user_id, device_tokens[], preferences{} }
Notification { notif_id, user_id, type, payload, status, created_at }
```

### Step 5 — High-Level Design (~15 min)
Draw components. Walk the happy path end-to-end. Justify each box in one sentence.

```
Client → API Gateway → Notification Service → [APNs / FCM / Twilio / SendGrid]
                              │
                         Message Queue
                              │
                         Notification DB
```

### Step 6 — Deep Dive (~10 min)
Pick 1–2 hardest components. Self-direct to best tradeoffs if interviewer doesn't direct you. This is where senior signal lives.

### Step 7 — Bottlenecks & Wrap-Up (~5 min)
Stress-test your own design before the interviewer does:
```
1. SCALE IT   "If traffic 10×, what breaks first?"
2. FAIL IT    "If X goes down, what happens?"
3. SECURE IT  "How do we prevent abuse/spoofing?"
4. MONITOR IT "What metrics would I alert on?"
```
Close with a one-sentence summary of the design and its main tradeoff.

---

## 6. Framework Flow Diagram

```
PROBLEM STATEMENT
      │
      ▼
┌─────────────┐
│ 1. CLARIFY  │  FR first → NFR → define out-of-scope
└──────┬──────┘
       ▼
┌─────────────┐
│ 2. ESTIMATE │  QPS / Storage / Bandwidth. State assumptions.
└──────┬──────┘
       ▼
┌─────────────┐
│  3. APIs    │  Contracts. Pagination + idempotency.
└──────┬──────┘
       ▼
┌─────────────┐
│ 4. DATA     │  Entities, schema, SQL vs NoSQL, shard key.
└──────┬──────┘
       ▼
┌─────────────┐
│  5. HLD     │  Draw boxes. Happy path. Justify each box.
└──────┬──────┘
       ▼
┌─────────────┐
│ 6. DEEP     │  1–2 hardest components. Best tradeoffs.
│   DIVE      │
└──────┬──────┘
       ▼
┌─────────────┐
│ 7. WRAP-UP  │  Scale → Fail → Secure → Monitor → Summarize
└─────────────┘
```

---

## 7. Meta-Skills (run through all steps)

**Think out loud.** Every thought spoken. Silence is your enemy.

**Drive the conversation.** Move through the framework yourself. Don't wait.

**Embrace requirement changes.** When the interviewer changes requirements mid-session:
```
First sentence: "Okay, that changes things —
                 let me think about where that impacts the design."

Then: "Going global affects: latency, data residency
       laws, and DB replication. Let me take them one by one…"

Pattern: Acknowledge → Identify what changed → Re-enter framework
```

**Manage time.** If still on HLD at 30-minute mark, speed up.

**Never ask the interviewer to make architectural decisions.**
❌ "Should we use microservices?"
❌ "What should we choose from CAP?"
✅ You reason, you decide, you defend.

---

## 8. Step 1 Deep-Dive: Notification System Example

```
FUNCTIONAL REQUIREMENTS
  ✓ What channels? Push / SMS / Email / In-app?    ← most design-changing FR
  ✓ What triggers notifications?
  ✓ User preference management (opt-out per type)?
  ✓ In-app notification inbox needed?

NON-FUNCTIONAL REQUIREMENTS
  ✓ Scale: DAU, notifications/day
  ✓ Latency: how fast must it arrive?
  ✓ Reliability: at-least-once, or dropping okay?  ← most design-changing NFR

OUT OF SCOPE (state explicitly)
  → Notification content generation
  → Anti-spam / DND hours (defer to v2)
```

---

## 9. Company Flavors

| Company | Emphasis |
|---|---|
| Google | Steps 5–6: deep technical correctness, edge cases |
| Meta | Step 1: product thinking, which features matter |
| Amazon | Step 7: failure modes, cost, operational ownership |
| Stripe | Steps 3–4: API correctness, idempotency |
| Uber | Steps 5–6: real-time geo, matching, latency |

---

## 10. Common Mistakes

1. **Skipping Step 1** — most expensive mistake (10–15% of score)
2. **Skipping Step 2** — designing blind to scale
3. **Asking interviewer architectural questions** — CAP choice, DB choice = your decision
4. **Never reaching Step 6** — no deep dive = no senior signal
5. **Forgetting Step 7** — not stress-testing your own design
6. **Treating framework as script** — robotic delivery kills rapport

---

## 11. Real Interview Questions

1. "Walk me through how you'd start a system design problem."
2. "You've been designing for 20 minutes — what's the most important thing you haven't discussed?"
3. "The interviewer changes the requirement mid-session. What do you say first?"
4. "How do you decide what to deep-dive in Step 6?"
5. "What's the first clarifying question you ask for any system?"

---

## 12. Cheat Sheet → [CheatSheets.md](../CheatSheets.md)

```
7-STEP FRAMEWORK
══════════════════════════════════════════════════════════
1. CLARIFY    FR ("what features?") → NFR ("what scale/latency?")
              Min 3 questions. Define out-of-scope.
2. ESTIMATE   QPS = DAU × req/day ÷ 86400. Peak = 2–5×. State assumptions.
3. APIs       Contracts first. Pagination + idempotency always.
4. DATA       Entities, schema, SQL vs NoSQL, shard key.
5. HLD        Boxes + arrows. Walk happy path. Justify each box.
6. DEEP DIVE  1–2 hardest components. Self-direct to best tradeoffs.
7. WRAP-UP    Scale → Fail → Secure → Monitor → Summarize

TIME (45 min): 5 / 5 / 5 / 5 / 15 / 10 / 5

META-SKILLS:
  Think out loud | Drive the conversation | Embrace changes
  Never ask interviewer to make architectural decisions
  First sentence on requirement change:
  "Okay, that changes things — let me think about where that impacts the design."
══════════════════════════════════════════════════════════
```

---

## 13. Summary

- The 7-step framework is the skeleton for every interview and every case study in this program.
- **Order is fixed. Time is flexible.**
- Step 1 (Clarify) is worth 10–15% of score alone — never skip it.
- Requirement changes mid-session are deliberate — the correct response is calm re-reasoning.
- You can now: name all 7 steps in order, run Step 1 for any system, and handle a mid-session requirement change without panicking.
