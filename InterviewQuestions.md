# Interview Question Bank

100+ real-style questions, growing after every lesson and case study.
Triple-indexed: by company, by difficulty, by topic.

---

## Index by Difficulty

### Easy
<!-- Added here -->

### Medium
<!-- Added here -->

### Hard
<!-- Added here -->

---

## Index by Topic
<!-- Sections added per topic as lessons complete -->

### Redis (Track R01) — 20 questions, provenance-labelled

The full question set lives in
[`Topics/R01_Redis_Consolidated_Module.md`](Topics/R01_Redis_Consolidated_Module.md) **§20**, split into:

- **✅ Reported / commonly documented (10)** — Redis vs Memcached · design a distributed rate limiter ·
  implement a distributed lock · design a leaderboard · what happens if Redis goes down · is Redis
  single-threaded and why it matters · how Cluster routes a key · Redis vs Kafka · cache-DB consistency ·
  RDB vs AOF and the crash loss window.
- **⚙️ Practice (10)** — generated for this curriculum, testing understanding rather than command recall.
- Plus the **follow-up chains**, **8 interview traps**, and **7 trade-off questions** in the same section.

> Labelling convention introduced here and worth reusing: ✅ = documented in published interview-experience
> write-ups and established prep collections as commonly asked; ⚙️ = generated for practice. Neither is a
> guarantee of what a specific interviewer will ask.

### Case Study #1 — TinyURL / URL Shortener — 12 questions

The full follow-up question list lives in
[`CaseStudies/CS001_TinyURL.md`](CaseStudies/CS001_TinyURL.md) **§16**, covering: auto-increment vs a
partitioned DB, the cache-miss redirect flow, why Snowflake's timestamp bits go first, sharding vs
replication at the cache layer, where rebalanced DynamoDB data actually comes from, 302 vs 301, the
security concern unique to URL shorteners, stampede detection vs generic high traffic, load-balancer
HA, Postgres-vs-DynamoDB at today's scale, why a conditional write beats a relational check-then-insert,
and the CDN-caching tradeoff.


---

## Index by Company

### Google
<!-- Added here -->

### Meta
<!-- Added here -->

### Amazon
<!-- Added here -->

### Microsoft
<!-- Added here -->

### Uber / DoorDash
<!-- Added here -->

### Stripe
<!-- Added here -->

### Netflix / Airbnb
<!-- Added here -->

### Datadog / Snowflake
<!-- Added here -->

### LinkedIn / Bloomberg / Adobe / Others
<!-- Added here -->

---

## Full Question List

<!-- Format:
- Q: <question>
  - Company(s): ...
  - Difficulty: Easy / Medium / Hard
  - Topics: ...
  - Follow-ups: <deeper probes>
  - Hidden traps: <what candidates miss>
  - Expected discussion: <what a strong answer covers>
  - Evaluation criteria: <how it's scored>
-->
