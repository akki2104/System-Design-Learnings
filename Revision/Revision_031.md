# Revision — Topic 031: Choosing a Database

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-30

---

## Q1. What are the six questions in the database-choice decision framework?

<details>
<summary>Answer</summary>

1. Access pattern (known/fixed vs ad-hoc/evolving)
2. Read:write ratio
3. Relationships/joins needed?
4. Consistency requirement (strong vs eventual)
5. Schema shape (flat, nested, wide/sparse)
6. Scale ceiling (will one machine become insufficient?)

</details>

---

## Q2. What is polyglot persistence, and why does naming it explicitly matter in an interview?

<details>
<summary>Answer</summary>

Using multiple databases in one system, each solving a different sub-problem, rather than one database for everything (e.g., Postgres for orders, Redis for sessions, Elasticsearch for search, S3 for images). Naming this explicitly signals you're matching tools to specific requirements rather than reciting one database's feature list — the difference between a strong and a shallow answer.

</details>

---

## Q3. A system needs both "store document content" and "full-text search across documents." Why is using one document store for both a mistake?

<details>
<summary>Answer</summary>

These are two different sub-problems. A document store's secondary indexes handle exact-match/range queries on structured fields well, but aren't built for full-text, relevance-ranked search across large content volumes at scale. The correct answer is polyglot: a document store for content, a dedicated search engine (Elasticsearch) for search.

</details>

---

## Q4. For a real-time collaborative document editor, why isn't "which database should I use" the most important question?

<details>
<summary>Answer</summary>

The hardest part of that system is merging concurrent edits from multiple users without conflicts or lost keystrokes — a real-time sync problem (WebSockets) plus a conflict-resolution problem (Operational Transform/CRDTs), solved at the application/protocol layer. No database choice solves this; the database only persists the already-resolved state. A strong answer flags this explicitly rather than only reasoning about storage.

</details>

---

## Q5. What's the standard interview format for justifying a database choice?

<details>
<summary>Answer</summary>

"For [this specific data/access pattern], I'll use [X] because [specific property]. I considered [Y] but rejected it because [what Y trades away]. This matters because [business impact]."

</details>

---

## 30-Second Elevator Pitch

> Choosing a database is a synthesis skill: walk through access pattern, read:write ratio, relationships, consistency needs, schema shape, and scale ceiling, then map the answers to Relational, Wide-Column, Document, Key-Value, or Graph. Real systems rarely use just one database — polyglot persistence means naming multiple stores, each for a different sub-problem, which is harder to actually apply under pressure than to state as a principle. The standard justification format names the property, rejects an alternative with its specific cost, and states the business impact. The biggest trap is picking a database by familiarity or hype instead of matching the actual requirement — and for some systems, the hardest problem isn't a database choice at all.

---

## Weak Areas to Watch

- Applying polyglot persistence under pressure — the instinct to pick "one database for everything" can resurface even right after learning the principle
- Recognize when a system's hardest problem lives outside the database layer entirely (e.g., real-time conflict resolution)
