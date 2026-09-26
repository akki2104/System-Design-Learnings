# Case Study #1: TinyURL / URL Shortener

**Tier:** 🔴 (Tier 1 Foundational — the universal warm-up question)
**Completed:** 2026-09-26
**Confidence:** 4/5
**Mode:** First case study under Sequencing Track v3 — guided whiteboard session, fully learner-driven (Master Guide §10.3), mentor correcting/deepening throughout.

---

## 1. Problem Statement

Design a service like TinyURL/bit.ly. Users submit a long URL and get back a short one; visiting the short URL redirects to the original long URL.

---

## 2. Requirements Gathering

### Functional Requirements
- Shorten a long URL into a short one.
- Redirect a short URL to its original long URL.
- Support an optional **custom alias** — if provided, it must be unique (reject with a clear conflict error if taken).
- Support an optional **expiry** date per URL.

### Non-Functional Requirements
- Redirect latency: **p99 < 100ms** — this is the hot path; users abandon a slow redirect.
- Scale: **~100 million new URLs created per month.**
- Read:write ratio: **~100:1** — redirects (reads) massively outnumber URL creation (writes).

### Clarifying Questions Asked (and why each mattered)
- *"Any aliases for the URLs?"* — determines whether the create flow needs a uniqueness-check/conflict path at all.
- *"Any expiry?"* — determines whether the data model needs a TTL concept and whether cleanup is needed.
- *"Redirection time from short to long URL?"* — pins down the dominant NFR before any design decision is made; this single answer (must feel instant) shapes the caching strategy, the ID-generation choice, and the DB choice later.
- *"DAU / scale?"* — without a number, capacity estimation and every downstream scaling decision (partitioning, whether NoSQL is even justified) has nothing to anchor to.

### Scope
- **In scope:** create, redirect, custom alias, expiry, basic security/abuse prevention.
- **Out of scope (not asked for, not designed):** analytics dashboards, full user-account system (a `user_id` field is included for future extensibility only), link editing/versioning.

---

## 3. Capacity Estimation

```
New URLs: 100,000,000 / month ÷ 30 days ≈ 3.33M/day (rounded to 3.5M/day for clean math)

WRITE QPS ≈ 3.5,000,000 ÷ 86,400 s/day ≈ 40/s
  (a quick mental-math approximation of 86,400 ≈ 10^5 gives ~35/s — both are
  acceptable back-of-envelope answers; the precise figure is ~40.5/s)

READ QPS ≈ WRITE QPS × 100 (the given read:write ratio) ≈ 3,500 – 4,000/s

STORAGE (row ≈ 100 bytes: short_code, long_url, alias, timestamp, expiry metadata)
  100,000,000 rows/month × 100 bytes = 1×10^10 bytes = 10 GB/month
  → ~120 GB/year
  → ~600 GB (0.6 TB) over 5 years
```
**A caught arithmetic error worth remembering:** the first pass computed write QPS as "3.5/s" by skipping the ÷86,400 conversion (daily count treated as if it were already a per-second rate), and separately computed storage as "10^9 bytes = 1GB/month" — an order-of-magnitude slip (100M × 100 bytes = 10^10 bytes = 10GB, not 10^9 = 1GB). Both were caught and corrected. This ties directly to a long-standing personal weak area (QPS-from-volume conversions) — worth a deliberate slow-down and a sanity-check pass on any estimation math before moving on.

---

## 4. API Design

```
CREATE:
  POST /api/v1/urls        (or an action-style /api/v1/shorten — either is defensible)
  Request body: { long_url, alias?, expiry? }
  Success:  201 Created    { short_url }
  Conflict: 409 Conflict   { "error": "DUPLICATE_ENTRY", "message": "alias already exists." }
    — the requested custom alias is already taken

REDIRECT:
  GET /{short_code}
  Success:  302 Found      Location: <long_url>   (no meaningful response body —
                                                    the Location header IS the redirect)
  Not found: 404 Not Found  (a code that never existed)
  (410 Gone is a more precise alternative specifically for an EXPIRED code —
   distinguishing "never existed" from "existed, now intentionally gone" —
   404 is an acceptable simplification too)
```

