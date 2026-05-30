# Designing Large-Scale Distributed Systems
## A Practitioner's Complete Reference
### — Table of Contents —

---

# PART I — FIRST PRINCIPLES & FOUNDATIONAL THINKING

## Chapter 1: The Anatomy of Scale
1.1 What Changes When You Add a Zero  
1.2 Vertical Scaling — Buying Bigger Machines  
1.3 Horizontal Scaling — Buying More Machines  
1.4 Diagonal Scaling — The Hybrid Reality  
1.5 The Scale Cube (X-axis Cloning, Y-axis Functional Split, Z-axis Data Partitioning)  
1.6 Scaling Dimensions: Data, Compute, Users, Geography  
1.7 Sub-linear, Linear, and Super-linear Scaling Behaviors  
1.8 Cost of Scale — Coordination Overhead and Diminishing Returns  

## Chapter 2: Latency, Throughput & Bandwidth
2.1 Latency — The Speed of One  
2.2 Throughput — The Speed of Many  
2.3 Bandwidth — The Width of the Pipe  
2.4 Latency Budgets and SLA Decomposition  
2.5 Little's Law and Its Implications for System Design  
2.6 The Latency-Throughput Tradeoff  
2.7 Tail Latency — Why P99 Matters More Than P50  
2.8 Numbers Every Engineer Should Know (Disk, Network, Memory, Cross-Region)  

## Chapter 3: Availability, Reliability & Fault Tolerance
3.1 Availability — Nines and What They Mean  
3.2 Reliability vs. Availability — The Subtle Difference  
3.3 Durability — Data That Survives Disasters  
3.4 MTBF, MTTR, and the Availability Equation  
3.5 Failure Domains — Blast Radius Thinking  
3.6 Failure Modes: Crash, Omission, Byzantine, Timing  
3.7 Redundancy — Active-Active, Active-Passive, N+1, N+2  
3.8 Single Points of Failure (SPOF) Identification and Elimination  
3.9 Graceful Degradation vs. Hard Failure  
3.10 Failure Budgets and Error Budgets (SRE Approach)  

## Chapter 4: Consistency Models
4.1 What Is Consistency? — A Precise Definition  
4.2 Strong Consistency (Linearizability)  
4.3 Sequential Consistency  
4.4 Causal Consistency  
4.5 Eventual Consistency — The Default of Distributed Systems  
4.6 Read-Your-Writes, Monotonic Reads, Monotonic Writes  
4.7 Session Consistency and Sticky Sessions  
4.8 Tunable Consistency (Quorum Reads/Writes)  
4.9 Timeline Consistency  
4.10 Consistency in the Real World — What Users Actually Need  

## Chapter 5: The CAP Theorem & Its Successors
5.1 CAP Theorem — The Original Impossibility Result  
5.2 Why "Pick Two" Is Misleading  
5.3 PACELC — Extending CAP to Normal Operations  
5.4 The Consistency-Availability Spectrum  
5.5 Network Partitions — How They Actually Happen  
5.6 Designing for Partition Tolerance — Practical Strategies  
5.7 Harvest vs. Yield — A Better Framework for Tradeoffs  

## Chapter 6: Back-of-the-Envelope Estimation
6.1 Why Estimation Is the First Step  
6.2 Traffic Estimation — QPS, DAU, Peak Multipliers  
6.3 Storage Estimation — Growth Rate, Retention, Compression  
6.4 Bandwidth Estimation — Ingress, Egress, Internal Fan-out  
6.5 Compute Estimation — CPU, Memory, GPU  
6.6 Cost Estimation — Cloud Primitives and Unit Economics  
6.7 Capacity Planning — From Estimates to Infrastructure  
6.8 Worked Examples: Estimating for Chat, Video, Search  

---

# PART II — NETWORKING & COMMUNICATION PATTERNS

## Chapter 7: Network Fundamentals for System Designers
7.1 TCP/IP — Connections, Handshakes, and Head-of-Line Blocking  
7.2 UDP — When Unreliable Is Good Enough  
7.3 DNS — The Internet's Phone Book (Resolution, TTL, Failover)  
7.4 HTTP/1.1, HTTP/2, HTTP/3 (QUIC) — Evolution of the Web Protocol  
7.5 TLS/SSL — The Cost of Encryption  
7.6 IP Anycast — Routing to the Nearest Node  
7.7 BGP and Internet Routing — Why Geography Matters  
7.8 Network Partitions, Split-Brain, and Asymmetric Failures  

## Chapter 8: Synchronous Communication Patterns
8.1 Request-Response — The Simplest Pattern  
8.2 REST — Resource-Oriented Design  
8.3 GraphQL — Query-Oriented Design  
8.4 gRPC and Protocol Buffers — High-Performance RPC  
8.5 Thrift, Avro, and Other RPC Frameworks  
8.6 Connection Pooling and Multiplexing  
8.7 Timeouts, Retries, and Idempotency in Synchronous Calls  
8.8 The Cascade Failure Problem in Synchronous Chains  

