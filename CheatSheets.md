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

### [005] How to Reason About Tradeoffs
```
THE TRADEOFF DECISION TREE
─────────────────────────────────────────────────
Scary number appears → READS or WRITES?
  READS  → ADD A CACHE       (cheap, reversible, same DB)
  WRITES → SWITCH/SHARD DB   (expensive, hard to reverse)

WHY CACHE FIXES READS, NOT WRITES
─────────────────────────────────────────────────
Read cache miss  = fall back to DB, just slower (no data loss)
Write cache loss = data gone forever (no fallback) — DB must absorb all writes

WHY JOINS GET EXPENSIVE
─────────────────────────────────────────────────
Big data, ONE machine       → joins fine (indexing handles it)
Data SPLIT across machines  → joins = network calls (the real cost)
Fix: denormalize to avoid cross-shard joins

DB CHOICE QUICK MAP
─────────────────────────────────────────────────
Flexible/nested schema, unknown shape   → MongoDB
Massive writes, KNOWN access pattern    → Cassandra / DynamoDB
Money, joins, ACID                      → PostgreSQL / MySQL (default)
Hot reads, sessions, counters           → Redis (cache, not truth)
Full-text/fuzzy search                  → Elasticsearch (index, not truth)
PB-scale blobs                          → S3 / object storage

THE 3-PART INTERVIEW SENTENCE
─────────────────────────────────────────────────
"I'll use [X] because [specific problem property].
 I considered [Y] but rejected it because [what Y trades away]."
```
---

### [006] The Client-Server Model
```
CLIENT = initiates requests | SERVER = listens & responds
Same machine can be BOTH — client to one system, server to another
Model is ASYMMETRIC: servers always listening; clients connect on demand
Failure propagates UP the chain: DB down → App Server times out → Browser errors

Client-Server (default) → centralized, easy to secure, but server = bottleneck
Peer-to-Peer (rare in interviews) → no bottleneck, hard to secure/coordinate
```
---

### [007] IP, Ports, Sockets
```
IP = WHICH MACHINE | PORT = WHICH PROCESS on that machine | SOCKET = the open connection
Socket = 5-tuple: (source IP, source port, dest IP, dest port, protocol)
Ports let ONE machine run MANY services (web:443, ssh:22, db:5432)
Multiple connections to same server:port are distinguished by different
CLIENT source ports (OS-assigned automatically, invisible to the user)
```
---

### [008] TCP vs UDP
```
TCP vs UDP
─────────────────────────────────────────────────
TCP: connection-oriented, guaranteed+ordered delivery, handshake first, slower
UDP: connectionless, no guarantees, no handshake, faster

3-WAY HANDSHAKE (TCP setup cost)
─────────────────────────────────────────────────
SYN → SYN-ACK → ACK   (1 full round trip before real data flows)

WHEN TO USE WHICH
─────────────────────────────────────────────────
TCP → data must be correct & complete: files, APIs, DB, trading systems
UDP → stale data is harmless, speed matters more: video/voice, gaming, DNS

THE REAL QUESTION (not "is speed important")
─────────────────────────────────────────────────
"Does losing/reordering ONE update cause real harm, or does the
 next update make the old one obsolete anyway?"
Harm → TCP.  Obsolete anyway → UDP.

QUIC (HTTP/3, Topic 010 preview)
─────────────────────────────────────────────────
UDP-based but adds its own reliability layer — skips TCP's handshake
cost while still guaranteeing delivery
```
---

### [009] DNS
```
DNS = translates domain names → IP addresses

WHY HIERARCHICAL (not one giant server)
─────────────────────────────────────────
Root → TLD (.com) → Authoritative — each layer handles a small, delegated slice
Same pattern as sharding: distribute responsibility, not centralize it

RESOLUTION FLOW
─────────────────────────────────────────
"google.com?" → Resolver → Root → TLD → Authoritative → IP returned

CACHING (TTL)
─────────────────────────────────────────
Records cached per TTL — skips the full chain on repeat lookups
Lower TTL before migration → faster propagation, less stale-cache risk
Raise TTL after → fewer lookups, cheaper/faster steady state

RECORD TYPES
─────────────────────────────────────────
A     → domain → IPv4
AAAA  → domain → IPv6
CNAME → domain → another domain (alias)
GeoDNS → different IP per requester location (cheap geo-routing)

PROTOCOL
─────────────────────────────────────────
DNS uses UDP (Topic 008) — tiny, stateless, cheap to retry if lost
```
---

