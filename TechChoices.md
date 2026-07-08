# Technology Decision Playbook — "When to Use What, and Why Not the Others"

> Created 2026-07-08 at the learner's request. This file grows after every lesson that touches a
> technology category. Before any case study, review the relevant sections here.
> The interview goal is never "name the tech" — it's "name the tech, the reason, AND the rejected alternative."

**How to use in an interview — the 3-part sentence:**
> "I'll use **[tech]** because **[the specific problem property]**. I considered **[alternative]** but rejected it because **[what it trades away]**."

---

## The 3 Questions Before Any Tech Choice

```
1. What problem am I actually solving?   (reads? writes? search? consistency? scale?)
2. What does this tech trade away?       (every tool gives something and takes something)
3. What happens if this choice is wrong at 10×?  (can I migrate? blast radius?)
```

---

## Databases — The Decision Table

| Database | Pick it when | Do NOT pick it when | Real users |
|----------|-------------|--------------------|-----------| 
| **PostgreSQL / MySQL** (relational) | You need ACID transactions, joins, structured data with relationships. Money, orders, inventory, bookings. | Data is PB-scale; write throughput exceeds what one primary can take (~10–50K writes/sec); schema changes constantly | Stripe (payments), Amazon (orders), Uber (trips core) |
| **MongoDB** (document) | Flexible/nested schema, product catalogs, user profiles, CMS content. Objects that are read/written whole. | You need multi-record transactions or heavy cross-entity joins. **The classic disaster: payments on Mongo** | eBay (catalog), forums, content platforms |
| **Cassandra / DynamoDB** (wide-column) | Massive write throughput (100K+/sec), known access patterns, time-series-ish data: messages, feeds, sensor data | You need ad-hoc queries, joins, or strong transactions. Query patterns unknown upfront | Discord (messages), Netflix (viewing history), Amazon (cart) |
| **Redis** (in-memory KV) | Hot reads, sessions, leaderboards, rate-limit counters, caching layer | As the primary durable store — it's memory-first; a crash can lose recent writes | Twitter (timeline cache), everyone (sessions) |
| **Elasticsearch** (search) | Full-text search, fuzzy matching, log queries, faceted filters | As the source of truth — it's an *index built from* your real DB, eventual-consistent | Uber Eats (restaurant search), GitHub (code search) |
| **S3 / object storage** (blob) | Images, videos, backups, anything PB-scale and read-whole | Low-latency queries on structured fields — it has no query engine | Netflix (video), Dropbox (files), Instagram (photos) |

**The golden default:** *Start with PostgreSQL. Leave it only when a specific number forces you out* — writes exceed one primary (~10s of K/sec), data exceeds a few TB of hot set, or the data is genuinely unstructured blobs. Saying this in an interview signals maturity; jumping to Cassandra for a 1K-QPS app signals buzzword-matching.

**Interview-ready "why not" phrases:**
- *"Not Mongo here — this is money; I need multi-row ACID transactions."*
- *"Not Cassandra here — our query patterns are ad-hoc and the scale (2K QPS) doesn't justify losing joins."*
- *"Not Postgres for the messages table — 500K writes/sec exceeds a single write primary; Cassandra partitions writes across nodes."*
- *"Redis for the leaderboard, but it's a cache — the source of truth stays in Postgres."*

---

## Estimation → Tech Choice (bridge from Topic 003)

| The number says | The tech conclusion |
|-----------------|--------------------|
| Read:write 100:1 | Redis/Memcached cache in front of the DB; read replicas |
| Read:write ≈1:1, high write QPS | Wide-column store (Cassandra) or sharded SQL — cache won't save you |
| Storage in PBs | S3-style object storage + metadata in a DB; never raw RDBMS |
| p99 latency < 50ms globally | CDN + regional deployments; single-region won't make it |
| Exact money movement | Relational DB + idempotency keys; nothing else is acceptable |

---

## Sections added by later topics

<!-- Topic 008 will add: TCP vs UDP decision box -->
<!-- Topic 015 will add: WebSockets vs SSE vs polling decision box -->
<!-- Topic 017 will add: L4 vs L7 load balancer decision box -->
<!-- Topic 059-060 will add: Kafka vs RabbitMQ vs SQS decision box -->
<!-- Topic 031 will consolidate the full DB decision framework -->
