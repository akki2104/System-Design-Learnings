# Topic 011 — HTTPS & TLS

**Module:** 1 — Networking & Communication Foundations
**Status:** Completed
**Date:** 2026-07-12
**Confidence:** 3-4/5
**Difficulty:** Medium

---

## 1. Why This Topic Exists

HTTP sends everything as plaintext. Any attacker sitting between client and server — a coffee shop router, a compromised ISP node — can read and modify every byte. HTTPS solves this by wrapping HTTP in TLS before it hits the wire.

---

## 2. What HTTPS Adds

```
HTTPS = HTTP + TLS (Transport Layer Security)
```

TLS provides three guarantees:

| Guarantee | What it means |
|-----------|--------------|
| **Encryption** | Data is unreadable in transit |
| **Integrity** | Any modification in transit is detected |
| **Authentication** | You're talking to the real server, not an imposter |

Without authentication, you could have perfect encryption — and be encrypting your data directly to an attacker.

---

## 3. Certificates & Certificate Authorities

Authentication is done via **certificates**. When your browser connects to `api.stripe.com`, Stripe's server sends a certificate saying "I am stripe.com." Your browser verifies this against a list of trusted **Certificate Authorities (CAs)** — like DigiCert, Let's Encrypt, or Comodo — that are pre-installed in your OS/browser.

```
Browser
  └── trusts a list of CAs (pre-installed)
        └── CA signs Stripe's certificate
              └── Stripe presents certificate on connection
                    └── Browser verifies: "CA I trust signed this → it's really Stripe"
```

If an attacker presents a fake certificate, the browser checks the CA signature → fails → connection blocked.

---

## 4. The TLS Handshake

Before any HTTP data can flow, client and server must agree on a shared encryption key. The problem: how do you exchange a secret over an insecure channel?

**Phase 1 — Asymmetric encryption (used once):**
- Server has a **public key** (shared with everyone) and a **private key** (never leaves server)
- Client uses public key to encrypt a secret → only server can decrypt with private key
- This is how they exchange a shared session key without it being visible on the wire

**Phase 2 — Symmetric encryption (used for all data):**
- Both sides now share a session key
- All HTTP data is encrypted with this key
- Symmetric encryption is ~1000× faster than asymmetric

```
Client                                Server
  |------ ClientHello (capabilities) ------->|
  |<----- Certificate + public key ----------|
  |--- Key material (encrypted w/ pub key) ->|
  |<============ encrypted HTTP data ========>|
```

**Why two phases?** Asymmetric encryption is mathematically expensive. It's used once to securely establish a shared secret, then discarded. Symmetric takes over for everything after.

---

## 5. TLS 1.2 vs TLS 1.3

| Version | Round trips before data flows | Notes |
|---------|------------------------------|-------|
| TLS 1.2 | 2 RTT | Older cipher suites, slower |
| TLS 1.3 | 1 RTT | Streamlined — client sends key material in first message |
| TLS 1.3 0-RTT | 0 RTT | Session resumption — data sent in first packet using prior session ticket |

**System design relevance:** at scale, 2-RTT adds latency for every new connection. TLS 1.3's 1-RTT is one reason HTTP/3 + QUIC performs better — QUIC has TLS 1.3 built in, so the crypto handshake happens alongside the transport handshake (no extra round trip).

---

## 6. TLS Termination — The System Design Decision

In a real system:
```
User → [Load Balancer] → [App Servers]
```

**Option A: TLS termination at the Load Balancer**
- LB decrypts HTTPS → forwards plain HTTP to app servers
- Traffic inside your network is unencrypted
- Simpler: one certificate to manage at the LB
- LB can inspect traffic for WAF, rate limiting, routing
- ✅ Right choice for most web apps and APIs

**Option B: End-to-end TLS (passthrough)**
- LB forwards encrypted traffic; app servers decrypt
- Traffic stays encrypted inside your network
- Harder: cert management on every app server
- ✅ Required for payment systems (PCI-DSS), healthcare (HIPAA)

```
Decision rule:
  General API / web app     → terminate at LB
  PCI-DSS / HIPAA / legal   → end-to-end TLS
```

**Why the LB default?** Internal networks run in private VPCs with no external access. Encryption overhead on every internal hop is often unnecessary cost for non-sensitive data.

**Why end-to-end for payments?** PCI-DSS (Payment Card Industry Data Security Standard) mandates that card data be encrypted everywhere in transit — including inside your own network. "Trust the internal network" is not a valid compliance argument.

---

## 7. mTLS — Mutual TLS

Normal TLS: only the **server** proves its identity to the client.

mTLS: **both sides** present certificates — client proves identity to server, server proves identity to client.

```
Normal TLS:   Client → nothing  |  Server → certificate
mTLS:         Client → certificate  |  Server → certificate
```

**Use case:** service-to-service authentication in microservices. Service A calls Service B — B needs to verify the request actually came from A, not from a compromised service impersonating A. mTLS gives B cryptographic proof of A's identity.

Used by: Stripe, Google (via Istio/Envoy), Uber for internal service auth.

---

## Decision Box — When to Use What

| Scenario | Choice | Reason |
|----------|--------|--------|
| General public API | HTTPS, terminate at LB | Simpler, standard |
| Payment / health data | HTTPS, end-to-end TLS | PCI-DSS / HIPAA compliance |
| Service-to-service in microservices | mTLS | Prevent impersonation; mutual auth |
| High-latency mobile clients | TLS 1.3 + 0-RTT | Minimize handshake overhead |

**When NOT to use mTLS everywhere:** cert management for every service is operationally expensive. Use a service mesh (Istio, Linkerd) to automate it at scale — don't roll your own.

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| "Internal network is always trusted" | Not for payment/health data — PCI-DSS and HIPAA don't agree |
| Conflating TLS 1.3 0-RTT with first-connection speed | 0-RTT is session *resumption*, not first connection |
| Thinking mTLS is just "stronger HTTPS" | It's a different trust model — mutual identity proof, not just encryption |

---

## Revision Questions

1. What three things does TLS guarantee beyond what HTTP provides?
2. Why does the TLS handshake use asymmetric encryption first, then switch to symmetric?
3. A startup processes medical records. Where should they terminate TLS? Why?
4. What is mTLS and when would you use it over regular TLS?
5. TLS 1.3 reduced the handshake from 2 RTT to 1 RTT. Why does this matter in a high-scale system?

---

## Summary

- **HTTPS = HTTP + TLS** — adds encryption, integrity, and server authentication
- **Certificates + CAs** — chain of trust that proves server identity
- **Handshake:** asymmetric to exchange session key → symmetric for all data (1000× faster)
- **TLS 1.3** = 1 RTT (down from 2); 0-RTT for session resumption
- **Terminate at LB** for general apps; **end-to-end TLS** when compliance requires it
- **mTLS** = both sides authenticate; standard for internal service-to-service auth in microservices
