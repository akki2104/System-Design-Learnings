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

### 2026-07-30 — [Topic 031: Choosing a Database]
- Mistake: In a practice scenario (collaborative document editor), used a single Document store (MongoDB) to solve both "store document content" and "search by title/content" — right after the lesson explicitly taught polyglot persistence as the fix for exactly this pattern.
- Why it's wrong: Full-text, relevance-ranked search at scale is a different problem than structured content storage — a document store's secondary indexes aren't built for it. This is the same category error as Topic 028's "write-heavy → MongoDB" trap, just recurring in a new context: defaulting to one familiar database for multiple distinct sub-problems.
- Correct understanding: Document store (MongoDB) for content/metadata; dedicated search engine (Elasticsearch) for full-text search — polyglot persistence applied, not just stated as a principle.
- How to remember: Knowing a principle and applying it under pressure are different skills — when a scenario has two distinct query shapes (structured lookup vs. full-text search), that's the signal to reach for two different stores, not one.
- Recurs? 1 (related in spirit to the Topic 028 write-heavy/MongoDB category error, but a distinct instance — different underlying gap: single-store-for-everything, not throughput-vs-flexibility conflation)

---

### 2026-07-25 — [Topic 026: Concurrency Control: Locks, 2PL, Deadlocks]
- Mistake: Proposed preventing Lost Update via "acquire shared lock to read, then upgrade to exclusive to write"
- Why it's wrong: Shared locks are compatible with each other — both transactions could hold a shared lock simultaneously and both read the same stale value. When both then try to upgrade to exclusive, neither can proceed (each blocked by the other's shared lock) — a lock-upgrade deadlock, not a clean fix.
- Correct understanding: Acquire the EXCLUSIVE lock at read time (`SELECT ... FOR UPDATE`), not a shared lock. This blocks the second transaction's READ itself until the first commits, so it always sees the up-to-date value.
- How to remember: "Lock what you'll write, from the moment you read it — don't start shared and hope to upgrade."
- Recurs? 1

### 2026-07-25 — [Topic 026: Concurrency Control: Locks, 2PL, Deadlocks]
- Mistake: Said deadlock detection would "cause more deadlocks to occur" compared to prevention
- Why it's wrong: Deadlocks occur (or don't) based purely on actual lock-request timing/ordering — independent of which resolution strategy the DB uses. Detection doesn't create deadlocks; it just doesn't stop them from forming, then resolves them reactively.
- Correct understanding: Detection = reactive, more flexible lock ordering, pays an abort/retry cost when a cycle does form. Prevention = proactive, structurally zero deadlock risk, but forces a fixed global lock order that can over-conservatively block transactions that would never have actually deadlocked.
- How to remember: The tradeoff is flexibility-vs-guaranteed-safety, not "which approach causes more deadlocks."
- Recurs? 1

### 2026-07-22 — [Revision — Topic 025: Isolation Levels & Anomalies]
- Mistake: Distinguished non-repeatable read from phantom read using the wrong axis ("without filter vs with filter, different count both times")
- Why it's wrong: Both real examples actually use a filter (e.g., `WHERE id='X'` is a filter too) — filter-presence isn't the distinguishing feature.
- Correct understanding: Non-repeatable read = an EXISTING, already-matched row's VALUE changes on re-read. Phantom read = the SET/COUNT of rows matching a range condition changes because new rows appear or vanish. One is "a row I already found now has a different value"; the other is "new rows now match my filter that didn't before."
- How to remember: "Non-repeatable = same row, new value. Phantom = new row, matches old filter."
- Recurs? 1

### 2026-07-22 — [Revision — Topic 025: Isolation Levels & Anomalies]
- Mistake: Answered "what's the actual isolation-level decision" with "according to the use case, right?" — correct instinct, too vague to count as a real interview answer
- Why it's wrong: An interviewer needs the specific framing, not just an acknowledgment that context matters.
- Correct understanding: The precise framing is "what's the WEAKEST isolation level that still satisfies my correctness requirement for THIS specific operation" — not "always Serializable" (kills throughput) and not "always the default" (risks real bugs).
- How to remember: Weakest-that-still-works, decided PER OPERATION, not per system.
- Recurs? 1

