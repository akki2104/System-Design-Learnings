# Topic 003 — Back-of-the-Envelope Estimation

**Module:** 0 — Orientation & Mental Models
**Status:** Completed
**Date:** 2026-06-30
**Confidence:** 3/5
**Difficulty:** Easy-Medium

---

## 1. Why This Topic Exists

Every design decision — how many servers, which database, whether you need a cache — is validated or invalidated by scale. Without numbers, you're drawing boxes with no idea if they'll survive real load. Estimation is how you size the problem before you solve it.

---

## 2. Real Production Problem

In 2012, Instagram added video uploads. One engineer estimated "maybe 100 videos a day" — no math, just gut feel. Within 24 hours users uploaded 5 million videos. Storage and bandwidth collapsed; the feature was pulled days later. A 5-minute calculation would have revealed the mismatch between intuition and reality.

---

## 3. Simple Intuition

Before building a highway, you estimate traffic volume. Before designing a water system, you estimate flow rate. Estimation in system design is the same act: size the load before you choose the pipes.

---

## 4. Core Concepts

### The 4 Numbers (always compute these)

| # | Metric | Formula |
|---|--------|---------|
| 1 | Write QPS (avg) | DAU × writes/user/day ÷ 86,400 |
| 2 | Peak QPS | avg QPS × 2–10× |
| 3 | Storage/day | writes/day × avg_object_size × replication_factor |
| 4 | Bandwidth | peak QPS × avg_payload_size |

### Critical rule — peak vs average

```
Peak QPS   → server/network capacity planning  (design to peak)
Avg QPS    → storage calculation               (data arrives at average rate)
```

This is the most commonly confused point. Storage is how much data *lands on disk per day* — governed by average arrival rate, not peak.

### Read:Write ratio — how to get reads/user/day

You reason from product behavior, not a formula:

| System | Read:Write | Why |
|--------|-----------|-----|
| Twitter/feed | ~100:1 | People consume far more than they produce |
| URL shortener | ~100:1 | Many redirects per short URL created |
| WhatsApp | ~1:1 | Every message sent is read roughly once |
| Dropbox | ~1:1 | Sync both ways |

If the interviewer doesn't give it, state your assumption: *"I'll assume 100:1 since it's a consumption-heavy feed."*

### The implication rule

The number's job is to tell you *what kind of problem* you have:

```
Read-heavy  (>10:1)   → cache layer is primary solution
Write-heavy (≈1:1)    → sharding/partitioning; caching helps less
PB-scale storage      → object/blob storage (S3-style), not RDBMS
```

---

## 5. Visualization

```
ESTIMATION FLOW

DAU × writes/day
      │
      ▼
writes/day ──÷ 86,400──► Avg Write QPS ──× 2-10──► Peak Write QPS
      │                                                     │
      │                                                     ▼
      │                                              × payload ──► Bandwidth
      │
      ▼
× object_size × replication ──► Storage/day ──× 365 × N──► Storage/Nyears
```

---

## 6. Animation Frames — Twitter Worked Example

```
Frame 1: Start with users
  100M DAU × 2 tweets/day = 200M tweets/day

Frame 2: Convert to QPS
  200M ÷ 10^5 (≈86,400) = 2,000 writes/sec (avg)

Frame 3: Peak
  2,000 × 5 = 10,000 writes/sec (peak)

Frame 4: Read QPS (100:1 ratio)
  10,000 × 100 = 1,000,000 reads/sec (peak)

Frame 5: Storage
  200M × 300 bytes × 3 = 1.8×10^11 = 180 GB/day

Frame 6: 5-year storage
  180 GB × 365 × 5 ≈ 180 × 2,000 = 360,000 GB = 360 TB

Frame 7: Bandwidth
  10,000 × 300 = 3,000,000 bytes/sec = 3 MB/s (write)
  1,000,000 × 300 = 300 MB/s (read)
```

---

## 7. Architecture Diagram

Estimation output feeds directly into design decisions:

```
┌────────────────────────────────────────────────────┐
│            ESTIMATION OUTPUT                       │
│  Write QPS: 10K/s   Read QPS: 1M/s                │
│  Storage: 360 TB/5yr  Bandwidth: 300 MB/s read     │
└──────────────┬─────────────────┬───────────────────┘
               │                 │
               ▼                 ▼
    Read-heavy (100:1)      PB-scale storage
    → Cache layer           → Object store (S3)
    → Read replicas         → Tiered archival
```

---

## 8. Real Engineering Examples

- **Twitter:** ~500K tweets/day peak during major events; storage tiered to cold storage after 30 days
- **WhatsApp:** 100B+ messages/day; write-balanced, sharded by phone number; messages deleted server-side after delivery
- **YouTube:** ~500 hours of video uploaded per minute; storage measured in exabytes; encoding multiplies raw upload size ~10×

---

## 9. Industry Examples

- **Google:** Estimates used in design docs before any infrastructure is provisioned
- **Amazon:** Capacity reviews require QPS + storage projections before any system is approved
- **Stripe:** Estimates payments/sec by country to size regional deployments

---

## 10. Tradeoffs

| Choice | Benefit | Cost |
|--------|---------|------|
| Use peak QPS for everything | Safe, never under-provisioned | Wastes money on storage (storage is average, not peak) |
| Use 2× peak multiplier | Conservative, cheap | Under-provisioned for event spikes |
| Use 10× peak multiplier | Safe for viral events | Expensive, may be overkill |
| Round aggressively (10^5 for 86,400) | Fast, interview-friendly | ~15% error — always acceptable for estimation |

---

## 11. Complexity Analysis

Estimation itself is O(1) — a handful of multiplications. The value is in the signal it produces, not the computation.

