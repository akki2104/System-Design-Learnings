# Progress Tracker

## Dashboard
```
Last Updated    : 2026-08-17
Current Module  : MODULE 4 — Scaling & Distributing Data
Current Topic   : 040 Replication Lag & Read-Your-Writes (Next) — 🔴 MUST, 45 min, Med-Hard
Topics Mastered : 0 / 114 (HLD)   0 / 19 (LLD)   0 / 54 (Case Studies)
Topics Completed: 39 HLD (001-039) — MODULES 1, 2 & 3 COMPLETE, MODULE 4 IN PROGRESS
                  + 2 LLD (L001, L002 — LLD Session 1/6, see tracker below)
Revisions Due   : 013 (+1d, PERSISTENT ×3), 025 (+1d, reset) both due 2026-07-23 — now well overdue. Learner revises on own cadence.
Top Weak Areas  : PERSISTENT (×3, needs mnemonic drill not re-explanation) — 013 gRPC browser blocker (3rd different wrong reason). PERSISTENT (×3, teaching method changed to numeric example) — 015 short-polling latency-vs-waste. PERSISTENT (×2, needs 60s drill next session) — 032 display-vs-transactional staleness. New: 037 L1-cache-as-cluster-node mix-up (attributed structural separation to replication lag; self-corrected on nudge). 036 "Redis fast enough to BE a database" inversion (self-corrected — same family as 032's durable-store trap). 034 hashmap-alone-for-LRU + TTL-refreshes-on-read misconceptions (both corrected). 033 first-pass cache-aside write path omitted cache invalidation (self-corrected). 031 polyglot-persistence-under-pressure gap. Long-standing: reads in write QPS formula. 035, L001, L002: clean passes, no new gaps.

── SEQUENCING TRACK v3 (adopted 2026-08-17) ────────────────────────────
Mode            : Case-study-driven JIT theory (Master Guide §0.2) — supersedes v2's
                  "finish all theory, then case studies" ordering. Teach only a case
                  study's specific missing prerequisites just-in-time (full lessons,
                  full tracking files, committed/pushed), then run the case study
                  learner-driven (Section 10.3). Reversible to v2 sequencing on request.
Trigger         : Aug 17 status check — v2 target date (Aug 18) arrived with case
                  studies 0/14 and mocks 0/3 despite theory ~60% done. Real bottleneck
                  was zero case-study reps, not remaining theory.
In progress     : Pre-TinyURL (Case Study #1) prerequisite batch — 038,039,040,041,
                  042,043,044 (Module 4) + 098 (Unique ID Generation), ~4.75h, before
                  Case Study #1 begins.
Scope reference : TopicPriority.md's tiers (MUST/SKIM/SKIP) and Schedule.md's content
                  (case-study list, time budgets) remain authoritative — v3 changes
                  ORDERING only, not scope.

── COMPRESSED TRACK v2 (adopted 2026-07-25; scope reference under v3) ──
Target Date     : 2026-08-18  (revised from 2026-08-09; +9 days) — effectively passed;
                  see Aug 17 status check. Real remaining work ≈ 44 hours (computed from
                  TopicPriority.md time budgets: ~19.7h MUST theory, ~4.9h SKIM theory,
                  ~11.8h case studies, ~3.8h LLD, ~3.75h mocks). Actual pace since adoption
                  has run ~7x slower than the ~2.25h/day assumed (≈6.9h of lesson content
                  completed in 23 days). No new fixed target date set — v3 tracks by
                  hours-of-work-remaining rather than a calendar date until a real
                  interview-driven deadline is named.
Remaining work  : 35 🔴 must-topics · 26 🟡 skims · 26 ⚫ skipped
                  + 14 case studies + 6 LLD sessions + 3 mocks  ≈ 51 hrs (original estimate)
Priority source : TopicPriority.md — mentor MUST show the Topic Briefing Card
                  (Master Guide §0.1) before every topic. Learner decides
                  full / quick / skip; deviations logged with schedule impact.

LLD Interleave  : (rule added 2026-08-02) Mentor suggests an LLD session ~every 2-3 HLD
  Tracker         topics; learner opts in ("LLD session") or continues HLD — never forced.
                  Pulls from the COMPRESSED 6-session list only (Schedule.md Phase 5:
                  L001+L002, L004, L008, L009+L010, L011, L019), not the full 19-lesson list.
                  Phase 5 (Aug 15-16) remains the default fallback if none are taken early.
                  LLD Sessions Done: 1 / 6 — Session 1 (L001 OOP + L002 SOLID) completed
                  2026-08-13, clean 3/3 pass, confidence 5/5. Next up in order: L004 (UML
                  Class+Sequence). Next suggestion due: ~after Topic 040-041 (2-3 HLD topics
                  from here, currently at 037/038).
────────────────────────────────────────────────────────────────────────

Overall Interview Readiness : 6%

Learner Profile : Some exposure — knows terms like CDN, load balancer; not design-confident yet.
                  Learns best from worked examples + scorecards. Wants every concept tied to
                  named tech choices: which DB/queue, why, and why NOT the alternatives (→ TechChoices.md).
                  Actively audits lesson completeness — asks "is this covered enough / are we
                  missing something" and catches real gaps (e.g. normalization forms, 2026-07-13).
                  PROCESS RULE: Topics/NNN_*.md must be a complete superset of the live chat,
                  never a shortened re-summary — cross-check master guide keywords before finalizing.
Target          : General product-based companies (FAANG-adjacent, to be refined)
Timeline Target : ~4 weeks new content + 2 weeks revision = INTERVIEW READY by ~mid-August 2026

── ACCELERATED TRACK ──────────────────────────────────────────────────
Session Cadence : 5–6 × 90-min sessions per week (15 hrs/week total)
Topics per session: 3–6 topics batched by theme (not one-by-one)
Mastery gate    : Applied only to 30 core high-signal topics (not all 114)
Case Studies    : 20–25 highest-signal ones (not all 54)
LLD             : 8 essential lessons (SOLID + Patterns + Concurrency)

Week 1 : Modules 0–3  (Networking, Storage, Caching)
Week 2 : Modules 4–6  (Scaling, Distributed Systems, Messaging)
Week 3 : Modules 7–10 (Reliability, Security, Building Blocks) + first 8 Case Studies
Week 4 : Module 11 + LLD essentials + 10 more Case Studies + 2 Mock Interviews
Wk 5–6 : Revision + 5 more Case Studies + company-flavored mocks
───────────────────────────────────────────────────────────────────────
```

---

