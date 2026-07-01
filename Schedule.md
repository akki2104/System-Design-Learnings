# Master Schedule — Accelerated 1-Month Track

**Program start:** June 28, 2026
**Interview-ready target:** August 9, 2026
**Pace:** 15 hrs/week | 5 study days/week | ~2–3 hrs/day

**Depth key:**
- `D` = Deep (45–60 min) — full treatment, high-signal topic
- `S` = Survey (15–20 min) — batched with others, concept + key tradeoffs
- `A` = Awareness — covered *inside* the parent topic listed; no separate session

If you're behind on a date, that's your slack alert. Check this weekly.

---

## HLD Topics — 114 Total

| ID | Title | Type | Target Date | Time |
|----|-------|------|-------------|------|
| 001 | Introduction to System Design | D | Jun 28 ✅ | 60 min |
| 002 | System Design Interview Framework | D | Jun 28 ✅ | 60 min |
| 003 | Back-of-the-Envelope Estimation | D | Jun 30 ✅ | 60 min |
| 004 | Non-Functional Requirements | D | Jun 30 ✅ | 60 min |
| 005 | How to Reason About Tradeoffs | D | Jul 1 | 45 min |
| 006 | The Client–Server Model | S | Jul 1 | 15 min |
| 007 | IP, Ports, Sockets | S | Jul 1 | 15 min |
| 008 | TCP vs UDP | D | Jul 2 | 50 min |
| 009 | DNS | S | Jul 2 | 15 min |
| 010 | HTTP/1.1, HTTP/2, HTTP/3 | D | Jul 2 | 50 min |
| 011 | HTTPS & TLS | S | Jul 3 | 15 min |
| 012 | REST API Design | D | Jul 3 | 50 min |
| 013 | RPC & gRPC | S | Jul 3 | 15 min |
| 014 | GraphQL | S | Jul 3 | 15 min |
| 015 | WebSockets, SSE, Polling, Long Polling | D | Jul 4 | 50 min |
| 016 | Forward Proxy, Reverse Proxy, API Gateway | S | Jul 4 | 15 min |
| 017 | Load Balancers | D | Jul 4 | 50 min |
| 018 | CDN | S | Jul 5 | 15 min |
| 019 | Content Compression & Encoding | A | Jul 5 | inside 018 |
| 020 | Storage Engine Fundamentals | S | Jul 5 | 15 min |
| 021 | Relational Databases & SQL | D | Jul 5 | 50 min |
| 022 | Indexing Deep Dive | D | Jul 6 | 50 min |
| 023 | B-Trees vs LSM-Trees | S | Jul 6 | 15 min |
| 024 | Transactions & ACID | D | Jul 6 | 50 min |
| 025 | Isolation Levels & Anomalies | D | Jul 7 | 50 min |
| 026 | Concurrency Control: Locks, 2PL, Deadlocks | S | Jul 7 | 15 min |
| 027 | MVCC | S | Jul 7 | 15 min |
| 028 | NoSQL Overview | S | Jul 7 | 15 min |
| 029 | Wide-Column Stores | S | Jul 8 | 15 min |
| 030 | Document Stores | S | Jul 8 | 15 min |
| 031 | Choosing a Database | S | Jul 8 | 15 min |
| 032 | Caching Fundamentals | S | Jul 8 | 15 min |
| 033 | Caching Patterns | D | Jul 9 | 50 min |
| 034 | Eviction Policies | S | Jul 9 | 15 min |
| 035 | Cache Problems (stampede, penetration, avalanche) | D | Jul 9 | 50 min |
| 036 | Distributed Caching with Redis/Memcached | D | Jul 10 | 50 min |
| 037 | Cache Consistency & Invalidation | D | Jul 10 | 50 min |
| 038 | Vertical vs Horizontal Scaling | S | Jul 11 | 15 min |
| 039 | Replication | D | Jul 11 | 50 min |
| 040 | Replication Lag & Read Consistency | D | Jul 11 | 50 min |
| 041 | Partitioning & Sharding | D | Jul 12 | 50 min |
| 042 | Consistent Hashing | D | Jul 12 | 50 min |
| 043 | Choosing a Shard Key | D | Jul 13 | 50 min |
| 044 | Rebalancing & Resharding | S | Jul 13 | 15 min |
| 045 | Why Distributed Systems Are Hard | S | Jul 13 | 15 min |
| 046 | CAP Theorem | D | Jul 14 | 50 min |
| 047 | PACELC | A | Jul 14 | inside 046 |
| 048 | Consistency Models | D | Jul 14 | 50 min |
| 049 | Time, Clocks & Ordering | S | Jul 15 | 15 min |
| 050 | Quorums | D | Jul 15 | 50 min |
| 051 | Consensus & Leader Election | S | Jul 15 | 15 min |
| 052 | Raft | D | Jul 16 | 60 min |
| 053 | Paxos | A | Jul 16 | inside 052 |
| 054 | Distributed Transactions: 2PC & 3PC | S | Jul 17 | 20 min |
| 055 | Sagas & Compensating Transactions | D | Jul 17 | 50 min |
| 056 | Idempotency & Exactly-Once Semantics | D | Jul 17 | 50 min |
| 057 | Conflict Resolution & CRDTs | S | Jul 18 | 15 min |
| 058 | Why Asynchronous Processing | S | Jul 18 | 15 min |
| 059 | Message Queues (RabbitMQ model) | S | Jul 18 | 15 min |
| 060 | Log-Based Streaming (Kafka) | D | Jul 18 | 60 min |
| 061 | Delivery Guarantees | D | Jul 19 | 50 min |
| 062 | Stream Processing | S | Jul 19 | 15 min |
| 063 | Event-Driven Architecture | S | Jul 19 | 15 min |
| 064 | Event Sourcing | S | Jul 20 | 15 min |
| 065 | CQRS | S | Jul 20 | 15 min |
| 066 | Change Data Capture & Outbox Pattern | S | Jul 20 | 15 min |
| 067 | Availability Math (SLA/SLO/SLI/Error budgets) | D | Jul 20 | 50 min |
| 068 | Fault Tolerance & Redundancy | S | Jul 21 | 15 min |
| 069 | Timeout, Retry, Exponential Backoff | D | Jul 21 | 50 min |
| 070 | Circuit Breaker & Bulkhead | D | Jul 21 | 50 min |
| 071 | Rate Limiting & Throttling | D | Jul 22 | 60 min |
| 072 | Load Shedding & Backpressure | S | Jul 22 | 15 min |
| 073 | Graceful Degradation & Failover | S | Jul 22 | 15 min |
| 074 | Disaster Recovery (RPO/RTO) | D | Jul 23 | 50 min |
| 075 | Health Checks, Heartbeats, Failure Detection | S | Jul 23 | 15 min |
| 076 | Monitoring & Metrics (RED/USE/4 golden signals) | D | Jul 23 | 50 min |
| 077 | Logging at Scale | S | Jul 24 | 15 min |
| 078 | Distributed Tracing | D | Jul 24 | 50 min |
| 079 | Alerting & On-Call | S | Jul 24 | 15 min |
| 080 | Security Fundamentals | S | Jul 25 | 15 min |
| 081 | Authentication vs Authorization | D | Jul 25 | 50 min |
| 082 | Sessions, Cookies, Tokens, JWT | D | Jul 25 | 50 min |
| 083 | OAuth 2.0 & OIDC | D | Jul 26 | 50 min |
| 084 | Encryption | S | Jul 26 | 15 min |
| 085 | Secrets Management & Certificate Rotation | S | Jul 26 | 15 min |
| 086 | API Security | S | Jul 27 | 15 min |
| 087 | Multi-Tenancy & Data Isolation | S | Jul 27 | 15 min |
| 088 | Privacy, Compliance & Data Residency | S | Jul 27 | 15 min |
| 089 | Bloom Filters | D | Jul 27 | 50 min |
| 090 | HyperLogLog | S | Jul 28 | 15 min |
| 091 | Count-Min Sketch | S | Jul 28 | 15 min |
| 092 | Geospatial Indexing | D | Jul 28 | 50 min |
| 093 | Full-Text Search & Inverted Indexes | S | Jul 28 | 15 min |
| 094 | Elasticsearch / OpenSearch | S | Jul 29 | 15 min |
| 095 | Time-Series Databases | S | Jul 29 | 15 min |
| 096 | Object/Blob Storage | S | Jul 29 | 15 min |
| 097 | Distributed File Systems | S | Jul 29 | 15 min |
| 098 | Unique ID Generation | S | Jul 30 | 15 min |
| 099 | Distributed Locks & Leases | D | Jul 30 | 50 min |
| 100 | Service Discovery & Service Mesh | S | Jul 30 | 15 min |
| 101 | Configuration & Feature Flags at Scale | S | Jul 30 | 15 min |
| 102 | Monolith vs Microservices vs Modular Monolith | D | Jul 31 | 50 min |
| 103 | Service Decomposition & DDD Basics | S | Jul 31 | 15 min |
| 104 | API Gateway & BFF | S | Jul 31 | 15 min |
| 105 | Serverless & FaaS | S | Aug 1 | 15 min |
| 106 | Containers & Docker | S | Aug 1 | 15 min |
| 107 | Orchestration & Kubernetes | S | Aug 1 | 15 min |
| 108 | CI/CD & Deployment Strategies | S | Aug 1 | 15 min |
| 109 | Infrastructure as Code & Environments | S | Aug 2 | 15 min |
| 110 | Cloud Building Blocks | S | Aug 2 | 15 min |
| 111 | The Full HLD Playbook (Capstone Refresher) | D | Aug 2 | 50 min |
| 112 | Estimation Mastery Drill | D | Aug 2 | 50 min |
| 113 | Bottleneck Hunting & Evolution | D | Aug 3 | 50 min |
| 114 | Cost Reasoning in Design | S | Aug 3 | 20 min |

