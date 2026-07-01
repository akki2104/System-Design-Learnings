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

### [003] Back-of-the-Envelope Estimation
```
THE 4 NUMBERS
─────────────────────────────────────────────────
Write QPS (avg)  = DAU × writes/user/day ÷ 86,400
Peak QPS         = avg × 2–10×
Storage/day      = writes/day × obj_size × replication  ← USE AVERAGE
Storage/N years  = storage/day × 365 × N
Bandwidth        = PEAK QPS × payload_size              ← USE PEAK

UNIT LADDER (strip 3 zeros per step up)
KB(10³) → MB(10⁶) → GB(10⁹) → TB(10¹²) → PB(10¹⁵)

IMPLICATION RULE
Read-heavy (>10:1)  → cache layer
Write-heavy (≈1:1)  → sharding/partitioning
PB-scale            → object storage (S3), not RDBMS

OBJECT SIZES
Tweet/message ~100–300 bytes  |  User row ~1KB
Photo (thumb) ~200KB          |  Photo (full) ~1–3MB
Audio (1 min) ~1MB            |  Video (1 min) ~50–100MB

INTERVIEW NARRATIVE (≤5 min)
1. State assumptions (DAU, ratio, sizes)
2. Compute 4 numbers (show working, round aggressively)
3. State implication ("write-heavy → need sharding")
4. Move on
```
---

### [004] Non-Functional Requirements
```
THE 5 NFRs
─────────────────────────────────────────────────────
Availability    → system UP; measured in nines
                  Series: A = A1×A2×A3 (degrades with each dependency)
                  Parallel: A = 1-(1-A1)(1-A2) (improves with redundancy)

Reliability     → correct results when UP (≠ Availability)
                  Available + wrong = most dangerous failure mode
                  Sub: fault tolerance, redundancy, graceful degradation

Scalability     → DB bottlenecks before server (stateful vs stateless)
                  Order: server → DB connections → DB writes → I/O

Maintainability → operable, simple, evolvable
                  Signal: add monitoring + logging to every design

Cost            → compute + storage + network + ops
                  Move: name most expensive component + how to reduce it

NFR FORMAT (use every time)
─────────────────────────────────────────────────────
[Quality] + [Threshold + p99] + [Business justification]
"< 2s at p99 — users abandon after 3s, direct revenue impact"

NINES TABLE
─────────────────────────────────────────────────────
99%     → 3.65 days/year   99.99%  → 52.6 min/year
99.9%   → 8.76 hours/year  99.999% → 5.26 min/year

NFR PRIORITY BY SYSTEM
─────────────────────────────────────────────────────
Payments → Reliability | Social → Availability | Healthcare → Consistency
Real-time → Latency    | Internal → Cost

TRADEOFF FRAMEWORK
─────────────────────────────────────────────────────
1. Name the conflict  2. Challenge whether stricter NFR is truly needed
3. Propose specific tradeoff + business justification
```
---
