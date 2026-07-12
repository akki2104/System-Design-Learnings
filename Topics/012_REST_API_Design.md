# Topic 012: REST API Design

**Module:** 1 — Networking & Communication Foundations
**Completed:** 2026-07-12
**Confidence:** 4/5

---

## 1. Why This Topic Exists

Every system design interview eventually needs an API contract between client and server. REST is the default convention — get it wrong (verbs in URLs, wrong status codes, offset pagination on a write-heavy feed) and you signal inexperience even if your architecture is solid.

---

## 2. What REST Actually Is

REST (Representational State Transfer) is a set of conventions, not a protocol. Two rules matter most in interviews:

**Rule 1 — Resources, not actions.** URLs name *nouns* (things), not verbs (actions). The HTTP method conveys the action.
```
BAD:  GET /getUser?id=5        GET /deleteTweet?id=9
GOOD: GET /users/5             DELETE /tweets/9
```

**Rule 2 — Statelessness.** The server holds no memory of previous requests. Every request carries everything needed to process it (e.g., an auth token). This is what makes REST APIs horizontally scalable — any server instance can handle any request, no session affinity required.

---

## 3. HTTP Methods, Safe & Idempotent

| Method | Purpose | Safe? | Idempotent? |
|--------|---------|-------|-------------|
| GET | Read a resource | Yes | Yes |
| POST | Create a resource | No | No |
| PUT | Replace a resource entirely | No | Yes |
| PATCH | Partially update a resource | No | No (usually) |
| DELETE | Remove a resource | No | Yes |

- **Safe** = doesn't change server state (read-only).
- **Idempotent** = calling it N times produces the **same final state** as calling it once. This is about **state**, not the HTTP response code.

**Common trap:** DELETE returns 200/204 the first time and 404 on repeat calls — different responses, but identical end state (resource doesn't exist either way). That's still idempotent. Idempotency is a state guarantee, not a response guarantee.

This maps directly to retry-safety: an idempotent endpoint (e.g., `POST /payments/{idempotencyKey}` designed to behave like PUT) can be safely retried by a client after a timeout, without risking a duplicate charge.

---

## 4. URL Design Conventions

| Rule | Bad | Good |
|------|-----|------|
| Plural nouns for collections | `/tweet` | `/tweets` |
| Collection before ID | `/{userId}/users` | `/users/{userId}` |
| Nested sub-resources express ownership | `/tweets?userId=5` | `/users/5/tweets` |
| Actions on a resource → model as a sub-resource, not a verb/field patch | `PATCH /tweets/9/like` | `POST /tweets/9/likes` |

The "like" case is the subtlest one: a like is itself a resource (created, deleted, has an owner) — not a field update on the tweet. Model it as its own collection:
```
POST   /tweets/{tweetId}/likes            → like
DELETE /tweets/{tweetId}/likes/{userId}   → unlike
```

---

## 5. Status Codes

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Successful GET/PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Malformed input |
| 401 | Unauthorized | Not authenticated — "who are you?" |
| 403 | Forbidden | Authenticated but not permitted — "I know you, you can't do this" |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate create |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Server Error | Server-side bug |

**401 vs 403** is a favorite interview gotcha: 401 = no/invalid credentials, 403 = valid credentials but insufficient permission.

---

## 6. Versioning

| Approach | Example | Tradeoff |
|----------|---------|----------|
| URL path (default choice) | `/v1/users/{id}` | Explicit, cacheable, easy to route at gateway/LB |
| Header | `Accept: application/vnd.api.v2+json` | Cleaner URLs, harder to test/debug manually |
| Query param | `/users/{id}?version=2` | Rarely used, easy to omit by accident |

Default interview answer: **URL path versioning.**

---

## 7. Pagination — Offset vs Cursor

**Offset-based:**
```
GET /tweets?offset=20&limit=10
```
Breaks under concurrent writes on a live feed: if a new row is inserted while a user pages through results, everything below it shifts by one position — the user sees a **duplicate** (offset-based "positions" moved) or **skips** an item (on deletes). This isn't a performance problem — it's a **correctness** problem caused by anchoring to a row count instead of a fixed reference point.

**Cursor-based:**
```
GET /tweets?cursor=tweet_82&limit=10
```
The cursor anchors to an actual unique, sortable ID (e.g., `tweet_id` or timestamp), not a position:
```sql
SELECT * FROM tweets WHERE tweet_id < 82 ORDER BY tweet_id DESC LIMIT 10
```
New inserts/deletes elsewhere in the table don't affect "the 10 rows after tweet_82" — the anchor never shifts. This is why every high-write feed (Twitter, Slack, Stripe API) uses cursor-based pagination.

**Bonus:** cursor-based is also faster at depth — `OFFSET 10000` forces the DB to scan and discard 10,000 rows; a cursor query jumps straight to the indexed position.

---

## 8. Decision Box — REST vs Alternatives

| Use REST when | Avoid REST when |
|---|---|
| Public API, broad client compatibility (web/mobile/3rd-party) | Need real-time bidirectional push → WebSockets |
| Resource-oriented domain (CRUD-shaped) | Client needs flexible/nested data shapes efficiently → GraphQL |
| HTTP caching semantics matter (GET is cacheable) | Internal service-to-service, need low latency + strict typed contracts → gRPC |

*(Full gRPC and GraphQL comparisons come in Topics 013–014.)*

---

## 9. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| ID before collection noun in URL (`/{id}/users`) | Collection first: `/users/{id}` |
| Singular collection nouns (`/tweet`) | Plural: `/tweets` |
| Modeling an action as a field patch (`PATCH .../like`) | Model as a sub-resource: `POST .../likes` |
| Idempotency defined by "same response" | Idempotency = same **final state**, response codes can differ |
| Offset pagination on a high-write feed | Cursor-based pagination |

---

## 10. Revision Questions
See `Revision/Revision_012.md`.

## 11. Summary
- REST = resources as nouns in URLs, HTTP method conveys the verb, statelessness for scalability.
- Idempotency is about final state, not response code — critical for retry-safe/payment APIs.
- URL conventions: plural collections, collection-before-ID, actions modeled as sub-resources.
- Cursor-based pagination is a correctness fix (not just a perf one) for high-write feeds.
- URL path versioning is the default interview answer.
