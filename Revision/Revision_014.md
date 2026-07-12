# Revision — Topic 014: GraphQL

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-12

---

## Q1. What are over-fetching and under-fetching, and how does GraphQL fix each?

<details>
<summary>Answer</summary>

**Over-fetching:** a REST endpoint returns a full object (e.g., all user fields) when the client only needs a few. **Under-fetching:** a REST client needs multiple sequential round trips to assemble related/nested data (user → posts → comment counts).

GraphQL fixes both with a single endpoint where the client specifies the exact fields and nesting it needs in one request — the response shape matches the query exactly, in one round trip.

</details>

---

## Q2. Why is GraphQL harder to cache at the HTTP layer than REST?

<details>
<summary>Answer</summary>

REST's `GET /users/5` is cacheable by URL — browsers, CDNs, and reverse proxies can cache it for free. GraphQL uses a single `POST /graphql` endpoint with a variable query body — there's no fixed URL to key a cache on, so HTTP-layer caching doesn't apply. GraphQL needs its own caching strategy (persisted queries, client-side caches like Apollo/Relay).

</details>

---

## Q3. GraphQL is supposed to reduce round trips — where can a new N+1 problem reappear, and how is it fixed?

<details>
<summary>Answer</summary>

It moves from the client to the server: if a resolver (e.g., `commentCount`) runs once per item naively, N items → N+1 DB queries for a single GraphQL request.

**Fix:** batching (the DataLoader pattern) — queue individual resolver requests within the same request tick, then fire one combined query (`WHERE post_id IN (...)`) instead of one query per item. Same principle as write-batching/Nagle's algorithm: collect small requests, fire one bulk operation.

</details>

---

## Q4. What's the classic use case that makes GraphQL worth its added complexity?

<details>
<summary>Answer</summary>

A product with multiple diverse client types (web, iOS, Android, smart TV) each needing a different slice of the same underlying data. Facebook, who invented GraphQL, is the canonical example — each client fetches exactly the fields it needs from a shared graph of data.

</details>

---

## Q5. What are GraphQL's three operation types, and what REST equivalent does each map to?

<details>
<summary>Answer</summary>

- **Query** → GET (read)
- **Mutation** → POST/PUT/DELETE (write)
- **Subscription** → similar to gRPC server streaming (real-time push after the client subscribes)

</details>

---

## 30-Second Elevator Pitch

> GraphQL fixes REST's over-fetching and under-fetching by letting the client specify exactly which fields, nested and all, it wants in one request to a single endpoint. It's strongly typed via a schema, with a resolver function behind each field. But it doesn't eliminate complexity — it moves the N+1 problem from client round trips into server-side per-item DB queries, which needs explicit batching (DataLoader) to fix, and it loses REST's free HTTP-layer caching since there's no fixed URL to key on. Best used when a product has many different client types needing different slices of the same data — Facebook's original use case.

---

## Weak Areas to Watch

- The N+1 problem is not eliminated by GraphQL — it moves server-side, needs the DataLoader/batching pattern to actually fix
- HTTP caching doesn't work the same way — single POST endpoint, no cacheable URL
