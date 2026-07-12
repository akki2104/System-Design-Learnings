# Revision — Topic 013: RPC & gRPC

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-12

---

## Q1. How does RPC's mental model differ from REST's?

<details>
<summary>Answer</summary>

REST models a network call as acting on a **resource** (`POST /users/5/tweets`). RPC models it as calling a **remote function** (`client.PostTweet(userId=5, text="hello")`). Same network call underneath — different abstraction.

</details>

---

## Q2. What two ingredients make gRPC fast, and what does each contribute?

<details>
<summary>Answer</summary>

1. **HTTP/2 transport** — multiplexed binary streams over one connection, no app-level head-of-line blocking.
2. **Protocol Buffers (protobuf)** — compact binary serialization defined by a strict `.proto` schema; both client/server generate code from it, so contracts are enforced at compile time; payloads are smaller/faster than JSON.

</details>

---

## Q3. Name gRPC's four call patterns and one example use case for each.

<details>
<summary>Answer</summary>

- **Unary** — 1 request → 1 response (normal REST-like call)
- **Server streaming** — 1 request → stream of responses (live price updates)
- **Client streaming** — stream of requests → 1 response (chunked file upload)
- **Bidirectional streaming** — both sides stream independently (live chat)

</details>

---

## Q4. Why can't a browser call a gRPC endpoint directly? Is the same true for a native mobile app?

<details>
<summary>Answer</summary>

Browsers can't control HTTP/2 trailers (needed to signal stream completion/status) through `fetch`/`XMLHttpRequest` — gRPC requires that level of control. So browsers need a **grpc-web proxy** to translate.

Native mobile apps (iOS/Android) have no such restriction — they can speak gRPC directly (Uber, Square do this in production).

</details>

---

## Q5. Why would a team choose REST over gRPC even for an internal service where gRPC would technically work fine?

<details>
<summary>Answer</summary>

Debuggability — REST/JSON payloads are human-readable and can be inspected with `curl`; protobuf's binary format needs special tooling. For APIs consumed by external third-party developers, or where quick manual inspection matters, REST's readability can outweigh gRPC's performance edge.

</details>

---

## 30-Second Elevator Pitch

> RPC models a remote call as calling a function, not acting on a resource — that's the core mental shift from REST. gRPC is Google's RPC framework: HTTP/2 gives it multiplexed streaming, and Protocol Buffers give it a compact binary format with a compile-time-checked schema, both faster and smaller than REST/JSON. gRPC supports four call patterns, including full bidirectional streaming, which REST has no equivalent for. The catch: browsers can't speak gRPC directly because they don't expose HTTP/2 trailer control — you need a grpc-web proxy. Native mobile apps don't have that restriction. In practice: gRPC for internal service-to-service calls where you control both ends and want speed and strict contracts; REST for public APIs where broad compatibility and human-readable debugging matter more.

---

## Weak Areas to Watch

- Browser vs native-mobile gRPC support are NOT the same — browsers are blocked, native mobile isn't
- The browser blocker is HTTP/2 trailer control, not "protobuf is unreadable" (that's a secondary/debuggability reason)
