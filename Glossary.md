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
| DNS (Domain Name System) | Translates human-readable domain names into IP addresses via a hierarchical resolver chain | [009](Topics/009_DNS.md) |
| TTL (Time To Live) | How long a DNS resolver may cache a record before re-checking the authoritative source | [009](Topics/009_DNS.md) |
| A Record | A DNS record mapping a domain to an IPv4 address | [009](Topics/009_DNS.md) |
| CNAME Record | A DNS record mapping a domain to another domain name (an alias) | [009](Topics/009_DNS.md) |
| GeoDNS | DNS that returns different IPs for the same domain based on the requester's location — a simple form of geo-routing | [009](Topics/009_DNS.md) |
| Head-of-Line Blocking | When one blocked/lost item delays all items behind it in a queue; occurs at the application layer (HTTP/1.1's connection cap) and transport layer (HTTP/2 on TCP's shared ordering) | [010](Topics/010_HTTP_1_1_HTTP_2_HTTP_3.md) |
| Multiplexing | Sending multiple independent request/response streams over a single connection simultaneously, interleaved as frames | [010](Topics/010_HTTP_1_1_HTTP_2_HTTP_3.md) |
| HPACK | HTTP/2's header compression scheme; avoids resending full headers on every request over the same connection | [010](Topics/010_HTTP_1_1_HTTP_2_HTTP_3.md) |
| 0-RTT | A connection resumption mode (used by QUIC/HTTP/3) that sends encrypted data in the very first packet, skipping a full handshake round trip | [010](Topics/010_HTTP_1_1_HTTP_2_HTTP_3.md) |

