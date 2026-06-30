# Interview Mistakes Log

Running log of every mistake made during lessons, mastery checks, and mock interviews.
Recurring mistakes (count ≥ 2) are flagged as persistent weak areas.

Format:
```
### YYYY-MM-DD — [Topic NNN: Title]
- Mistake: <what was said/done>
- Why it's wrong: <...>
- Correct understanding: <...>
- How to remember: <mnemonic or hook>
- Recurs? <count>
```

---

### 2026-06-28 — [Topic 002: The System Design Interview Framework]
- Mistake: Asked "What should we choose from CAP?" as a clarifying question to the interviewer
- Why it's wrong: CAP reasoning is YOUR architectural decision to make and defend. Asking the interviewer signals you can't reason independently.
- Correct understanding: You gather requirements in Step 1, then YOU decide the consistency/availability tradeoff in Step 4–5 and justify it.
- How to remember: Clarifying questions ask about REQUIREMENTS. Architectural questions are for YOU to answer, not the interviewer.
- Recurs? 1

### 2026-06-28 — [Topic 002: The System Design Interview Framework]
- Mistake: Missed the most design-changing FR for a notification system (delivery channel)
- Why it's wrong: Channel choice (push/SMS/email/in-app) determines how many third-party integrations and what architecture is needed. It's the highest-leverage Step 1 question.
- Correct understanding: For any notification-type system, channel is always the first FR to clarify.
- How to remember: "How does it reach the user?" = the channel question = always ask this first.
- Recurs? 1

### 2026-06-28 — [Topic 001: Introduction to System Design]
- Mistake: When asked for NFRs, gave solutions instead ("use SQL", "add a load balancer")
- Why it's wrong: NFRs describe HOW WELL the system must perform, not WHAT technology to use. Solutions come AFTER NFRs.
- Correct understanding: An NFR starts with "The system must…" and describes a quality/constraint (availability, latency, durability). The solution is chosen to satisfy it.
- How to remember: NFR = the problem. Solution = the answer. Never give the answer before stating the problem.
- Recurs? 1

### 2026-06-28 — [Topic 001: Introduction to System Design]
- Mistake: Described a Functional Requirement ("users can sign in from anywhere") as an NFR
- Why it's wrong: FRs describe WHAT the system does (features). NFRs describe HOW WELL (quality constraints). "Sign in from anywhere" is a feature, not a quality.
- Correct understanding: Ask "can this be satisfied by writing one function?" → Yes = FR. "Does satisfying this require architectural decisions?" → Yes = NFR.
- How to remember: FR = What. NFR = How well.
- Recurs? 1

### 2026-06-30 — [Topic 003: Back-of-the-Envelope Estimation]
- Mistake: Used peak QPS (5×10^5) instead of average QPS (10^5) for storage/day calculation
- Why it's wrong: Storage is how much data lands on disk per day — governed by average arrival rate. Peak QPS is a burst multiplier for capacity planning (servers, network), not for disk accounting.
- Correct understanding: Storage/day = writes/day × obj_size × replication. writes/day = DAU × writes/user/day (or avg QPS × 86,400). Never use peak QPS here.
- How to remember: "Storage = accounting (what actually arrived). Bandwidth = capacity (worst case)."
- Recurs? 1

### 2026-06-30 — [Topic 003: Back-of-the-Envelope Estimation]
- Mistake: Multiplied storage/day by 5 (years) without multiplying by 365 (days/year) for multi-year estimate
- Why it's wrong: 5-year storage = storage/day × 365 days/year × 5 years. Skipping ×365 gives an answer off by 365× — turning PBs into TBs.
- Correct understanding: Multi-year = /day × 365 × N. Shortcut: ≈ /day × 400 × N (rounds 365 to 400).
- How to remember: "Per day → per year → per N years. Two multiplications, not one."
- Recurs? 1

### 2026-06-30 — [Topic 003: Back-of-the-Envelope Estimation]
- Mistake: Stated implication as "multiple servers with load balancers" — correct but generic
- Why it's wrong: Every scaled system has multiple servers. The implication of estimation numbers must name the *type of constraint* (read-heavy → cache; write-heavy → sharding; PB-scale → object storage). Generic answers don't demonstrate architectural thinking.
- Correct understanding: After computing numbers, ask "what kind of problem is this?" and name it specifically. 1:1 ratio = write-heavy = sharding. 100:1 = read-heavy = cache. PB = object storage.
- How to remember: "The number is the symptom. Name the diagnosis, not just the symptom."
- Recurs? 1
