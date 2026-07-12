# Revision — Topic 011: HTTPS & TLS

**Format:** Active recall — answer before reading the answer.
**Completed:** 2026-07-12

---

## Q1. What three things does TLS guarantee that plain HTTP does not?

<details>
<summary>Answer</summary>

1. **Encryption** — data is unreadable in transit (prevents eavesdropping)
2. **Integrity** — data cannot be modified without detection (prevents tampering)
3. **Authentication** — you're talking to the real server, proven via CA-signed certificate (prevents impersonation)

</details>

---

## Q2. Why does the TLS handshake use asymmetric encryption first, then switch to symmetric?

<details>
<summary>Answer</summary>

Asymmetric encryption is used **once** to safely exchange a shared session key over an insecure channel — the public key encrypts, only the private key decrypts, so the attacker can't learn the session key even if they intercept the handshake.

Symmetric encryption is used for **all data** after that because it is ~1000× faster than asymmetric. Using asymmetric for bulk data would be computationally prohibitive at scale.

</details>

---

## Q3. A startup processes medical records (HIPAA-regulated). Where should they terminate TLS, and why?

<details>
<summary>Answer</summary>

**End-to-end TLS** — not at the load balancer.

HIPAA mandates encryption of protected health information (PHI) everywhere in transit, including internal network segments. TLS termination at the LB would leave PHI unencrypted between the LB and app servers, violating HIPAA.

Same logic applies to PCI-DSS (payment card data).

</details>

---

## Q4. What is mTLS, and when would you use it over regular TLS?

<details>
<summary>Answer</summary>

**Mutual TLS (mTLS):** both the client and the server present certificates — each proves its identity to the other.

Regular TLS: only the server authenticates itself.
mTLS: both sides authenticate.

**Use when:** service-to-service calls in a microservices architecture, where you need cryptographic proof that the calling service is who it claims to be (not a compromised or impersonating service).

Tools that automate mTLS at scale: Istio, Linkerd (service mesh).

</details>

---

## Q5. TLS 1.3 reduced the handshake from 2 RTT to 1 RTT. Why does this matter at scale?

<details>
<summary>Answer</summary>

At scale, every new TCP connection requires a TLS handshake before any data flows. On TLS 1.2, that's 2 round trips of pure overhead before the first byte of HTTP data.

On high-latency networks (mobile, cross-region), a round trip can be 100–200ms. Saving 1 RTT means 100–200ms less latency per new connection.

At millions of requests/day, the aggregate gain is significant — and it's one reason HTTP/3 performs better on mobile (QUIC has TLS 1.3 built in, so the transport and crypto handshakes are combined).

0-RTT further lets existing sessions resume with data in the first packet — no handshake at all for repeat connections.

</details>

---

## 30-Second Elevator Pitch

> HTTPS is HTTP wrapped in TLS. TLS provides three things: encryption so nobody can read the data in transit, integrity so nobody can tamper with it, and authentication so you know you're talking to the real server. The handshake uses asymmetric encryption to safely exchange a session key, then switches to symmetric encryption for all data because it's orders of magnitude faster. TLS 1.3 cut the handshake from 2 round trips to 1. In system design, you terminate TLS at the load balancer for most apps, but regulations like PCI-DSS and HIPAA require end-to-end encryption for sensitive data. mTLS adds mutual authentication for service-to-service calls in microservices.

---

## Weak Areas to Watch

- TLS 1.3 0-RTT: session *resumption* only, not first connection
- mTLS is a trust model change, not just stronger encryption
- PCI-DSS / HIPAA = end-to-end TLS, not LB termination
