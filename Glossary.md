# Glossary

Every term introduced in a lesson. One-line definition + link to the source topic.
Kept in alphabetical order.

| Term | Definition | Topic |
|------|-----------|-------|
| Availability | The percentage of time a system is operational and accessible | [001](Topics/001_Introduction_To_System_Design.md) |
| Fault Tolerance | A system's ability to continue operating despite component failures | [001](Topics/001_Introduction_To_System_Design.md) |
| Functional Requirement (FR) | WHAT the system does — a feature or behavior | [001](Topics/001_Introduction_To_System_Design.md) |
| High-Level Design (HLD) | Bird's-eye architecture: components, connections, data flow | [001](Topics/001_Introduction_To_System_Design.md) |
| Horizontal Scaling | Adding more servers to handle increased load | [001](Topics/001_Introduction_To_System_Design.md) |
| Low-Level Design (LLD) | Code-level design: classes, patterns, interfaces | [001](Topics/001_Introduction_To_System_Design.md) |
| Non-Functional Requirement (NFR) | HOW WELL the system performs — quality constraints like latency, availability, scalability | [001](Topics/001_Introduction_To_System_Design.md) |
| Observability | The ability to understand system internals via Metrics, Logs, and Traces | [001](Topics/001_Introduction_To_System_Design.md) |
| Reliability | A system's ability to perform its function correctly and consistently | [001](Topics/001_Introduction_To_System_Design.md) |
| Scalability | A system's ability to handle growing load without degrading performance | [001](Topics/001_Introduction_To_System_Design.md) |
| Tradeoff | A design decision where gaining one benefit requires accepting a cost elsewhere | [001](Topics/001_Introduction_To_System_Design.md) |
| Vertical Scaling | Increasing the resources (CPU/RAM) of a single server | [001](Topics/001_Introduction_To_System_Design.md) |
| Back-of-the-Envelope Estimation | A rough order-of-magnitude calculation used to size a system before designing it | [003](Topics/003_Back_of_the_Envelope_Estimation.md) |
| Bandwidth | The volume of data transferred per second across a network; estimated as peak QPS × payload size | [003](Topics/003_Back_of_the_Envelope_Estimation.md) |
| DAU (Daily Active Users) | The number of unique users who interact with a system in a 24-hour period; the base unit for estimation | [003](Topics/003_Back_of_the_Envelope_Estimation.md) |
| Peak QPS | The highest requests-per-second a system must handle; typically 2–10× average QPS | [003](Topics/003_Back_of_the_Envelope_Estimation.md) |
| QPS (Queries Per Second) | The number of requests a system receives per second; the primary throughput metric | [003](Topics/003_Back_of_the_Envelope_Estimation.md) |
| Read:Write Ratio | The ratio of read operations to write operations; drives whether caching or write-scaling is the primary concern | [003](Topics/003_Back_of_the_Envelope_Estimation.md) |
| Replication Factor | The number of copies of each data item stored; multiplies raw storage requirements | [003](Topics/003_Back_of_the_Envelope_Estimation.md) |
| Error Budget | The allowable downtime/failure within an SLO period; once exhausted, all changes freeze | [004](Topics/004_Non_Functional_Requirements.md) |
| Fault Tolerance | A system's ability to continue operating correctly despite individual component failures | [004](Topics/004_Non_Functional_Requirements.md) |
| Graceful Degradation | Under partial failure, the system does less but doesn't crash — core path stays alive, non-critical features shed | [004](Topics/004_Non_Functional_Requirements.md) |
| Maintainability | A system's ability to be understood, changed, and operated by humans over time (operability + simplicity + evolvability) | [004](Topics/004_Non_Functional_Requirements.md) |
| p99 (99th Percentile) | The latency below which 99% of requests are served; the standard percentile for latency NFRs in interviews | [004](Topics/004_Non_Functional_Requirements.md) |
| SLA (Service Level Agreement) | A contractual commitment with external parties about availability/performance, with penalties for breach | [004](Topics/004_Non_Functional_Requirements.md) |
| SLI (Service Level Indicator) | The actual measured metric (e.g., request success rate, p99 latency) | [004](Topics/004_Non_Functional_Requirements.md) |
| SLO (Service Level Objective) | The internal target for an SLI (e.g., "99.9% of requests succeed") | [004](Topics/004_Non_Functional_Requirements.md) |
| Denormalization | Deliberately duplicating data (e.g., embedding related fields) to avoid expensive cross-shard joins | [005](Topics/005_How_to_Reason_About_Tradeoffs.md) |
| Leaderless Replication | A write model (used by Cassandra/DynamoDB) where any node can accept a write, avoiding a single-primary write bottleneck | [005](Topics/005_How_to_Reason_About_Tradeoffs.md) |
| Sharding | Splitting a dataset across multiple machines; the real cause of expensive joins (not raw data volume) | [005](Topics/005_How_to_Reason_About_Tradeoffs.md) |
| Write-Back Cache | A cache that acknowledges writes before flushing to the durable database; fast but risks data loss on crash | [005](Topics/005_How_to_Reason_About_Tradeoffs.md) |
| Write-Through Cache | A cache that writes to both cache and database synchronously; safe but doesn't reduce database write load | [005](Topics/005_How_to_Reason_About_Tradeoffs.md) |
| Client | Anything that initiates a request to consume a resource/service | [006](Topics/006_The_Client_Server_Model.md) |
| Server | Anything that provides a resource/service and listens for requests | [006](Topics/006_The_Client_Server_Model.md) |
| Peer-to-Peer (P2P) | An architecture where nodes communicate directly without a central server; no single bottleneck but harder to secure/coordinate | [006](Topics/006_The_Client_Server_Model.md) |
| IP Address | Identifies WHICH MACHINE on a network | [007](Topics/007_IP_Ports_Sockets.md) |
| Port | Identifies WHICH PROCESS/SERVICE on a machine; lets one IP run many services | [007](Topics/007_IP_Ports_Sockets.md) |
| Socket | An open network connection uniquely identified by the 5-tuple: source IP, source port, dest IP, dest port, protocol | [007](Topics/007_IP_Ports_Sockets.md) |
| Ephemeral Port | A temporary port automatically assigned by the OS to a client's outgoing connection | [007](Topics/007_IP_Ports_Sockets.md) |
| TCP (Transmission Control Protocol) | Connection-oriented transport protocol with guaranteed, ordered delivery via handshake, ACKs, and retransmission | [008](Topics/008_TCP_vs_UDP.md) |
| UDP (User Datagram Protocol) | Connectionless transport protocol with no delivery/ordering guarantees; faster, no handshake | [008](Topics/008_TCP_vs_UDP.md) |
| 3-Way Handshake | TCP's connection setup sequence (SYN, SYN-ACK, ACK) before any real data flows; costs one round trip | [008](Topics/008_TCP_vs_UDP.md) |
| QUIC | A UDP-based transport protocol (used in HTTP/3) that adds its own reliability layer, skipping TCP's handshake cost while keeping delivery guarantees | [008](Topics/008_TCP_vs_UDP.md) |

<!-- Rows added after each lesson -->