## Chapter 9: Asynchronous Communication Patterns
9.1 Why Async? — Decoupling in Time and Space  
9.2 Message Queues — Point-to-Point (SQS, RabbitMQ)  
9.3 Publish-Subscribe — Fan-out (SNS, Kafka Topics, Google Pub/Sub)  
9.4 Event Streaming — The Ordered, Replayable Log (Kafka, Kinesis, Pulsar)  
9.5 Message Ordering Guarantees — FIFO, Partition-Ordered, Best-Effort  
9.6 Delivery Guarantees — At-Most-Once, At-Least-Once, Exactly-Once  
9.7 Dead Letter Queues and Poison Message Handling  
9.8 Backpressure — What Happens When the Consumer Can't Keep Up  
9.9 Message Serialization — JSON, Protobuf, Avro, MessagePack  
9.10 Idempotent Consumers and Deduplication Strategies  

## Chapter 10: Event-Driven Architecture
10.1 Events vs. Commands vs. Queries  
10.2 Event Notification Pattern  
10.3 Event-Carried State Transfer  
10.4 Event Sourcing — Recording Every State Change  
10.5 Event Choreography vs. Orchestration  
10.6 Event Schemas and Evolution (Schema Registry)  
10.7 Event Replay, Reprocessing, and Time Travel  
10.8 Designing Event Contracts — Thin Events vs. Fat Events  
10.9 The Dual-Write Problem and Transactional Outbox  

## Chapter 11: API Design & Gateway Patterns
11.1 API Design Principles — Consistency, Discoverability, Versioning  
11.2 API Gateway Pattern — The Single Entry Point  
11.3 Backend for Frontend (BFF) Pattern  
11.4 API Composition and Aggregation Layer  
11.5 API Versioning Strategies (URL, Header, Content-Type)  
11.6 Pagination — Offset, Cursor, Keyset  
11.7 Bulk and Batch APIs  
11.8 Long-Running Operations and Polling  
11.9 Webhooks — Server-to-Client Push  
11.10 Server-Sent Events (SSE) and WebSockets  
11.11 API Rate Limiting and Quotas (Consumer-Facing)  
11.12 API Contracts, OpenAPI, and Contract Testing  

---

# PART III — DATA MANAGEMENT PATTERNS

## Chapter 12: Storage Engines & Data Structures
12.1 B-Trees and B+ Trees — The Read-Optimized Workhorse  
12.2 LSM Trees and SSTables — The Write-Optimized Alternative  
12.3 Write-Ahead Log (WAL) — Durability Before Acknowledgment  
12.4 Bloom Filters — Probabilistic Existence Checks  
12.5 Skip Lists, Red-Black Trees — In-Memory Ordered Structures  
12.6 Inverted Indexes — Full-Text Search  
12.7 Column-Oriented Storage — Analytics at Speed  
12.8 Row-Oriented vs. Column-Oriented — Choosing the Right Engine  
12.9 Compaction Strategies — Size-Tiered, Leveled, FIFO  

## Chapter 13: Database Selection & Modeling
13.1 Relational Databases — When Schema Matters  
13.2 Document Databases — Flexible, Denormalized (MongoDB, DynamoDB)  
13.3 Key-Value Stores — The Simplest Abstraction (Redis, DynamoDB, Memcached)  
13.4 Wide-Column Stores — Sparse, Distributed Tables (Cassandra, HBase, Bigtable)  
13.5 Graph Databases — Relationship-First Modeling (Neo4j, Neptune)  
13.6 Time-Series Databases — Append-Optimized (InfluxDB, TimescaleDB)  
13.7 Vector Databases — Similarity Search (Pinecone, Milvus, pgvector)  
13.8 Search Engines — Inverted Index as a Database (Elasticsearch, Solr)  
13.9 Multi-Model Databases and Polyglot Persistence  
13.10 Data Modeling: Normalization vs. Denormalization  
13.11 Schema Design for Read-Heavy vs. Write-Heavy Workloads  

## Chapter 14: Data Partitioning (Sharding)
14.1 Why Partition? — Breaking the Single-Node Ceiling  
14.2 Horizontal Partitioning (Sharding) vs. Vertical Partitioning  
14.3 Hash-Based Partitioning  
14.4 Range-Based Partitioning  
14.5 Consistent Hashing — Minimizing Redistribution  
14.6 Virtual Nodes (Vnodes) — Balancing the Ring  
14.7 Directory-Based Partitioning (Lookup Table)  
14.8 Geographic (Geo) Partitioning  
14.9 Partition Key Selection — The Most Important Decision  
14.10 Hot Partitions and Skew — Detection and Mitigation  
14.11 Cross-Partition Queries and Scatter-Gather  
14.12 Resharding and Online Partition Migration  

## Chapter 15: Data Replication
15.1 Why Replicate? — Availability, Performance, Durability  
15.2 Single-Leader (Primary-Secondary) Replication  
15.3 Multi-Leader Replication — Active-Active Writes  
15.4 Leaderless Replication — Quorum-Based (Dynamo-style)  
15.5 Synchronous vs. Asynchronous Replication  
15.6 Semi-Synchronous Replication — The Practical Middle Ground  
15.7 Replication Lag — Causes, Detection, Mitigation  
15.8 Conflict Detection and Resolution in Multi-Leader Systems  
15.9 Last-Write-Wins (LWW) and Its Dangers  
15.10 CRDTs — Conflict-Free Replicated Data Types  
15.11 Merkle Trees for Anti-Entropy and Replica Repair  
15.12 Chain Replication  