## Session Log
| Date | What Was Covered | What's Next |
|------|-----------------|-------------|
| 2026-06-28 | Bootstrap complete + Topic 001 (Introduction to System Design) — Completed. Key mistakes: NFR vs Solution confusion, FR vs NFR distinction. | Topic 002 — The System Design Interview Framework |
| 2026-06-28 | Topic 002 (The System Design Interview Framework) — Completed. Key mistakes: CAP not a clarifying question; missed channel FR; missed reliability NFR. | Topic 003 — Back-of-the-Envelope Estimation |
| 2026-06-30 | Topic 003 (Back-of-the-Envelope Estimation) — Completed. Got QPS and bandwidth correct on two practice systems. Storage errors: used peak instead of avg QPS; forgot ×365 for multi-year. Implication answers were generic. Confidence: 3/5. | Topics 001 + 002 overdue revisions, then Topic 004 |
| 2026-06-30 | Topic 004 (Non-Functional Requirements) — Completed. Strong tradeoff reasoning (Zomato PM question). Graceful degradation instincts good. Weak: missing p99 percentile on latency NFRs; warm-up NFRs vague before drilling. Confidence: 3/5. | Revisions 001+002+003+004 due tomorrow, then Topic 005 |
| 2026-07-08 | Topic 005 (How to Reason About Tradeoffs) — Completed after a multi-day gap (learner requested TechChoices.md playbook — added to curriculum standard). Correctly applied read/write decision tree to Twitter and WhatsApp. Fixed mid-lesson: conflated MongoDB flexible-schema with Cassandra write-throughput; attributed join cost to data volume instead of sharding. Mastery check passed. Confidence: 3/5. | Revisions overdue (001,002,003,004,005) — batch next session; then Topics 006+007 |
| 2026-07-09 | Topics 006 (Client-Server Model) + 007 (IP, Ports, Sockets) — Completed as survey batch, entering Module 1. Clean understanding of role-asymmetry and failure propagation (006); minor correction on socket 5-tuple mechanics (007 — thought "socket ID" was separate from the tuple). Confidence: 4/5 both. | Revisions still overdue (001-005) — batch soon; then Topic 008 TCP vs UDP |
| 2026-07-09 | Topic 008 (TCP vs UDP) — Completed same session as 006+007. Clean pass on the core decision framework AND the trap question (stock trading system — correctly rejected UDP despite "needs speed" framing). No mistakes logged. Confidence: 4/5. Learner has deferred all revisions (001-007) to this weekend by choice. | Topic 009 (DNS) — continue Module 1; revision catch-up this weekend |
| 2026-07-09 | Topic 009 (DNS) — Completed same session. Clean pass on TTL/migration reasoning, no mistakes logged. Confidence: 4/5. | Topic 010 (HTTP/1.1, HTTP/2, HTTP/3) — continue Module 1; revision catch-up still due this weekend (001-009) |
| 2026-07-09 | Topic 010 (HTTP/1.1, HTTP/2, HTTP/3) — Completed same session (5 topics in one day: 006-010). Correctly traced app-level HOL blocking (HTTP/1.1) and identified the cause; needed the transport-level HOL blocking mechanism (HTTP/2 on TCP) explained fully, then passed mastery check on why HTTP/3 needed a new transport (QUIC). Confidence: 4/5. Git identity for this repo switched to akki2104 (personal GitHub) per learner request — local config only, not global. | Topic 011 (HTTPS & TLS); big revision catch-up (001-010) still due this weekend |
| 2026-07-11 | Revision Blitz — all 10 topics (001-010) covered. Avg vs peak QPS CLEARED. Persistent new weak area: reads in write QPS formula (×2). New mistakes: Port 80/443 swapped; DNS chain missing browser+OS cache; HTTP/3 HoL mechanism confused; Tradeoff steps 1-2 missing; P2P difficulty too vague. Clean passes: TCP/UDP, NFR template, series availability, Postgres read vs write trigger, CAP forbidden questions. Topic 007 updated with port 80/443 reference table. | Topic 011 — HTTPS & TLS |
| 2026-07-12 | Topic 011 (HTTPS & TLS) — Completed. Correctly identified MITM as the HTTP threat and PCI-DSS as the reason for end-to-end TLS on payment APIs. Good grasp of the 3 TLS guarantees and why the handshake switches from asymmetric to symmetric. Didn't know authentication was the third guarantee (required prompting). Confidence: 3-4/5. | Topic 012 — REST API Design |
| 2026-07-12 | Topic 012 (REST API Design) — Completed same session as 011. Strong grasp of statelessness and the cursor-vs-offset pagination correctness argument (self-derived the duplicate/skip mechanism correctly). Corrected: idempotency defined by response code instead of final state; URL design exercise had collection/ID ordering reversed and modeled "like" as a PATCH field instead of a sub-resource. Clean on 401 vs 403. Confidence: 4/5. | Topic 013 — RPC & gRPC |
| 2026-07-12 | Topic 013 (RPC & gRPC) — Completed same session as 011+012. Good grasp of gRPC's HTTP/2 + protobuf combo and the four streaming patterns. Corrected: attributed gRPC's browser incompatibility to protobuf's unreadability (secondary reason) instead of the actual hard blocker — browsers can't control HTTP/2 trailers, while native mobile apps can use gRPC directly. Confidence: 4/5. | Topic 014 — GraphQL |
| 2026-07-12 | Topic 014 (GraphQL) — Completed same session. Learner requested a pacing change: explain the full topic uninterrupted, ask all questions at the end (saved as standing behavioral preference). Correctly identified over-fetching, the caching problem (no fixed URL), and correctly self-diagnosed that GraphQL's N+1 problem moves server-side — but did not know the DataLoader/batching fix, which was explained fully. Confidence: 4/5. | Topic 015 — WebSockets, SSE, Polling, Long Polling |
| 2026-07-13 | Topic 015 (WebSockets, SSE, Polling, Long Polling) — Completed, following the uninterrupted-explanation-then-questions pacing preference. Strong performance on directionality reasoning (SSE vs WebSockets) and the collaborative-editor design question (clean interview-sentence answer). Needed sharpening: short-polling latency-vs-waste mechanism (clients × frequency framing), long-polling's second unsolved limitation (re-request overhead), and the precise reason WebSocket upgrades concern firewalls (protocol Upgrade support, not "no indefinite connections"). Confidence: 4/5. | Topic 016 — Forward Proxy, Reverse Proxy, API Gateway; big revision backlog (001-002,006-014) due later today |
| 2026-07-13 | Topic 016 (Forward Proxy, Reverse Proxy, API Gateway) — Completed same session as 015. Clean 4/4 on all checkpoint questions, including the load-balancer-classification trap and the reverse-proxy-vs-API-Gateway distinction. No mistakes logged. Confidence: 4/5. | Topic 017 — Load Balancers; big revision backlog (001-002,006-015) still due later today |
| 2026-07-14 | Topic 017 (Load Balancers) — Completed. Learner confirmed revision cadence: revises on weekends or whenever he feels enough is covered, not strictly on RevisionSchedule.md due dates (saved as standing preference — backlog will keep being surfaced but not pushed). Clean on health checks (correctly identified passive checks catch functional failures active pings miss) and on why sticky sessions undercut horizontal scaling. Needed clarification: initially conflated IP Hash and Redis-backed sessions as the same solution — clarified they're different mechanisms (routing trick vs true statelessness) with different tradeoffs. Confidence: 4/5. | Topic 018 — CDN |
| 2026-07-14 | Topic 018 (CDN) — Completed same session. Clean 4/4 on all checkpoint questions — correctly tied edge servers back to Topic 016 (reverse proxy) and GeoDNS back to Topic 009, correctly chose Push CDN for a zero-cold-cache-miss launch scenario, and correctly used explicit purge (not TTL wait) for a critical fix. No mistakes logged. Confidence: 4/5. | Topic 019 — Content Compression & Encoding (final Module 1 topic) |
| 2026-07-14 | Topic 019 (Content Compression & Encoding) — Completed same session. MODULE 1 COMPLETE (Topics 006-019). Clean on Accept-Encoding/Content-Encoding negotiation, the "compress once, serve millions" amortization insight, and lossy/lossless distinction with a correct legal-document example. Partial on the video codec trap: correctly flagged CPU/encode cost but missed decode cost (on the USER's device) and compatibility/fallback needs as the other two dimensions. Confidence: 4/5. | Topic 020 — Storage Engine Fundamentals (begins MODULE 2 — Data Storage Foundations) |
| 2026-07-14 | Topic 020 (Storage Engine Fundamentals) — Completed, opening Module 2. Clean on the page-as-I/O-unit mechanism and the precise data-loss distinction between crash-before-fsync vs crash-after-fsync. Correctly explained buffer pool's role but conflated it with checkpointing when asked why data pages are still flushed to disk instead of relying on WAL replay forever — clarified as two separate mechanisms (read speed vs bounded recovery time). Confidence: 4/5. | Topic 021 — Relational Databases & SQL |
| 2026-07-15 | Topic 021 (Relational Databases & SQL) — Completed. PROCESS ISSUE: chat lesson initially covered normalization only as philosophy, skipping the formal 1NF/2NF/3NF forms the master guide explicitly names for this topic — learner proactively asked "is this covered enough," caught the gap, and it was taught in full afterward. Learner then raised a broader, valid concern that chat content sometimes doesn't fully match what lands in Topics/NNN_*.md files. Standing process fix saved to memory: the .md file must be a complete superset of the live chat, never a shortened re-summary; cross-check master guide keywords before finalizing. Once corrected, learner constructed his OWN 2NF/3NF violation examples correctly (stronger signal than reciting memorized ones) and gave clean answers on join cost and the normalize-vs-denormalize practical rule. Confidence: 3-4/5. | Topic 022 — Indexing Deep Dive |
| 2026-07-19 | REVISION BLITZ — full backlog cleared, 18 topics (001-002, 006-021). 13 passed/advanced, 5 RESET to +1 day: 009 (DNS hierarchy reasoning — new miss), 011 (TLS Authentication guarantee — PERSISTENT ×2), 013 (gRPC browser blocker — PERSISTENT ×2, different wrong reason each time), 015 (short-polling latency-vs-waste — PERSISTENT ×2), 017 (active/passive health checks — new miss, explained as speed not failure-type). 019 passed but weak (codec costs 1/3→2/3, improving). Full scorecard in RevisionSchedule.md's blitz block. | Reset revisions (009,011,013,015,017) due 2026-07-20; then Topic 022 — Indexing Deep Dive |
| 2026-07-20 | DATE CORRECTION — learner caught that all dates from Topic 017 onward through the Revision Blitz were mislabeled "2026-07-13" (mentor anchored to a stale date early in this long multi-day chat and never re-verified it as real days passed). Corrected via git commit timestamps: 017-020 → 2026-07-14, 021 → 2026-07-15, Revision Blitz → 2026-07-19. All revision due-dates recalculated from correct anchors. Process fix saved to memory: verify the real system date before writing any date, every time. | Continue with Topic 023 — B-Trees vs LSM-Trees |
| 2026-07-20 | Topic 022 (Indexing Deep Dive) — Completed same session. Excellent performance — clean 4/4 including the bookmark-lookup mechanism (Q2) and the write-heavy indexing trap (Q4, correctly recommended minimal/zero secondary indexes given a 500:1 write:read ratio). Q1's initial answer was correct but light on mechanism (why binary search applies to a B-Tree) — sharpened with the "tree height stays ~log₂(n) regardless of table size" explanation during feedback. No mistakes logged. Confidence: 4-5/5. | Topic 023 — B-Trees vs LSM-Trees |
| 2026-07-20 | Topic 023 (B-Trees vs LSM-Trees) — Completed same session. Core misconception surfaced and corrected: believed SSTables were unsorted, so LSM reads couldn't binary search — corrected that SSTables ARE sorted (the name says so), and the real read cost is checking MULTIPLE sorted structures, not losing sorting within one. Q1 (B-Tree random writes) only half-answered initially, missing why LSM avoids the same cost. Q4 (LSM not "strictly better") was clean and correct on the first try. Confidence: 3/5. | Topic 024 — Transactions & ACID |
| 2026-07-20 | Topic 024 (Transactions & ACID) — Completed same session. Clean 4/4 pass — correctly distinguished ACID Consistency from CAP Consistency (a common interview trap), correctly identified Durability as literally the WAL mechanism from Topic 020 rather than something new, and gave well-reasoned answer on when to skip transactions (analytics/logging). No mistakes logged. Confidence: 4-5/5. | Topic 025 — Isolation Levels & Anomalies |
| 2026-07-20 | Topic 025 (Isolation Levels & Anomalies) — Completed same session. Correctly identified the minimum isolation level (Repeatable Read) for a non-repeatable-read scenario and correctly reasoned through why Read Committed doesn't prevent a double-spend race (named it as "both commit before either reads the other's write") — but initially named the anomaly by the isolation level that permits it instead of its actual name; corrected to Non-Repeatable Read and additionally taught Lost Update (the anomaly the bank-transfer race actually is) as a superset addition beyond the master guide's three named anomalies, since it directly completes the reasoning. Confidence: 4/5. | Topic 026 — Concurrency Control: Locks, 2PL, Deadlocks |
| 2026-07-22 | Revision session — 9 topics (009,011,013,015,017,022,023,024,025). 7 passed/advanced (011 and 017 RESOLVED after being persistent); 2 reset: 013 (gRPC browser blocker, PERSISTENT ×3, needs mnemonic drill not re-explanation), 025 (1st revision, failed both sub-questions). | Reset revisions (013,015,025) due 2026-07-23; then Topic 026 |
| 2026-07-25 | Topic 026 (Concurrency Control: Locks, 2PL, Deadlocks) — Completed. Correctly explained Strict 2PL's cascading-rollback rationale and the flexibility-vs-safety shape of the detection-vs-prevention tradeoff. Corrected: proposed a shared-lock-then-upgrade sequence for preventing Lost Update, which actually risks a lock-upgrade deadlock (fix: exclusive lock at read time via SELECT...FOR UPDATE); also said deadlock detection "causes" more deadlocks than prevention (corrected — deadlock occurrence is timing-dependent, not strategy-dependent). Confidence: 4/5. | Topic 027 — MVCC |
| 2026-07-25 | Topic 027 (MVCC) — Completed same session. Strong pass, no mistakes logged. Correctly explained why readers/writers never block each other (different physical row versions), correctly reasoned through why Repeatable Read's fixed per-transaction snapshot prevents re-reads from seeing concurrent commits, and correctly identified (with a request for elaboration) that MVCC alone doesn't prevent Lost Update — Postgres's extra write-conflict detection does the actual work depending on isolation level. Confidence: 4/5. Closes out the Topics 020/024/025/026/027 transactions-and-concurrency arc. | Topic 028 — NoSQL Overview |
| 2026-07-25 | COMPRESSED TRACK v2 ADOPTED — new TopicPriority.md created (tier system: 🔴 MUST/🟡 SKIM/⚫ SKIP for all remaining topics), target date revised 2026-08-09 → 2026-08-18, case studies trimmed 54→14, LLD trimmed 19→6 sessions. New process requirement: show a Topic Briefing Card (tier/time/complexity/why) before every topic going forward. | Topic 028 — NoSQL Overview, with Briefing Card shown first |
| 2026-07-26 | Topic 028 (NoSQL Overview) — Completed. First topic taught under the new Briefing Card protocol (🔴 MUST, 40m, Medium). Clean pass on all 3 checkpoints: correctly identified "write-heavy → MongoDB" as a category error (Wide-Column is the actual fit), correctly named leaderless/multi-master replication as the real mechanism behind NoSQL write throughput (not "the data model"), and correctly defined schema-on-write vs schema-on-read. No mistakes logged. Confidence: 4-5/5. | Topic 029 — Wide-Column Stores |
| 2026-07-26 | Topic 029 (Wide-Column Stores) — Completed same session (🔴 MUST, 45m, Med-Hard). Excellent pass — precisely distinguished "leaderless replication" (any replica of the specific partition's replica set) from "any node in the cluster," correctly reasoned through why LSM-Tree and leaderless replication are independently necessary (losing either reintroduces a specific known cost), and correctly concluded two tables are needed for two access patterns (sharpened with the underlying reason: a partition key can only efficiently serve one query shape). No mistakes logged. Confidence: 4-5/5. Third clean lesson in a row. | Topic 030 — Document Stores (🟡 SKIM) |
| 2026-07-26 | Topic 030 (Document Stores) — Completed same session (🟡 SKIM tier — shaped live-session pace only). Learner corrected mentor's initial compromise: gave a condensed Topics file and skipped the Revision file, citing tier. Learner clarified tiers are for HIS calibration of attention/time, never a reason to shorten documentation — standing rule saved to memory. Topic 030 was then rebuilt to full depth: nesting vs flat columns, flexible secondary indexing, embed-vs-reference modeling decision, and MongoDB's leader-follower (NOT leaderless) replication model as a genuine contrast with Cassandra. Full revision file created. | Topic 031 — Choosing a Database |
| 2026-07-30 | Topic 031 (Choosing a Database) — Completed (🔴 MUST, 45m, Medium). MODULE 2 COMPLETE (Topics 020-031). Delivered the 6-question decision framework, decision tree, polyglot persistence, and interview justification format, then posed a practice scenario (real-time collaborative document editor). Learner correctly rejected Wide-Column with good reasoning, but defaulted to using ONE document store for both content storage AND full-text search — the exact polyglot-persistence trap just taught, showing that knowing a principle and applying it under pressure are different skills. Also missed that the system's actual hardest problem (concurrent edit conflict resolution) isn't a database choice at all. Both corrected in detail. Confidence: 4/5. | Topic 032 — Caching Fundamentals (begins MODULE 3 — Caching) |
| 2026-07-31 | Topic 032 (Caching Fundamentals) — Completed (🔴 MUST, 35m, Easy), opening MODULE 3 — Caching. Covered hit/miss/hit-ratio, why a cache hit beats even a well-indexed DB query (DB overhead — parsing/locking/MVCC checks/WAL — not just RAM-vs-disk), the invalidation problem (TTL vs explicit), good/bad caching candidates, and cache stampede. Learner correctly rejected the shopping cart as a caching candidate and correctly identified stale pricing as unacceptable, with the right fix (explicit invalidation) — but reasoning leaned on UX vibes rather than naming the specific framework criteria (read:write ratio, read-your-own-write), and treated "acceptable staleness" as one fixed property instead of splitting by display-read vs transactional-read use. Both sharpened in feedback. Confidence: 4/5. | Topic 033 — Caching Patterns (cache-aside, write-through, write-behind) |
| 2026-07-31 | Topic 032 merged (parallel-session lesson combined with this session's version — no content lost) and Topic 033 (Caching Patterns) completed in a parallel session (🔴 MUST, per commit history). Covered cache-aside/write-through/write-behind mechanics; first-pass answer on the cache-aside write path initially omitted cache invalidation, self-corrected within the same session when reconciled against a later answer. A master-guide clarification was also added this session: file state showing a topic complete (e.g. from a parallel session) is never a substitute for this session grading its own live checkpoint answers before advancing — MUST-topic no-confirmation-to-start is separate from the Mastery Check gate. | Topic 034 — Eviction Policies |
| 2026-08-02 | Topic 034 (Eviction Policies) — Completed (🟡 SKIM, 8m, Easy — full documentation depth maintained per standing rule). Covered LRU (hashmap+doubly-linked-list mechanism), LFU (stale-popularity blind spot), FIFO, and TTL vs eviction-policy as orthogonal mechanisms. Two corrections needed: (1) claimed a hashmap alone could mimic LRU — corrected that ordering requires the linked list, hashmap alone can't identify "least recently used" without an O(n) scan; (2) assumed TTL automatically refreshes on every read — corrected that standard TTL is a fixed clock from insertion, sliding/refresh-on-read TTL is opt-in, not default. Also asked a good clarifying question (how does "cache fills up" apply to one repeatedly-read item) — answered: it's a global-capacity event across many distinct keys, not this item's own re-reads. Confidence: 4/5. Status check this session also surfaced the schedule gap: Schedule.md v2 expected ~056-069 by 2026-08-02; actual position was 033/034 — flagged as BEHIND in the dashboard (updated from the previously-stale "ON TRACK" claim). | Topic 035 — Cache Problems (stampede, penetration, avalanche) |
| 2026-08-02 | Topic 035 (Cache Problems) — Completed same session (🔴 MUST, 45m, Med-Hard). Covered the three distinct failure modes and their triggers: cache penetration (key never existed in cache or DB — fix: cache negative results + Bloom filter), cache stampede (one hot key expires under concurrent load — fix: mutex/lock or logical/soft expiration), and cache avalanche (mass simultaneous TTL expiry, or a total cache-cluster outage — fix: jittered TTLs for the former, HA cache cluster + circuit breaker + multi-tier caching for the latter). Clean 3/3 pass — correctly classified a gaming-leaderboard scenario as avalanche with the jitter fix, correctly explained the Bloom filter's one-sided error (false positives possible, never false negatives) as why it's a penetration-only fix, and correctly explained why logical expiration avoids the blocking downside of a plain mutex. No mistakes logged. Confidence: 5/5. | Topic 036 — Distributed Caching with Redis/Memcached |
| 2026-08-04 to 2026-08-06 | Topics 036 (Distributed Caching with Redis/Memcached, completed 2026-08-04) and 037 (Cache Consistency & Invalidation, completed 2026-08-06) — completed in parallel sessions, closing out MODULE 3 — Caching (032-037). Topic 034 also got a small addendum this window: naming Redis's actual `maxmemory-policy` config values (allkeys-lru, allkeys-lfu, volatile-ttl, etc.) as a stronger interview answer than "I'd evict old stuff." New weak areas logged: 036 briefly inverted "Redis is fast enough to act like a database" into treating it as a durable primary store (self-corrected, same family as Topic 032's durable-store trap); 037 mixed up an in-process L1 cache with a distributed cluster node, attributing their structural separation to replication lag instead of physical placement (self-corrected on a nudge). | Topic 038 — Vertical vs Horizontal Scaling (begins MODULE 4 — Scaling & Distributing Data) |
| 2026-08-13 | Status check surfaced the widening schedule gap (Schedule.md v2 Phase 4 expected case studies starting today; actual position still Topic 038/theory) — flagged, not yet acted on. Learner then opted to take the standing LLD-interleave suggestion: **LLD Session 1/6 — L001 (OOP Fundamentals) + L002 (SOLID Principles)**, per the compressed 6-session list (🔴 MUST, ~50m combined). Covered all four OOP pillars (with the encapsulation-vs-abstraction distinction emphasized) and all five SOLID letters, tying each back to the OOP pillar it depends on (O↔polymorphism, D↔abstraction, L↔inheritance-soundness-check). Clean 3/3 pass: correctly identified a RubberDuck.fly()-override as a Liskov violation, correctly split a dual-purpose ReportGenerator via Single Responsibility (with a forward-looking generic sender name), and produced a strong self-constructed parking-lot example distinguishing encapsulation from abstraction. No mistakes logged. Confidence: 5/5. | Topic 038 — Vertical vs Horizontal Scaling; next LLD session (L004 UML) suggested after ~2-3 more HLD topics |
| 2026-07-22 | BIG REVISION SESSION — 9 overdue topics covered (009, 011, 013, 015, 017, 022, 023, 024, 025), following the new "cover the full topic, not just weak points" protocol. Two real wins: 011's TLS-guarantee gap and 017's health-check-mechanism gap both RESOLVED after persisting across 2 prior attempts. Two topics remain PERSISTENT and now need a different remediation approach: 013's gRPC browser blocker has produced a 3rd distinct wrong reason across 3 sessions — flagging for a mnemonic-first drill instead of more conceptual re-teaching. 015's short-polling latency-vs-waste mechanism failed a 3rd time with the same shallow answer — switched teaching method to a concrete numeric walkthrough (10,000 clients × poll-interval change) rather than re-explaining the concept abstractly; retest with a similarly concrete question next time. New minor gaps: 009 still missing "management" as DNS's 3rd hierarchy reason; 017 forgot the basic L4 vs L7 distinction; 025 (first revision) got both sub-questions wrong — non-repeatable-vs-phantom distinguishing axis, and the isolation-level decision framing was too vague — RESET to +1 day. 022, 023, 024 all passed clean. | Topic 026 — Concurrency Control: Locks, 2PL, Deadlocks; retest 013/015 with new approaches next revision cycle |

---

## HLD Topics

| ID | Title | Status | Start | Completed | Mastered | Rev# | Last Rev | Next Due | Conf | Diff | Weak Areas |
|----|-------|--------|-------|-----------|----------|------|----------|----------|------|------|------------|
| 001 | Introduction to System Design | Completed | 2026-06-28 | 2026-06-28 | — | 4 | 2026-07-19 | 2026-08-18 | 4 | Easy | Blitz pass — minor: said "traceability" instead of "Traces" |
| 002 | The System Design Interview Framework | Completed | 2026-06-28 | 2026-06-28 | — | 4 | 2026-07-19 | 2026-08-18 | 4 | Easy | Blitz pass — clean |
| 003 | Back-of-the-Envelope Estimation | Completed | 2026-06-30 | 2026-06-30 | — | 3 | 2026-07-11 | 2026-07-15 | 2 | Easy-Med | Reads in write QPS formula (RECURRING ×2 — persistent); avg vs peak CLEARED |
| 004 | Non-Functional Requirements | Completed | 2026-06-30 | 2026-06-30 | — | 3 | 2026-07-11 | 2026-07-15 | 3 | Medium | Missing p99 percentile on latency; initial NFRs were vague |
| 005 | How to Reason About Tradeoffs | Completed | 2026-07-08 | 2026-07-08 | — | 2 | 2026-07-11 | 2026-07-15 | 3 | Medium | Tradeoff method steps 1-2 missing in revision; Postgres/write confusion fully corrected |
| 006 | The Client–Server Model | Completed | 2026-07-09 | 2026-07-09 | — | 2 | 2026-07-19 | 2026-08-03 | 4 | Easy | Blitz pass — clean |
| 007 | IP, Ports, Sockets | Completed | 2026-07-09 | 2026-07-09 | — | 2 | 2026-07-19 | 2026-08-03 | 4 | Easy | Blitz pass — clean (5-tuple recalled precisely) |
| 008 | TCP vs UDP | Completed | 2026-07-09 | 2026-07-09 | — | 2 | 2026-07-19 | 2026-08-03 | 4 | Medium | Blitz pass (soft) — named right dimensions, missed precise "obsolete anyway" framing |
| 009 | DNS | Completed | 2026-07-09 | 2026-07-09 | — | 2 | 2026-07-22 | 2026-07-29 | 3 | Easy | Improved — named scale+availability correctly, still missing "management" as 3rd reason |
| 010 | HTTP/1.1, HTTP/2, HTTP/3 | Completed | 2026-07-09 | 2026-07-09 | — | 2 | 2026-07-19 | 2026-08-03 | 4 | Med-Hard | Blitz pass — clean, HOL blocking mechanism solid |
| 011 | HTTPS & TLS | Completed | 2026-07-12 | 2026-07-12 | — | 1 | 2026-07-22 | 2026-07-25 | 4 | Medium | RESOLVED — all 3 TLS guarantees recalled cleanly, no longer persistent |
| 012 | REST API Design | Completed | 2026-07-12 | 2026-07-12 | — | 1 | 2026-07-19 | 2026-07-22 | 4 | Medium | Blitz pass — clean cursor-pagination reasoning |
| 013 | RPC & gRPC | Completed | 2026-07-12 | 2026-07-12 | — | 0 | 2026-07-22 (FAILED) | 2026-07-23 | 3 | Medium | RESET — gRPC browser blocker wrong for a 3RD different reason. PERSISTENT ×3 — needs mnemonic drill, not re-explanation |
| 014 | GraphQL | Completed | 2026-07-12 | 2026-07-12 | — | 1 | 2026-07-19 | 2026-07-22 | 4 | Medium | Blitz pass — clean N+1/DataLoader recall |
| 015 | WebSockets, SSE, Polling, Long Polling | Completed | 2026-07-13 | 2026-07-13 | — | 0 | 2026-07-19 (FAILED) | 2026-07-20 (TODAY) | 3 | Medium | RESET — short-polling latency-vs-waste mechanism wrong AGAIN. PERSISTENT ×2, needs dedicated drill. |
| 016 | Forward Proxy, Reverse Proxy, API Gateway | Completed | 2026-07-13 | 2026-07-13 | — | 1 | 2026-07-19 | 2026-07-22 | 4 | Easy | Blitz pass — clean |
| 017 | Load Balancers | Completed | 2026-07-14 | 2026-07-14 | — | 1 | 2026-07-22 | 2026-07-25 | 4 | Medium | RESOLVED — health check distinction now correct; NEW minor gap: forgot L4 vs L7 core difference |
| 018 | CDN | Completed | 2026-07-14 | 2026-07-14 | — | 1 | 2026-07-19 | 2026-07-22 | 4 | Easy | Blitz pass — clean |
| 019 | Content Compression & Encoding | Completed | 2026-07-14 | 2026-07-14 | — | 1 | 2026-07-19 | 2026-07-22 | 4 | Easy | Blitz pass (weak) — codec costs improved 1/3→2/3, decode-on-device still missing, watch next pass |
| 020 | Storage Engine Fundamentals | Completed | 2026-07-14 | 2026-07-14 | — | 1 | 2026-07-19 | 2026-07-22 | 4 | Medium | Blitz pass — clean checkpointing/WAL reasoning |
| 021 | Relational Databases & SQL | Completed | 2026-07-15 | 2026-07-15 | — | 1 | 2026-07-19 | 2026-07-22 | 4 | Med-Hard | Blitz pass — excellent self-constructed 3NF example |
| 022 | Indexing Deep Dive | Completed | 2026-07-20 | 2026-07-20 | — | 1 | 2026-07-22 | 2026-07-25 | 4-5 | Med-Hard | Revision pass clean — bookmark-lookup mechanism solid |
| 023 | B-Trees vs LSM-Trees | Completed | 2026-07-20 | 2026-07-20 | — | 1 | 2026-07-22 | 2026-07-25 | 4 | Medium | Revision pass clean — LSM read-stopping reasoning solid |
| 024 | Transactions & ACID | Completed | 2026-07-20 | 2026-07-20 | — | 1 | 2026-07-22 | 2026-07-25 | 4-5 | Med-Hard | Revision pass clean — ACID-vs-CAP consistency and WAL rollback both solid |
| 025 | Isolation Levels & Anomalies | Completed | 2026-07-20 | 2026-07-20 | — | 0 | 2026-07-22 (FAILED) | 2026-07-23 | 3 | Med-Hard | RESET (1st revision) — non-repeatable vs phantom distinguishing axis wrong; isolation-level decision framing too vague |
| 026 | Concurrency Control: Locks, 2PL, Deadlocks | Completed | 2026-07-25 | 2026-07-25 | — | 0 | — | 2026-07-26 | 4 | Med-Hard | Proposed shared-then-upgrade locking for Lost Update (risks lock-upgrade deadlock; corrected to X-lock at read time); said detection "causes" more deadlocks than prevention (corrected — timing-dependent, not strategy-dependent) |
| 027 | MVCC | Completed | 2026-07-25 | 2026-07-25 | — | 0 | — | 2026-07-26 | 4 | Med-Hard | None — clean pass, correctly self-identified that MVCC alone doesn't prevent Lost Update and asked for the completing mechanism (Postgres's isolation-level-dependent write-conflict detection) |
| 028 | NoSQL Overview | Completed | 2026-07-26 | 2026-07-26 | — | 0 | — | 2026-07-27 | 4-5 | Medium | None — clean pass on all 3 checkpoints (write-heavy category error, leaderless replication mechanism, schema-on-write vs schema-on-read) |
| 029 | Wide-Column Stores | Completed | 2026-07-26 | 2026-07-26 | — | 0 | — | 2026-07-27 | 4-5 | Med-Hard | None — clean pass on all 3 checkpoints (leaderless precision, LSM+replication interdependence, partition-key/one-table-per-query reasoning) |
| 030 | Document Stores | Completed | 2026-07-26 | 2026-07-26 | — | 0 | — | 2026-07-27 | 4 | Easy | Full lesson + revision file (tier only shaped live-session pace, not documentation depth). Clean understanding of nesting, flexible secondary indexing, embed-vs-reference, and MongoDB's leader-follower (not leaderless) replication model |
| 031 | Choosing a Database | Completed | 2026-07-30 | 2026-07-30 | — | 0 | — | 2026-07-31 | 4 | Medium | Practice scenario: used one document store for both content storage AND full-text search (polyglot persistence not applied under pressure, despite just learning it); missed that the system's hardest problem (concurrent edit conflict resolution) isn't a DB choice at all — both corrected |
| 032 | Caching Fundamentals | Completed | 2026-07-31 | 2026-07-31 | — | 0 | — | 2026-08-01 | 4 | Easy | Correct on cart rejection and stale-price call, but reasoning leaned on UX vibes over named criteria (read:write ratio, read-your-own-write); also treated staleness acceptability as fixed rather than display-vs-transactional-read dependent — both sharpened |
| 033 | Caching Patterns | Completed | 2026-07-31 | 2026-07-31 | — | 0 | — | 2026-08-01 | 4 | Medium | First-pass cache-aside write path omitted cache invalidation, self-corrected same session |
| 034 | Eviction Policies | Completed | 2026-08-02 | 2026-08-02 | — | 0 | — | 2026-08-03 | 4 | Easy | Claimed hashmap alone could mimic LRU (missed the linked-list ordering role); assumed TTL auto-refreshes on read (it doesn't by default) — both corrected |
| 035 | Cache Problems | Completed | 2026-08-02 | 2026-08-02 | — | 0 | — | 2026-08-03 | 5 | Med-Hard | None — clean pass on all 3 (avalanche classification, Bloom filter one-sided-error reasoning, logical-expiration-vs-mutex distinction) |
| 036 | Distributed Caching with Redis/Memcached | Not Started | — | — | — | 0 | — | — | — | — | — |
| 037 | Cache Consistency & Invalidation | Not Started | — | — | — | 0 | — | — | — | — | — |
| 038 | Vertical vs Horizontal Scaling | Not Started | — | — | — | 0 | — | — | — | — | — |
| 039 | Replication | Not Started | — | — | — | 0 | — | — | — | — | — |
| 040 | Replication Lag & Read Consistency | Not Started | — | — | — | 0 | — | — | — | — | — |
| 041 | Partitioning & Sharding | Not Started | — | — | — | 0 | — | — | — | — | — |
| 042 | Consistent Hashing | Not Started | — | — | — | 0 | — | — | — | — | — |
| 043 | Choosing a Shard Key | Not Started | — | — | — | 0 | — | — | — | — | — |
| 044 | Rebalancing & Resharding | Not Started | — | — | — | 0 | — | — | — | — | — |
| 045 | Why Distributed Systems Are Hard | Not Started | — | — | — | 0 | — | — | — | — | — |
| 046 | CAP Theorem | Not Started | — | — | — | 0 | — | — | — | — | — |
| 047 | PACELC | Not Started | — | — | — | 0 | — | — | — | — | — |
| 048 | Consistency Models | Not Started | — | — | — | 0 | — | — | — | — | — |
| 049 | Time, Clocks & Ordering | Not Started | — | — | — | 0 | — | — | — | — | — |
| 050 | Quorums | Not Started | — | — | — | 0 | — | — | — | — | — |
| 051 | Consensus & Leader Election | Not Started | — | — | — | 0 | — | — | — | — | — |
| 052 | Raft | Not Started | — | — | — | 0 | — | — | — | — | — |
| 053 | Paxos | Not Started | — | — | — | 0 | — | — | — | — | — |
| 054 | Distributed Transactions: 2PC & 3PC | Not Started | — | — | — | 0 | — | — | — | — | — |
| 055 | Sagas & Compensating Transactions | Not Started | — | — | — | 0 | — | — | — | — | — |
| 056 | Idempotency & Exactly-Once Semantics | Not Started | — | — | — | 0 | — | — | — | — | — |
| 057 | Conflict Resolution & CRDTs | Not Started | — | — | — | 0 | — | — | — | — | — |
| 058 | Why Asynchronous Processing | Not Started | — | — | — | 0 | — | — | — | — | — |
| 059 | Message Queues | Not Started | — | — | — | 0 | — | — | — | — | — |
| 060 | Log-Based Streaming (Kafka) | Not Started | — | — | — | 0 | — | — | — | — | — |
| 061 | Delivery Guarantees | Not Started | — | — | — | 0 | — | — | — | — | — |
| 062 | Stream Processing | Not Started | — | — | — | 0 | — | — | — | — | — |
| 063 | Event-Driven Architecture | Not Started | — | — | — | 0 | — | — | — | — | — |
| 064 | Event Sourcing | Not Started | — | — | — | 0 | — | — | — | — | — |
| 065 | CQRS | Not Started | — | — | — | 0 | — | — | — | — | — |
| 066 | Change Data Capture & Outbox Pattern | Not Started | — | — | — | 0 | — | — | — | — | — |
| 067 | Availability Math (SLA/SLO/SLI) | Not Started | — | — | — | 0 | — | — | — | — | — |
| 068 | Fault Tolerance & Redundancy | Not Started | — | — | — | 0 | — | — | — | — | — |
| 069 | Timeout, Retry, Backoff | Not Started | — | — | — | 0 | — | — | — | — | — |
| 070 | Circuit Breaker & Bulkhead | Not Started | — | — | — | 0 | — | — | — | — | — |
| 071 | Rate Limiting & Throttling | Not Started | — | — | — | 0 | — | — | — | — | — |
| 072 | Load Shedding & Backpressure | Not Started | — | — | — | 0 | — | — | — | — | — |
| 073 | Graceful Degradation & Failover | Not Started | — | — | — | 0 | — | — | — | — | — |
| 074 | Disaster Recovery (RPO/RTO) | Not Started | — | — | — | 0 | — | — | — | — | — |
| 075 | Health Checks, Heartbeats, Failure Detection | Not Started | — | — | — | 0 | — | — | — | — | — |
| 076 | Monitoring & Metrics | Not Started | — | — | — | 0 | — | — | — | — | — |
| 077 | Logging at Scale | Not Started | — | — | — | 0 | — | — | — | — | — |
| 078 | Distributed Tracing | Not Started | — | — | — | 0 | — | — | — | — | — |
| 079 | Alerting & On-Call | Not Started | — | — | — | 0 | — | — | — | — | — |
| 080 | Security Fundamentals | Not Started | — | — | — | 0 | — | — | — | — | — |
| 081 | Authentication vs Authorization | Not Started | — | — | — | 0 | — | — | — | — | — |
| 082 | Sessions, Cookies, Tokens, JWT | Not Started | — | — | — | 0 | — | — | — | — | — |
| 083 | OAuth 2.0 & OIDC | Not Started | — | — | — | 0 | — | — | — | — | — |
| 084 | Encryption | Not Started | — | — | — | 0 | — | — | — | — | — |
| 085 | Secrets Management & Certificate Rotation | Not Started | — | — | — | 0 | — | — | — | — | — |
| 086 | API Security | Not Started | — | — | — | 0 | — | — | — | — | — |
| 087 | Multi-Tenancy & Data Isolation | Not Started | — | — | — | 0 | — | — | — | — | — |
| 088 | Privacy, Compliance & Data Residency | Not Started | — | — | — | 0 | — | — | — | — | — |
| 089 | Bloom Filters | Not Started | — | — | — | 0 | — | — | — | — | — |
| 090 | HyperLogLog | Not Started | — | — | — | 0 | — | — | — | — | — |
| 091 | Count-Min Sketch | Not Started | — | — | — | 0 | — | — | — | — | — |
| 092 | Geospatial Indexing | Not Started | — | — | — | 0 | — | — | — | — | — |
| 093 | Full-Text Search & Inverted Indexes | Not Started | — | — | — | 0 | — | — | — | — | — |
| 094 | Elasticsearch / OpenSearch | Not Started | — | — | — | 0 | — | — | — | — | — |
| 095 | Time-Series Databases | Not Started | — | — | — | 0 | — | — | — | — | — |
| 096 | Object/Blob Storage | Not Started | — | — | — | 0 | — | — | — | — | — |
| 097 | Distributed File Systems | Not Started | — | — | — | 0 | — | — | — | — | — |
| 098 | Unique ID Generation | Not Started | — | — | — | 0 | — | — | — | — | — |
| 099 | Distributed Locks & Leases | Not Started | — | — | — | 0 | — | — | — | — | — |
| 100 | Service Discovery & Service Mesh | Not Started | — | — | — | 0 | — | — | — | — | — |
| 101 | Configuration & Feature Flags at Scale | Not Started | — | — | — | 0 | — | — | — | — | — |
| 102 | Monolith vs Microservices vs Modular Monolith | Not Started | — | — | — | 0 | — | — | — | — | — |
| 103 | Service Decomposition & DDD Basics | Not Started | — | — | — | 0 | — | — | — | — | — |
| 104 | API Gateway & BFF | Not Started | — | — | — | 0 | — | — | — | — | — |
| 105 | Serverless & FaaS | Not Started | — | — | — | 0 | — | — | — | — | — |
| 106 | Containers & Docker | Not Started | — | — | — | 0 | — | — | — | — | — |
| 107 | Orchestration & Kubernetes | Not Started | — | — | — | 0 | — | — | — | — | — |
| 108 | CI/CD & Deployment Strategies | Not Started | — | — | — | 0 | — | — | — | — | — |
| 109 | Infrastructure as Code & Environments | Not Started | — | — | — | 0 | — | — | — | — | — |
| 110 | Cloud Building Blocks | Not Started | — | — | — | 0 | — | — | — | — | — |
| 111 | The Full HLD Playbook (Capstone Refresher) | Not Started | — | — | — | 0 | — | — | — | — | — |
| 112 | Estimation Mastery Drill | Not Started | — | — | — | 0 | — | — | — | — | — |
| 113 | Bottleneck Hunting & Evolution | Not Started | — | — | — | 0 | — | — | — | — | — |
| 114 | Cost Reasoning in Design | Not Started | — | — | — | 0 | — | — | — | — | — |

---

## LLD Topics

| ID | Title | Status | Start | Completed | Mastered | Rev# | Last Rev | Next Due | Conf | Diff | Weak Areas |
|----|-------|--------|-------|-----------|----------|------|----------|----------|------|------|------------|
| L001 | OOP Fundamentals | Completed | 2026-08-13 | 2026-08-13 | — | 0 | — | 2026-08-14 | 5 | Easy | None — clean pass, strong self-constructed encapsulation-vs-abstraction example (parking lot) |
| L002 | SOLID Principles | Completed | 2026-08-13 | 2026-08-13 | — | 0 | — | 2026-08-14 | 5 | Medium | None — clean pass on Liskov violation, SRP split with forward-looking generic sender name |
| L003 | GRASP Principles | Not Started | — | — | — | 0 | — | — | — | — | — |
| L004 | UML: Class Diagrams | Not Started | — | — | — | 0 | — | — | — | — | — |
| L005 | UML: Sequence Diagrams | Not Started | — | — | — | 0 | — | — | — | — | — |
| L006 | UML: State & Activity Diagrams | Not Started | — | — | — | 0 | — | — | — | — | — |
| L007 | Design Principles (DRY, KISS, YAGNI…) | Not Started | — | — | — | 0 | — | — | — | — | — |
| L008 | Creational Patterns | Not Started | — | — | — | 0 | — | — | — | — | — |
| L009 | Structural Patterns | Not Started | — | — | — | 0 | — | — | — | — | — |
| L010 | Behavioral Patterns | Not Started | — | — | — | 0 | — | — | — | — | — |
| L011 | Concurrency & Thread Safety | Not Started | — | — | — | 0 | — | — | — | — | — |
| L012 | Concurrency Patterns | Not Started | — | — | — | 0 | — | — | — | — | — |
| L013 | Domain & Object Modeling | Not Started | — | — | — | 0 | — | — | — | — | — |
| L014 | Dependency Injection & IoC | Not Started | — | — | — | 0 | — | — | — | — | — |
| L015 | Clean Code & Code Smells | Not Started | — | — | — | 0 | — | — | — | — | — |
| L016 | Refactoring Techniques | Not Started | — | — | — | 0 | — | — | — | — | — |
| L017 | Testing & Testability | Not Started | — | — | — | 0 | — | — | — | — | — |
| L018 | Conducting & Surviving Design Reviews | Not Started | — | — | — | 0 | — | — | — | — | — |
| L019 | The Machine-Coding Round Playbook | Not Started | — | — | — | 0 | — | — | — | — | — |

---

## Case Studies

| ID | Title | Tier | Status | Completed | Mastered | Conf | Notes |
|----|-------|------|--------|-----------|----------|------|-------|
| CS001 | TinyURL / URL Shortener | 1 | Not Started | — | — | — | — |
| CS002 | Pastebin | 1 | Not Started | — | — | — | — |
| CS003 | Rate Limiter | 1 | Not Started | — | — | — | — |
| CS004 | Key-Value Store / Distributed Cache | 1 | Not Started | — | — | — | — |
| CS005 | Unique ID Generator | 1 | Not Started | — | — | — | — |
| CS006 | Web Crawler | 1 | Not Started | — | — | — | — |
| CS007 | Notification System | 1 | Not Started | — | — | — | — |
| CS008 | Consistent Hashing Service | 1 | Not Started | — | — | — | — |
| CS009 | Nearby Friends / Proximity Service | 1 | Not Started | — | — | — | — |
| CS010 | Leaderboard / Top-K Service | 1 | Not Started | — | — | — | — |
| CS011 | Twitter/X Timeline & News Feed | 2 | Not Started | — | — | — | — |
| CS012 | Instagram | 2 | Not Started | — | — | — | — |
| CS013 | Facebook News Feed (Fanout Deep Dive) | 2 | Not Started | — | — | — | — |
| CS014 | WhatsApp / Messenger | 2 | Not Started | — | — | — | — |
| CS015 | Discord / Slack | 2 | Not Started | — | — | — | — |
| CS016 | YouTube | 2 | Not Started | — | — | — | — |
| CS017 | Netflix | 2 | Not Started | — | — | — | — |
| CS018 | Dropbox / Google Drive | 2 | Not Started | — | — | — | — |
| CS019 | Google Docs (Collaborative Editing) | 2 | Not Started | — | — | — | — |
| CS020 | Typeahead / Search Autocomplete | 2 | Not Started | — | — | — | — |
| CS021 | Search Engine | 2 | Not Started | — | — | — | — |
| CS022 | E-commerce / Amazon Product System | 2 | Not Started | — | — | — | — |
| CS023 | BookMyShow / Ticketmaster | 2 | Not Started | — | — | — | — |
| CS024 | Hotel/Flight Booking (Airbnb-style) | 2 | Not Started | — | — | — | — |
| CS025 | Spotify | 2 | Not Started | — | — | — | — |
| CS026 | Uber / Lyft (Ride Matching) | 3 | Not Started | — | — | — | — |
| CS027 | DoorDash / Food Delivery | 3 | Not Started | — | — | — | — |
| CS028 | Google Maps | 3 | Not Started | — | — | — | — |
| CS029 | Payment Gateway (Stripe-style) | 3 | Not Started | — | — | — | — |
| CS030 | Digital Wallet (Paytm/Google Pay) | 3 | Not Started | — | — | — | — |
| CS031 | Stock Exchange / Trading System | 3 | Not Started | — | — | — | — |
| CS032 | Ad Click Aggregation / Ad Serving | 3 | Not Started | — | — | — | — |
| CS033 | Live Sports Score (Real-Time Fanout) | 3 | Not Started | — | — | — | — |
| CS034 | Zoom / Video Conferencing | 3 | Not Started | — | — | — | — |
| CS035 | Ride ETA & Surge Pricing | 3 | Not Started | — | — | — | — |
| CS036 | Distributed Logging / Log Aggregation | 4 | Not Started | — | — | — | — |
| CS037 | Metrics & Monitoring (Datadog-style) | 4 | Not Started | — | — | — | — |
| CS038 | Distributed Task Scheduler / Cron | 4 | Not Started | — | — | — | — |
| CS039 | Workflow / Orchestration Engine | 4 | Not Started | — | — | — | — |
| CS040 | Analytics Platform (Clickstream → OLAP) | 4 | Not Started | — | — | — | — |
| CS041 | Recommendation Engine | 4 | Not Started | — | — | — | — |
| CS042 | Distributed Message Queue (Kafka-lite) | 4 | Not Started | — | — | — | — |
| CS043 | Distributed File System (GFS/HDFS-style) | 4 | Not Started | — | — | — | — |
| CS044 | Object Storage (S3-style) | 4 | Not Started | — | — | — | — |
| CS045 | Email Service | 4 | Not Started | — | — | — | — |
| CS046 | Calendar System | 4 | Not Started | — | — | — | — |
| CS047 | Auction System | 4 | Not Started | — | — | — | — |
| CS048 | Inventory Management System | 4 | Not Started | — | — | — | — |
| CS049 | CRM / Multi-Tenant Platform | 4 | Not Started | — | — | — | — |
| CS050 | Web Analytics / A/B Testing Platform | 4 | Not Started | — | — | — | — |
| CS051 | Code Deployment / CI-CD System | 4 | Not Started | — | — | — | — |
| CS052 | Online Code Judge / Sandbox | 4 | Not Started | — | — | — | — |
| CS053 | Collaborative Whiteboard | 4 | Not Started | — | — | — | — |
| CS054 | Content Moderation Pipeline | 4 | Not Started | — | — | — | — |
