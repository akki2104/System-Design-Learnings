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