## Chapter 16: Caching Patterns
16.1 Why Cache? — Trading Freshness for Speed  
16.2 Cache-Aside (Lazy Loading)  
16.3 Read-Through Cache  
16.4 Write-Through Cache  
16.5 Write-Behind (Write-Back) Cache  
16.6 Refresh-Ahead Cache  
16.7 Cache Eviction Policies — LRU, LFU, FIFO, TTL, ARC  
16.8 Cache Invalidation — The Hard Problem  
16.9 Distributed Caching — Memcached, Redis Cluster  
16.10 Cache Stampede (Thundering Herd) and Prevention  
16.11 Cache Warming and Pre-Population  
16.12 Cache Penetration, Cache Breakdown, Cache Avalanche  
16.13 Multi-Tier Caching — L1 (Local) + L2 (Distributed) + CDN  
16.14 Negative Caching — Caching the Absence  
16.15 Consistent Hashing for Cache Distribution  

## Chapter 17: Indexing & Query Optimization Patterns
17.1 Primary vs. Secondary Indexes  
17.2 Covering Indexes and Index-Only Scans  
17.3 Composite (Multi-Column) Indexes and Key Ordering  
17.4 Partial and Filtered Indexes  
17.5 Global Secondary Indexes vs. Local Secondary Indexes  
17.6 Inverted Indexes for Full-Text Search  
17.7 Geospatial Indexes — R-Trees, Geohashes, S2 Cells, H3  
17.8 Bitmap Indexes — Low-Cardinality Analytics  
17.9 Materialized Views — Pre-Computed Query Results  
17.10 Denormalized Read Models and Query Tables  
17.11 Search Index as a Secondary Read Path (Elasticsearch Pattern)  
17.12 The Index Tax — Write Amplification and Storage Cost  

---

# PART IV — COMPUTE & PROCESSING PATTERNS

## Chapter 18: Job Scheduling & Triggering
18.1 What Is a Job? — Tasks, Steps, DAGs, and Workflows  
18.2 Cron-Based Scheduling — Time-Triggered Execution  
18.3 Event-Triggered Execution — React to State Changes  
18.4 Distributed Cron — Ensuring Exactly-One Execution (Leader Election for Cron)  
18.5 Job Queues — Decoupling Submission from Execution  
18.6 Priority Queues and Multi-Level Scheduling  
18.7 Delayed and Scheduled Message Delivery  
18.8 Workflow Orchestration Engines (Temporal, Airflow, Step Functions, Cadence)  
18.9 DAG-Based Dependency Resolution  
18.10 Job Idempotency — Safe to Retry, Safe to Re-Run  
18.11 Job Deduplication — Preventing Duplicate Execution  
18.12 Dead Job Detection and Zombie Reaping  
18.13 Fan-Out / Fan-In — Parallel Execution and Result Aggregation  

## Chapter 19: Timer Service & Delayed Execution Patterns
19.1 The Timer Wheel — Efficient Timeout Management  
19.2 Hierarchical Timer Wheels — Multi-Resolution Scheduling  
19.3 Distributed Timer Service — Scalable Delayed Execution  
19.4 Scheduler-Based Timers vs. Queue-Based Delays  
19.5 Reminder and Alarm Patterns  
19.6 Lease-Based Expiration and TTL Enforcement  
19.7 Debounce and Throttle Patterns  
19.8 Heartbeat and Liveness Detection  
19.9 Calendar-Aware Scheduling (Business Days, Timezone Handling)  
19.10 Timer Persistence and Crash Recovery  

## Chapter 20: Batch Processing
20.1 Batch vs. Real-Time — When Batches Win  
20.2 MapReduce — The Foundational Pattern  
20.3 ETL Pipelines — Extract, Transform, Load  
20.4 Data Lakes and the ELT Inversion  
20.5 Partitioned Parallel Processing  
20.6 Sort-Merge Join for Large-Scale Joins  
20.7 Checkpointing, Restartability, and Idempotent Batches  
20.8 Backfill and Reprocessing Patterns  
20.9 Batch Windowing — Micro-Batch as a Bridge to Streaming  

## Chapter 21: Stream Processing
21.1 Unbounded Data and the Streaming Mindset  
21.2 Event Time vs. Processing Time  
21.3 Windowing — Tumbling, Sliding, Session, Global  
21.4 Watermarks — Tracking Event Time Completeness  
21.5 Late Data and Allowed Lateness  
21.6 Triggers and Accumulation Modes  
21.7 Stateful Stream Processing — Managed State, Changelogs  
21.8 Stream-Table Duality (Kafka Streams, Flink)  
21.9 Exactly-Once Semantics in Stream Processing  
21.10 Stream Joins — Stream-Stream, Stream-Table  
21.11 Change Data Capture (CDC) — Database as a Stream  
21.12 Kappa Architecture — Streams as the Single Source of Truth  
21.13 Lambda Architecture — Batch + Speed Layers  

