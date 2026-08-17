# Topic Priority & Time Budget

> **Created:** 2026-07-25 · **Owner:** mentor maintains, learner decides
> **Purpose:** Every remaining topic rated by interview value, so the learner can make an informed
> keep/compress/skip call *before* each lesson instead of discovering the cost afterwards.
> **Calibration:** Software Engineer, ~2 years experience. Targets: Indian product companies
> (Flipkart, Razorpay, Swiggy, PhonePe, CRED…), FAANG + global product (Google, Meta, Amazon,
> Uber, Stripe, Netflix), and well-funded startups. **LLD/machine-coding rounds expected.**

---

## The Tiers

| Tier | Meaning | Typical time | Mentor behaviour |
|------|---------|--------------|------------------|
| 🔴 **MUST** | Asked directly or needed as vocabulary in most case studies. Skipping creates a visible gap. | 35–50 min | Full 24-step lesson + revision file |
| 🟡 **SKIM** | Worth recognising and name-dropping; not worth a full LIVE lesson at this level. | 8–10 min | Batched 5–6 per session, condensed LIVE treatment. **Documentation is still full-depth** — complete `Topics/NNN_*.md` + `Revision/Revision_NNN.md`, real examples, common mistakes, interview questions, full revision Q&A. Tier shapes teaching pace, never documentation depth (2026-08-17 correction) |
| ⚫ **SKIP** | Staff-level, specialist, or already-known-by-any-engineer. Very rarely decides a 2-YOE interview. | 0 (45+ if done anyway) | Not scheduled. Available on request with stated cost |

---

## Budget Model (how "impact" is calculated)

```
Assumed study rate : ~2.25 hrs/day of focused work
Conversion         : 45 unplanned minutes ≈ 0.33 day of schedule slip
                     3 unplanned MUST-sized topics ≈ 1 full day slip
Target date        : 2026-08-18  (revised from 2026-08-09 on 2026-07-25)
```

| Work item | Volume | Hours |
|---|---|---|
| 🔴 Must theory topics | 35 × 40 min | ~23.5 |
| 🟡 Skim batches | 26 × 8 min | ~3.5 |
| LLD essentials | 6 × 45 min | ~4.5 |
| Case studies | 14 × 50 min | ~11.5 |
| Mock interviews | 3 × 75 min | ~3.75 |
| Revision (weekend cadence) | — | ~4.0 |
| **TOTAL REMAINING** | | **~51 hrs ≈ 23 days** |

**This budget is zero-sum.** Adding a ⚫ SKIP topic in full does not come free — it comes out of
case-study time, which is the highest-value block in the plan. That trade is always stated explicitly
in the briefing card before the topic starts.

---

## MODULE 2 — Data Storage (remaining)

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 028 | NoSQL Overview | 🔴 MUST | 40m | Medium | Frames every "SQL or NoSQL?" question |
| 029 | Wide-Column Stores (Cassandra/DynamoDB) | 🔴 MUST | 45m | Med-Hard | Every write-heavy design lands here |
| 030 | Document Stores (MongoDB) | 🟡 SKIM | 10m | Easy | Fold into 028; you already know the flexible-schema tradeoff |
| 031 | Choosing a Database | 🔴 MUST | 45m | Medium | This *is* the interview skill — the decision framework itself |

## MODULE 3 — Caching

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 032 | Caching Fundamentals | 🔴 MUST | 35m | Easy | Foundation for the whole module |
| 033 | Caching Patterns (cache-aside, write-through…) | 🔴 MUST | 45m | Medium | Cache-aside is asked in nearly every design |
| 034 | Eviction Policies | 🟡 SKIM | 8m | Easy | Know LRU + TTL; LFU/FIFO are trivia |
| 035 | Cache Problems (stampede, penetration, avalanche) | 🔴 MUST | 45m | Med-Hard | Favourite deep-dive follow-up |
| 036 | Distributed Caching with Redis/Memcached | 🔴 MUST | 45m | Medium | Named tech + data structures you'll cite constantly |
| 037 | Cache Consistency & Invalidation | 🔴 MUST | 45m | Hard | "The hard part" — strong differentiator when handled well |