### [010] HTTP/1.1, HTTP/2, HTTP/3
```
HTTP/1.1 vs HTTP/2 vs HTTP/3
─────────────────────────────────────────────────
HTTP/1.1 → TCP, ~6 connections/domain limit, app-level head-of-line blocking
HTTP/2   → TCP, ONE connection, multiplexed binary streams, HPACK header
           compression — fixes app-level blocking, TCP-level blocking remains
HTTP/3   → QUIC (UDP-based), independent per-stream delivery, no HOL blocking,
           faster setup (0-RTT), handshake+TLS bundled together

HEAD-OF-LINE BLOCKING — TWO DIFFERENT LAYERS
─────────────────────────────────────────────────
App-level (HTTP/1.1)   → limited connections force requests to queue
Transport-level (HTTP/2 on TCP) → one lost packet blocks ALL multiplexed
                                   streams sharing that TCP connection

WHY HTTP/3 NEEDED A NEW TRANSPORT (not just new HTTP framing)
─────────────────────────────────────────────────
TCP's ordering guarantee applies to the WHOLE connection — can't be fixed
by changing HTTP's layer again. QUIC replaces TCP itself.

WHEN TO DESIGN AROUND WHICH
─────────────────────────────────────────────────
HTTP/1.1 → legacy support, simple low-resource-count APIs
HTTP/2   → default for modern web apps (many resources)
HTTP/3   → high-latency/lossy networks, mobile, video streaming
```
---

### [011] HTTPS & TLS
```
HTTPS = HTTP + TLS

TLS GUARANTEES (3)
─────────────────────────────────────────────────
Encryption    → data unreadable in transit
Integrity     → tampering detected
Authentication → server identity proven via CA-signed certificate

CHAIN OF TRUST
─────────────────────────────────────────────────
Browser trusts CAs (pre-installed) → CA signs server cert → server presents cert
→ browser verifies → connection proceeds

TLS HANDSHAKE
─────────────────────────────────────────────────
Asymmetric (once) → securely exchange session key
Symmetric (all data) → ~1000× faster, used for everything after

TLS VERSIONS
─────────────────────────────────────────────────
TLS 1.2 → 2 RTT before data flows
TLS 1.3 → 1 RTT (key material sent with ClientHello)
0-RTT   → session resumption, data in first packet (repeat connections only)

TLS TERMINATION DECISION
─────────────────────────────────────────────────
General API / web app   → terminate at LB (simpler, standard)
PCI-DSS / HIPAA         → end-to-end TLS (card/health data must be encrypted
                          everywhere, including internal network)

mTLS
─────────────────────────────────────────────────
Both sides present certs → mutual identity proof
Use for: service-to-service auth in microservices
Tools: Istio, Linkerd (service mesh automates cert management)

PORT REMINDER
─────────────────────────────────────────────────
80 = HTTP (unencrypted)   |   443 = HTTPS (TLS-encrypted)
```
---