## Chapter 22: Aggregation & Projection Patterns
22.1 Pre-Aggregation — Compute Once, Query Many  
22.2 Rollup Tables — Time-Bucketed Summaries  
22.3 HyperLogLog — Approximate Distinct Counts at Scale  
22.4 Count-Min Sketch — Approximate Frequency Counts  
22.5 T-Digest and DDSketch — Approximate Percentiles  
22.6 Materialized Projections — Purpose-Built Read Views  
22.7 CQRS — Separate Read and Write Models  
22.8 Incremental Aggregation vs. Full Recomputation  
22.9 Multi-Dimensional Aggregation (OLAP Cubes, Druid, Pinot)  
22.10 Leaderboard Pattern — Top-K with Real-Time Updates  
22.11 The Counter Pattern — Scalable Distributed Counting  
22.11.1 Naive Counters and Their Limits  
22.11.2 Sharded Counters — Distributing Write Load  
22.11.3 Approximate Counters — Trading Accuracy for Speed  
22.11.4 Hierarchical Counter Aggregation  
22.11.5 Cell-Based Counters (Google Zanzibar-style)  
22.11.6 Counter with Exact Read — Fan-Out Write, Fan-In Read  
22.12 Eventual Projection — Async View Materialization  
22.13 Time-Series Downsampling and Retention Policies  

---

# PART V — RESOURCE MANAGEMENT & FAIRNESS PATTERNS

## Chapter 23: Rate Limiting
23.1 Why Rate Limit? — Protecting Systems and Ensuring Fairness  
23.2 Token Bucket Algorithm  
23.3 Leaky Bucket Algorithm  
23.4 Fixed Window Counter  
23.5 Sliding Window Log  
23.6 Sliding Window Counter (Hybrid)  
23.7 Adaptive and Dynamic Rate Limiting  
23.8 In-Memory Rate Limiting (Single Node)  
23.9 Distributed Rate Limiting  
23.9.1 Centralized (Redis-Based) Rate Limiter  
23.9.2 Gossip-Based Approximate Rate Limiting  
23.9.3 Local + Sync Hybrid Rate Limiting  
23.9.4 Cell-Based Rate Limiting  
23.10 Multi-Dimensional Rate Limiting (Per-User, Per-IP, Per-API, Per-Region)  
23.11 Rate Limiting at Different Layers (L3/L4, L7, Application)  
23.12 Rate Limit Headers and Client Communication (429, Retry-After)  
23.13 Cost-Based Rate Limiting (Weighted Request Costing)  
23.14 Rate Limiting vs. Throttling vs. Backpressure  

## Chapter 24: Resource Allocation & Fairness
24.1 The Shared Resource Problem  
24.2 Quota Management — Hard Limits, Soft Limits, Burst Allowances  
24.3 Fair Queuing and Weighted Fair Queuing  
24.4 Max-Min Fairness  
24.5 Dominant Resource Fairness (DRF) — Multi-Resource Allocation  
24.6 Priority-Based Resource Allocation  
24.7 Preemption — Reclaiming Resources from Lower Priorities  
24.8 Admission Control — Rejecting Work Before It Starts  
24.9 Resource Reservation and Capacity Booking  
24.10 Token-Based Resource Allocation  
24.11 Slot-Based Allocation (Fixed Resource Partitioning)  
24.12 Work-Stealing and Load Rebalancing  
24.13 Resource Pools and Pool Sizing (Connection Pools, Thread Pools)  
24.14 Bin Packing — Fitting Workloads to Machines  
24.15 Multi-Tenant Resource Isolation  
24.15.1 Noisy Neighbor Detection and Mitigation  
24.15.2 Tenant-Aware Scheduling  
24.15.3 Resource Sandboxing (Cgroups, Namespaces, VMs)  
24.15.4 Blast Radius Containment per Tenant  

## Chapter 25: Load Balancing & Traffic Distribution
25.1 Why Load Balance? — From Single Server to Fleet  
25.2 DNS-Based Load Balancing  
25.3 L4 (Transport Layer) Load Balancing  
25.4 L7 (Application Layer) Load Balancing  
25.5 Algorithms: Round Robin, Weighted Round Robin  
25.6 Algorithms: Least Connections, Least Response Time  
25.7 Algorithms: Consistent Hashing for Stateful Routing  
25.8 Algorithms: Power of Two Random Choices (P2C)  
25.9 Algorithms: Locality-Aware Routing  
25.10 Global Server Load Balancing (GSLB)  
25.11 Health Checks — Active, Passive, Deep  
25.12 Connection Draining and Graceful Shutdown  
25.13 Load Shedding — Controlled Rejection Under Overload  
25.14 Subsetting — Limiting the Fan-Out  

## Chapter 26: Auto-Scaling Patterns
26.1 Reactive Scaling — Threshold-Based  
26.2 Predictive Scaling — Time-Series Forecasting  
26.3 Schedule-Based Scaling — Known Traffic Patterns  
26.4 Target Tracking — Maintaining a Metric  
26.5 Scaling Metrics: CPU, Memory, Queue Depth, Latency, Custom  
26.6 Cooldown Periods and Flapping Prevention  
26.7 Scale-to-Zero and Cold Start Strategies  
26.8 Cluster Auto-Scaling (Kubernetes HPA, VPA, Cluster Autoscaler)  
26.9 Serverless Scaling Model — Implicit Auto-Scaling  
26.10 Database Auto-Scaling — Read Replicas, Serverless RDS  

---

# PART VI — RELIABILITY & RESILIENCE PATTERNS

