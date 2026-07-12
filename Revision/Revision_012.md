# Revision — Topic 012: REST API Design

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-12

---

## Q1. What are REST's two core rules?

<details>
<summary>Answer</summary>

1. **Resources, not actions** — URLs name nouns; the HTTP method conveys the verb (`DELETE /tweets/9`, not `GET /deleteTweet?id=9`).
2. **Statelessness** — the server holds no memory between requests; every request carries everything needed (e.g., auth token), enabling any server instance to handle any request.

</details>

---

## Q2. Is DELETE idempotent? Explain using the response codes.

<details>
<summary>Answer</summary>

Yes. The first DELETE returns 200/204, subsequent calls on the same resource return 404 — **different responses**, but the **final state is identical** (resource doesn't exist either way). Idempotency is a state guarantee, not a response guarantee.

</details>

---

## Q3. Why does offset-based pagination break on a live, high-write feed, but cursor-based doesn't?

<details>
<summary>Answer</summary>

Offset-based pagination anchors to a **row count** ("skip N rows"). If a new row is inserted while paging, every row below it shifts by one position — causing duplicates (on insert) or skips (on delete) in what the user sees next.

Cursor-based pagination anchors to a fixed, unique, sortable ID (e.g., `tweet_id < 82`) — inserts/deletes elsewhere don't change what "after tweet_82" means, so no duplicates or skips occur.

</details>

---

## Q4. How would you model "liking a tweet" as a REST endpoint, and why not PATCH?

<details>
<summary>Answer</summary>

`POST /tweets/{tweetId}/likes` (and `DELETE /tweets/{tweetId}/likes/{userId}` to unlike).

A like is its own resource — it has an owner, can be created and deleted — not a field update on the tweet itself. Modeling it as `PATCH /tweets/{id}/like` incorrectly treats a create/delete action as a partial field update.

</details>

---

## Q5. What's the difference between 401 and 403?

<details>
<summary>Answer</summary>

**401 Unauthorized** — no or invalid credentials ("who are you?").
**403 Forbidden** — valid credentials, but insufficient permission for this resource ("I know who you are, you can't do this").

</details>

---

## 30-Second Elevator Pitch

> REST models resources as nouns in URLs with the HTTP method conveying the action, and is stateless so any server can handle any request — the basis of horizontal scalability. Idempotency means repeated calls produce the same final state, not necessarily the same response — critical for safely retrying payment APIs. URL conventions: plural collection nouns, collection-before-ID, and actions like "like" modeled as their own sub-resource rather than a field patch. For pagination on high-write feeds, cursor-based beats offset-based because it anchors to a fixed ID instead of a row count, avoiding duplicate/skipped results under concurrent writes. Default versioning approach: URL path (`/v1/...`).

---

## Weak Areas to Watch

- Idempotency = state-based, not response-code-based (was initially described by response code)
- URL ordering: collection noun always precedes the ID
- "Like" as a sub-resource, not a PATCH field