**Why 302, not 301, for the redirect — this is the single most TinyURL-specific API decision in the whole design:**
```
301 (Permanent): browsers/CDNs cache the redirect aggressively. After the FIRST
  click, repeat clicks may never reach our servers again.
  + Less load on our servers
  - We lose click-count visibility after the first click
  - Expiry/destination changes won't be seen by clients with a cached redirect

302 (Found/Temporary): browser does NOT cache — every click hits our servers.
  + Full click analytics (a near-universal real product requirement)
  + Expiry and destination changes take effect immediately, always
  - More requests reach our infrastructure
```
Real shorteners (bit.ly-class systems) use 302 specifically because they want analytics and immediate update visibility — the "extra load" cost is handled server-side via caching (Section 9), not by giving up those properties for free browser-side caching.

---

## 5. Data Model & Database Choice

### The Decision: DynamoDB (KV/Wide-Column) over Postgres (Relational)

Running the system through the standard database decision framework (access pattern, read:write ratio, relationships, consistency requirement, schema shape, scale ceiling):

| Criterion | What it says for TinyURL |
|---|---|
| Access pattern | Known and fixed — pure point lookup, `short_code → long_url`, no joins ever on the redirect path |
| Read:write ratio | ~100:1 — read-heavy but not extreme |
| Relationships | None needed for the core table |
| Consistency | A newly-created link taking a few seconds to propagate everywhere is **cosmetic staleness** (self-resolving, harmless) — eventual consistency is confirmed acceptable here |
| Schema shape | Flat, simple, a handful of fields |
| Scale ceiling | ~600GB/5yr, ~4K read QPS, ~40 write QPS — comfortably within a single well-indexed Postgres instance too |

**The honest finding:** at *today's* modest numbers, Postgres would also technically work — none of the specific thresholds (writes exceeding 10s of K/sec, data exceeding a few TB) that force you off a relational default are being approached. But the *firm* recommendation is still DynamoDB, because:
- Relational's actual value-add (joins, multi-row ACID transactions) is **never exercised** by this workload — the redirect path is a pure KV lookup, and any secondary query pattern (e.g., "list my URLs") gets its own dedicated index rather than a live join (see Section 6 on shard-key choice) — so staying relational buys nothing here.
- DynamoDB gives **horizontal write-scaling and multi-region reads natively** — Postgres would require hand-building sharding (exactly the work in Sections 6-8) if it ever needed to scale past one primary; DynamoDB starts already built for that.
- The workload's shape (pure KV access + confirmed-acceptable eventual consistency + a global, latency-critical, unpredictable-growth-potential service) matches DynamoDB's sweet spot more precisely than Postgres's.

**Interview sentence:** *"I'll use DynamoDB — the redirect path is a pure key-value lookup that never needs joins or multi-row transactions, eventual consistency is confirmed acceptable, and I want horizontal write-scaling and multi-region reads built in rather than something I'd have to engineer myself later. I rejected Postgres — not because it couldn't handle today's volume, but because it would be optimizing for relational capabilities this workload structurally never exercises."*

### Schema (Item Structure)

```
Table: Urls   (Partition Key: short_code)
{
  short_code:  string   ← PK. Either auto-generated (Section 6) or the user's custom alias — the SAME field either way, no separate "id" needed
  long_url:    string
  created_at:  number (epoch)
  expiry_time: number (epoch)  ← designated as DynamoDB's NATIVE TTL attribute — DynamoDB
                                  automatically deletes expired items in the background,
                                  no manual cleanup job needed
  user_id:     string, optional ← not used by the primary access pattern; if "list my
                                  URLs" ever becomes a real requirement, served via a
                                  Global Secondary Index (GSI) on this field — a
                                  separate, automatically-maintained secondary index,
                                  never a compromise on the primary partition key
}
```

---

## 6. Deep Dive #1: Short Code Generation