## Chapter 27: Fault Tolerance Patterns
27.1 Retry Pattern  
27.1.1 Simple Retry  
27.1.2 Exponential Backoff  
27.1.3 Exponential Backoff with Jitter  
27.1.4 Retry Budgets — Limiting System-Wide Retry Amplification  
27.1.5 Retry Storms and Metastable Failure  
27.2 Circuit Breaker Pattern  
27.2.1 Closed, Open, Half-Open States  
27.2.2 Failure Counting vs. Failure Rate Thresholds  
27.2.3 Circuit Breaker per Downstream, per Endpoint  
27.2.4 Circuit Breaker + Fallback Composition  
27.3 Bulkhead Pattern — Failure Isolation  
27.3.1 Thread Pool Isolation  
27.3.2 Semaphore Isolation  
27.3.3 Process-Level Isolation  
27.3.4 Infrastructure-Level Bulkheads (Cells, Swimlanes)  
27.4 Timeout Pattern  
27.4.1 Connect Timeout vs. Read Timeout  
27.4.2 End-to-End Timeout (Deadline Propagation)  
27.4.3 Adaptive Timeouts Based on P99 Latency  
27.5 Fallback Pattern  
27.5.1 Static Fallback (Default Values)  
27.5.2 Cached Fallback (Stale Data)  
27.5.3 Graceful Degradation (Reduced Functionality)  
27.5.4 Failover to Secondary System  
27.6 Hedge Requests — Racing Redundant Calls  
27.7 Speculative Execution  

## Chapter 28: Distributed Transactions & Consistency Patterns
28.1 ACID in a Distributed World  
28.2 Two-Phase Commit (2PC) — The Classic Protocol  
28.3 Three-Phase Commit (3PC) — Reducing Blocking  
28.4 Saga Pattern — Long-Running Distributed Transactions  
28.4.1 Choreography-Based Sagas  
28.4.2 Orchestration-Based Sagas  
28.4.3 Compensating Transactions  
28.4.4 Saga Failure Modes and Pivot Transactions  
28.5 Outbox Pattern — Reliable Event Publishing  
28.6 Transactional Outbox + CDC — Guaranteed Delivery  
28.7 Try-Confirm/Cancel (TCC) Pattern  
28.8 Reservation Pattern — Tentative Resource Locking  
28.9 Idempotency Keys — Making Operations Safely Repeatable  
28.10 Optimistic Concurrency Control (Versioning, ETags)  
28.11 Pessimistic Locking and Its Distributed Pitfalls  

## Chapter 29: Leader Election & Coordination
29.1 Why Leader Election? — Single-Writer and Coordination  
29.2 Bully Algorithm  
29.3 Ring-Based Election  
29.4 Paxos — The Foundational Consensus Algorithm  
29.5 Multi-Paxos and Paxos Variants  
29.6 Raft — Understandable Consensus  
29.7 ZAB (ZooKeeper Atomic Broadcast)  
29.8 Lease-Based Leadership  
29.9 Distributed Locks (ZooKeeper, etcd, Redis Redlock)  
29.10 Fencing Tokens — Preventing Stale Leader Actions  
29.11 Split-Brain Detection and Resolution  
29.12 Epoch and Generation Numbers  

## Chapter 30: Disaster Recovery & Data Protection
30.1 RPO and RTO — Defining Recovery Requirements  
30.2 Backup Strategies — Full, Incremental, Differential  
30.3 Point-in-Time Recovery (PITR)  
30.4 Cross-Region Replication — Active-Passive Failover  
30.5 Cross-Region Replication — Active-Active  
30.6 Pilot Light, Warm Standby, Hot Standby  
30.7 Chaos Engineering — Proactive Failure Injection  
30.8 Game Days and Disaster Recovery Drills  
30.9 Data Integrity Verification — Checksums, Scrubbing  
30.10 Immutable Backups and Ransomware Protection  

---

# PART VII — ARCHITECTURE PATTERNS AT SCALE

## Chapter 31: Monolith to Microservices — Evolution of Architecture
31.1 The Monolithic Architecture — Benefits and Limits  
31.2 Modular Monolith — Structure Without Distribution  
31.3 Microservices — Independent Deployability  
31.4 Service Mesh — Infrastructure for Service-to-Service Communication  
31.5 Nano-Services and the Over-Decomposition Anti-Pattern  
31.6 Strangler Fig Pattern — Incremental Migration  
31.7 Anti-Corruption Layer — Bridging Old and New  
31.8 Service Boundaries — Aligning with Domain-Driven Design  

## Chapter 32: Multi-Tier & Layered Architecture Patterns
32.1 The Classic Three-Tier Architecture  
32.2 N-Tier with Caching and Queue Layers  
32.3 Hexagonal Architecture (Ports and Adapters)  
32.4 Clean Architecture — Dependency Inversion at Scale  
32.5 Sidecar Pattern  
32.6 Ambassador Pattern  
32.7 Adapter Pattern at the Infrastructure Boundary  

## Chapter 33: Cell-Based & Isolation Architecture
33.1 Cell Architecture — AWS Route 53, Slack, Teams  
33.2 Swimlane Architecture  
33.3 Shuffle Sharding — Fault Isolation Without Full Isolation  
33.4 Bulkhead at the Infrastructure Level  
33.5 Blast Radius Reduction Through Cellular Deployment  
33.6 Cell Routing — Mapping Users to Cells  
33.7 Cross-Cell Operations and Data Rebalancing  

