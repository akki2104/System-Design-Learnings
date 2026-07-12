# Topic 013: RPC & gRPC

**Module:** 1 — Networking & Communication Foundations
**Completed:** 2026-07-12
**Confidence:** 4/5

---

## 1. Why This Topic Exists

REST models a system as resources. Internal microservices often need something faster and more strictly typed than REST — RPC, and specifically gRPC, is the default answer for service-to-service calls at companies like Google, Uber, and Netflix.

---

## 2. What RPC Is

RPC (Remote Procedure Call) models a network call as a **function call** on a remote machine, instead of REST's "resource + HTTP verb" model.

```
REST:  POST /users/5/tweets   {text: "hello"}
RPC:   client.PostTweet(userId=5, text="hello")
```

Same network call underneath — different mental model. REST: "here's a resource, act on it." RPC: "here's a procedure, run it."

---

## 3. gRPC — Google's RPC Framework

Two ingredients make gRPC fast:

1. **HTTP/2 as transport** (Topic 010) — multiplexed binary streams over one connection, no app-level head-of-line blocking.
2. **Protocol Buffers (protobuf)** — a compact binary serialization format defined by a strict schema (`.proto` file). Both client and server generate code from the same file, so the contract is enforced at **compile time**, not just convention.

```protobuf
service TweetService {
  rpc PostTweet (TweetRequest) returns (TweetResponse);
}
message TweetRequest {
  int64 user_id = 1;
  string text = 2;
}
```

Protobuf payloads are smaller than JSON (binary, no repeated field names per message) — meaningfully faster at high call volume.

---

## 4. gRPC's Four Call Patterns

| Pattern | Shape | Example |
|---|---|---|
| Unary | 1 request → 1 response | Normal REST-like call |
| Server streaming | 1 request → stream of responses | Subscribe to live price updates |
| Client streaming | Stream of requests → 1 response | Upload a file in chunks |
| Bidirectional streaming | Both sides stream independently | Live chat |

These patterns exist because gRPC rides on HTTP/2's native stream multiplexing — REST/HTTP has no equivalent primitive.

---

## 5. REST vs gRPC — Decision Table

| | REST | gRPC |
|---|---|---|
| Contract | Loose (docs, convention) | Strict (`.proto`, compile-time checked) |
| Payload | JSON — human-readable, larger | Protobuf — binary, smaller, faster |
| Browser support | Native | **Not possible directly** — browsers can't control HTTP/2 trailers gRPC needs; requires a **grpc-web proxy** |
| Native mobile support | Fine | Also fine — Uber/Square use gRPC directly on mobile |
| Debuggability | `curl` it, human-readable | Needs special tooling — binary isn't readable, third-party devs can't easily inspect it |
| Best for | Public APIs, broad client compatibility | Internal service-to-service, low latency, strict contracts |

**Key correction:** the reason gRPC can't be exposed to browsers isn't primarily "protobuf is unreadable" (a debuggability annoyance, not a hard blocker) — it's that gRPC needs direct control over HTTP/2 trailers to signal stream completion/status, and browser APIs (`fetch`, `XMLHttpRequest`) don't expose that level of control. Native mobile apps don't have this restriction and can speak gRPC directly.

---

## 6. Common Mistakes

| Mistake | Correction |
|---------|-----------|
| "gRPC can't be used on mobile because of unreadable payloads" | The real mobile/browser distinction: native mobile apps CAN use gRPC directly (Uber, Square do); browsers CANNOT, because they don't expose the HTTP/2 control gRPC needs — that's what grpc-web solves |
| Treating protobuf's binary format as the main reason for browser incompatibility | It's a secondary debuggability cost, not the hard technical blocker |

---

## 7. Decision Box

| Use gRPC when | Use REST when |
|---|---|
| Internal service-to-service calls, low latency matters | Public API, broad client compatibility needed |
| Strict typed contracts across teams/services | Browser clients are in the mix (without a grpc-web proxy layer) |
| Streaming (live updates, chunked uploads, bidirectional chat) needed | Human-readable/debuggable payloads matter (3rd-party devs, `curl`-ability) |

---

## 8. Revision Questions
See `Revision/Revision_013.md`.

## 9. Summary
- RPC models a network call as a remote function call; gRPC is Google's RPC framework.
- gRPC = HTTP/2 (transport) + Protobuf (serialization) — compile-time-checked contracts, smaller/faster payloads than JSON.
- Four call patterns: unary, server streaming, client streaming, bidirectional streaming — enabled by HTTP/2 multiplexing.
- Browsers cannot speak gRPC directly (no HTTP/2 trailer control) — needs grpc-web proxy. Native mobile apps can use gRPC directly.
- Default: gRPC for internal service-to-service; REST for public APIs.