### [012] REST API Design
```
TWO CORE RULES
─────────────────────────────────────────────────
1. Resources (nouns) in URLs, HTTP method = the verb
   GET /users/5   not   GET /getUser?id=5
2. Statelessness — no server memory between requests (auth token every call)

HTTP METHODS (Safe? / Idempotent?)
─────────────────────────────────────────────────
GET     read       Safe=Y  Idempotent=Y
POST    create     Safe=N  Idempotent=N
PUT     replace    Safe=N  Idempotent=Y
PATCH   partial    Safe=N  Idempotent=N (usually)
DELETE  remove     Safe=N  Idempotent=Y

IDEMPOTENCY = SAME FINAL STATE, NOT SAME RESPONSE
─────────────────────────────────────────────────
DELETE: 1st call → 200/204, repeat calls → 404 (different responses,
same end state: resource gone). Ties directly to payment retry-safety.

URL CONVENTIONS
─────────────────────────────────────────────────
Plural collection nouns      → /tweets not /tweet
Collection before ID         → /users/{id} not /{id}/users
Nested = ownership           → /users/{id}/tweets
Action → model as resource   → POST /tweets/{id}/likes (not PATCH .../like)

STATUS CODES
─────────────────────────────────────────────────
200 OK | 201 Created | 204 No Content
400 Bad Request | 401 Unauthorized (who are you?)
403 Forbidden (know you, can't do this) | 404 Not Found
409 Conflict | 429 Too Many Requests | 500 Server Error

VERSIONING — default: URL path (/v1/...)
─────────────────────────────────────────────────
Path > Header > Query param (in order of interview-default preference)

PAGINATION — OFFSET VS CURSOR
─────────────────────────────────────────────────
Offset (?offset=20&limit=10)  → breaks on live feeds: insert/delete
                                  shifts row positions → dup/skip results
Cursor (?cursor=id82&limit=10) → anchored to fixed ID, immune to shifts
                                  used by Twitter/Slack/Stripe

DECISION BOX
─────────────────────────────────────────────────
REST    → public API, CRUD-shaped, HTTP caching matters
WebSocket → need real-time bidirectional push
GraphQL → client needs flexible/nested data shapes
gRPC    → internal service-to-service, low latency, strict contracts
```
---

### [013] RPC & gRPC
```
RPC MENTAL MODEL
─────────────────────────────────────────────────
REST → act on a resource:      POST /users/5/tweets
RPC  → call a remote function: client.PostTweet(userId=5, text=...)

gRPC = HTTP/2 (transport) + PROTOBUF (serialization)
─────────────────────────────────────────────────
HTTP/2   → multiplexed streams, no app-level HOL blocking (Topic 010)
Protobuf → binary, schema-defined (.proto), compile-time-checked
           contract, smaller/faster than JSON

FOUR CALL PATTERNS
─────────────────────────────────────────────────
Unary                 → 1 request → 1 response
Server streaming      → 1 request → stream of responses (live prices)
Client streaming      → stream of requests → 1 response (chunked upload)
Bidirectional stream  → both sides stream (live chat)

BROWSER VS NATIVE MOBILE (the real distinction)
─────────────────────────────────────────────────
Browser       → CANNOT speak gRPC directly (no HTTP/2 trailer control
                 via fetch/XHR) → needs grpc-web proxy
Native mobile → CAN speak gRPC directly (Uber, Square do this)
Debuggability → binary payload isn't curl-able — secondary reason
                 teams still pick REST for public APIs

DECISION
─────────────────────────────────────────────────
gRPC → internal service-to-service, low latency, strict typed contracts,
        streaming needed
REST → public API, browser clients involved, human-readable debugging
```
---

### [014] GraphQL
```
PROBLEMS IT SOLVES
─────────────────────────────────────────────────
Over-fetching  → REST returns full object, client needs 2 fields
Under-fetching → REST needs N sequential round trips for nested data
GraphQL fix    → 1 endpoint (POST /graphql), client specifies exact
                  fields+nesting in 1 query, 1 round trip

SCHEMA / TYPES / RESOLVERS
─────────────────────────────────────────────────
Strongly typed schema (like protobuf, but for a data graph)
Each field → backed by a resolver function that fetches that data

THREE OPERATIONS
─────────────────────────────────────────────────
Query        → read   (like GET)
Mutation     → write  (like POST/PUT/DELETE)
Subscription → real-time push (like gRPC server streaming)

THE N+1 PROBLEM MOVES, DOESN'T DISAPPEAR
─────────────────────────────────────────────────
Naive: N posts → N+1 DB queries (1 for posts + 1 per post's commentCount)
Fix: DataLoader/batching — queue per-item requests within a tick,
      fire ONE combined query (WHERE id IN (...)) instead of N queries
Same principle as write-batching/Nagle's algorithm elsewhere

CACHING IS HARDER
─────────────────────────────────────────────────
REST: GET /users/5 → cacheable by URL (CDN/browser, free)
GraphQL: single POST endpoint, variable body → no URL to key on
          → needs its own cache layer (persisted queries, Apollo/Relay)

DECISION
─────────────────────────────────────────────────
GraphQL → many diverse client types (web/iOS/Android/TV) needing
           different data shapes from the same domain (Facebook's case)
REST    → simple CRUD, cacheable reads, public API simplicity
```
---