## Chapter 34: Multi-Region & Global Architecture
34.1 Single-Region, Multi-AZ — The Starting Point  
34.2 Multi-Region Active-Passive  
34.3 Multi-Region Active-Active — The Holy Grail  
34.4 Follow-the-Sun Routing  
34.5 Data Sovereignty and Residency Requirements  
34.6 Global Tables and Cross-Region Replication  
34.7 Conflict Resolution in Multi-Region Writes  
34.8 Global Load Balancing and Latency-Based Routing  
34.9 Region Evacuation and Failover Procedures  

## Chapter 35: Multi-Tenancy Patterns
35.1 Shared Everything (Single Pool)  
35.2 Shared Infrastructure, Isolated Data (Schema-per-Tenant, Database-per-Tenant)  
35.3 Shared Nothing (Dedicated Infrastructure)  
35.4 Hybrid Tiered Tenancy (Free → Shared, Enterprise → Dedicated)  
35.5 Tenant Isolation at Compute, Storage, Network Layers  
35.6 Tenant-Aware Routing and Context Propagation  
35.7 Tenant Onboarding and Offboarding Automation  
35.8 Per-Tenant Billing, Metering, and Cost Attribution  

---

# PART VIII — IDENTITY, SECURITY & ACCESS PATTERNS

## Chapter 36: Authentication Patterns
36.1 Session-Based Authentication  
36.2 Token-Based Authentication (JWT, Opaque Tokens)  
36.3 OAuth 2.0 Flows — Authorization Code, Client Credentials, PKCE  
36.4 OpenID Connect (OIDC) — Identity Layer on OAuth  
36.5 Single Sign-On (SSO) — SAML, OIDC Federation  
36.6 Multi-Factor Authentication (MFA)  
36.7 API Key Authentication and Rotation  
36.8 Mutual TLS (mTLS) for Service-to-Service  
36.9 Token Refresh, Rotation, and Revocation  
36.10 Passwordless Authentication (WebAuthn, Magic Links)  

## Chapter 37: Authorization Patterns
37.1 RBAC — Role-Based Access Control  
37.2 ABAC — Attribute-Based Access Control  
37.3 ReBAC — Relationship-Based Access Control (Google Zanzibar)  
37.4 Policy Engines (OPA, Cedar)  
37.5 Permission Checking at Scale — Caching, Denormalization  
37.6 Hierarchical Permissions and Inheritance  
37.7 Cross-Service Authorization and Token Exchange  
37.8 Audit Logging and Access Trail  

## Chapter 38: Data Security Patterns
38.1 Encryption at Rest — Full Disk, Column-Level, Application-Level  
38.2 Encryption in Transit — TLS, mTLS  
38.3 Envelope Encryption and Key Hierarchy  
38.4 Key Management (KMS, HSM, Vault)  
38.5 Data Masking and Tokenization  
38.6 Secrets Management (HashiCorp Vault, AWS Secrets Manager)  
38.7 Data Classification and Sensitivity Tagging  
38.8 Zero-Trust Architecture Principles  

---

# PART IX — OBSERVABILITY & OPERATIONAL PATTERNS

## Chapter 39: Logging Patterns
39.1 Structured Logging — JSON, Key-Value  
39.2 Centralized Log Aggregation (ELK, Loki, Splunk)  
39.3 Log Levels and Dynamic Log Level Changes  
39.4 Correlation IDs and Request Tracing  
39.5 Log Sampling — Reducing Volume Without Losing Signal  
39.6 Audit Logging — Compliance and Forensics  
39.7 Log Retention, Rotation, and Archival  

## Chapter 40: Metrics & Monitoring Patterns
40.1 The Four Golden Signals — Latency, Traffic, Errors, Saturation  
40.2 RED Method — Rate, Errors, Duration  
40.3 USE Method — Utilization, Saturation, Errors  
40.4 Metric Types — Counters, Gauges, Histograms, Summaries  
40.5 Push vs. Pull Metric Collection  
40.6 Dimensional Metrics and High-Cardinality Challenges  
40.7 SLIs, SLOs, SLAs — Defining and Measuring Reliability  
40.8 Error Budgets and Burn Rate Alerts  
40.9 Anomaly Detection and Adaptive Thresholds  
40.10 Dashboard Design — Operator Dashboards, Executive Dashboards  

## Chapter 41: Distributed Tracing
41.1 Trace Context Propagation (W3C TraceContext, B3)  
41.2 Span Hierarchies and Trace Trees  
41.3 Sampling Strategies — Head-Based, Tail-Based, Adaptive  
41.4 Tracing Across Async Boundaries (Queues, Events)  
41.5 Correlating Traces, Logs, and Metrics  
41.6 Distributed Tracing Tools (Jaeger, Zipkin, OpenTelemetry)  