**→ All 114 HLD topics done by: August 3, 2026**

---

## LLD Track — 8 Essential Lessons
*(Parallel track — starts July 19, interleaved with HLD)*

| ID | Title | Type | Target Date | Time |
|----|-------|------|-------------|------|
| L001 | OOP Fundamentals | D | Jul 19 | 45 min |
| L002 | SOLID Principles | D | Jul 21 | 45 min |
| L003 | GRASP Principles | S | Jul 23 | 20 min |
| L004 | UML: Class + Sequence Diagrams | D | Jul 25 | 45 min |
| L008 | Creational Design Patterns | D | Jul 28 | 45 min |
| L009 | Structural Design Patterns | S | Jul 30 | 30 min |
| L010 | Behavioral Design Patterns | D | Aug 1 | 45 min |
| L011 | Concurrency & Thread Safety | D | Aug 3 | 45 min |
| L019 | Machine-Coding Round Playbook | D | Aug 4 | 45 min |

**→ LLD essentials done by: August 4, 2026**

---

## Case Studies — 20 Highest-Signal
*(Starts July 14 — one every 1–2 days alongside HLD)*

| ID | System | Tier | Target Date | Time |
|----|--------|------|-------------|------|
| CS001 | TinyURL / URL Shortener | 1 | Jul 14 | 45 min |
| CS002 | Rate Limiter | 1 | Jul 16 | 45 min |
| CS003 | Notification System | 1 | Jul 18 | 45 min |
| CS004 | Unique ID Generator | 1 | Jul 20 | 45 min |
| CS005 | Key-Value Store / Distributed Cache | 1 | Jul 22 | 45 min |
| CS006 | Web Crawler | 1 | Jul 24 | 45 min |
| CS007 | Leaderboard / Top-K Service | 1 | Jul 26 | 45 min |
| CS008 | Nearby Friends / Proximity Service | 1 | Jul 28 | 45 min |
| CS011 | Twitter / News Feed | 2 | Jul 30 | 60 min |
| CS014 | WhatsApp / Messenger | 2 | Aug 1 | 60 min |
| CS016 | YouTube | 2 | Aug 3 | 60 min |
| CS018 | Dropbox / Google Drive | 2 | Aug 4 | 60 min |
| CS020 | Typeahead / Search Autocomplete | 2 | Aug 5 | 45 min |
| CS023 | BookMyShow / Ticketmaster | 2 | Aug 5 | 60 min |
| CS026 | Uber / Ride Matching | 3 | Aug 6 | 60 min |
| CS029 | Payment Gateway (Stripe-style) | 3 | Aug 6 | 60 min |
| CS028 | Google Maps | 3 | Aug 7 | 60 min |
| CS037 | Metrics & Monitoring (Datadog-style) | 4 | Aug 7 | 60 min |
| CS031 | Stock Exchange / Trading System | 3 | Aug 8 | 60 min |
| CS013 | Facebook News Feed (Fanout deep dive) | 2 | Aug 8 | 60 min |