### [015] WebSockets, SSE, Polling, Long Polling
```
THE REAL-TIME SPECTRUM
─────────────────────────────────────────────────
Short Polling → repeat request on timer; wasted "no" responses; cost = clients × frequency
Long Polling  → server holds request open until data/timeout; near-instant once data exists;
                still re-requests every cycle + many idle open connections at scale
SSE           → ONE persistent connection, server pushes continuously; ONE-DIRECTIONAL
                (server→client only); auto-reconnect built into spec; text only
WebSockets    → HTTP upgrades (101 Switching Protocols) to full-duplex persistent connection;
                BOTH directions, binary or text; needs sticky LB routing at scale

DECISION MAP
─────────────────────────────────────────────────
Latency doesn't matter much        → Short Polling
Moderate need, broad compatibility → Long Polling
Server pushes, client never replies → SSE
True two-way, high-frequency       → WebSockets

THE 101 STATUS
─────────────────────────────────────────────────
Confirms protocol upgrade from HTTP → WebSocket.
Middleboxes that don't understand Upgrade may block/mishandle it —
why wss:// typically runs on port 443 (looks like normal HTTPS)
```
---

### [016] Forward Proxy, Reverse Proxy, API Gateway
```
THE DISTINGUISHING QUESTION
─────────────────────────────────────────────────
"Whose side is this thing standing on — client's or server's?"

FORWARD PROXY (client-side)          REVERSE PROXY (server-side)
──────────────────────────           ──────────────────────────
Hides CLIENT from server             Hides SERVER from client
Corporate filtering, VPNs,           SSL termination, load balancing,
client-side caching                  hiding backend topology, caching

API GATEWAY = reverse proxy + cross-cutting concerns for microservices
─────────────────────────────────────────────────
Centralized auth, rate limiting, routing, transformation, logging
— things a plain reverse proxy does NOT provide
```
---

### [017] Load Balancers
```
LB = specialized reverse proxy distributing traffic across a backend pool
     → solves scaling (Topic 001) + redundancy (Topic 004) at once

L4 vs L7
─────────────────────────────────────────────────
L4 → IP+port only (5-tuple), fast, protocol-agnostic, no content awareness
L7 → understands HTTP (path/headers/cookies), enables TLS termination,
      more overhead. Default for web apps.

ALGORITHMS
─────────────────────────────────────────────────
Round Robin           → equal-capacity servers
Weighted Round Robin  → heterogeneous server pool
Least Connections     → long-lived connections (WebSockets, streaming)
IP Hash               → session affinity without a shared store

HEALTH CHECKS
─────────────────────────────────────────────────
Active  → LB pings /health proactively
Passive → LB observes real traffic; catches functional failures
           (e.g. exhausted DB pool) that active pings MISS

STICKY SESSIONS / IP HASH vs REDIS
─────────────────────────────────────────────────
Sticky/IP Hash → routing-layer trick, session still fragile
                  (pinned to ONE server, lost if it dies)
Redis (shared store) → true statelessness, any server serves any
                  request, +1 hop cost, needs its own infra
Redis is NOT just "a better sticky session" — it removes the need
for affinity entirely.

LB REDUNDANCY (LB itself is now a SPOF)
─────────────────────────────────────────────────
DNS-based failover | Active-passive VIP pair (keepalived/VRRP)
Cloud-managed LB (AWS ALB/NLB) → redundancy handled for you

DECISION
─────────────────────────────────────────────────
L4 → raw throughput, protocol-agnostic (DB poolers)
L7 → web apps, content routing, TLS termination
Tools: HAProxy, Nginx, AWS ALB/NLB, Envoy
```
---

