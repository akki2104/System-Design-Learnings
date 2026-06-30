# Revision — Topic 003: Back-of-the-Envelope Estimation

Active recall only. No re-reading. Answer each question from memory before expanding.

---

## 30-Second Elevator Explanation

*Recite this from memory before any estimation section in an interview:*

> "I'll start by estimating scale. I'll assume [DAU], [writes/user/day], [read:write ratio], and [object size]. That gives me [write QPS], [peak QPS], [storage/day], and [bandwidth]. The key implication is [one sentence about design constraint]."

---

## Active-Recall Q&A

<details>
<summary>Q1: What are the 4 numbers you always compute in estimation?</summary>

1. Write QPS (average)
2. Peak QPS (= avg × 2–10×)
3. Storage/day and storage/N years
4. Bandwidth

</details>

<details>
<summary>Q2: Formula for average write QPS?</summary>

`Write QPS = DAU × writes/user/day ÷ 86,400`

Shortcut: 86,400 ≈ 10^5. So `Write QPS ≈ DAU × writes/day × 10^-5`

</details>

<details>
<summary>Q3: Why use average QPS for storage but peak QPS for bandwidth?</summary>

**Storage** is about how much data lands on disk per day — governed by how much data actually arrives, which is average rate, not the peak burst.

**Bandwidth** is about how fat your network pipes need to be — that's a capacity question, so you design to the worst case (peak).

Memory hook: *Storage = accounting (what arrived). Bandwidth = capacity planning (worst case).*

</details>

<details>
<summary>Q4: WhatsApp (1:1 read:write) vs Twitter (100:1). What different design conclusions?</summary>

**Twitter (100:1 read-heavy):**
- Cache is extremely effective — same tweet read 100× = huge cache hit rate
- Read replicas solve most read scaling problems
- Write load is manageable

**WhatsApp (1:1 balanced):**
- Every message is read roughly once → caching barely helps
- Bottleneck is write throughput
- Needs horizontal write scaling = sharding (e.g., by user/conversation ID)
- Object storage for media; no RDBMS at petabyte scale

</details>

<details>
<summary>Q5: 50M DAU, 10 writes/user/day, 1KB objects, replication 3. Storage/day in GB?</summary>

```
writes/day = 50M × 10 = 500M = 5 × 10^8
obj size   = 1KB = 10^3 bytes
replication = 3

storage/day = 5 × 10^8 × 10^3 × 3
            = 15 × 10^11
            = 1.5 × 10^12 bytes
            = 1,500 GB/day  (~1.5 TB/day)
```

</details>

<details>
<summary>Q6: Convert 6 × 10^13 bytes to TB.</summary>

1 TB = 10^12 bytes

6 × 10^13 ÷ 10^12 = 60 TB

</details>

<details>
<summary>Q7: Peak multiplier — when 2× vs 10×?</summary>

- **2–3×** — stable, predictable traffic (enterprise SaaS, B2B tools)
- **5×** — consumer product with moderate event spikes
- **10×** — consumer product with extreme event spikes (live sports, viral moments, product launches)

Always state which you're using and why.

</details>

<details>
<summary>Q8: What does PB-scale storage tell you about database choice?</summary>

Relational databases (Postgres, MySQL) are not designed for petabyte-scale raw storage. PB-scale means:
- Use object/blob storage (S3-style) for bulk data
- Use tiered archival (hot → warm → cold → glacier)
- Keep only metadata and indexes in a relational DB
- Consider columnar storage (Redshift, BigQuery) if analytical queries are needed

</details>

---

## Key Diagrams

```
ESTIMATION FLOW
───────────────────────────────────────────────────
DAU × writes/day
      │
      ├──÷ 86,400──► Avg Write QPS ──× 2-10x──► Peak QPS
      │                                               │
      │                                         × payload──► Bandwidth
      │
      └──× obj_size × replication──► Storage/day ──× 365 × N──► Storage/Nyears

UNIT LADDER
───────────────────────────────────────────────────
bytes → KB(10³) → MB(10⁶) → GB(10⁹) → TB(10¹²) → PB(10¹⁵)
          strip 3 zeros per step up
```

---

## My Weak Areas (from lesson 2026-06-30)

- **Used peak QPS for storage** — storage must use average writes, not peak
- **Forgot ×365 for multi-year** — multi-year = /day × 365 × N, not just × N
- **Generic implication** — "need more servers" is not enough; say *what type of problem* and *why*

---

## Past Mistakes

See [InterviewMistakes.md](../InterviewMistakes.md) — entries dated 2026-06-30.
