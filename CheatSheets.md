# Cheat Sheets

One cheat sheet per completed topic + a Master Cheat Sheet at the bottom.
Each cheat sheet is the condensed one-screen summary from Section 23 of the lesson.

---

## Master Cheat Sheet (grows over time)
<!-- Updated continuously; becomes the go-to pre-interview review -->

---

## Per-Topic Cheat Sheets

### [001] Introduction to System Design
```
WHAT IT IS: Designing systems that work at scale, reliably, under real constraints.
Interviews test: thinking process + tradeoffs + adaptability — NOT trivia.

HLD = components, connections, data flow, DB choice (bird's eye)
LLD = classes, patterns, interfaces, thread safety (ground level)

FR  = WHAT the system does        ("users can save a note")
NFR = HOW WELL the system does it ("notes must save in < 200ms")
NFR drives architecture. Solution satisfies NFR.

FIVE CORE IDEAS:
1. System = components talking over a network
2. HLD vs LLD — two separate interview rounds
3. Every decision is a tradeoff
4. NFRs drive architecture more than features do
5. Interviews test thinking, not memory

OBSERVABILITY = Metrics + Logs + Traces

TOP MISTAKES:
1. Jump to solution before clarifying requirements
2. Recite tech without justifying why
3. Ignore NFRs
4. Over-engineer
5. Go silent — it's a conversation

RULE: Never draw a single box until you've asked 3 clarifying questions.
```
---