## MODULE 4 — Scaling & Distributing Data *(highest-signal module remaining)*

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 038 | Vertical vs Horizontal Scaling | 🟡 SKIM | 8m | Easy | Already understood from Topics 001/004 |
| 039 | Replication | 🔴 MUST | 45m | Medium | Leader-follower shows up in every DB discussion |
| 040 | Replication Lag & Read-Your-Writes | 🔴 MUST | 45m | Med-Hard | Classic follow-up: "user posts then can't see it" |
| 041 | Partitioning & Sharding | 🔴 MUST | 50m | Med-Hard | Top-3 highest-signal topic in the whole curriculum |
| 042 | Consistent Hashing | 🔴 MUST | 45m | Med-Hard | Asked by name; also its own case study |
| 043 | Choosing a Shard Key | 🔴 MUST | 45m | Hard | Hot-partition reasoning separates strong candidates |
| 044 | Rebalancing & Resharding | 🟡 SKIM | 8m | Medium | Fold into 042 |

## MODULE 5 — Distributed Systems Theory *(largest cuts)*

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 045 | Why Distributed Systems Are Hard | 🟡 SKIM | 10m | Easy | 8 fallacies — good vocabulary, cheap |
| 046 | CAP Theorem | 🔴 MUST | 45m | Medium | Named in interviews constantly; misconceptions are penalised |
| 047 | PACELC | 🟡 SKIM | 8m | Medium | One-liner extension of CAP |
| 048 | Consistency Models | 🔴 MUST | 45m | Hard | Strong vs eventual vs causal — needed for any replicated design |
| 049 | Time, Clocks & Ordering (Lamport/vector) | ⚫ SKIP | (50m) | Hard | Staff-level. Rare at 2 YOE |
| 050 | Quorums (R + W > N) | 🔴 MUST | 30m | Medium | Short, high-yield, pairs with Cassandra/Dynamo |
| 051 | Consensus & Leader Election | 🟡 SKIM | 10m | Hard | Know the *problem*, not the algorithms |
| 052 | Raft | 🟡 SKIM | 15m | Hard | "Leader election + log replication" is enough; skip the protocol |
| 053 | Paxos | ⚫ SKIP | (50m) | Very Hard | Never required at this level |
| 054 | Distributed Transactions: 2PC & 3PC | 🟡 SKIM | 10m | Hard | "It exists; it blocks on coordinator failure" |
| 055 | Sagas & Compensating Transactions | 🔴 MUST | 45m | Med-Hard | The microservices distributed-txn answer interviewers want |
| 056 | Idempotency & Exactly-Once Semantics | 🔴 MUST | 45m | Med-Hard | Very high signal; ties to payments + retries |
| 057 | Conflict Resolution & CRDTs | ⚫ SKIP | (45m) | Very Hard | Only if a collaborative-editing case study comes up |

## MODULE 6 — Messaging, Streaming & Async

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 058 | Why Asynchronous Processing | 🔴 MUST | 25m | Easy | Short; sets up the whole module |
| 059 | Message Queues (RabbitMQ model) | 🔴 MUST | 40m | Medium | Queues appear in most designs |
| 060 | Log-Based Streaming (Kafka) | 🔴 MUST | 55m | Hard | Highest-signal topic in this module |
| 061 | Delivery Guarantees | 🔴 MUST | 40m | Med-Hard | At-least-once + idempotency is a standard combo answer |
| 062 | Stream Processing | 🟡 SKIM | 10m | Hard | Windowing/watermarks — recognition only |
| 063 | Event-Driven Architecture | 🟡 SKIM | 10m | Medium | Mostly a naming layer over 058–061 |
| 064 | Event Sourcing | ⚫ SKIP | (40m) | Hard | Niche; rarely decides an interview |
| 065 | CQRS | 🟡 SKIM | 8m | Medium | One paragraph is enough |
| 066 | CDC & Outbox Pattern | 🟡 SKIM | 10m | Med-Hard | Outbox is a nice-to-have flourish |

## MODULE 7 — Reliability, Resilience & Operations

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 067 | Availability Math (SLA/SLO/SLI) | 🟡 SKIM | 10m | Medium | Largely covered already in Topic 004 |
| 068 | Fault Tolerance & Redundancy | 🟡 SKIM | 8m | Easy | Covered across 004/017 |
| 069 | Timeout, Retry, Exponential Backoff + Jitter | 🔴 MUST | 45m | Medium | Very commonly probed; jitter is the detail people miss |
| 070 | Circuit Breaker & Bulkhead | 🔴 MUST | 40m | Medium | Circuit breaker asked by name |
| 071 | Rate Limiting & Throttling | 🔴 MUST | 55m | Med-Hard | Both a topic *and* a full case study |
| 072 | Load Shedding & Backpressure | 🟡 SKIM | 8m | Medium | Recognition-level |
| 073 | Graceful Degradation & Failover | 🟡 SKIM | 8m | Easy | Already covered in 004 |
| 074 | Disaster Recovery (RPO/RTO) | 🟡 SKIM | 10m | Medium | Know what the two acronyms mean |
| 075 | Health Checks & Failure Detection | 🟡 SKIM | 5m | Easy | Already done in Topic 017 |
| 076 | Monitoring & Metrics (4 golden signals) | 🔴 MUST | 30m | Easy | Cheap points on the Operations rubric dimension |
| 077 | Logging at Scale | 🟡 SKIM | 8m | Easy | Structured logging + aggregation, one pass |
| 078 | Distributed Tracing | 🟡 SKIM | 10m | Medium | Spans + context propagation, recognition-level |
| 079 | Alerting & On-Call | ⚫ SKIP | (30m) | Easy | Practitioner knowledge, not interview content |