| TLS (Transport Layer Security) | The cryptographic protocol that wraps HTTP to create HTTPS; provides encryption, integrity, and server authentication | [011](Topics/011_HTTPS_TLS.md) |
| Certificate Authority (CA) | A trusted third party (e.g., DigiCert, Let's Encrypt) that signs server certificates, enabling browsers to verify server identity | [011](Topics/011_HTTPS_TLS.md) |
| Asymmetric Encryption | Encryption using a public/private key pair; public key encrypts, only private key decrypts — used in TLS handshake to safely exchange a session key | [011](Topics/011_HTTPS_TLS.md) |
| Symmetric Encryption | Encryption using a single shared key; ~1000× faster than asymmetric — used for all data after TLS handshake completes | [011](Topics/011_HTTPS_TLS.md) |
| TLS Termination | The point where HTTPS is decrypted; at the load balancer (standard) or end-to-end (required for PCI-DSS/HIPAA) | [011](Topics/011_HTTPS_TLS.md) |
| mTLS (Mutual TLS) | A TLS mode where both client and server present certificates, providing mutual authentication — standard for service-to-service calls in microservices | [011](Topics/011_HTTPS_TLS.md) |
| PCI-DSS | Payment Card Industry Data Security Standard; mandates end-to-end encryption for card data in transit, including inside internal networks | [011](Topics/011_HTTPS_TLS.md) |

| REST (Representational State Transfer) | An API convention where URLs name resources (nouns) and HTTP methods convey actions; stateless by design | [012](Topics/012_REST_API_Design.md) |
| Idempotent | A property where calling an operation N times produces the same final state as calling it once — a state guarantee, not a response guarantee | [012](Topics/012_REST_API_Design.md) |
| Statelessness | The server holds no memory between requests; each request carries everything needed to process it, enabling any instance to serve any request | [012](Topics/012_REST_API_Design.md) |
| Cursor-Based Pagination | Pagination anchored to a fixed, unique, sortable ID rather than a row count; immune to duplicate/skipped results caused by concurrent inserts/deletes | [012](Topics/012_REST_API_Design.md) |
| Offset-Based Pagination | Pagination anchored to a row count ("skip N, take M"); breaks under concurrent writes on live feeds | [012](Topics/012_REST_API_Design.md) |

| RPC (Remote Procedure Call) | A model where a network call is treated as calling a function on a remote machine, rather than acting on a resource (REST's model) | [013](Topics/013_RPC_gRPC.md) |
| gRPC | Google's RPC framework; combines HTTP/2 transport with Protocol Buffers for fast, strictly-typed service-to-service calls | [013](Topics/013_RPC_gRPC.md) |
| Protocol Buffers (protobuf) | A compact binary serialization format defined by a schema (`.proto` file); contracts are enforced at compile time, unlike JSON | [013](Topics/013_RPC_gRPC.md) |
| grpc-web | A proxy layer that translates between browser-compatible requests and native gRPC, needed because browsers can't control HTTP/2 trailers directly | [013](Topics/013_RPC_gRPC.md) |

| GraphQL | A query language for APIs where the client specifies exact fields/nesting in one request to a single endpoint, fixing REST's over-/under-fetching | [014](Topics/014_GraphQL.md) |
| Over-fetching | A REST response containing more fields than the client needs, wasting bandwidth | [014](Topics/014_GraphQL.md) |
| Under-fetching | Needing multiple sequential REST round trips to assemble related/nested data | [014](Topics/014_GraphQL.md) |
| Resolver | A function behind a GraphQL schema field that knows how to fetch that specific piece of data | [014](Topics/014_GraphQL.md) |
| DataLoader (Batching) | A pattern that queues individual per-item data requests within a request tick and combines them into one bulk query, fixing GraphQL's server-side N+1 problem | [014](Topics/014_GraphQL.md) |
| Short Polling | Client repeatedly requests data on a fixed timer; simple but wastes requests and bounds latency to the poll interval | [015](Topics/015_WebSockets_SSE_Polling_Long_Polling.md) |
| Long Polling | Server holds a request open until data is available or a timeout hits, then the client immediately re-requests | [015](Topics/015_WebSockets_SSE_Polling_Long_Polling.md) |
| SSE (Server-Sent Events) | A one-directional (server→client), persistent HTTP-based connection for continuous server push, with built-in auto-reconnect | [015](Topics/015_WebSockets_SSE_Polling_Long_Polling.md) |
| WebSocket | A protocol that upgrades an HTTP connection (via 101 Switching Protocols) into a persistent, full-duplex, bidirectional connection | [015](Topics/015_WebSockets_SSE_Polling_Long_Polling.md) |
| 101 Switching Protocols | The HTTP status code confirming a connection has upgraded from HTTP to WebSocket | [015](Topics/015_WebSockets_SSE_Polling_Long_Polling.md) |
| Forward Proxy | Sits in front of the client, making requests on its behalf; hides the client's identity from the server | [016](Topics/016_Forward_Proxy_Reverse_Proxy_API_Gateway.md) |
| Reverse Proxy | Sits in front of one or more servers; hides the server's identity/topology from the client | [016](Topics/016_Forward_Proxy_Reverse_Proxy_API_Gateway.md) |
| API Gateway | A reverse proxy specialized for microservices APIs, adding centralized auth, rate limiting, routing, and transformation | [016](Topics/016_Forward_Proxy_Reverse_Proxy_API_Gateway.md) |

| Load Balancer | A specialized reverse proxy that distributes traffic across a pool of backend servers for scaling and redundancy | [017](Topics/017_Load_Balancers.md) |
| L4 Load Balancing | Routing based on IP + port only (transport layer); fast, protocol-agnostic, no content awareness | [017](Topics/017_Load_Balancers.md) |
| L7 Load Balancing | Routing based on HTTP content (path/headers/cookies); enables TLS termination, more processing overhead | [017](Topics/017_Load_Balancers.md) |
| Sticky Sessions (Session Affinity) | Routing the same client to the same backend server, usually because that server holds session state in memory | [017](Topics/017_Load_Balancers.md) |
| Active Health Check | The load balancer proactively pings a health endpoint to detect dead backends | [017](Topics/017_Load_Balancers.md) |
| Passive Health Check | The load balancer infers backend health by observing real traffic outcomes (errors/timeouts), catching failures active pings miss | [017](Topics/017_Load_Balancers.md) |
| CDN (Content Delivery Network) | A geographically distributed network of edge servers (reverse proxies) that cache content close to users to reduce latency | [018](Topics/018_CDN.md) |
| Edge Server | A CDN node physically close to end users; architecturally a reverse proxy in front of the origin | [018](Topics/018_CDN.md) |
| Pull CDN | A CDN model where edge servers fetch and cache content from origin on first request (cache miss), then serve subsequent requests from cache | [018](Topics/018_CDN.md) |
| Push CDN | A CDN model where the origin proactively uploads content to all edge servers ahead of time, guaranteeing zero cold-cache misses | [018](Topics/018_CDN.md) |
| Cache Invalidation (CDN) | Removing stale cached content via TTL expiry or an explicit purge API call, mirroring DNS TTL tradeoffs | [018](Topics/018_CDN.md) |
| Accept-Encoding | An HTTP request header where the client advertises which compression formats it supports | [019](Topics/019_Content_Compression_and_Encoding.md) |
| Content-Encoding | An HTTP response header indicating which compression format the server actually used | [019](Topics/019_Content_Compression_and_Encoding.md) |
| Lossy Compression | Compression that permanently discards some data to achieve smaller size (e.g., JPEG) — acceptable when the loss is imperceptible | [019](Topics/019_Content_Compression_and_Encoding.md) |
| Lossless Compression | Compression that preserves all original data exactly (e.g., PNG) — required when every detail matters | [019](Topics/019_Content_Compression_and_Encoding.md) |
| Codec | An algorithm for encoding/decoding video (or audio); newer codecs trade better compression for higher encode/decode compute cost | [019](Topics/019_Content_Compression_and_Encoding.md) |

| Page | The fixed-size unit of disk I/O (e.g., 4KB/8KB/16KB); even a tiny query reads/writes a full page | [020](Topics/020_Storage_Engine_Fundamentals.md) |
| Buffer Pool | A DB's in-memory cache of recently-used pages, serving reads and holding dirty writes before they're flushed to disk | [020](Topics/020_Storage_Engine_Fundamentals.md) |
| WAL (Write-Ahead Log) | An append-only log that records changes before they're applied to data pages, giving durability via fast sequential writes | [020](Topics/020_Storage_Engine_Fundamentals.md) |
| Checkpointing | Periodically flushing dirty pages to disk, bounding how much WAL history must be replayed on crash recovery | [020](Topics/020_Storage_Engine_Fundamentals.md) |
| Primary Key | A column (or set of columns) that uniquely identifies a row within its own table | [021](Topics/021_Relational_Databases_and_SQL.md) |
| Foreign Key | A column referencing a primary key in another table, forming a relationship between the two | [021](Topics/021_Relational_Databases_and_SQL.md) |
| Normalization | Structuring a schema so each fact is stored exactly once, eliminating update anomalies from duplicated data | [021](Topics/021_Relational_Databases_and_SQL.md) |
| 1NF (First Normal Form) | Requires every column to hold a single, atomic value — no repeating groups/lists in one field | [021](Topics/021_Relational_Databases_and_SQL.md) |
| 2NF (Second Normal Form) | Requires every non-key column to depend on the WHOLE composite primary key, not just part of it | [021](Topics/021_Relational_Databases_and_SQL.md) |
| 3NF (Third Normal Form) | Requires no non-key column to depend on another non-key column (no transitive dependency) | [021](Topics/021_Relational_Databases_and_SQL.md) |
| Query Planner | The database component that decides how to actually execute a declarative SQL query (index choice, join order, etc.) | [021](Topics/021_Relational_Databases_and_SQL.md) |
| Index | A separate, sorted data structure mapping column values to row locations, enabling O(log n) lookups instead of O(n) full table scans | [022](Topics/022_Indexing_Deep_Dive.md) |
| B-Tree | A sorted, balanced tree structure supporting both exact-match and range queries; the default index structure in relational databases | [022](Topics/022_Indexing_Deep_Dive.md) |
| Clustered Index | An index where the table's actual row data is physically stored in index order; only one per table; a lookup costs a single hop | [022](Topics/022_Indexing_Deep_Dive.md) |
| Secondary (Non-Clustered) Index | A separate structure pointing to a row's location elsewhere; many per table; a lookup costs two hops (the "bookmark lookup") | [022](Topics/022_Indexing_Deep_Dive.md) |
| Bookmark Lookup | The extra hop a secondary-index query pays to fetch the actual row after finding its location in the index | [022](Topics/022_Indexing_Deep_Dive.md) |
| Leftmost-Prefix Rule | A composite index on (A, B) only helps queries filtering on a prefix of its columns in order (A, or A+B) — not B alone | [022](Topics/022_Indexing_Deep_Dive.md) |
| Covering Index | An index containing every column a query needs, allowing an index-only scan that skips the bookmark lookup entirely | [022](Topics/022_Indexing_Deep_Dive.md) |
| Cardinality | The number of distinct values in a column; high-cardinality columns (email, user_id) benefit from indexing far more than low-cardinality ones (booleans) | [022](Topics/022_Indexing_Deep_Dive.md) |
| LSM-Tree (Log-Structured Merge-Tree) | A write-optimized storage structure that converts all writes to sequential appends via a memtable and immutable SSTables, trading read simplicity for write throughput | [023](Topics/023_B_Trees_vs_LSM_Trees.md) |
| Memtable | An in-memory sorted buffer that holds recent writes before they're flushed to disk as an SSTable | [023](Topics/023_B_Trees_vs_LSM_Trees.md) |
| SSTable (Sorted String Table) | An immutable, sorted file on disk produced by flushing a memtable; binary search works within it, but a read may need to check several | [023](Topics/023_B_Trees_vs_LSM_Trees.md) |
| Compaction | The background process that merges multiple SSTables into fewer, larger ones, removing stale/duplicate/deleted entries and bounding read-side cost | [023](Topics/023_B_Trees_vs_LSM_Trees.md) |
| ACID | Atomicity, Consistency, Isolation, Durability — the four guarantees a database transaction provides against partial-failure bugs | [024](Topics/024_Transactions_and_ACID.md) |
| Atomicity | A transaction either fully completes or fully rolls back — no partial state is ever visible | [024](Topics/024_Transactions_and_ACID.md) |
| Consistency (ACID) | A transaction moves the database from one valid state to another, respecting all schema constraints — NOT the same concept as CAP's Consistency | [024](Topics/024_Transactions_and_ACID.md) |
| Isolation | Concurrent transactions behave as if each were the only one running; exact rules covered in Topic 025 | [024](Topics/024_Transactions_and_ACID.md) |
| Durability | Once a transaction commits, its changes survive a crash — implemented via the WAL fsync-before-acknowledge mechanism (Topic 020) | [024](Topics/024_Transactions_and_ACID.md) |

| Dirty Read | Reading data another transaction wrote but hasn't committed yet; if it rolls back, you acted on data that never existed | [025](Topics/025_Isolation_Levels_and_Anomalies.md) |
| Non-Repeatable Read | Re-reading the same row within a transaction and getting a different value because another transaction committed a change in between | [025](Topics/025_Isolation_Levels_and_Anomalies.md) |
| Phantom Read | Re-running the same query and getting a different SET of matching rows because another transaction inserted/deleted qualifying rows in between | [025](Topics/025_Isolation_Levels_and_Anomalies.md) |
| Lost Update | Two transactions read the same value before either commits, both write based on that stale read, and one update silently overwrites the other | [025](Topics/025_Isolation_Levels_and_Anomalies.md) |
| Read Committed | Isolation level preventing dirty reads only; Postgres's default | [025](Topics/025_Isolation_Levels_and_Anomalies.md) |
| Repeatable Read | Isolation level preventing dirty reads + non-repeatable reads via a consistent snapshot; MySQL/InnoDB's default | [025](Topics/025_Isolation_Levels_and_Anomalies.md) |
| Serializable | The strictest isolation level; transactions behave as if run one after another sequentially, preventing all anomalies at the highest performance cost | [025](Topics/025_Isolation_Levels_and_Anomalies.md) |

| Shared Lock (S-lock) | A read lock compatible with other shared locks — multiple transactions can hold it on the same row simultaneously | [026](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) |
| Exclusive Lock (X-lock) | A write lock incompatible with any other lock (shared or exclusive) on the same row | [026](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) |
| Two-Phase Locking (2PL) | A protocol with a growing phase (acquire-only) followed by a shrinking phase (release-only), guaranteeing serializability | [026](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) |
| Strict 2PL | 2PL variant holding all exclusive locks until commit/abort, preventing cascading rollback | [026](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) |
| Cascading Rollback | A chain reaction where rolling back one transaction forces every transaction that read its uncommitted data to also roll back | [026](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) |
| Deadlock | Two or more transactions each hold a lock the other needs, so neither can proceed | [026](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) |
| Wait-For Graph | A graph of which transaction is waiting on which; a cycle indicates a deadlock | [026](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) |
| Deadlock Prevention (Lock Ordering) | Requiring all transactions to acquire locks in a fixed global order, making deadlocks structurally impossible | [026](Topics/026_Concurrency_Control_Locks_2PL_Deadlocks.md) |

| MVCC (Multi-Version Concurrency Control) | Keeping multiple versions of each row so readers and writers act on different versions and never block each other | [027](Topics/027_MVCC.md) |
| Snapshot | A record of which transactions had already committed at a given moment, used to decide which row versions a transaction can see | [027](Topics/027_MVCC.md) |
| xmin / xmax | Hidden row metadata: the transaction that created a row version (xmin) and the one that superseded it (xmax) | [027](Topics/027_MVCC.md) |
| EvalPlanQual | Postgres's Read Committed mechanism that re-applies an UPDATE's WHERE clause against a newly-committed row version instead of erroring | [027](Topics/027_MVCC.md) |
| VACUUM | Postgres's background process that reclaims old row versions no longer visible to any active transaction | [027](Topics/027_MVCC.md) |
| Table Bloat | Storage buildup from old row versions accumulating faster than VACUUM can reclaim them | [027](Topics/027_MVCC.md) |

<!-- Rows added after each lesson -->
