# Topic 014: GraphQL

**Module:** 1 — Networking & Communication Foundations
**Completed:** 2026-07-12
**Confidence:** 4/5

---

## 1. Why This Topic Exists

REST has two structural pain points at scale: over-fetching (returning full objects when only a few fields are needed) and under-fetching (needing multiple round trips for related/nested data). GraphQL is the industry answer, especially when many different client types (web, iOS, Android, TV) need different slices of the same data.

---

## 2. The Two Problems GraphQL Solves

**Over-fetching:** `GET /users/5` returns the entire user object — name, email, bio, avatar, settings — even if the client only needs the username. Wasted bandwidth on fields that are never rendered.

**Under-fetching:** Fetching a user's profile + their last 5 posts + each post's comment count with REST often takes multiple sequential round trips (a "waterfall"):
```
GET /users/5
GET /users/5/posts?limit=5
GET /posts/{id}/comments/count   (× 5)
```
Each round trip adds full network latency — costly on high-latency (mobile) networks.

---

## 3. What GraphQL Is

A **query language for APIs**. Instead of many fixed REST endpoints, there's one endpoint (`POST /graphql`), and the client specifies exactly which fields it wants, nested and all, in a single request:

```graphql
query {
  user(id: 5) {
    name
    avatar
    posts(limit: 5) {
      title
      commentCount
    }
  }
}
```

One request, one round trip, response shape exactly matches the query — no over-fetching or under-fetching.

---

## 4. Schema, Types, Resolvers

GraphQL is strongly typed (like gRPC's protobuf, but for a data graph):
```graphql
type User {
  id: ID!
  name: String!
  posts: [Post!]!
}
type Post {
  title: String!
  commentCount: Int!
}
```
Each field is backed by a **resolver** — a function that knows how to fetch that piece of data. The server walks the query tree, invoking the resolver for each requested field, and assembles the nested response.

---

## 5. Queries, Mutations, Subscriptions

| Operation | Analogous to | Purpose |
|---|---|---|
| Query | GET | Read data |
| Mutation | POST/PUT/DELETE | Write data (`mutation { likePost(id: 9) { likeCount } }`) |
| Subscription | gRPC server streaming | Real-time push (client subscribes, server pushes updates) |

---

## 6. The N+1 Problem Moves — It Doesn't Disappear

GraphQL fixes the **client's** round-trip problem, but can silently recreate an N+1 problem **inside the server**: if the `commentCount` resolver runs once per post naively:

```
resolve posts(userId=5)        → 1 query
resolve commentCount(post=1)   → 1 query
resolve commentCount(post=2)   → 1 query
resolve commentCount(post=3)   → 1 query
... (N posts → N+1 total queries)
```

100 posts on a page → 101 DB queries for a single GraphQL request, even though the client made only one call.

**Fix — batching (DataLoader pattern):** instead of each resolver firing its own query immediately, individual requests are queued within the same request tick, then combined into **one batched query**:
```
Collected: post_ids = [1,2,3,4,5]
One query: SELECT post_id, COUNT(*) FROM comments
           WHERE post_id IN (1,2,3,4,5) GROUP BY post_id
```
Now it's 1 (posts) + 1 (batched counts) = 2 queries regardless of N. This is the same batching principle as Nagle's algorithm / write-batching elsewhere in networking — collect small requests, fire one bulk operation. Facebook's `dataloader` library is the reference implementation. Production GraphQL servers need this batching layer explicitly, or the N+1 problem just moves from client→server instead of disappearing.

---

## 7. Caching Is Harder

REST's `GET /users/5` is cacheable by URL for free at the HTTP layer (browsers, CDNs, reverse proxies). GraphQL's single `POST /graphql` endpoint with a variable query body has no URL to key a cache on — HTTP caching doesn't apply. GraphQL needs its own caching strategy (persisted queries, or client-side caches like Apollo/Relay).

---

## 8. Decision Box — REST vs GraphQL vs gRPC

| | REST | GraphQL | gRPC |
|---|---|---|---|
| Client controls response shape | No | Yes | No (fixed `.proto` contract) |
| HTTP caching (CDN/browser) | Native | Hard — needs its own caching layer | N/A |
| Best for | Public APIs, simple CRUD, cacheable reads | Complex/nested data, diverse client types (web vs mobile vs TV) | Internal service-to-service, low latency, streaming |
| Operational cost | Low | Higher — resolver N+1, query complexity limits needed | Moderate — schema management, browser proxy needed |

Classic GraphQL use case: one product, many client types, each needing a different slice of the same data (Facebook, who invented GraphQL, is the canonical example).

---

## 9. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| Thinking GraphQL eliminates the N+1 problem entirely | It moves the N+1 problem from client-round-trips to server-side DB queries; requires explicit batching (DataLoader) to actually fix |
| Assuming GraphQL is cacheable the same way REST is | Single POST endpoint with variable query body has no cacheable URL — needs its own caching layer |

---

## 10. Revision Questions
See `Revision/Revision_014.md`.

## 11. Summary
- GraphQL fixes REST's over-fetching (full objects for partial needs) and under-fetching (multiple round trips for nested data) via a single endpoint and client-specified query shape.
- Strongly typed schema + resolvers per field, similar spirit to gRPC's protobuf contracts but for a data graph.
- Queries = reads, Mutations = writes, Subscriptions = real-time push.
- The N+1 problem doesn't vanish — it moves server-side into resolver-per-item DB calls; fixed via batching (DataLoader pattern).
- HTTP caching is much harder — no fixed URLs to key on; needs its own caching layer.
- Best for: multiple diverse client types needing different data shapes from the same domain.