### 2026-07-22 — [Revision — Topic 009: DNS]
- Mistake: On second attempt, correctly named scale and availability as reasons for DNS's hierarchical design, but still missed the third reason (management)
- Why it's wrong: Management — domain owners need to control their own records without needing central authority permission — is the third pillar alongside scale and availability.
- Correct understanding: DNS is hierarchical for three reasons: scale, availability, AND management. Two out of three isn't the full picture.
- How to remember: "Scale, Availability, Management — S-A-M owns his own DNS records."
- Recurs? 1 (this specific sub-gap; overall DNS-hierarchy question has now been asked twice with different wrong/incomplete answers each time)

### 2026-07-20 — [Topic 023: B-Trees vs LSM-Trees]
- Mistake: Believed SSTables are unsorted, so LSM-Tree reads can't use binary search and must do random disk seeks
- Why it's wrong: SSTable literally stands for "Sorted String Table" — each individual SSTable IS sorted, and binary search works fine within one. The actual read cost is having to check MULTIPLE separate sorted structures (memtable + several SSTables), not losing sorting within any single one.
- Correct understanding: LSM-Tree read cost is roughly O(k × log n), where k = number of SSTables checked — multiplying binary search across files, not abandoning it. A key might live in an older SSTable if it wasn't touched since an earlier flush, which is why the newest SSTable alone isn't sufficient.
- How to remember: "The S in SSTable stands for Sorted. The cost is MANY sorted files, not NO sorting."
- Recurs? 1

### 2026-07-15 — [Topic 021: Relational Databases & SQL]
- Mistake: Stated the normalize-vs-denormalize practical rule circularly ("normalize when we have to remove redundancy") rather than naming the actual trigger condition
- Why it's wrong: This restates normalization's definition rather than answering "when do you choose one over the other" — an interviewer wants the decision trigger, not the definition repeated back.
- Correct understanding: Default to 3NF (removes redundancy). Denormalize specifically WHEN sharding would make joins expensive (network hops) OR a read-heavy access pattern benefits from pre-joined/embedded data.
- How to remember: The answer should name a CONDITION ("when X happens, do Y"), not just restate what normalization/denormalization each mean.
- Recurs? 1

### 2026-07-19 — [Revision Blitz — Topic 009: DNS]
- Mistake: Attributed DNS's hierarchical design to "maintaining caching at multiple levels" rather than the real reasons
- Why it's wrong: Caching (TTL) is a separate mechanism layered on top for speed — it's not why DNS resolution is hierarchical in the first place.
- Correct understanding: DNS is hierarchical for three reasons: scale (billions of domains, no single machine/company can hold it all), availability (one server = single point of failure for the entire internet), and management (domain owners control their own records without needing central authority permission). Same pattern as sharding — delegate responsibility instead of centralizing it.
- How to remember: "Hierarchy = delegation of responsibility (scale/availability/management). Caching = a separate speed layer on top."
- Recurs? 1

### 2026-07-19 — [Revision Blitz — Topic 011: HTTPS & TLS]
- Mistake: Could not recall the third TLS guarantee (Authentication) during revision
- Why it's wrong: This is the exact same gap from the original Topic 011 lesson — Confidentiality and Integrity are recalled easily, but Authentication (the certificate proving the server is who it claims to be) keeps getting dropped.
- Correct understanding: TLS provides three guarantees — Confidentiality (encryption), Integrity (tamper detection), Authentication (certificate-verified server identity). Mnemonic: C-I-A.
- How to remember: "C-I-A — like the agency. Confidentiality, Integrity, Authentication."
- Recurs? 2 — first missed in the original 2026-07-12 lesson, then again in the 2026-07-19 revision blitz. **RESOLVED 2026-07-21** — correctly recalled all three (Authentication, Encryption, Integrity) clean on the next revision pass. No longer flagged as persistent.

### 2026-07-19 — [Revision Blitz — Topic 017: Load Balancers]
- Mistake: Explained passive vs. active health checks as a difference in detection SPEED ("passive knows instantly, active has a delay") rather than the difference in WHAT KIND of failure each can see
- Why it's wrong: The real distinction is that active health checks only test a synthetic `/health` endpoint — which can return 200 OK even when the real service is functionally broken (e.g., DB connection pool exhausted, but the shallow health check never touches the DB). Passive health checks observe REAL production traffic outcomes, catching functional failures a synthetic ping structurally cannot see.
- Correct understanding: A server can pass every active health check while being functionally broken for real users — that's precisely the scenario passive checks exist to catch. It's about failure TYPE visibility, not detection speed.
- How to remember: "Active checks ask a scripted question. Passive checks watch what actually happens to real customers."
- Recurs? 1