## MODULE 8 — Security & Identity

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 080 | Security Fundamentals (CIA, OWASP) | 🟡 SKIM | 10m | Easy | Vocabulary layer |
| 081 | Authentication vs Authorization | 🔴 MUST | 25m | Easy | Short, and asked directly |
| 082 | Sessions, Cookies, Tokens, JWT | 🔴 MUST | 45m | Medium | JWT pitfalls are a common follow-up |
| 083 | OAuth 2.0 & OIDC | 🟡 SKIM | 12m | Hard | Know the flow shape, not every grant type |
| 084 | Encryption | 🟡 SKIM | 8m | Medium | Mostly covered in Topic 011 |
| 085 | Secrets Management & Cert Rotation | ⚫ SKIP | (30m) | Medium | Ops concern |
| 086 | API Security | 🟡 SKIM | 10m | Medium | Rate limiting + input validation already covered elsewhere |
| 087 | Multi-Tenancy & Data Isolation | ⚫ SKIP | (40m) | Med-Hard | Only for B2B SaaS-specific interviews |
| 088 | Privacy, Compliance & Data Residency | ⚫ SKIP | (30m) | Medium | PCI-DSS/HIPAA already touched in Topic 011 |

## MODULE 9 — Specialized Building Blocks

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 089 | Bloom Filters | 🔴 MUST | 40m | Med-Hard | Classic; already previewed in Topic 023 |
| 090 | HyperLogLog | 🟡 SKIM | 8m | Hard | "Approximate distinct count" is enough |
| 091 | Count-Min Sketch | ⚫ SKIP | (40m) | Hard | Rarely required |
| 092 | Geospatial Indexing (geohash, quadtree, S2) | 🔴 MUST | 50m | Hard | **Upgraded to MUST** — Uber/Maps/Swiggy/Zomato all in your target list |
| 093 | Full-Text Search & Inverted Indexes | 🟡 SKIM | 12m | Medium | Concept only |
| 094 | Elasticsearch / OpenSearch | 🟡 SKIM | 10m | Medium | Name-drop + when to use |
| 095 | Time-Series Databases | ⚫ SKIP | (30m) | Medium | Niche |
| 096 | Object/Blob Storage (S3) | 🔴 MUST | 25m | Easy | Short; appears in almost every design |
| 097 | Distributed File Systems (GFS/HDFS) | ⚫ SKIP | (40m) | Hard | Historical/academic at this level |
| 098 | Unique ID Generation (Snowflake, ULID) | 🔴 MUST | 40m | Medium | Classic topic *and* a case study |
| 099 | Distributed Locks & Leases | 🟡 SKIM | 12m | Hard | Know Redlock exists + fencing tokens |
| 100 | Service Discovery & Service Mesh | ⚫ SKIP | (35m) | Medium | Partially covered via mTLS in Topic 011 |
| 101 | Configuration & Feature Flags | ⚫ SKIP | (25m) | Easy | Practitioner knowledge |

## MODULE 10 — Architecture Styles & Delivery *(almost entirely cuttable)*

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 102 | Monolith vs Microservices vs Modular Monolith | 🔴 MUST | 35m | Medium | Opinion question you must answer crisply |
| 103 | Service Decomposition & DDD Basics | 🟡 SKIM | 10m | Medium | Bounded contexts, one pass |
| 104 | API Gateway & BFF | 🟡 SKIM | 8m | Easy | Already covered in Topic 016 |
| 105 | Serverless & FaaS | ⚫ SKIP | (30m) | Easy | Cold starts — one sentence if it ever comes up |
| 106 | Containers & Docker | ⚫ SKIP | (30m) | Easy | You already know this as an engineer |
| 107 | Orchestration & Kubernetes | ⚫ SKIP | (40m) | Medium | Not an HLD-interview decider at 2 YOE |
| 108 | CI/CD & Deployment Strategies | 🟡 SKIM | 10m | Easy | Blue-green / canary / rolling — names + when |
| 109 | Infrastructure as Code | ⚫ SKIP | (25m) | Easy | Not interview content |
| 110 | Cloud Building Blocks | ⚫ SKIP | (30m) | Easy | Absorbed implicitly throughout |

