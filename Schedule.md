# Master Schedule — COMPRESSED TRACK (v2)

> **Rewritten:** 2026-07-25, from the learner's *actual* position (Topic 027 complete), replacing the
> original v1 calendar which assumed a zero-gap run from June 28 and had drifted ~51 topics out of date.
> **Priority ratings live in `TopicPriority.md`** — read it before every topic and show the Topic
> Briefing Card (Master Guide §0.1).

**Position at rewrite:** Topics 001–027 complete · Modules 0 and 1 done · Module 2 in progress
**Revised interview-ready target:** **2026-08-18** *(was Aug 9; +9 days)*
**Assumed rate:** ~2.25 hrs/day focused · ~51 hrs of work remaining ≈ 23 days

**Why the extension:** the original date required ~87 theory topics + 20 case studies + 9 LLD lessons
in 15 days, which was not achievable. Compressing to 35 must-topics / 26 skims / 26 skips and holding
14 case studies + 6 LLD sessions fits in 23 days. The learner is mid-active applying — a 9-day
extension was chosen over dropping case studies, because case-study reps are the single highest-value
block at this stage.

**Legend:** 🔴 full lesson · 🟡 condensed, batched · ⚫ not scheduled (see `TopicPriority.md`)

---

## PHASE 1 — Finish the Theory Core (Jul 25 → Jul 31, 7 days)

| Date | Work | Time |
|------|------|------|
| **Jul 25** *(today)* | 🔴 028 NoSQL Overview · 🔴 029 Wide-Column · 🟡 030 Document Stores · 🔴 031 Choosing a Database | ~2.3h |
| **Jul 26** | 🔴 032 Caching Fundamentals · 🔴 033 Caching Patterns · 🟡 034 Eviction · 🔴 035 Cache Problems | ~2.3h |
| **Jul 27** | 🔴 036 Redis/Memcached · 🔴 037 Cache Consistency · 🟡 038 Vert-vs-Horiz · 🔴 039 Replication | ~2.3h |
| **Jul 28** | 🔴 040 Replication Lag · 🔴 041 Partitioning & Sharding · 🟡 044 Rebalancing | ~1.8h |
| **Jul 29** | 🔴 042 Consistent Hashing · 🔴 043 Shard Key · 🟡 045 Why Distributed Is Hard | ~1.8h |
| **Jul 30** | 🔴 046 CAP · 🟡 047 PACELC · 🔴 048 Consistency Models · 🔴 050 Quorums | ~2.2h |
| **Jul 31** | 🟡 **SKIM BATCH A** (051 Consensus, 052 Raft, 054 2PC/3PC, 062 Stream Proc, 063 EDA, 065 CQRS, 066 CDC/Outbox) · 🔴 055 Sagas | ~1.9h |

> ⚫ Not scheduled in this phase: 049 Clocks/Vector · 053 Paxos · 057 CRDTs · 064 Event Sourcing

## PHASE 2 — Messaging, Reliability, Security (Aug 1 → Aug 4, 4 days)

| Date | Work | Time |
|------|------|------|
| **Aug 1** | 🔴 056 Idempotency · 🔴 058 Why Async · 🔴 059 Message Queues | ~1.8h |
| **Aug 2** | 🔴 060 Kafka · 🔴 061 Delivery Guarantees · 🔴 069 Retry/Backoff+Jitter | ~2.3h |
| **Aug 3** | 🔴 070 Circuit Breaker · 🔴 071 Rate Limiting · 🔴 076 Monitoring | ~2.1h |
| **Aug 4** | 🟡 **SKIM BATCH B** (067 Availability Math, 068 Fault Tolerance, 072 Load Shedding, 073 Graceful Degradation, 074 DR/RPO-RTO, 075 Health Checks, 077 Logging, 078 Tracing) · 🔴 081 AuthN/AuthZ · 🔴 082 JWT/Sessions | ~2.4h |

> ⚫ Not scheduled: 079 Alerting · 085 Secrets · 087 Multi-Tenancy · 088 Compliance

## PHASE 3 — Building Blocks + Capstone (Aug 5 → Aug 8, 4 days)