**→ 20 Case Studies done by: August 8, 2026**

---

## Mock Interviews
| # | Type | Target Date |
|---|------|-------------|
| Mock 1 | Tier 1 system (any CS001–CS008) | Aug 4 |
| Mock 2 | Timed full 45-min (company-flavored) | Aug 7 |
| Mock 3 | Requirement-change round | Aug 9 |

---

## Summary Timeline

```
Jun 28 – Jul  4 : Module 0 + Module 1 (Topics 001–017)       ← YOU ARE HERE
Jul  5 – Jul 11 : Module 2 + Module 3 (Topics 018–037)
Jul 12 – Jul 18 : Module 4 + Module 5 (Topics 038–056)        + CS001–CS003
Jul 19 – Jul 25 : Module 6 + Module 7 (Topics 057–079)        + CS004–CS007 + LLD 1–4
Jul 26 – Aug  2 : Modules 8–10 (Topics 080–110)               + CS008–CS014 + LLD 5–8
Aug  3 – Aug  5 : Module 11 + capstone (Topics 111–114)        + CS015–CS016 + Mock 1
Aug  6 – Aug  9 : Remaining case studies (CS017–CS020)         + Mocks 2–3
Aug  9          : INTERVIEW READY ✅
```

---

## Your Slack Checker

Print this and stick it somewhere visible. At the end of each day, check:

> *"What date is it? What topic should be done by today? Is it done?"*

If you're more than **3 days behind** on any topic — flag it immediately and batch double the next session.
If you're more than **1 week behind** — add one extra session per day until caught up.
