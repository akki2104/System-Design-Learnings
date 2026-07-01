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

<!-- Rows added after each lesson -->