---

## 12. Scaling Considerations

```
10×  traffic  → revisit server count, check peak QPS headroom
100× traffic  → storage tier changes (GB→TB→PB), sharding becomes mandatory
1000× traffic → multi-region, CDN becomes non-optional, cold storage archival
```

---

## 13. Failure Scenarios

- **Under-estimating peak multiplier** → servers fall over during traffic spikes (Instagram video launch)
- **Using peak for storage** → over-budget on disk by 5–10×; procurement delays
- **Wrong object size** → off by 700× if you confuse 300 bytes (tweet) with 300KB (photo)
- **Forgetting replication factor** → actual disk usage is 3× estimated; capacity shortfall

---

## 14. Monitoring

Metrics to watch that validate your estimates:
- Actual QPS (compare to estimated)
- Disk usage growth rate/day (compare to estimated storage/day)
- Bandwidth utilization (compare to estimated)
- Cache hit rate (validates read-heavy assumption)

---

## 15. Observability

- Alert when actual QPS exceeds 80% of peak design capacity
- Dashboard: estimated vs actual for QPS, storage growth, bandwidth
- Capacity planning review triggered when 6-month storage projection > 80% of provisioned

---

## 16. Security

N/A for estimation itself. However: storage estimates should account for encrypted data (encryption overhead ~10–20%), and bandwidth estimates should include TLS handshake overhead for HTTPS-heavy workloads.

---

## 17. Interview Discussion

Estimation comes in Step 2 of the 7-step framework, right after requirements. Interviewers evaluate:
- Do you state assumptions before calculating?
- Do you use powers of 10 comfortably?
- Do you arrive at the right order of magnitude?
- Do you extract design implications from the numbers?
- Do you finish in ≤5 minutes and move on?

The last point matters: candidates who spend 20 minutes perfecting estimation don't get to the design. Move on at "good enough."

---

## 18. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Peak QPS for storage (most common) | Storage = average writes × size × replication |
| Forgetting ×365 for multi-year | Multi-year = /day × 365 × N years |
| No implication after numbers | Always add one sentence: "this means X because Y" |
| Wrong object size | Tweet=300B, photo=1-3MB, message=100B, video=50-100MB/min |
| Spending >5 minutes | Set a mental timer; rough is better than perfect |
| Skipping assumptions | Always say DAU, read:write ratio, object size out loud |

---

## 19. Advanced Topics

- **Hotspot estimation:** 80/20 rule — 20% of content gets 80% of reads. Cache sizing = 20% of working set.
- **Encoding overhead:** Video/image compression ratios; raw video ~10× encoded video
- **Network vs disk bandwidth:** Internal datacenter bandwidth (~10–100 Gbps) vs cross-region (much lower, metered)
- **Cost estimation:** ~$0.02–$0.05/GB/month on S3; storage cost = an interview differentiator in 2025–2026

---

## 20. Real Interview Questions

1. "Design Twitter. Start by estimating the scale." (Twitter, Meta, Google — universal)
2. "How many servers does Google Search need?" (Google — Fermi estimation)
3. "WhatsApp sends 100B messages/day. Estimate storage requirements." (Meta)
4. "Estimate the storage required for a photo-sharing app with 500M DAU." (Instagram-style)
5. "How much bandwidth does Netflix need for 1M concurrent video streams?" (Netflix)
6. "If each Uber ride generates a location ping every 5 seconds, estimate the write QPS for 1M active rides." (Uber)
7. "A URL shortener has 100M DAU creating 1 URL/day each. How big is the database after 5 years?" (Amazon, general)

---

## 21. Exercises

1. **Twitter:** Confirm the numbers from the lesson from scratch without notes.
2. **WhatsApp:** 200M DAU, 50 msgs/day, 100 bytes/msg, 1:1 ratio, replication 3. All 4 numbers.
3. **YouTube:** 2B DAU, 5 videos watched/day (100MB each), 0.1 video uploaded/day (500MB raw). Estimate read/write QPS, storage/day.
4. **Uber:** 10M active rides at peak, location ping every 5 sec, 50 bytes/ping. Write QPS?
5. **What changes?** You're told WhatsApp adds media: 10% of messages are 2MB photos. Recalculate storage/day. What's the dominant term?

---

## 22. Revision Questions

1. What are the 4 numbers you always compute in estimation?
2. Formula for write QPS given DAU and writes/user/day?
3. Why do you use average QPS for storage but peak QPS for bandwidth?
4. WhatsApp (1:1 ratio) vs Twitter (100:1) — what different design conclusions do you draw?
5. A system has 50M DAU, 10 writes/user/day, 1KB objects, replication 3. Storage/day in GB?
6. Convert `6 × 10^13 bytes` to TB.
7. What multiplier do you use for peak QPS, and when do you pick 2× vs 10×?
8. What does PB-scale storage immediately tell you about database choice?

---

## 23. Cheat Sheet

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

## 24. Summary

- **Estimation = sizing the problem before solving it.** Wrong order = wrong design.
- **Four numbers cover everything:** write QPS, peak QPS, storage, bandwidth.
- **Average vs peak matters:** storage uses average writes; bandwidth uses peak QPS.
- **The implication sentence** is more important than the numbers themselves.
- **Read:write ratio** tells you whether to invest in caching (read-heavy) or write scaling (balanced/write-heavy).
- **Round aggressively.** 86,400 ≈ 10^5 is fine. Order of magnitude is what matters.
- **5 minutes maximum.** Spending longer signals poor time management and kills the design phase.

> **You now can:** compute QPS, storage, and bandwidth for any system in under 5 minutes and state what the numbers mean for the design.