## MODULE 11 — Capstone *(do all — this is where it comes together)*

| ID | Topic | Tier | Time | Complexity | Why |
|----|-------|------|------|-----------|-----|
| 111 | The Full HLD Playbook | 🔴 MUST | 50m | Medium | Ties all 7 framework steps into one repeatable motion |
| 112 | Estimation Mastery Drill | 🔴 MUST | 50m | Med-Hard | Directly targets your longest-standing weak area (write-QPS formula) |
| 113 | Bottleneck Hunting & Evolution | 🔴 MUST | 50m | Hard | "Now 10× the traffic" — the standard live-pressure follow-up |
| 114 | Cost Reasoning in Design | 🟡 SKIM | 12m | Medium | Name the expensive component + how to reduce it |

---

## LLD Track — 6 Sessions *(kept at 6, not 4 — machine-coding rounds expected)*

| ID | Topic | Tier | Time | Why |
|----|-------|------|------|-----|
| L001+L002 | OOP Fundamentals + SOLID (combined) | 🔴 MUST | 50m | SOLID is asked by name in nearly every LLD round |
| L004 | UML: Class + Sequence Diagrams | 🔴 MUST | 40m | You must be able to draw these live |
| L008 | Creational Patterns (Factory, Builder, Singleton) | 🔴 MUST | 45m | Most-asked pattern family |
| L009+L010 | Structural + Behavioral Patterns (combined) | 🔴 MUST | 55m | Strategy, Observer, Decorator, Adapter, State — the high-frequency five |
| L011 | Concurrency & Thread Safety | 🔴 MUST | 45m | "Now make it thread-safe" is the standard machine-coding twist |
| L019 | Machine-Coding Round Playbook | 🔴 MUST | 45m | The round itself — how to structure 60–90 min under pressure |
| L003, L005–L007, L012–L018 | GRASP, remaining UML, DI, clean code, refactoring, testing, reviews | ⚫ SKIP | — | Absorbed implicitly or not interview-decisive at 2 YOE |

---

## Case Studies — 14 *(the highest-ROI block in the entire plan)*

Ordered by interview frequency. Do them in this order; if time runs short, the tail is what gets cut.

| # | Case Study | Tier | Time | Why it's on the list |
|---|---|---|---|---|
| 1 | TinyURL / URL Shortener | 🔴 | 45m | The universal warm-up question |
| 2 | Rate Limiter | 🔴 | 45m | Extremely common; pairs with Topic 071 |
| 3 | Twitter / News Feed | 🔴 | 60m | The canonical fanout problem |
| 4 | WhatsApp / Chat | 🔴 | 60m | Pairs with WebSockets (015) |
| 5 | Notification System | 🔴 | 45m | Queue + fanout + third-party integration |
| 6 | Unique ID Generator | 🔴 | 40m | Short; pairs with Topic 098 |
| 7 | Key-Value Store / Distributed Cache | 🔴 | 50m | Consistent hashing + quorums in practice |
| 8 | YouTube / Video Streaming | 🔴 | 60m | CDN + blob storage + encoding pipeline |
| 9 | Uber / Ride Matching | 🔴 | 60m | Geospatial in practice (Topic 092) |
| 10 | Payment Gateway (Stripe-style) | 🔴 | 60m | Idempotency + consistency under money constraints |
| 11 | Typeahead / Autocomplete | 🔴 | 45m | Tries + caching + ranking |
| 12 | Web Crawler | 🟡 | 45m | Good for queue/dedup reasoning |
| 13 | Leaderboard / Top-K | 🟡 | 40m | Redis sorted sets |
| 14 | BookMyShow / Ticketing | 🟡 | 55m | Locking + inventory under contention |

**Cut first if behind:** #12, #13, #14 (the 🟡 tail). **Never cut:** #1–#6.

---

## Mock Interviews — 3

| # | Type | Time | When |
|---|---|---|---|
| Mock 1 | Tier-1 system, untimed, coached | 75m | After case study #6 |
| Mock 2 | Full 45-min timed, company-flavoured | 75m | After case study #11 |
| Mock 3 | Requirement-change / curveball round | 75m | Final week |

---

## Deviation Log

Recorded whenever the learner overrides a tier recommendation, so schedule drift is traceable
rather than mysterious.

| Date | Topic | Recommended | Chosen | Time cost | Cumulative slip |
|------|-------|-------------|--------|-----------|-----------------|
| — | — | — | — | — | 0.0 days |