### [018] CDN
```
CDN = distributed network of edge servers (reverse proxies) caching
      content physically close to users — fixes latency that's PHYSICAL,
      not code-optimizable

ROUTING: GeoDNS (Topic 009) sends users to their nearest edge

PUSH vs PULL
─────────────────────────────────────────────────
Pull → lazy, cache miss on first request per region, self-managing (default)
Push → pre-distributed everywhere, zero cold-cache misses, for major launches

CACHE INVALIDATION (same as DNS TTL, Topic 009)
─────────────────────────────────────────────────
TTL expiry     → auto-refresh after N time
Explicit purge → force-remove NOW via invalidation API (critical fixes)
```
---

### [019] Content Compression & Encoding
```
NEGOTIATION
─────────────────────────────────────────────────
Accept-Encoding (client's supported formats) → Content-Encoding (server's choice)

TEXT: gzip (safe default) vs brotli (better ratio, more CPU) — compress ONCE
      at build/deploy for static assets, amortized across millions of requests

IMAGES: lossy (JPEG, small, fine for photos) vs lossless (PNG, pixel-perfect,
        required for legal docs/transparency) vs modern (WebP/AVIF + fallback)

VIDEO CODEC TRADEOFF (not just compression ratio)
─────────────────────────────────────────────────
1. Encode cost   → server CPU/GPU
2. Decode cost   → USER'S device CPU/battery
3. Compatibility → may need fallback (H.264 baseline + newer codec)

WHERE: reverse proxy / CDN layer (Topics 016, 018) — transparent, centralized
```
---

### [020] Storage Engine Fundamentals
```
THE CONSTRAINT: RAM vs DISK SPEED GAP
─────────────────────────────────────────────────
RAM ~100ns | SSD random read ~100-150μs (1000x) | HDD seek ~10ms (100,000x)
→ minimize seeks, maximize sequential I/O — drives everything below

PAGES = fixed-size unit of disk I/O (4KB/8KB/16KB)
─────────────────────────────────────────────────
Tiny query still pulls a FULL page. Random access = many seeks (slow).
Sequential access = adjacent pages, prefetchable (fast).

BUFFER POOL (RAM cache)
─────────────────────────────────────────────────
Reads: buffer pool hit → return from RAM | miss → disk → cache → return
Writes: applied to in-memory page first (dirty), not flushed immediately

WAL (WRITE-AHEAD LOG) — durability without random-write cost
─────────────────────────────────────────────────
Write → WAL entry appended+fsynced (sequential, FAST) → ack to client
      → data page dirty in memory → [later] checkpoint flushes to disk
Crash before fsync → write lost. Crash after fsync → replay WAL, recovered.

WHY STILL FLUSH DATA PAGES (not WAL-replay forever)
─────────────────────────────────────────────────
Unbounded WAL → replay = entire write history = infeasible at scale
Checkpointing → materialized snapshot → recovery only replays SINCE
                 last checkpoint, not from the beginning

Buffer pool (RAM, read speed) ≠ Checkpointing (disk, bounds WAL replay)
— two separate mechanisms, easy to conflate

FORWARD LINKS
─────────────────────────────────────────────────
WAL → ACID Durability (024) | WAL shipping → Replication (039)
Sequential-write principle → LSM-Trees (023), Kafka
```
---