## Chapter 42: Deployment & Release Patterns
42.1 Blue-Green Deployment  
42.2 Canary Deployment  
42.3 Rolling Deployment  
42.4 Feature Flags — Decoupling Deploy from Release  
42.5 Dark Launches — Testing in Production Without Exposure  
42.6 A/B Testing Infrastructure  
42.7 Traffic Shifting and Weighted Routing  
42.8 Rollback Strategies — Instant, Gradual, Data-Aware  
42.9 Database Migration Safety — Expand-Contract Pattern  
42.10 Immutable Infrastructure and Infrastructure as Code  

---

# PART X — SPECIALIZED SYSTEM PATTERNS

## Chapter 43: Content Delivery & Edge Patterns
43.1 CDN Architecture — PoPs, Origin Shield, Edge Caching  
43.2 Cache-Control Headers — max-age, s-maxage, stale-while-revalidate  
43.3 Cache Invalidation at CDN Scale (Purge, Soft Purge, Tagging)  
43.4 Edge Compute — Running Logic at the CDN  
43.5 Static vs. Dynamic Content Acceleration  
43.6 Video Streaming — Adaptive Bitrate, Chunked Delivery (HLS, DASH)  
43.7 Image Optimization Pipeline — Resize, Compress, Format on the Fly  
43.8 Edge-Side Includes (ESI) — Partial Page Caching  

## Chapter 44: Search System Patterns
44.1 Indexing Pipeline — Crawl, Parse, Index  
44.2 Inverted Index Construction and Updates  
44.3 Relevance Scoring — TF-IDF, BM25, Learning to Rank  
44.4 Query Understanding — Tokenization, Stemming, Synonyms  
44.5 Typeahead / Autocomplete — Trie and Prefix Index  
44.6 Faceted Search and Aggregation  
44.7 Search Result Caching  
44.8 Near-Real-Time Indexing (NRT)  
44.9 Federated Search — Querying Across Multiple Indexes  
44.10 Vector Search and Semantic Search (Embeddings + ANN)  

## Chapter 45: Notification & Messaging System Patterns
45.1 Notification Fan-Out — Push to Many Devices  
45.2 Push Notification Architecture (APNS, FCM)  
45.3 Email Delivery Pipeline — Queuing, Templating, Throttling, Reputation  
45.4 SMS and Voice Notification Delivery  
45.5 In-App Notification Store and Read Tracking  
45.6 Notification Preferences and Channel Routing  
45.7 Notification Deduplication and Collapsing  
45.8 Chat and Real-Time Messaging Architecture  
45.9 Presence Detection — Online/Offline Status  

## Chapter 46: Unique ID Generation Patterns
46.1 UUID (v4) — Universally Unique, Unordered  
46.2 UUID v7 — Time-Sortable UUIDs  
46.3 Snowflake IDs — Time + Machine + Sequence  
46.4 ULID — Universally Unique Lexicographically Sortable  
46.5 Database Sequence Generators and Their Limits  
46.6 Ticket Servers (Flickr-style)  
46.7 Clock Skew and Its Impact on ID Ordering  
46.8 Short IDs, Vanity IDs, and URL Shortening  
46.9 Choosing the Right ID Strategy for Your System  

## Chapter 47: File & Object Storage Patterns
47.1 Object Storage Architecture (S3-style)  
47.2 Chunked Upload and Multipart Upload  
47.3 Resumable Uploads  
47.4 Content-Addressable Storage (CAS)  
47.5 Deduplication — Block-Level, File-Level  
47.6 Pre-Signed URLs — Secure Direct Upload/Download  
47.7 Storage Tiering — Hot, Warm, Cold, Archive  
47.8 Lifecycle Policies and Automated Tiering  
47.9 Metadata Store Design for File Systems  
47.10 Large File Distribution — BitTorrent, P2P-Assisted CDN  

## Chapter 48: Geo & Location Patterns
48.1 Geohashing — Grid-Based Spatial Encoding  
48.2 S2 Geometry and H3 — Hierarchical Spatial Indexes  
48.3 Quadtrees — Adaptive Spatial Partitioning  
48.4 R-Trees — Bounding Box Spatial Index  
48.5 Proximity Search — Finding Nearby Entities  
48.6 Geofencing — Region-Based Triggers  
48.7 Location Tracking and History Storage  
48.8 Map Tile Serving and Caching  

---

# PART XI — DATA PIPELINE & INTEGRATION PATTERNS

## Chapter 49: Data Pipeline Architecture
49.1 Batch Pipelines — Scheduled, Bulk Processing  
49.2 Streaming Pipelines — Continuous, Real-Time  
49.3 Hybrid (Lambda) Pipelines — Batch + Stream  
49.4 Change Data Capture (CDC) — Debezium, DynamoDB Streams  
49.5 Schema Registry and Schema Evolution (Avro, Protobuf)  
49.6 Data Quality Gates — Validation, Anomaly Detection  
49.7 Exactly-Once Delivery in Pipelines  
49.8 Pipeline Orchestration — Airflow, Dagster, Prefect  
49.9 Data Lineage and Provenance Tracking  

## Chapter 50: Integration & Interoperability Patterns
50.1 Anti-Corruption Layer — Protecting Domain Integrity  
50.2 Adapter Pattern — Translating Between Systems  
50.3 Sidecar Pattern for Cross-Cutting Integration  
50.4 Shared Database — The Anti-Pattern and When It's Acceptable  
50.5 Event-Driven Integration — Decoupled System Boundaries  
50.6 File-Based Integration — Batch File Transfer  
50.7 API-Led Connectivity — System, Process, Experience APIs  
50.8 Data Synchronization — One-Way, Two-Way, Conflict Resolution  

