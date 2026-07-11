# Topic 007 — IP, Ports, Sockets

**Module:** 1 — Networking & Communication Foundations
**Status:** Completed
**Date:** 2026-07-09
**Confidence:** 4/5
**Difficulty:** Easy (Survey)

---

## 1. Why This Topic Exists

Topic 006 established client-server roles. This topic answers the mechanical question: how does a client actually locate and open a connection to a server?

---

## 2. Core Concepts

```
IP Address   → WHICH MACHINE on the network (like a street address)
Port         → WHICH PROCESS/SERVICE on that machine (like an apartment number)
Socket       → the actual open CONNECTION combining IP + Port + protocol
```

```
192.168.1.10 : 443
   ↑             ↑
   IP          Port (HTTPS)
```

**Why ports exist:** one machine (one IP) runs many services — web server on 443, SSH on 22, database on 5432. Without ports, only one network service could run per machine. Ports let the OS route incoming traffic to the correct process.

### The Socket 5-Tuple

A socket is uniquely identified by 5 values:
```
(source IP, source port, dest IP, dest port, protocol)
```

**Multiple connections to the same server:port are distinguished by the CLIENT'S source port** — the OS automatically assigns a different, temporary ("ephemeral") port to each outgoing connection. This is invisible to the user but is exactly how two browser tabs can be open to the same website simultaneously.

```
Connection 1: (Your IP, PORT_A, Server IP, 443, TCP)
Connection 2: (Your IP, PORT_B, Server IP, 443, TCP)
                          ↑
              different SOURCE ports distinguish them
```

Any ONE value differing in the 5-tuple = a distinct connection.

---

## Why This Matters in Interviews

- **Connection limits:** servers handle thousands of simultaneous clients because each gets a unique source port (up to ~65,535 ports per IP, though OS/NAT limits are usually lower)
- **Load balancers:** route based on destination IP:port; understanding the 5-tuple explains sticky sessions and connection draining
- **NAT:** translates the 5-tuple at network boundaries — this is why NAT tables have connection limits

---

## Well-Known Ports to Know

| Port | Protocol | Notes |
|------|----------|-------|
| 80 | HTTP | Unencrypted web traffic |
| 443 | HTTPS | Encrypted web traffic (TLS) |
| 22 | SSH | Secure shell |
| 5432 | PostgreSQL | Default DB port |
| 6379 | Redis | Default cache port |

**WebSockets do not have their own port.** ws:// runs over HTTP on port 80; wss:// runs over HTTPS on port 443. WebSocket connections start as HTTP and upgrade via the `Upgrade: websocket` header.

Memory hook: 443 = HTTPS = the *secure* one. 80 = HTTP = plain.

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Thinking a "socket ID" is a separate identifier | There's no separate ID — the 5-tuple itself is the identity |
| Assuming two tabs to the same site need different destination ports | Only the CLIENT'S source port differs; destination IP:port stays the same |
| "Port 80 is for WebSockets" | Port 80 = HTTP. WebSockets run *over* HTTP/HTTPS, not on their own port |

---

## Cheat Sheet

```
IP = WHICH MACHINE | PORT = WHICH PROCESS on that machine | SOCKET = the open connection
Socket = 5-tuple: (source IP, source port, dest IP, dest port, protocol)
Ports let ONE machine run MANY services (web:443, ssh:22, db:5432)
Multiple connections to same server:port are distinguished by different
CLIENT source ports (OS-assigned automatically, invisible to the user)
```

---

## Revision Questions

1. What's the difference between an IP address and a port, using the street address analogy?
2. What are the 5 values in a socket's identifying tuple?
3. Two browser tabs open the same website at the same time — what actually differs between the two connections?
4. Why do ports exist at all — what would break without them?

---

## Summary

- **IP** identifies the machine; **Port** identifies the process/service on that machine
- A **socket** is the 5-tuple: source IP, source port, dest IP, dest port, protocol
- Any single differing value in the 5-tuple creates a distinct connection
- Multiple client connections to the same server:port are distinguished by different **client source ports**, assigned automatically by the OS

> **You now can:** explain how a client locates and opens a connection to a server, and why one server can handle many simultaneous clients on the same port.