### [021] Relational Databases & SQL
```
RELATIONAL MODEL
─────────────────────────────────────────────────
Tables + rows + columns, related via primary/foreign keys

NORMALIZATION — store each fact ONCE
─────────────────────────────────────────────────
1NF → atomic values, no repeating groups in one column
2NF → no partial dependency on a composite key
3NF → no transitive dependency (non-key depending on non-key)
Default to 3NF; denormalize deliberately (Topic 005) for read speed/sharding

JOINS
─────────────────────────────────────────────────
Cheap on ONE machine (in-memory/page matching)
Expensive once SHARDED (network hop per join)
INNER / LEFT / RIGHT / FULL

SQL = declarative — query planner decides execution (uses Topic 020's
      buffer pool / page internals to decide what's cheap)

SCHEMA ENFORCEMENT
─────────────────────────────────────────────────
Fixed schema + types + constraints, enforced at write time —
opposite of MongoDB's flexible schema (Topic 005)

WHEN TO USE RELATIONAL (extends Topic 005)
─────────────────────────────────────────────────
Clear relationships + integrity matters + frequent joins + ACID > infinite
horizontal write scale → still the DEFAULT starting point
```
---

### [022] Indexing Deep Dive
```
FULL SCAN vs INDEX
─────────────────────────────────────────────────
Full table scan: O(n)          — check every row
B-Tree index lookup: O(log n)   — height stays tiny even at huge scale
                                   (10M rows → ~24 page reads)

CLUSTERED vs SECONDARY (NON-CLUSTERED)
─────────────────────────────────────────────────
Clustered   → row data physically stored in index order; ONE per table; 1 hop
Secondary   → separate structure pointing to row location; MANY per table; 2 hops
              (the "bookmark lookup" cost)

COMPOSITE INDEX — LEFTMOST PREFIX RULE
─────────────────────────────────────────────────
INDEX (A, B) helps: WHERE A=x | WHERE A=x AND B=y
INDEX (A, B) does NOT help: WHERE B=y alone

COVERING INDEX
─────────────────────────────────────────────────
Index contains ALL columns a query needs → index-only scan,
no bookmark lookup to the actual table at all

THE COST SIDE
─────────────────────────────────────────────────
Every index → slower writes (must update on every INSERT/UPDATE/DELETE)
            → more storage
Add when: high cardinality + frequently queried + read-favoring ratio
Avoid when: write-heavy table, low-cardinality column, already over-indexed
```
---

### [023] B-Trees vs LSM-Trees
```
B-TREE vs LSM-TREE
─────────────────────────────────────────────────
B-Tree:    in-place random writes (slow writes), ONE sorted structure (fast reads)
LSM-Tree:  sequential appends only (fast writes), MULTIPLE sorted structures (slower reads)

LSM WRITE PATH
─────────────────────────────────────────────────
Write → WAL (append) + memtable (in-memory, sorted)
Memtable full → flush as new immutable SSTable (sorted, sequential write)

LSM READ PATH
─────────────────────────────────────────────────
Check memtable → check newest SSTable (binary search WITHIN it, it IS sorted)
→ not found → check next SSTable → ... (key may be in an OLDER SSTable)
Cost: O(k × log n) — k = number of SSTables checked, not "no sorting"

COMPACTION
─────────────────────────────────────────────────
Merges multiple SSTables → fewer files to check + removes stale/duplicate/
deleted entries. Background cost that bounds read-side degradation.

BLOOM FILTERS (Topic 089 preview)
─────────────────────────────────────────────────
Per-SSTable structure: quickly says "definitely NOT here" → skip that
SSTable without a disk read

DECISION (extends Topic 005)
─────────────────────────────────────────────────
Write-heavy → LSM-Tree (Cassandra, RocksDB, LevelDB, HBase)
Read-heavy/balanced → B-Tree (Postgres, MySQL/InnoDB)
NOT strictly better either way — tradeoff based on read:write ratio
```
---

### [024] Transactions & ACID
```
ACID
─────────────────────────────────────────────────
Atomicity   → all-or-nothing; WAL/undo log enables rollback of incomplete txns
Consistency → valid state → valid state; schema constraints hold (Topic 021)
              ≠ CAP's Consistency (cross-node agreement, Topic 046) — different concept!
Isolation   → concurrent txns don't interfere (levels detailed in Topic 025)
Durability  → committed = survives crash; THIS IS WAL fsync-before-ack (Topic 020)

WHEN TO USE FULL TRANSACTIONS
─────────────────────────────────────────────────
Money/inventory/anything where partial writes cause real harm → YES
Analytics/logging/event data → often NO, cost > benefit

MECHANISM MAP
─────────────────────────────────────────────────
Atomicity   ← WAL/undo log
Consistency ← constraint checks (Topic 021)
Isolation   ← concurrency control (Topics 026/027)
Durability  ← WAL fsync (Topic 020)
```
---