| Date | Work | Time |
|------|------|------|
| **Aug 5** | 🟡 **SKIM BATCH C** (080 Security Fund., 083 OAuth, 084 Encryption, 086 API Security, 090 HyperLogLog, 093 Full-Text, 094 Elasticsearch, 099 Distributed Locks) · 🔴 089 Bloom Filters | ~1.9h |
| **Aug 6** | 🔴 092 Geospatial Indexing · 🔴 096 Object Storage · 🔴 098 Unique ID Generation | ~2.0h |
| **Aug 7** | 🔴 102 Monolith vs Microservices · 🟡 **SKIM BATCH D** (103 DDD, 104 BFF, 108 CI/CD, 114 Cost Reasoning) · 🔴 111 Full HLD Playbook | ~1.9h |
| **Aug 8** | 🔴 112 Estimation Mastery Drill · 🔴 113 Bottleneck Hunting | ~1.7h |

> ⚫ Not scheduled: 091 CMS · 095 TSDB · 097 DFS · 100 Service Mesh · 101 Feature Flags · 105–107, 109–110

**→ All prioritized HLD theory complete: Aug 8** *(35 full lessons, 26 skims, 26 skipped)*

## PHASE 4 — Case Studies (Aug 9 → Aug 14, 6 days) — *the highest-value block*

| Date | Case Studies | Time |
|------|-------------|------|
| **Aug 9** | CS1 URL Shortener · CS2 Rate Limiter | ~1.5h |
| **Aug 10** | CS3 Twitter/News Feed · CS4 WhatsApp | ~2.0h |
| **Aug 11** | CS5 Notification System · CS6 Unique ID Generator · **Mock 1** (coached) | ~2.6h |
| **Aug 12** | CS7 Key-Value Store · CS8 YouTube | ~1.8h |
| **Aug 13** | CS9 Uber/Ride Matching · CS10 Payment Gateway | ~2.0h |
| **Aug 14** | CS11 Typeahead · **Mock 2** (timed, company-flavoured) | ~2.0h |

> 🟡 Tail — cut first if behind: CS12 Web Crawler · CS13 Leaderboard · CS14 BookMyShow

## PHASE 5 — LLD Essentials (Aug 15 → Aug 16, 2 days)

*Kept at 6 sessions because machine-coding rounds are expected at Indian product companies.*

| Date | Work | Time |
|------|------|------|
| **Aug 15** | L001+L002 OOP & SOLID · L004 UML Class+Sequence · L008 Creational Patterns | ~2.3h |
| **Aug 16** | L009+L010 Structural+Behavioral · L011 Concurrency · L019 Machine-Coding Playbook | ~2.4h |

## PHASE 6 — Consolidation (Aug 17 → Aug 18, 2 days)

| Date | Work | Time |
|------|------|------|
| **Aug 17** | Tail case studies (CS12–CS14 if time) · full weak-area drill (persistent misses: 013, 015, estimation formula) | ~2.5h |
| **Aug 18** | **Mock 3** (requirement-change round) · final cheat-sheet review | ~2.0h |

**→ INTERVIEW READY: 2026-08-18** ✅

---

## Slack Checker

At the end of each day, ask: *"What did the schedule say for today? Did I do it?"*

| Drift | Action |
|-------|--------|
| ≤ 1 day behind | Normal. Absorb it in the next session. |
| 2–3 days behind | Cut the 🟡 case-study tail (CS12–CS14) and reclaim ~2.3h. |
| 4+ days behind | Downgrade the remaining 🔴 topics in Modules 8–10 to 🟡, keep all case studies and mocks. **Never** protect theory at the cost of case studies. |
| Ahead | Add back ⚫ topics in this order: 064 Event Sourcing → 049 Clocks → 100 Service Mesh → 091 Count-Min Sketch. |

**Rule of thumb:** if forced to choose between one more theory topic and one more case study,
**always take the case study.** At 2 YOE, interviews evaluate you on a design conversation, not a quiz.

---

## Revision Cadence

The learner revises on his own schedule (weekends / when enough accumulates), not on strict due
dates. `RevisionSchedule.md` remains the queue of record; the mentor surfaces the backlog in each
daily briefing but does **not** block new content on it.

**Current persistent misses to target in the next revision pass:**
- Topic 013 — gRPC browser blocker (failed ×3, needs a mnemonic, not re-explanation)
- Topic 015 — short-polling latency-vs-waste (failed ×3, numeric-example approach being tried)
- Topic 003 — reads-in-write-QPS formula (long-standing; Topic 112 drill will target it directly)

---

## Archived: v1 Schedule

The original June 28 – Aug 9 calendar is preserved in git history (any commit before 2026-07-25).
It assumed no gaps and no compression; it is superseded by this file and should not be consulted
for pacing decisions.
