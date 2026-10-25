# Case Study #2 (compressed track): Rate Limiter

**Numbering:** file number = position on the compressed 14-case list (Rate Limiter is #2). The old 54-row table in Progress.md still labels it `CS003` (Pastebin was `CS002` there); that table predates the compressed plan.
**Tier:** 🔴 (Tier 1 — extremely common; pairs with Topic 071)
**Completed:** 2026-10-24
**Confidence:** 3/5
**Mode:** Second case study under Sequencing Track v3 — guided whiteboard, learner-driven (Master Guide §10.3). Prerequisites taught just-in-time beforehand: Topic 071 (Rate Limiting & Throttling) and the CS2-relevant slice of R01 (Redis). **Honest note:** Step 4 (the token-bucket data model and Lua walk-through) was taught by the mentor after the learner said they didn't know it and weren't confident in the underlying module; the learner then reproduced it in the Step 5 diagram (with bugs, listed in §6). Steps 6-7 needed substantial mentor help (see §17).

---

## 1. Problem Statement

Design a rate limiter that restricts how many requests a client can make to our API within a given time period.

---

## 2. Requirements Gathering

### Functional Requirements
- Limit requests per client, identified by **API key** (developers) or **user ID** (logged-in users); fall back to **IP** for unauthenticated endpoints such as login.
- Limits are **configurable per endpoint** (`/login` is much tighter than `/search`).
- **Allow short bursts** but enforce a steady average over time (legitimate clients retry after network blips).
- Enforced **server-side only**. A client-side limit is advisory and can be ignored by a malicious or buggy client.
- When blocked, return **HTTP 429** with `Retry-After` and `X-RateLimit-*` headers so well-behaved clients back off.

### Non-Functional Requirements
- Scale: **100M DAU, ~50 API calls/user/day**, every call passes through the limiter.
- Added latency must be tiny (a few ms) — the limiter sits on the critical path of every request.
- **Failure behavior is a requirement the designer sets:** general endpoints **fail open** (availability beats perfect enforcement); `/login` **fails closed** (unlimited attempts would defeat the limiter's purpose), softened with a local fallback.

### Clarifying Questions Asked (and why each mattered)
- *"How should the system identify the client — user ID, IP, API key?"* — determines the key format, and (later caught) that `/login` can't use user ID because the caller isn't authenticated yet.
- *"Burst or smooth?"* — directly selects the algorithm (token bucket vs leaky bucket).
- *"QPS and DAU?"* — anchors every sizing decision.
- *"Client side as well?"* — settles trust boundary (server-side only).
- *"What status when limited?"* — 429 plus headers.
- *"What if the limiter itself breaks?"* — the single most design-shaping NFR for this system: fail open vs fail closed.

### Scope
- **In scope:** per-client limiting, per-endpoint rules, burst-tolerant algorithm, distributed state, failure behavior.
- **Out of scope:** billing/quota plans, cross-region global limits (named in §15), analytics UI.

---

## 3. Capacity Estimation

```
Requests/day  = 100,000,000 DAU × 50 calls = 5 × 10^9
Avg QPS       = 5×10^9 ÷ 86,400 ≈ 58K/s   (≈50K with the ÷10^5 shortcut — both acceptable)
Peak QPS      = ~5× avg ≈ 250K–290K checks/s     (state the 5× assumption out loud)

STATE (per active client)
  Learner's estimate: 50 B × 10^8 clients = 5 GB   (arithmetic correct)
  More defensible:    100–200 B/client (key ~20-30 B + Redis per-key overhead + a 2-field
                      Hash) → ~10–20 GB. Still tiny.
  Framing correction: limiter state EXPIRES (TTL). Memory is the clients active within a
                      rate-limit window, not everyone seen across a whole day — so 5-20 GB
                      is a safe upper bound.
```
**The implication (the step people skip):** one Redis instance handles ~50–100K simple ops/sec; peak is ~250K checks/sec, and **each check is a write** (it updates the counter). Minimum 5 shards; **8–10 primaries** with headroom for spikes and uneven key distribution. **QPS forces the cluster, memory does not** — 10–20 GB fits on one node (a node comfortably holds ~25–50 GB). This is exactly Part 4's point: Sentinel gives failover but doesn't split writes; Cluster does.

**Estimation went clean this time** — no missed ÷86,400, no order-of-magnitude slip (the "count the zeros twice" habit from CS1 held). The only refinements were the optimistic per-client byte count and the "memory in a day" framing.

---

## 4. API Design

**Client-facing (when blocked):**
```
HTTP/1.1 429 Too Many Requests
Retry-After: 7
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: <epoch seconds when a token is available>
```
`Retry-After` is what stops clients hammering harder (ties to Topic 069 retry/backoff). Allowed responses also carry `X-RateLimit-Remaining`.

**Internal interface (gateway → Redis):** one call per request —
```
EVALSHA <token_bucket_script>  1  rl:{rule}:{clientId}   <capacity> <refill_rate>
  → returns { allowed (0/1), remaining_tokens, retry_after_seconds }
```
**Rules** are not supplied by the client (that would be a security hole). They live in a config store (source of truth), are **cached in each gateway instance's memory**, and refresh periodically or on change notification. A rule change reaching all gateways a few seconds late is harmless (cosmetic staleness, Topic 040).

---

## 5. Data Model

```
Key:     rl:{rule}:{clientId}        e.g.  rl:search:apikey_8841   |   rl:login:ip_203.0.113.7
Type:    Redis Hash
Fields:  tokens, last_refill
TTL:     capacity ÷ rate seconds
```
- **The rule is part of the key** — `/login` and `/search` are independent buckets with different limits.
- **`/login` is keyed by IP** (optionally + the username being attempted): the user isn't authenticated yet, so no user ID/API key exists to key on.
- **Why the TTL is lossless:** after `capacity ÷ rate` seconds with no requests the bucket would be full anyway, and a missing key is *defined* as a full bucket — so expiry loses no information, and it bounds memory to active clients.

**The Lua script (correct order — refill BEFORE the check):**
```
1. now = Redis server TIME                       ← ONE clock (the shard's), not each gateway's
2. read tokens, last_refill; if no key → tokens = capacity, last_refill = now
3. tokens = min(capacity, tokens + (now − last_refill) × rate)     ← cap applies to the TOTAL
4. if tokens ≥ 1:  tokens −= 1 ; allowed        (≥ 1, not > 1)
   else:           rejected ; retry_after = (1 − tokens) ÷ rate
5. write tokens + last_refill = now ; set TTL = capacity ÷ rate
6. return { allowed, tokens, retry_after }
```
**Why one clock:** many gateway instances have drifting clocks; each passing its own "now" gives the same client inconsistent elapsed times (even negative). Using Redis's time inside the script gives every request for a client the same clock.

**Why a Lua script:** steps 2-5 are read → decide → write. As separate network steps that is the lost-update race (Topic 026): two concurrent requests both read `tokens = 1`, both pass, both write `0`. `MULTI/EXEC` can't read-then-branch; `WATCH` is correct but retry-storms under contention. Lua is atomic *and* can branch.

---

## 6. High-Level Architecture

```
 Clients ──► Load Balancer ──► API Gateway instances (many, stateless)
                                 │   ├─ rules cache (in memory; source of truth in a config store)
                                 │   ├─ builds key rl:{rule}:{id}
                                 │   ├─ cluster-aware Redis client (slot → node map)
                                 │   └─ local "blocked-until" cache + local fallback limiter
                                 │
                                 │  one EVALSHA call (short timeout)
                                 ▼
                    ┌──────── Redis Cluster ────────┐
                    │  8–10 primary shards           │   each shard: primary + 1–2 replicas
                    │  16,384 hash slots across them │   (availability only — NOT write scaling,
                    │  token-bucket script runs HERE │    and never read counters from replicas)
                    └────────────────────────────────┘
                                 │ allowed
                                 ▼
                           App servers          (blocked → gateway returns 429 itself;
                                                 the request never reaches the app)
```
**Allowed request:** client → gateway → rule lookup (cache) → key → `EVALSHA` to the owning shard → script refills, spends a token, returns allowed → gateway forwards to the app server (with `X-RateLimit-Remaining`).
**Rejected request:** same path, script returns `allowed=0, retry_after` → gateway returns **429 + headers**, spends no token, never contacts the app server.
**Routing:** the *whole key* is hashed — `slot = CRC16(key) mod 16384`; each shard owns a slot range; the cluster-aware client routes directly and follows `MOVED` redirects when its map is stale. One key per check → trivially single-slot, no hash tags needed.

**Corrections made to the learner's first diagram** (kept for the record):
1. The "Lua logic" was drawn *inside the gateway* as read → compute → write — three separate network steps, i.e. the very race the script exists to prevent. The script runs **inside Redis**; the gateway makes one call.
2. Refill was placed *after* the check — it must come first.
3. `token > 1` should be `token ≥ 1`.
4. `token += min(cap, elapsed × rate)` can overflow the bucket (8 + min(10,5) = 13); the cap must wrap the total: `min(cap, tokens + elapsed × rate)`.
5. Missing: shard count/replicas/routing, the rules cache, the failure path + Redis timeout, multiple gateway instances, the rule in the key, and IP-keying for `/login`.

---

## 7. Deep Dives

**Deep dive 1 — atomic check-and-update.** The whole design rests on the token-bucket script being one indivisible step on one shard. Everything else (cluster, replicas, routing) exists to keep that single-key atomic operation fast and available.

**Deep dive 2 — a shard's primary dies at peak.** Promotion is not instant: Cluster needs a few seconds to detect the failure and elect a replica. During that window every call for that shard's slice (~1/10 of clients) fails or times out. Therefore:
- The Redis call needs a **short timeout** (a few ms) so the limiter never becomes the slowest part of the request path.
- On timeout, apply the policy: **general endpoints fail open**; **`/login` fails closed via a stricter local in-memory limit** (per-server counters multiply the limit by N — Topic 071 — but as a temporary fallback it is bounded, not unlimited, and keeps logins possible).
- Only that shard's clients are affected. Turn **off** `cluster-require-full-coverage` for the limiter so one shard's trouble doesn't disable limiting on the other nine.
- **State after failover:** async replication means the promoted replica may be behind, so it holds *more* tokens than it should. Clients are briefly **over-allowed, never wrongly blocked** — the error is **one-sided, bounded** (at most one full bucket of extra burst) and **self-correcting** (state is rewritten on the very next request). That bounded, one-sided, soft-state error — not "eventual consistency" as a slogan — is why it's acceptable.

**Deep dive 3 — a single very heavy client (hot key).** A partner legitimately sending 100K+/s on one API key: one key = one slot = one shard = one command thread, so **adding shards does nothing** (Part 4), and it starves every other client on that shard. Fixes, simplest first:
1. **Local token leasing** — each gateway takes a block of tokens (e.g. 100) from Redis and spends them locally; Redis sees ~1 call per 100 requests (100K/s → ~1K/s). Cost: enforcement can be off by up to (#gateways × block size) — a rounding error for a large limit.
2. **Key splitting** — divide the partner's budget across N sub-keys (`rl:partner#0…#N-1`, each 1/N of the limit); spreads across shards; cost is one sub-bucket running dry while others have tokens.
3. **Isolate** the heavy tenant on a dedicated shard (noisy-neighbour fix).

---

## 8. Scaling

- **10× (2.5M checks/s):** the gateway is stateless — add instances. The Redis tier is the real bottleneck (shared, write-bound): ~50 shards minimum, 80–100 with headroom, via live resharding (slots move — Topic 044; gate it behind a grace period so a blip doesn't trigger a huge migration).
- **Reduce Redis calls, don't only add nodes:** cache "client X is blocked until T" in the gateway so a blocked (often abusive) client's repeat requests are rejected in memory — abusive clients generate the most traffic, so this removes a large share of the load.
- **Hot keys:** see Deep dive 3.

---

## 9. Caching

- **Rules cache** in each gateway (periodic refresh or change notification) — staleness of seconds is harmless.
- **Local blocked-until cache** — short-lived, per gateway, avoids Redis for known-blocked clients.
- **Local token leasing** for hot keys (Deep dive 3).
- Redis *is* the shared state here, not a cache in front of a DB — there is no database behind it; the buckets are the source of truth for limiting, and they are reconstructible (a missing bucket = a full bucket).

---

## 10. Messaging / Async

N/A for the enforcement path — it is a synchronous check on the critical path. Optional asynchronous extras: publish rule-change notifications to gateways; stream block events to an analytics/abuse-detection pipeline (out of scope here).

---

## 11. Failure Handling

| Component | What breaks | Mitigation |
|---|---|---|
| **Shard primary dies** | That slice's checks fail for a few seconds | Replica promoted by Cluster; short timeout + fail-open (general) / local-fallback-closed (`/login`); state after failover slightly over-permissive but bounded |
| **Whole Redis layer down** | No shared counters anywhere | Same policy per endpoint; local in-memory fallback limiter; alert immediately (fallback rate is the key signal) |
| **Gateway instance dies** | Its in-flight requests fail | Stateless — the LB health-checks and routes around it; state lives in Redis, nothing is lost |
| **Rule store unavailable** | New rules can't load | Gateways keep serving the **last known rules** from memory; never fail because config is unreachable |
| **Bad rule deployed** | Legitimate users blocked en masse | Alert on a per-rule 429-rate spike; staged rollout of rule changes; fast rollback |
| **Gateway clock skew** | Inconsistent refill math | Avoided by design — the script uses the Redis shard's own clock |

---

## 12. Monitoring & Observability

```
BASELINE (the four basics — a gap repeated from CS1, called out explicitly)
  Latency  : limiter-check p99 — on the critical path, budget a few ms, alert on breach
  Traffic  : checks/sec vs the 250K peak estimate
  Errors   : Redis timeout/error rate
  Saturation: per-shard ops/sec vs the ~50-100K ceiling, memory, CPU

SPECIFIC TO A RATE LIMITER
  Fallback rate     : how often fail-open / fail-closed paths fire — the most important
                      limiter-specific signal (it means the limiter is not actually limiting)
  Allowed vs blocked: and the 429 rate PER RULE — a sudden spike = attack OR a bad rule
                      change blocking real users; you must be able to tell which
  Hot shards / hot keys, top blocked clients (abuse detection)
  Failover & replication-lag events, rules-version skew across gateways
```
Learner covered: traffic, allowed/blocked counts, hot shards, misbehaving clients. Missed (and filled in): limiter latency, Redis error rate, shard saturation, per-rule 429 spikes, fallback rate.

---

## 13. Security

- **IP spoofing via `X-Forwarded-For`.** The header is client-controlled; trusting it lets an attacker rotate fake IPs to dodge the limit. Trust only the address appended by your own load balancer / trusted proxy hop (the connection source as seen at the edge), never the raw client-supplied value.
- **Distributed attack from many IPs (botnet / credential stuffing).** Per-IP limits don't stop an attacker with thousands of IPs, each staying under the limit. Add limits on a different axis: **per target username/account**, a global limit on the endpoint, CAPTCHA/step-up after failures, and WAF/CDN-level bot protection upstream.
- **NAT collateral damage.** Many legitimate users share one IP (offices, mobile carriers); a strict per-IP limit blocks innocents. Combine per-IP with per-account limits and keep IP thresholds generous where identity isn't available.
- **Leaked API key.** A per-key limit caps the damage of a stolen key and makes abuse visible; rotate on detection.
- **The limiter as a DoS target.** Keep the rejection path cheap (local blocked-until cache, short Redis timeout) so an attacker can't exhaust the limiter itself; TLS everywhere.

*(The learner was not asked to finish this step — at their request the mentor answered it and closed the case study.)*

---

## 14. Tradeoffs — the Biggest Decisions

| Decision | Chose | Rejected | Why |
|---|---|---|---|
| Placement | Middleware inside the API gateway | Separate limiter service | No extra network hop on a 250K QPS path, nothing extra to operate. Cost: limiter deploys/scales with the gateway; other internal services can't share it. *(Correct rejection reason is hop + operational burden — **not** "single point of failure"; a separate service can be replicated.)* |
| Algorithm | Token bucket in a Lua script | Fixed window, sliding log, leaky bucket | Allows legitimate bursts with a steady average. Fixed window has the 2× boundary burst; the log costs memory per request; leaky bucket forbids all bursts |
| State store | Redis Cluster, 8–10 shards | Single node + Sentinel; per-server memory | **QPS forces it** (250K vs ~50–100K/node), not memory. Per-server counters multiply the limit by N |
| Failure behavior | Fail open generally; fail closed on `/login` + local fallback | Fail open everywhere | Availability matters for general APIs; unlimited login attempts defeats the limiter |
| Hot-key handling | Local token leasing | Adding shards | One key can't be split across shards; leasing trades a little accuracy for ~100× fewer Redis calls |

---

## 15. Alternative Designs

- **Sliding Window Counter on Redis** (two `INCR`s + weighted previous window): simpler than a Lua token bucket and cheap in memory; no burst allowance beyond the window math. Pick it when bursts don't need explicit modelling and simplicity matters most.
- **Separate rate-limiter service** (Envoy-style global rate-limit service): the right choice when many different services/gateways need one shared policy engine, accepting the extra hop.
- **Approximate local limiting with periodic synchronization** (each gateway limits locally and syncs counts asynchronously): far fewer Redis calls and lower latency, at the cost of accuracy (over-allowance up to the sync interval). A reasonable choice for very high scale or **multi-region** limits, where a single global Redis is too slow — each region enforces locally and shares coarse totals.

---

## 16. Interviewer Follow-Up Questions (rehearse these)

1. Why can't each app server keep its own counter?
2. Token bucket vs leaky bucket — when would you pick each?
3. What's the boundary-burst flaw in a fixed window, and which algorithm avoids it cheaply?
4. Why does the check have to be atomic, and why Lua instead of `MULTI/EXEC` or `WATCH`?
5. Whose clock computes "now," and why?
6. Why is the TTL on a bucket key safe? What would be unsafe?
7. Fail open or fail closed if Redis is down — does the answer differ per endpoint?
8. A shard's primary dies. What do clients on that shard experience, and what is the state after failover?
9. One client sends 100K req/s on a single key. What breaks and how do you fix it?
10. Why does a rate limiter need Redis *Cluster* — is it memory or QPS that forces it?
11. How do you rate limit `/login` before the user is authenticated?
12. How does an attacker defeat IP-based limiting, and what do you do about it?
13. How would you roll out a rule change safely, and how would you detect a bad one?
14. How would you extend this to multiple regions?

---

## 17. Evaluation Rubric Mapping (Section 2.1 dimensions)

| Dimension | How this session scored |
|---|---|
| Requirements & scoping | **Strong** — seven targeted questions across identification, burst behavior, scale, trust boundary, response, and failure behavior |
| High-level architecture | **Mixed** — core flow right (gateway → Redis shard by client key → 429), but the first diagram put the Lua logic in the gateway (re-creating the race), omitted shards/replicas/routing/rules cache/failure path, and carried 3 logic bugs |
| Data model & API | **Mixed → rebuilt** — the model and script were taught by the mentor after the learner said they didn't know it; the learner then reproduced the TTL-lossless reasoning correctly and applied the model in the diagram |
| Deep dive & bottlenecks | **Needed help** — failover-window behavior and the hot-key fix were not produced independently; the learner's *diagnoses* were right (replica may be stale; shard gets hot) but fixes had to be supplied |
| Tradeoffs & justification | **Mixed** — one of three decisions named, with a wrong rejection reason ("SPOF"); the learner did correctly derive that QPS, not memory, drives Cluster once pushed on which number mattered |
| Operational maturity | **Weak (repeat)** — monitoring again led with specific traffic counts and missed the baseline signals (latency, errors, saturation); recurs from CS1. Fail-open/closed reasoning for `/login` had the right conclusion with a muddled reason |
| Communication | **Strong** — asked directly for help when unsure ("i dont know this answer", "please help"), and challenged whether material was needed for the case study — honest signal rather than guessing past gaps |

**Estimation was clean** — the CS1 order-of-magnitude slips did not recur.

---

## 18. Cheat Sheet
See the `CS002 Rate Limiter` entry appended to `CheatSheets.md`.