### 2026-07-19 — [Revision Blitz — Topic 019: Content Compression & Encoding]
- Mistake: Named 2 of 3 video codec adoption costs (encode/CPU cost, compatibility) but still missed the third (decode cost specifically draining the USER'S device battery/CPU, separate from server-side encode cost)
- Why it's wrong: Decode cost lives on a device you don't control — older/lower-end phones lack hardware decode support for newer codecs, forcing software decoding that drains battery and can cause stutter. This is a distinct cost from server-side encoding compute.
- Correct understanding: Three costs before adopting a newer codec: (1) encode cost — server CPU/GPU, (2) decode cost — USER'S device CPU/battery, (3) compatibility — may need a fallback (e.g., H.264 baseline + newer codec for supporting clients).
- How to remember: "Encode is your cost. Decode is THEIR cost. Compatibility is whether they can even try."
- Recurs? 2 — improved from the original lesson (only named 1 of 3 then, now 2 of 3) but the decode-on-device dimension still hasn't stuck. Track for one more clean pass before considering it resolved.

### 2026-07-13 — [Topic 015: WebSockets, SSE, Polling, Long Polling]
- Mistake: Explained short polling's latency-vs-waste tension only as "connection overhead + server load" rather than the precise cost mechanism
- Why it's wrong: The precise insight is that latency COULD be reduced by polling more frequently, but cost scales with clients × frequency, making it economically unbearable at scale — and latency can never drop below network RTT regardless.
- Correct understanding: You're always trading wasted requests for freshness, at a ratio that gets worse as client count grows — not a hard impossibility, an economic one.
- How to remember: "Cost = clients × frequency. You CAN buy lower latency, you just can't afford to at scale."
- Recurs? 3 — 2026-07-19 revision blitz: "increases server load" again. 2026-07-21 revision: same shallow "wasted requests + network overhead" answer a THIRD time. PERSISTENT WEAK AREA — three sessions of conceptual re-explanation haven't worked. Switched teaching method to a concrete numeric walkthrough (10,000 clients × poll interval change from 3s→300ms = 10× load for marginal latency gain) — retest next revision with a similar numeric-style question, not a conceptual one, to check if the concrete framing sticks better.

### 2026-07-13 — [Topic 015: WebSockets, SSE, Polling, Long Polling]
- Mistake: Named only one unsolved limitation of long polling (many idle open connections) and missed the second (re-request overhead every cycle)
- Why it's wrong: Long polling still forces the client to fire a brand-new HTTP request immediately after every response, re-paying connection/header setup cost each time — a separate cost from the idle-connections problem.
- Correct understanding: Long polling has TWO remaining costs: (1) many idle open connections at scale, (2) repeated connection/header overhead on every re-request cycle.
- How to remember: Long polling fixes the "wasted no" problem, not the "repeated handshake" problem.
- Recurs? 1

### 2026-07-13 — [Topic 015: WebSockets, SSE, Polling, Long Polling]
- Mistake: Attributed WebSocket firewall/proxy issues to "firewalls won't allow indefinite connections" rather than the precise mechanism
- Why it's wrong: The real issue is that intermediaries need to explicitly understand and support the HTTP protocol Upgrade. Middleboxes that only understand plain HTTP semantics may block, buffer incorrectly, or terminate a connection that silently becomes non-HTTP — it's a protocol-comprehension issue, not simply a duration/idle-timeout issue.
- Correct understanding: 101 Switching Protocols requires proxy/firewall support for the Upgrade mechanism itself; this is why wss:// typically runs on port 443, disguised as normal HTTPS traffic to less sophisticated network equipment.
- How to remember: "It's not that they hate long connections — it's that they don't understand what the connection turned into."
- Recurs? 1

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

### 2026-06-30 — [Topic 004: Non-Functional Requirements]
- Mistake: Stated latency NFR without a percentile qualifier ("under 1.5 seconds" instead of "under 1.5 seconds at p99")
- Why it's wrong: Without a percentile, a latency threshold is unmeasurable. "Under 1.5s" for 50% of requests is very different from 99% of requests. Interviewers and SREs always require percentile precision.
- Correct understanding: Always say "< Xms at p99" (or p95 for less strict). p99 = 99% of all requests are below this threshold.
- How to remember: Latency without a percentile = a speed limit sign with no number. Always add "at p99."
- Recurs? 1

### 2026-07-01 — [Topic 002 Revision: Interview Framework]
- Mistake: Named Step 6 as "LLD" instead of "Deep Dive"
- Why it's wrong: LLD is a completely separate interview round (classes, patterns, thread safety). The 7-step HLD framework has no LLD step. Step 6 = Deep Dive into the hardest 1–2 HLD components.
- Correct understanding: Deep Dive = pick the spicy HLD components (e.g., the feed ranking algorithm, the video encoding pipeline) and go deep on internals. LLD = different day, different round.
- How to remember: The 7 steps are all HLD. LLD never appears in the HLD framework.
- Recurs? 1

### 2026-07-01 — [Topic 003 Revision: Back-of-the-Envelope Estimation]
- Mistake: Write QPS formula stated as "DAU × (reads+writes)/user/day ÷ 86,400" — included reads
- Why it's wrong: Write QPS formula is for writes only. Reads and writes are calculated separately. Including reads inflates write QPS and corrupts downstream storage and bandwidth estimates.
- Correct understanding: Write QPS = DAU × writes/user/day ÷ 86,400. Read QPS = DAU × reads/user/day ÷ 86,400. Separate formulas, separate numbers.
- How to remember: QPS has a direction. Write QPS = writes only. Read QPS = reads only. Reads don't land on disk.
- Recurs? 2 — also occurred in 2026-07-11 revision blitz; PERSISTENT WEAK AREA

### 2026-07-01 — [Topic 003 Revision: Back-of-the-Envelope Estimation]
- Mistake: Explained bandwidth correctly but could not explain why storage uses average QPS
- Why it's wrong: Without understanding the why, the rule breaks down under interviewer pressure ("why average and not peak?")
- Correct understanding: Storage = accounting (what actually arrived on disk per day = average rate). Bandwidth = capacity planning (how wide the pipes need to be = peak rate). "Storage is what happened. Bandwidth is what could happen."
- Recurs? 2 (repeated from original lesson — now persistent weak area)

### 2026-07-01 — [Topic 004 Revision: NFRs]
- Mistake: Answered the nines table instead of calculating series availability (0.999³)
- Why it's wrong: The question asked for a multiplication — 3 components at 99.9% in series. Series availability = A1 × A2 × A3 = 0.997 = 99.7%. Reciting the table is not the same as computing the result.
- Correct understanding: Series availability compounds multiplicatively. Three 99.9% components → 99.7%. Every dependency added degrades availability.
- How to remember: Series = multiply probabilities. 99.9% × 99.9% × 99.9% = 99.7%.
- Recurs? 1

### 2026-07-08 — [Topic 005: How to Reason About Tradeoffs]
- Mistake: Claimed "high read to write ratio" would push you off Postgres
- Why it's wrong: High read:write ratio (e.g., 100:1) is the case Postgres/relational DBs handle BEST — that's exactly what read replicas and caching solve. The real trigger to leave Postgres is high WRITE throughput exceeding what one primary can absorb.
- Correct understanding: Read-heavy → stay on Postgres, add cache/replicas. Write-heavy (exceeding one primary's capacity) → shard or switch to a leaderless store (Cassandra/DynamoDB).
- How to remember: Read pressure never kicks you off Postgres by itself. Only write throughput does.
- Recurs? 1

### 2026-07-08 — [Topic 005: How to Reason About Tradeoffs]
- Mistake: Attributed expensive joins to "huge amount of data" rather than sharding
- Why it's wrong: A single machine joins large tables (hundreds of GB) fine with good indexes — that's a local, in-memory operation. Joins only become expensive network operations once data is SPLIT across multiple machines (sharding).
- Correct understanding: Big data on ONE machine → joins still work. Data spread across machines → joins become network calls. This is why sharded/NoSQL systems denormalize (duplicate data) to avoid cross-shard joins.
- How to remember: It's not the size of the table, it's whether the table lives on one machine or many.
- Recurs? 1

### 2026-07-08 — [Topic 005: How to Reason About Tradeoffs]
- Mistake: Picked MongoDB for a high-write-throughput problem (WhatsApp, 500K writes/sec), justified with "flexible schema" and "need to view data from different angles"
- Why it's wrong: Flexible schema (MongoDB's strength) and high write throughput (Cassandra's strength) are two different properties — conflating them leads to the wrong pick. Also, "view data from different angles" (ad-hoc queries) is actually an argument AGAINST wide-column stores, but WhatsApp messages have a well-known, fixed access pattern (by conversation_id, ordered by time), so that argument didn't even apply here. MongoDB shards still route writes through a per-shard primary — same write bottleneck as sharded SQL.
- Correct understanding: High write throughput + known access pattern → Cassandra/DynamoDB (leaderless writes, any node accepts a write). Flexible/unknown schema shape → MongoDB. Don't use one tech's justification for a different tech's problem.
- How to remember: Ask "what property does this system actually need?" before naming a database — write throughput and schema flexibility are different axes.
- Recurs? 1

### 2026-06-30 — [Topic 004: Non-Functional Requirements]
- Mistake: Warm-up NFRs were vague ("low latency", "high availability", "serve 10k users") — no thresholds or justifications
- Why it's wrong: Vague NFRs give no architectural anchor. "Low latency" doesn't tell you whether to invest in a cache or a CDN. "High availability" doesn't tell you how many nines to design for.
- Correct understanding: Every NFR = [quality] + [measurable threshold + percentile] + [business justification]. Only then can it drive architecture.
- How to remember: An NFR without a number is an opinion, not a requirement.
- Recurs? 1

### 2026-06-30 — [Topic 001 Revision: Introduction to System Design]
- Mistake: Could not recall the three pillars of observability
- Why it's wrong: Metrics, Logs, Traces are foundational — asked in every deep dive and every operations discussion. Forgetting them = blank on Operational Maturity dimension.
- Correct understanding: **M**etrics (numbers over time), **L**ogs (events with context), **T**races (request journey across services). MLT.
- How to remember: MLT — like a sandwich. Every system needs all three layers.
- Recurs? 1

### 2026-06-30 — [Topic 002 Revision: The System Design Interview Framework]
- Mistake: Time budget recalled as 5/5/5/5/10/10/5 — assigned only 10 min to HLD instead of 15
- Why it's wrong: HLD is the core of the interview — it's where the design lives. It gets the most time.
- Correct understanding: 5/5/5/5/**15**/10/5. HLD = 15 min. Deep dive = 10. Everything else = 5.
- How to remember: HLD is the biggest block. The two 'design' steps (HLD + deep dive) = 25 of 45 minutes.
- Recurs? 2 — 2026-07-11: got HLD right (15) but Deep Dive wrong (said 5 instead of 10). Different slot, same pattern.

### 2026-06-30 — [Topic 002 Revision: The System Design Interview Framework]
- Mistake: Could not recall the two questions you must never ask the interviewer
- Why it's wrong: Asking "should we use microservices?" or "what to choose from CAP?" hands your architectural thinking to the interviewer. It signals you can't reason independently — automatic low score.
- Correct understanding: Never ask the interviewer to make architectural decisions. You clarify REQUIREMENTS; you decide ARCHITECTURE.
- How to remember: Clarifying questions = about the problem. Architectural questions = yours to answer.
- Recurs? 2 (CAP question was also a mistake in original Topic 002 lesson — now recurring)

### 2026-06-30 — [Topic 003: Back-of-the-Envelope Estimation]
- Mistake: Stated implication as "multiple servers with load balancers" — correct but generic
- Why it's wrong: Every scaled system has multiple servers. The implication of estimation numbers must name the *type of constraint* (read-heavy → cache; write-heavy → sharding; PB-scale → object storage). Generic answers don't demonstrate architectural thinking.
- Correct understanding: After computing numbers, ask "what kind of problem is this?" and name it specifically. 1:1 ratio = write-heavy = sharding. 100:1 = read-heavy = cache. PB = object storage.
- How to remember: "The number is the symptom. Name the diagnosis, not just the symptom."
- Recurs? 1

---

### 2026-07-11 — [Revision Blitz: Topics 001–010]

### 2026-07-11 — [Topic 007 Revision: IP, Ports, Sockets]
- Mistake: Stated "Port 80 is for WebSockets, Port 443 is for web server"
- Why it's wrong: Port 80 = HTTP (unencrypted). Port 443 = HTTPS (TLS-encrypted). WebSockets do not own a port — ws:// rides HTTP on port 80, wss:// rides HTTPS on port 443.
- Correct understanding: 80 = HTTP. 443 = HTTPS. WebSockets start as an HTTP connection and upgrade via the `Upgrade: websocket` header.
- How to remember: 443 = HTTPS = Secure. The S in HTTPS = the bigger port. 80 = plain HTTP, no encryption.
- Recurs? 1

### 2026-07-11 — [Topic 009 Revision: DNS]
- Mistake: Walked the DNS chain starting from root nameserver — missed browser cache and OS cache as the first two steps
- Why it's wrong: In practice, most DNS resolutions never reach the root — they're served from browser or OS cache. Starting from root signals you don't know the full resolution path, which interviewers test.
- Correct understanding: Full chain: browser cache → OS cache → recursive resolver → root nameserver → TLD nameserver → authoritative nameserver.
- How to remember: "Before any network packet leaves the machine, check local caches first." Caches are always step 1.
- Recurs? 1

### 2026-07-11 — [Topic 010 Revision: HTTP/1.1, HTTP/2, HTTP/3]
- Mistake: Said HTTP/3's QUIC "guarantees ordered delivery" — describing TCP's behavior, not QUIC's fix
- Why it's wrong: TCP's guaranteed ordered delivery at the connection level is the problem that causes HoL blocking. QUIC fixes it by enforcing ordering per-stream independently — a lost packet in stream 1 only stalls stream 1, not stream 2.
- Correct understanding: QUIC = per-stream ordering over UDP. TCP = connection-level ordering. This is the architectural reason HTTP/3 eliminates cross-stream HoL blocking.
- How to remember: TCP orders at the connection → one loss blocks ALL. QUIC orders per stream → one loss blocks THAT stream only.
- Recurs? 1

### 2026-07-11 — [Topic 005 Revision: Tradeoffs]
- Mistake: Gave the spirit of tradeoff reasoning (steps 3–4) without naming steps 1–2 (Name the decision, Identify the axes)
- Why it's wrong: Interviewers score on structure, not just outcome. Jumping to "I chose X because Y" without naming the tradeoff axis and competing forces skips the reasoning framework that earns points on the Tradeoffs dimension.
- Correct understanding: Full 4 steps: (1) Name the decision, (2) Identify the axes (latency vs consistency, reads vs writes, etc.), (3) Evaluate each option against those axes, (4) Choose + Justify + Acknowledge the cost.
- How to remember: "Name → Axes → Evaluate → Choose+Cost." You can't skip to step 4 without steps 1–2.
- Recurs? 1

### 2026-07-11 — [Topic 006 Revision: Client-Server Model]
- Mistake: Described P2P difficulty as "complexity" without naming the specific type
- Why it's wrong: "Complexity" is filler — it signals you're reaching for a word, not a concept. The actual hardnesses are: security (no central checkpoint), coordination (no authority to enforce consistency), and discovery (no central registry).
- Correct understanding: P2P is harder because of (1) security — no single point to verify/authenticate nodes, (2) coordination — distributed consensus without a leader, (3) discovery — how peers find each other.
- How to remember: Three P's: Protection (security), Protocol agreement (coordination), Peer finding (discovery).
- Recurs? 1

### 2026-07-12 — [Topic 012: REST API Design]
- Mistake: Defined DELETE's idempotency as "same response every time" instead of "same final state"
- Why it's wrong: DELETE actually returns different responses on repeat calls (200/204 first, then 404) — if idempotency meant same response, DELETE would fail the test. Idempotency is a guarantee about system state, not HTTP response codes.
- Correct understanding: Idempotent = calling N times produces the same final state as calling once. DELETE's state (resource gone) is identical after 1 or 100 calls, even though the response code changes.
- How to remember: Look at the DATA after the dust settles, not the response you got. "Same world, not same words."
- Recurs? 1

### 2026-07-12 — [Topic 012: REST API Design]
- Mistake: URL design exercise — reversed path order in 3/5 answers (e.g., `/{userId}/users` instead of `/users/{userId}`), used singular collection nouns (`/tweet`), and modeled "like" as `PATCH .../like` instead of a sub-resource
- Why it's wrong: REST convention always puts the collection noun before the specific ID — reversing it breaks the resource-hierarchy readability the convention exists for. Singular nouns break the "collection" semantics. Modeling a create/delete action ("like") as a PATCH field update conflates a resource creation with a field mutation.
- Correct understanding: `/users/{userId}`, `/tweets` (plural), `/users/{userId}/tweets` (nested ownership), `POST /tweets/{tweetId}/likes` (like = its own resource).
- How to remember: "Collection, then ID" — read the URL left to right as narrowing scope from general to specific. Actions that create/delete records of their own (likes, follows, subscriptions) are always sub-resources, never PATCH fields.
- Recurs? 1

### 2026-07-12 — [Topic 013: RPC & gRPC]
- Mistake: Said gRPC can't be used on mobile apps because "the protobuf payload is humanly unreadable"
- Why it's wrong: Unreadability is a real but secondary debuggability cost — it doesn't actually block gRPC from running anywhere. The real hard blocker is specific to browsers: they can't control HTTP/2 trailers (needed by gRPC to signal stream completion/status) through fetch/XMLHttpRequest. Native mobile apps have no such restriction — Uber and Square use gRPC directly on mobile.
- Correct understanding: Browsers → gRPC blocked at the transport-control level → need grpc-web proxy. Native mobile → gRPC works directly. Debuggability (binary payload) is a separate, secondary reason teams choose REST for public APIs even when gRPC would technically work.
- How to remember: "Browsers can't hold the wheel (HTTP/2 trailers), mobile apps can." Unreadable payload is an annoyance, not a blocker.
- Recurs? 3 — 2026-07-19 revision blitz: gave a DIFFERENT wrong reason ("browsers don't know the server's procedures"). 2026-07-21 revision: got the HTTP/2 API restriction half right, but added a THIRD different wrong reason ("browsers lack native protobuf support" — false, JS protobuf libraries exist) and still didn't state the native-mobile comparison unprompted. PERSISTENT WEAK AREA — three sessions, three different wrong reasons, same underlying fact (HTTP/2 trailer control) not sticking. Needs a mnemonic-first drill, not another conceptual re-explanation.

### 2026-07-12 — [Topic 014: GraphQL]
- Mistake: Correctly diagnosed that GraphQL's N+1 problem moves server-side (per-item DB calls) but did not know the fix (batching/DataLoader)
- Why it's wrong: Without the batching fix, a production GraphQL server silently reintroduces N+1 DB queries per request — this is a real operational cost interviewers probe on when GraphQL is proposed.
- Correct understanding: DataLoader pattern — queue individual per-item resolver requests within the same request tick, then fire one combined batched query (`WHERE id IN (...)`) instead of one query per item. Same principle as write-batching/Nagle's algorithm.
- How to remember: "Collect, then combine." Diagnosis was right (N+1 moved to the server) — the missing piece was the standard fix name and mechanism.
- Recurs? 1

### 2026-07-14 — [Topic 020: Storage Engine Fundamentals]
- Mistake: Explained the buffer pool correctly (fast reads via RAM cache) but didn't know why the DB still flushes data pages to disk instead of relying on WAL replay forever
- Why it's wrong: An unbounded WAL means reconstructing current state requires replaying the entire write history from the beginning of time — infeasible after enough volume. Conflated the buffer pool's job (fast reads) with checkpointing's job (bounding WAL replay time).
- Correct understanding: Checkpointing flushes dirty pages to disk periodically, creating a materialized snapshot of state so crash recovery only replays the log since the last checkpoint. Buffer pool (RAM) = read speed. Checkpointing (disk flush) = bounded recovery time. Two separate mechanisms.
- How to remember: "Buffer pool serves today's reads. Checkpoints stop yesterday's history from piling up forever."
- Recurs? 1

### 2026-07-20 — [Topic 025: Isolation Levels & Anomalies]
- Mistake: Named an anomaly by the isolation level that allows it ("a Read Committed type of anomaly") instead of its actual name (Non-Repeatable Read)
- Why it's wrong: Interviewers ask "what anomaly is this?" expecting the anomaly's name — the isolation level is a separate fact (which levels prevent it), not the anomaly's identity.
- Correct understanding: The anomaly is Non-Repeatable Read; Read Committed is the isolation level that PERMITS it to happen (prevented starting at Repeatable Read).
- How to remember: Anomaly = the failure mode's name. Isolation level = which failure modes are allowed at that tier. Two different vocabularies — don't swap them.
- Recurs? 1
