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