### [025] Isolation Levels & Anomalies
```
THE ANOMALIES (name them, not the level that allows them)
─────────────────────────────────────────────────
Dirty Read           → read UNCOMMITTED data from another txn
Non-Repeatable Read  → re-read SAME ROW, value changed (other txn committed)
Phantom Read         → re-run SAME QUERY, ROW SET changed (insert/delete)
Lost Update          → two txns read same value pre-commit, both write,
                        one update silently overwritten (read+write not atomic)

FOUR ISOLATION LEVELS (progressively stricter, progressively costlier)
─────────────────────────────────────────────────
                    Dirty  NonRepeat  Phantom  LostUpdate
Read Uncommitted     ✗        ✗         ✗          ✗
Read Committed       ✓        ✗         ✗          ✗      ← Postgres default
Repeatable Read      ✓        ✓         ✗*         ✗      ← MySQL/InnoDB default
Serializable         ✓        ✓         ✓          ✓
(✓ = prevented, ✗ = possible; *Postgres's MVCC impl also blocks phantoms in practice)

WHY NOT ALWAYS SERIALIZABLE
─────────────────────────────────────────────────
Heavier locking (more blocking) OR abort/retry under contention (optimistic)
→ throughput cost. Pick level based on what YOUR business logic tolerates.

LOST UPDATE WALKTHROUGH (bank transfer)
─────────────────────────────────────────────────
A reads balance=100 → B reads balance=100 (still committed, A hasn't written)
→ A withdraws, writes 0, commits → B withdraws (based on ITS OWN read of 100),
writes 0, commits → A's withdrawal LOST, $200 left but balance shows one $100
Fix: Serializable OR explicit row lock (SELECT ... FOR UPDATE) — Topic 026

FORWARD LINKS
─────────────────────────────────────────────────
Locking mechanism → Topic 026 (2PL, Deadlocks)
Snapshot isolation impl → Topic 027 (MVCC)
```
---

### [026] Concurrency Control: Locks, 2PL, Deadlocks
```
LOCK TYPES
─────────────────────────────────────────────────
Shared (S)    → reading, compatible with other S-locks
Exclusive (X) → writing, incompatible with ANY other lock on that row

PREVENTING LOST UPDATE — THE TRAP
─────────────────────────────────────────────────
WRONG: S-lock to read, upgrade to X to write
  → both txns hold S concurrently, both read stale value,
    then BOTH block on upgrade → lock-upgrade DEADLOCK
RIGHT: X-lock AT READ TIME → SELECT ... FOR UPDATE
  → 2nd txn's READ itself blocks until 1st commits (sees NEW value)

TWO-PHASE LOCKING (2PL) — guarantees SERIALIZABILITY
─────────────────────────────────────────────────
Growing phase  → acquire only, release none
Shrinking phase → once you release ONE lock, acquire NONE more
Strict 2PL: hold ALL X-locks until commit/abort (not just shrinking
   phase start) → avoids CASCADING ROLLBACK (dirty read → chain of rollbacks)

DEADLOCKS
─────────────────────────────────────────────────
A holds Row1 wants Row2 | B holds Row2 wants Row1 → both wait forever
Detection: wait-for graph, cycle → abort a VICTIM, app retries
Prevention: fixed global lock-ACQUISITION ORDER → structurally impossible

DETECTION vs PREVENTION — the tradeoff
─────────────────────────────────────────────────
Neither strategy CAUSES more/fewer deadlocks (timing-dependent, not
strategy-dependent). Detection = flexible ordering, pays abort/retry
cost when cycles form. Prevention = zero deadlock risk, but forces a
fixed order → can over-conservatively block txns that'd never deadlock.

COST: more locking = safer, less concurrent → MVCC (027) is the
mostly-lock-free alternative
```
---