Three options considered (see Topic 098's full framework), walked through with explicit self-flagged rejections — **naming the flaw in your own proposal before being asked is a high-signal interview habit**:

```
AUTO-INCREMENT (rejected): a single global counter is a centralized bottleneck —
  directly contradicts the very reason DynamoDB (decentralized, horizontally
  partitioned) was chosen two steps earlier. Not a "deal with it later once we
  scale" problem — it contradicts the architecture as already designed, right now.

RANDOM UUID v4 (rejected): not sortable — can't efficiently answer "give me the
  last 20 created links." 122 bits of true randomness makes collision
  probability negligible, but sortability is lost entirely.

SNOWFLAKE (chosen): 64-bit ID = [41-bit timestamp][10-bit machine ID][12-bit sequence]
  - No per-ID coordination: each machine has its own fixed worker ID (assigned
    once, at startup — a much smaller one-time cost than a per-insert counter hit)
    and generates IDs from its own clock + local counter.
  - Timestamp occupies the MOST SIGNIFICANT bits specifically so that numeric/
    lexicographic ID order approximates GLOBAL chronological order across every
    machine — not just order within one machine's own stream (which is the much
    weaker guarantee a machine-ID-first layout would give).
  - Encoded in BASE62 ([0-9, a-z, A-Z], 62 characters) for the final short code:
    base62 needs ZERO escaping in a URL (unlike base64's `+`, `/`, `=`, which
    require a URL-safe variant workaround) while maximizing character density —
    a 64-bit Snowflake ID becomes at most an ~11-character string.
```

---

## 7. Deep Dive #2: Custom Alias Collision Handling

Snowflake only generates the **auto-generated** code path — a custom alias is a user-supplied string, requiring its own uniqueness mechanism entirely separate from ID generation.

**The race without protection:** two users simultaneously request the same custom alias. Both check "does it exist?" (read), both see "no," both write — a classic check-then-act race.

**The fix — DynamoDB's conditional write:**
```
PutItem(
  Item = { short_code: "myalias", long_url: "...", ... },
  ConditionExpression = "attribute_not_exists(short_code)"
)
```
This bundles the check *and* the write into a single atomic server-side operation — DynamoDB guarantees exactly one of two racing `PutItem`s succeeds; the loser gets `ConditionalCheckFailedException`, translated directly into the 409 Conflict from Section 4. This is stronger than even a relational `SELECT ... FOR UPDATE` fix (which still has two logical steps, lock-then-write) — here there's no separate read-then-write window at all, not just a narrowed one. Applied to the Snowflake-generated path too, as cheap insurance against its astronomically-unlikely-but-not-zero collision risk.

---

## 8. High-Level Architecture

```
                                     ┌─────────┐
                Client ────────────► │   CDN   │  (short, BOUNDED TTL — viral-spike
                  ▲                  └────┬────┘   stampede protection ONLY, see Section 9;
                  │                       │         NOT full redirect caching, which would
                  │                       ▼         undermine analytics the same way 301 would)
                  │                ┌─────────────┐
                  │                │ Load Balancer│  (health-checked; HA via managed cloud
                  │                └──────┬──────┘   LB, or DNS failover / floating-VIP+VRRP
                  │                       │           if self-hosted — see Section 11)
                  │              ┌────────┼────────┐
                  │              ▼        ▼         ▼
                  │         ┌────────┐┌────────┐┌────────┐
                  │         │ App 1  ││ App 2  ││ App 3  │  (stateless, horizontally scaled)
                  │         └───┬────┘└───┬────┘└───┬────┘
                  │             │  read    │         │
                  │             ▼          │         │
                  │        ┌─────────┐     │         │
                  │        │  Redis  │◄────┴─────────┘
                  │        │ (cache- │   write (DB only — no cache write on
                  │        │  aside) │   create; see below)
                  │        └────┬────┘
                  │       cache miss
                  │             ▼
                  │        ┌──────────────────────┐
                  └────────┤   DynamoDB           │
                           │ (sharded by short_code│
                           │  + per-shard replicas)│
                           └──────────────────────┘
```

**Create flow:** Client → LB → App server → conditional `PutItem` to DynamoDB (Section 7) → 201 response. **No cache write here at all** — the cache is populated lazily by the next read (correct cache-aside, Topic 033's pattern).

**Redirect flow:** Client → LB → App server → check Redis (cache-aside read) → **hit**: return 302 immediately; **miss**: query DynamoDB → populate cache → return 302 with `Location` header.

**A design insight worth stating explicitly in an interview:** URL mappings are **immutable** once created (no in-place updates to `long_url`) — so the entire cache-invalidation-race problem (delete-before-update vs. after) doesn't even apply here the way it would for mutable data. There's nothing to invalidate, only create-once and eventually expire. This is *why* the write path can skip the cache entirely without any consistency risk.

---

## 9. Caching

- **Redis, cache-aside, read path only.** Populated lazily on cache miss; never written to on create (see Section 8's immutability point).
- **Cache hit ratio** is the key metric — it directly measures how much read load Redis absorbs before it reaches DynamoDB, and a silently dropping ratio is an early warning sign of trouble before redirects actually slow down.
- **CDN edge caching — narrowly scoped, with an explicit tradeoff stated.** Caching the 302 response at the CDN reintroduces the *exact* problem 301 was rejected for (lost analytics, stale updates) — UNLESS the TTL is short and bounded, used specifically to absorb a stampede-level burst on one viral link (the same shape as a cache-stampede problem, just handled one layer further out). Framing it as a free win, or as a way to "scale to billions of records," is a mistake — that's a *storage* scaling claim (already solved by DynamoDB's partitioning), conflated with a *request-volume/latency* scaling tool (what CDN actually does). Keep the two scaling dimensions separate.

---

## 10. Messaging / Async

N/A for this topic — TinyURL's core create/redirect flow is fully synchronous request/response; there's no natural queue or event stream in the base design (a possible extension — asynchronously logging click events for analytics via a stream — was out of scope per Section 2).

---

## 11. Failure Handling

| Component | What breaks | Mitigation |
|---|---|---|
| **App server dies** | Requests routed to it fail | LB detects via heartbeat/health check, stops routing to it; survivors absorb load until it's fixed/replaced |
| **Redis node dies** | That node's slice of cached keys is gone | **Sharding alone doesn't protect against this** — it only spreads capacity. Each shard needs its own replica(s) (primary + follower) so a different node can take over serving that slice — the same partitioning-vs-replication distinction drawn for the database layer: sharding solves capacity, replication solves availability, both are needed |
| **DynamoDB shard unavailable** | That shard's data temporarily inaccessible | Consistent hashing decides the *new* owner, but the actual bytes must come from a **surviving replica** — the dead node itself can't be a copy source. Only works because replication factor > 1 existed beforehand. A grace period (sustained unreachability, not an instant reaction) prevents an unnecessary, expensive rebalance over a brief transient blip |
| **Load balancer dies** | New/existing clients can't reach any app server | LB needs its own HA: DNS-based failover (health-check-aware DNS, though subject to client-side DNS caching delay) or a floating/virtual IP with VRRP (`keepalived`) for near-instant takeover. Pragmatically: use a managed cloud LB (AWS ALB/GCP LB), which is already a distributed, self-healing service behind one stable endpoint — not something to hand-roll HA for yourself |

---

## 12. Monitoring & Observability

```
BASELINE SYSTEM HEALTH (needed regardless of any single event):
  - Latency (p99) — are we actually meeting the <100ms redirect NFR?
  - Error rate — 5xx from app servers, DynamoDB throttling, Redis connection failures
  - Traffic/throughput vs. estimated capacity (~40 write/s, ~4K read/s)
  - Resource saturation — CPU/memory/connections, DynamoDB consumed capacity units

SPECIFIC TO THIS ARCHITECTURE:
  - Cache hit ratio — directly measures read load absorbed before hitting DynamoDB;
    a silent drop is an early warning sign, distinct from any single viral event
  - Per-link click count — viral-link detection (a business/product signal)
  - Per-key cache-miss concentration — the PRECISE stampede-detection signal;
    distinct from aggregate system traffic, since one hot key's stampede can be a
    tiny fraction of total traffic and still take down the database behind it
```
**A gap worth remembering:** the first pass at this list jumped straight to "per-link click count" and treated that as sufficient monitoring — it catches only one specific failure mode (a single link going viral) and says nothing about whether the system is healthy day-to-day. The baseline signals (latency/errors/traffic/saturation) have to be there regardless of whether any single link ever goes viral.

---

## 13. Security

```
INPUT VALIDATION: long_url must be well-formed at the API boundary — not
  "SQL injection" (we're on DynamoDB, no SQL), but still validate/sanitize
  (e.g., reject a javascript: URI submitted as the "long URL")

RATE LIMITING on create: token bucket / sliding window per user or IP — prevents
  one actor from exhausting write capacity or spamming storage costs

★ MALICIOUS URL / PHISHING SCREENING — the concern most SPECIFIC to this exact
  system, not generic checklist security: a URL shortener's whole function is
  HIDING the real destination, making it a favorite phishing/malware vector.
  Check submitted long_urls against a known-malicious blocklist (e.g., Google
  Safe Browsing API) before shortening, and periodically re-scan existing links
  since a destination can turn malicious AFTER creation.

★ OPEN REDIRECT — name this as its own recognized vulnerability class: the
  system's entire purpose (redirect to an arbitrary destination) IS the
  textbook open-redirect pattern, just intentional. Mitigated by the same
  URL-screening step, optionally plus an interstitial warning for unverified
  destinations.

ID-GUESSABILITY CAVEAT: Snowflake IDs are sortable/decentralized but NOT
  cryptographically unguessable (structure is somewhat predictable) — a real
  tradeoff against our own ID choice IF "unlisted" link privacy were ever a
  stated requirement (it wasn't, here — but worth naming as a caveat, not a bug)

ROUNDING OUT: TLS/HTTPS everywhere; auth for custom-alias accountability
  (stronger signal than IP alone, which is spoofable/shared); distinguish
  single-actor abuse (rate limiting handles it) from genuine distributed DDoS
  from many IPs (needs an upstream CDN/WAF-level layer, not app-layer alone)
```

---

## 14. Tradeoffs — the Biggest Decisions

| Decision | Chose | Rejected | Why |
|---|---|---|---|
| Database | DynamoDB | Postgres | Workload never exercises relational's value-add (joins/transactions); confirmed-acceptable eventual consistency removes Postgres's main advantage; native horizontal write-scaling matches unpredictable growth |
| Redirect status code | 302 | 301 | Analytics + immediate expiry/update visibility, at the cost of every click reaching our servers (absorbed via caching) |
| ID generation | Snowflake + base62 | Auto-increment, UUID v4 | Auto-increment recreates a centralized bottleneck contradicting the chosen architecture; UUID v4 isn't sortable. Snowflake is decentralized (one-time worker-ID cost only) and globally time-sortable |
| CDN caching scope | Short, bounded TTL for viral-spike protection only | Full/aggressive redirect caching | Full caching would reintroduce the exact analytics/staleness cost 301 was rejected for; narrow scope bounds that cost deliberately |

---

## 15. Alternative Design

A materially different, simpler architecture: **single-region Postgres, no sharding, range/block-allocated IDs.** This is fully viable at today's modest estimated scale (Section 3) — none of the thresholds forcing a NoSQL/sharded design are being approached. The tradeoff: it requires manually building the exact sharding/consistent-hashing/rebalancing machinery (Sections 6-8's DynamoDB-native behavior) if the product's growth ever exceeds a single primary's write capacity — a costly later migration this design chose to avoid by starting on DynamoDB instead. Worth naming as a legitimate, simpler alternative for a smaller-scale or early-stage version of this product, not a wrong answer.

---

## 16. Interviewer Follow-Up Questions (rehearse these)

1. Why can't you just use auto-increment IDs once you've already chosen a horizontally-partitioned database?
2. Walk through exactly what happens, step by step, on a cache miss for a redirect.
3. Why does Snowflake put the timestamp in the most significant bits instead of the machine ID?
4. What's the actual difference between sharding and replication, and why do you need both at the Redis layer specifically?
5. If a DynamoDB shard becomes unavailable, where does the data for its rebalanced keys actually come from?
6. Why 302 instead of 301 for the redirect — what would break if you used 301?
7. What security concern is *unique* to a URL-shortening service, as opposed to generic web-app security?
8. How do you detect a cache stampede specifically, as opposed to just "high traffic"?
9. What happens if your load balancer itself dies — who load-balances the load balancers?
10. Would Postgres actually work for this system at today's scale? If so, why did you still pick DynamoDB?
11. Your custom-alias uniqueness check — why is a DynamoDB conditional write stronger than a relational "SELECT then INSERT" approach?
12. What's the tradeoff of caching the redirect response at the CDN layer?

---

## 17. Evaluation Rubric Mapping (Section 2.1 dimensions)

| Dimension | How this session scored |
|---|---|
| Requirements & scoping | Strong — four targeted clarifying questions before any design commitment |
| High-level architecture | Strong after correction — initial draft had the redirect narration skip the cache; corrected to full cache-aside flow |
| Data model & API | Strong — schema, status codes, and the DB-choice reasoning were all pushed to a precise, complete justification, not just "it works" |
| Deep dive & bottlenecks | Strong — ID generation and collision handling were both self-corrected and fully reasoned through, including proactively naming the auto-increment flaw before being asked |
| Tradeoffs & justification | Strong, with real growth during the session — the CDN/scaling conflation and the "why not Postgres" reasoning both sharpened considerably across iterations |
| Operational maturity | Mixed → strong — failure handling required correction twice (Redis sharding-vs-replication, DynamoDB rebalance source) before landing precisely; monitoring needed the baseline-signals gap filled in |
| Communication | Strong — proactively flagged uncertainty ("I don't know the answer"; "point out better solutions") rather than guessing past gaps |

---

## 18. Cheat Sheet
See the `CS001 TinyURL` entry appended to `CheatSheets.md`.