---

# PART XII — DESIGN PATTERN QUICK REFERENCE CATALOG

## Chapter 51: Structural Patterns Summary

| Pattern | Core Idea | Chapter Ref |
|---|---|---|
| Sidecar | Attach auxiliary functionality alongside primary | 32.5 |
| Ambassador | Proxy for outbound connections | 32.6 |
| Anti-Corruption Layer | Translate between bounded contexts | 31.7, 50.1 |
| Backend for Frontend (BFF) | Tailored API per client type | 11.3 |
| Gateway Aggregation | Combine multiple backend calls | 11.4 |
| Strangler Fig | Incremental migration | 31.6 |

## Chapter 52: Behavioral Patterns Summary

| Pattern | Core Idea | Chapter Ref |
|---|---|---|
| Circuit Breaker | Stop calling failing dependency | 27.2 |
| Retry with Backoff | Tolerate transient failures | 27.1 |
| Bulkhead | Isolate failure to a partition | 27.3 |
| Saga | Distributed transaction via compensations | 28.4 |
| Outbox | Reliable event publishing | 28.5 |
| CQRS | Separate read and write paths | 22.7 |
| Event Sourcing | Persist events, derive state | 10.4 |

## Chapter 53: Resource & Traffic Patterns Summary

| Pattern | Core Idea | Chapter Ref |
|---|---|---|
| Rate Limiting | Protect from excess requests | 23 |
| Load Shedding | Reject to preserve the whole | 25.13 |
| Admission Control | Gate entry to protect SLOs | 24.8 |
| Throttle | Slow down without rejecting | 23.14 |
| Fair Queuing | Equal share per client/tenant | 24.3 |
| Quota | Hard/soft limits per tenant | 24.2 |
| Token Bucket | Smooth burst + sustained rate | 23.2 |
| Backpressure | Consumer controls producer speed | 9.8 |
| Sharded Counter | Distribute write load for counting | 22.11.2 |

## Chapter 54: Data Patterns Summary

| Pattern | Core Idea | Chapter Ref |
|---|---|---|
| Cache-Aside | Lazy load into cache | 16.2 |
| Write-Behind | Async persist from cache | 16.5 |
| Read-Through | Cache mediates all reads | 16.3 |
| Materialized View | Pre-computed query result | 17.9 |
| Sharding | Horizontal data partitioning | 14 |
| Replication | Copies for availability + reads | 15 |
| CDC | Stream database changes | 21.11 |
| Denormalization | Trade storage for read speed | 13.10 |

---

# APPENDICES

## Appendix A: System Design Interview Framework
A.1 Requirements Gathering Checklist  
A.2 Back-of-the-Envelope Estimation Templates  
A.3 High-Level Design → Detailed Design Walkthrough  
A.4 Bottleneck Identification and Scaling Discussion  
A.5 Tradeoff Articulation Framework  

## Appendix B: Technology Reference Matrix
B.1 Database Comparison Matrix (SQL, NoSQL, NewSQL, Specialized)  
B.2 Message Queue Comparison (Kafka, RabbitMQ, SQS, Pulsar)  
B.3 Cache Technology Comparison (Redis, Memcached, Hazelcast)  
B.4 Orchestration Comparison (Temporal, Step Functions, Airflow)  
B.5 Load Balancer Comparison (HAProxy, NGINX, ALB/NLB, Envoy)  
B.6 Consensus System Comparison (ZooKeeper, etcd, Consul)  

## Appendix C: Cheat Sheets
C.1 Latency Numbers at a Glance  
C.2 CAP / PACELC Quick Reference  
C.3 Consistency Model Hierarchy  
C.4 Common Failure Modes and Mitigations  
C.5 Partitioning Strategy Decision Tree  
C.6 Caching Strategy Decision Tree  
C.7 When to Use Sync vs. Async Communication  

## Appendix D: System Design Case Study Index
D.1 URL Shortener  
D.2 Pastebin / Code Sharing  
D.3 Rate Limiter Service  
D.4 Distributed Key-Value Store  
D.5 Unique ID Generator  
D.6 News Feed / Timeline  
D.7 Chat System (1:1, Group, Presence)  
D.8 Notification Service  
D.9 Search Autocomplete / Typeahead  
D.10 Web Crawler  
D.11 Video Streaming Platform (YouTube-scale)  
D.12 Ride-Sharing Service (Uber/Lyft)  
D.13 Proximity Service / Nearby Friends  
D.14 Google Maps / Navigation  
D.15 Distributed Task Scheduler  
D.16 Payment System  
D.17 Hotel / Ticket Reservation System  
D.18 Social Graph Service  
D.19 Metrics and Monitoring System  
D.20 Distributed File System (GFS/HDFS)  
D.21 Object Storage (S3-scale)  
D.22 Real-Time Gaming Leaderboard  
D.23 Ad Click Event Aggregation  
D.24 Email Service  
D.25 Collaborative Document Editor (Google Docs)  
D.26 Stock Exchange / Order Book  

---

*Total: 54 Chapters · 12 Parts · 4 Appendices · ~550+ Sections*
