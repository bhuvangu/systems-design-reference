# The Architecture Cookbook
## Battle-Tested Patterns, Their Variants, and the Tradeoffs That Decide Which One You Pick

*A recipe-style reference. Each pattern states the problem it solves, how it works, the leading approaches with pros and cons, and how to choose.*

---

## Table of Contents

### Part I — Data Layer Patterns
1\. Single-Leader Replication (Primary–Replica)  
2\. Multi-Leader Replication (Active–Active)  
3\. Leaderless / Quorum Replication  
4\. Sharding (Horizontal Partitioning)  
5\. CQRS (Command Query Responsibility Segregation)  
6\. Event Sourcing  
7\. Outbox Pattern (Reliable Event Publishing)  
8\. Change Data Capture (CDC)  
9\. Materialized View / Read Model  

### Part II — Application & Workflow Patterns
10\. Notification Delivery Pattern  
11\. Counter Pattern (Distributed Counting)  
12\. Feed / Timeline Pattern  
13\. Booking / Reservation Pattern  
14\. Job Scheduling & Triggering Pattern  
15\. Timer / Delayed-Execution Pattern  
16\. Rate Limiter Pattern  
17\. Idempotency Pattern  
18\. Saga Pattern (Distributed Transactions)  
19\. Leaderboard / Top-K Pattern  
20\. Search & Autocomplete Pattern  

### Part III — Resilience & Traffic Patterns
21\. Circuit Breaker  
22\. Retry with Backoff and Jitter  
23\. Bulkhead Isolation  
24\. Load Shedding & Admission Control  
25\. Cell-Based Architecture  

---

# PART I — DATA LAYER PATTERNS

## 1. Single-Leader Replication (Primary–Replica)

- **Problem:** Scale reads and gain redundancy without dealing with write conflicts.
- **How it works:**
  - One node accepts all writes; one or more replicas receive the changes and serve reads.
  - Because only the leader mutates state, write ordering is total and unambiguous — the default for Postgres, MySQL, etc.
  - Reads scale by adding replicas behind a proxy or app-level routing.
- **Key tradeoffs:**
  - Writes do **not** scale — every write funnels through one node.
  - Replication lag means a read right after a write can be stale; mitigate with read-your-writes routing or reading from the leader for critical paths.
  - Failover has a downtime window and risks split-brain if the old leader returns; needs fencing tokens + consensus coordination (e.g., Patroni + etcd).
- **Pros:** simple, strong write ordering, mature tooling.
- **Cons:** write throughput capped at one node, failover downtime, stale reads from lag.
- **When to use:** the default choice unless you specifically need to scale writes or survive a regional outage.

## 2. Multi-Leader Replication (Active–Active)

- **Problem:** Keep accepting writes during partitions, or give geographically spread users low write latency.
- **How it works:**
  - Multiple nodes (typically one per region) each accept writes and replicate to the others asynchronously.
  - A user writes to their local leader and feels local latency; the change propagates to other regions.
  - Used by CockroachDB multi-region mode, Cassandra (in its way), classic CouchDB.
- **The hard part — conflict resolution:**
  - Two regions can accept conflicting writes before they sync.
  - Last-Write-Wins by timestamp silently drops data and is dangerous under clock skew.
  - Better: encode merge semantics with CRDTs (counters sum, sets union) or surface conflicts to the app via version vectors.
  - Genuine write-write conflicts (e.g., decrementing inventory) make correctness genuinely hard.
- **Pros:** write availability survives regional failure, low per-region write latency, no single write bottleneck.
- **Cons:** conflict resolution is unavoidable and often lossy, weaker consistency, much more operational complexity.
- **When to use:** only when you truly need multi-region write availability — many teams want single-leader with a fast failover region instead.

## 3. Leaderless / Quorum Replication

- **Problem:** Eliminate the failover cliff and tune consistency vs. latency per operation.
- **How it works:**
  - No leader; writes go to all replicas, succeeding when **W** of **N** acknowledge.
  - Reads hit enough replicas that **R + W > N**, guaranteeing overlap with the latest write.
  - The Dynamo model — powers Cassandra and Riak.
- **Staleness handling:**
  - Read-repair fixes lagging replicas on the read path.
  - Anti-entropy uses Merkle trees to find and reconcile divergent ranges efficiently.
  - "Consistency" here is quorum consistency, not linearizability; concurrent writes still need version vectors.
- **Pros:** no failover downtime, tunable per-operation consistency, excellent availability and write scaling.
- **Cons:** eventual consistency by default, conflict handling pushed to the app, read/write amplification, quorum math to reason about.
- **When to use:** high-availability, write-scalable stores where you can tolerate eventual consistency.

## 4. Sharding (Horizontal Partitioning)

- **Problem:** A dataset or its write volume exceeds what one node can hold or handle.
- **How it works:**
  - Split data across nodes by a partition key — hash (even spread), range (range scans), or directory (flexible).
  - Each shard owns a disjoint slice and serves its own reads/writes — the only way to scale writes past one machine.
  - Consistent hashing with virtual nodes minimizes data movement when nodes are added/removed.
- **The decisive choice — the partition key:**
  - A good key spreads load evenly and co-locates data queried together.
  - A bad key creates hot partitions (one celebrity, one viral product) that overwhelm a single node.
- **Query-time pain:**
  - Queries without the partition key become scatter-gather across all shards.
  - Cross-shard transactions lose ACID, forcing sagas or two-phase commit.
- **Pros:** near-linear write/storage scaling, fault isolation per shard.
- **Cons:** hot-partition risk, painful cross-shard queries/joins, resharding is a major operation, wrong key is expensive to undo.
- **When to use:** once a single node is the proven bottleneck — not preemptively.

## 5. CQRS (Command Query Responsibility Segregation)

- **Problem:** A single schema can't efficiently serve both complex writes and complex reads.
- **How it works:**
  - Commands mutate a normalized, validation-heavy write store optimized for correctness.
  - Queries are served from denormalized read models tailored to specific screens.
  - The two are synced asynchronously, usually via events on every write.
  - You can serve reads from Elasticsearch, a graph store, and a cache off the same write stream.
- **Best fit:** read/write workloads that are wildly asymmetric, or where one schema can't serve both (e.g., normalized product catalog writes vs. a fat denormalized read document).
- **Pros:** independent scaling, read models tailored per use case, natural fit with event-driven systems.
- **Cons:** eventual consistency between sides, more infrastructure, pure overhead on simple CRUD.
- **When to use:** complex read/write asymmetry — *don't CQRS a contact form.*

## 6. Event Sourcing

- **Problem:** You need a perfect audit trail and the ability to reconstruct past state.
- **How it works:**
  - Store the ordered log of events, not current state (balance = fold of every deposit/withdrawal).
  - Derive current state by replaying events; cache snapshots to avoid replaying from zero.
  - Pairs naturally with CQRS — the event log is the write side, projections are the read side.
- **Superpowers:**
  - Complete immutable audit trail for free.
  - Reconstruct state as of any past moment.
  - Build new read projections retroactively by replaying history through new logic.
- **Pros:** complete audit trail, time-travel, retroactive projections, natural event-driven integration.
- **Cons:** high complexity, event-versioning burden (old events readable forever), unbounded log growth, steep learning curve.
- **When to use:** where auditability is a hard requirement — not as a default.

## 7. Outbox Pattern (Reliable Event Publishing)

- **Problem:** Update the DB *and* publish an event atomically, when the DB and broker are separate systems (the dual-write problem).
- **How it works:**
  - Write the event into an `outbox` table **in the same local DB transaction** as the business change.
  - A relay process reads unpublished rows and publishes them to the broker, marking them sent.
  - The relay uses CDC (lower latency) or polling.
- **Guarantee delivered:** at-least-once — the relay may publish, crash before marking sent, and republish, so **consumers must be idempotent.**
- **Pros:** eliminates the dual-write problem, guarantees the event if the data changed, works with any database.
- **Cons:** at-least-once (needs idempotent consumers), added relay infrastructure and latency, outbox cleanup to manage.
- **When to use:** the correct default for "save data and emit an event" — safer than publishing directly after commit.

## 8. Change Data Capture (CDC)

- **Problem:** Propagate every database change to downstream systems without modifying the application.
- **How it works:**
  - Tap the DB's write-ahead log (binlog/WAL) and turn each committed row change into a stream.
  - Tools like Debezium publish inserts/updates/deletes to Kafka.
  - The app keeps doing plain writes; search indexes, caches, warehouses react to the stream.
- **Strengths:** captures *every* change with correct ordering — including from legacy systems, batch jobs, and manual SQL; powers the streaming half of the Outbox pattern.
- **Pros:** zero application changes, complete and ordered change stream, great for sync and integration.
- **Cons:** couples consumers to physical schema (schema changes ripple outward), needs log access and connector ops, large initial snapshots are heavy, captures row changes not business intent.
- **When to use:** keeping search indexes, caches, or read models in sync with a source-of-truth database.

## 9. Materialized View / Read Model

- **Problem:** A query is too slow to run live (expensive joins/aggregations on every read).
- **How it works:**
  - Pre-compute the answer and store it as a flat table; reads become a simple lookup.
  - Refresh on a schedule, incrementally as data changes, or driven by an event stream.
- **Refresh strategy (the whole decision):**
  - **Full recompute:** simple but expensive; staleness = refresh interval.
  - **Incremental:** updates only affected rows; cheap at scale but hard to get correct for aggregates/joins.
  - **Event-driven:** rebuild as change events arrive; near-real-time at the cost of streaming infra.
- **Pros:** dramatically faster reads, offloads work from the source DB, purpose-built views per access pattern.
- **Cons:** staleness between refreshes, storage duplication, incremental-maintenance complexity.
- **When to use:** read-heavy paths where you can trade some freshness and storage for read latency.

---

# PART II — APPLICATION & WORKFLOW PATTERNS

## 10. Notification Delivery Pattern

- **Problem:** One business event ("payment received") must trigger a multi-step, long-lived sequence — email, then push, then a reminder if unread after 24h — reliably, across days, with retries and channel preferences, without losing steps on crash.

- **Approach A — Workflow-per-event (orchestration engine):**
  - Each event starts a durable workflow instance in Step Functions / Temporal / Cadence.
  - The workflow encodes the whole sequence; the engine persists progress after each step, so a crash resumes where it left off.
  - State, retries, and timeouts are the engine's job; logic reads top-to-bottom like ordinary code.
  - *Pros:* full flow in one readable place; durability/retries/long-waits handled; great visibility into in-flight workflows.
  - *Cons:* heavyweight stateful dependency; high event volumes strain it and get expensive; vendor/framework lock-in.

- **Approach B — Event + Timer-registered continuation (choreography):**
  - Each event is processed for its immediate step; if a later step is needed, the handler registers a callback with a **timer service** at the next-eligible-time.
  - When that time arrives, the timer service fires an invocation that drives the next step, which may register the following one.
  - No central definition — the flow emerges from each step scheduling the next. This is how reminder/drip systems scale.
  - *Pros:* extremely scalable and cheap; handlers are stateless and short-lived; state lives in the timer store; handles multi-day delays naturally.
  - *Cons:* end-to-end flow is implicit and scattered (harder to see/debug); you must build idempotency and dedup (timers can fire twice); reasoning requires tracing.

- **Approach C — Fan-out queue with per-channel workers:**
  - The event is published once to a topic; independent email/push/SMS consumers each deliver in parallel with their own retry and rate-limit policy.
  - Handles the *parallel* dimension cleanly, but on its own doesn't sequence delayed steps — usually combined with B's timer for "remind later."
  - *Pros:* channels scale and fail independently; clean separation; easy to add a channel.
  - *Cons:* no built-in sequencing or cross-channel coordination; "stop sending once read" must be layered on.

- **Choosing:**
  - Complex, auditable, business-critical, modest volume → orchestration engine (A).
  - High-volume, simple-but-delayed (reminders, drip) → event + timer continuation (B).
  - Most real platforms combine **B for scheduling + C for parallel channel fan-out.**

## 11. Counter Pattern (Distributed Counting)

- **Problem:** Count something hot (likes, views, quota usage) where a single row updated by millions of writers becomes a contention bottleneck — naive `value = value + 1` serializes on one lock.

- **Approach A — Sharded counters:**
  - Split the logical counter into N physical sub-counters; each increment hits a random shard.
  - The true value is the sum of all shards, computed at read time.
  - More shards = less contention but pricier reads (tune N).
  - *Pros:* eliminates write hotspots, scales with N, exact total.
  - *Cons:* reads must aggregate all shards; N must be tuned; exact-write cost per increment.

- **Approach B — Approximate counters (probabilistic):**
  - HyperLogLog (distinct counts) / Count-Min Sketch (frequencies) answer within a known error bound using tiny fixed memory.
  - Great for analytics and unique-visitor counts where ~1% error is irrelevant.
  - *Pros:* tiny constant memory, very fast, mergeable across nodes.
  - *Cons:* approximate by design; unsuitable for billing/inventory.

- **Approach C — Write-buffered counters (batch-and-flush):**
  - Increments accumulate in memory (Redis) and flush periodically as one aggregated DB write.
  - Converts a million `+1`s into one `+1,000,000`.
  - *Pros:* massive write throughput, smooths bursts, cheap durable writes.
  - *Cons:* unflushed counts lost on crash; read freshness lags the flush interval.

- **Choosing:**
  - Exact, high-write, read-rarely → sharded (A).
  - Analytics where approximation is fine → probabilistic (B).
  - Extreme burst absorption with small-loss tolerance → buffered (C).
  - Per-request quota counter → a Redis-backed atomic counter is often enough.

## 12. Feed / Timeline Pattern

- **Problem:** Build a personalized, ordered feed from everyone a user follows — fast, for hundreds of millions of users with very different follower counts. Core tension: do the assembly work at write time or read time?

- **Approach A — Fan-out on write (push):**
  - On post, write the post ID into every follower's precomputed feed.
  - Reading is a trivial lookup of an already-assembled list.
  - *Pros:* extremely fast reads; feed is ready before it's requested.
  - *Cons:* write amplification explodes for high-follower accounts (the celebrity problem); wasteful for inactive followers; storage duplication.

- **Approach B — Fan-out on read (pull):**
  - On read, fetch recent posts from everyone the user follows and merge them.
  - *Pros:* cheap writes; no storage duplication; no celebrity write-explosion.
  - *Cons:* slow, expensive reads; heavy read-time load; poor for users following thousands.

- **Approach C — Hybrid (the production answer):**
  - Fan-out on write for normal accounts; fan-out on read for celebrities.
  - A feed = precomputed feed (regular follows) merged with a live pull of recent celebrity posts.
  - *Pros:* balances read/write cost; handles long-tail and celebrity cases.
  - *Cons:* most complex; requires classifying accounts and merging two paths at read time.

- **Choosing:** Nearly every large feed (Twitter/X, Instagram) lands on the hybrid. Start with fan-out on write; add the pull path for high-follower accounts once write amplification bites.

## 13. Booking / Reservation Pattern

- **Problem:** Allocate a finite, exclusive resource (seat, room, slot) to exactly one of many concurrent users — no double-booking, and no holding inventory hostage while a user dawdles through checkout.

- **Approach A — Pessimistic locking:**
  - Lock the resource row when a user starts booking until the transaction completes.
  - *Pros:* simple, airtight correctness.
  - *Cons:* poor concurrency under contention; lock held during slow user steps; deadlock risk. Bad for flash sales.

- **Approach B — Optimistic concurrency with a reservation hold:**
  - Don't lock; create a short-lived `HELD` reservation with an expiry (e.g., 10 min).
  - User converts HELD → CONFIRMED by paying within the window; otherwise a timer service expires it and returns inventory.
  - Concurrent holds on the same resource fail on a version check or unique constraint.
  - This is how ticketing and flight booking actually work.
  - *Pros:* high concurrency, no long-held locks, mirrors real user flows, gives a fair window.
  - *Cons:* needs an expiry/timer mechanism; "held but unpaid" inventory reduces effective availability; careful hold→confirm handling.

- **Approach C — Two-phase reserve/confirm across services (TCC / Saga):**
  - When a booking spans services (reserve seat, charge card, issue ticket), each step *tries* (tentatively reserves), then all *confirm*; on failure, compensations *cancel*.
  - Extends the hold concept across a distributed transaction with no shared DB lock.
  - *Pros:* correctness across services without distributed locks; graceful partial-failure handling.
  - *Cons:* significant complexity; compensation logic per step; eventual consistency during the flow.

- **Choosing:**
  - Single DB, low contention → pessimistic (A).
  - High contention / flash sales → optimistic hold with timer expiry (B).
  - Multi-service bookings → TCC/saga (C).
  - Real systems layer a hold (B) inside a saga (C) spanning inventory and payment.

## 14. Job Scheduling & Triggering Pattern

- **Problem:** Run non-request work (reports, retries, deferred processing, ETL, cleanups) reliably — at the right time, in the right order, **exactly once** despite many scheduler replicas running for availability, and surviving worker crashes mid-execution.

- **The pipeline (mental model):** every robust system separates four concerns, so a failure in one doesn't corrupt the others:
  - **Trigger** — decides *when* a job becomes eligible.
  - **Dispatch** — decides *which instance* enqueues it (the exactly-once choke point).
  - **Queue** — buffers eligible work and decouples submission from execution.
  - **Execute** — workers pull, run, and acknowledge.

### 14a. Triggering Modes (what makes a job eligible)

- **Time-based (cron):** fire at 02:00 daily / every 5 min. Needs a cron-expression parser and explicit **timezone + DST** handling (store the zone, compute next-fire in it).
  - *Pros:* predictable, simple to reason about. *Cons:* clustering at round times (00:00 stampede); DST gaps/overlaps cause skipped or doubled runs.
- **Event-based:** fire when something happens (message arrives, file lands, row changes via CDC).
  - *Pros:* reactive, no polling lag. *Cons:* event volume can overwhelm; needs dedup.
- **Dependency-based (DAG):** fire when upstream jobs succeed (job C after A *and* B).
  - *Pros:* models real pipelines. *Cons:* needs a dependency resolver and failure-propagation rules.
- **Manual / API-triggered:** on-demand kick (operator, webhook, backfill request).
  - *Pros:* flexible, good for reruns. *Cons:* must guard against accidental double-submits (idempotency keys).
- **Continuation / self-scheduling:** a job's last act is to schedule its successor (see Notification pattern B). Good for long, sparse chains; flow is implicit.

### 14b. Dispatch — Ensuring Exactly-One Trigger

The hard part in a distributed setting: N scheduler replicas must not all fire the same scheduled job. Approaches:

- **Approach A — Leader election:** replicas elect a leader (ZooKeeper, etcd, Consul, or a DB lease); only the leader dispatches; a standby takes over on leader death.
  - *Pros:* simple model, one clear dispatcher. *Cons:* the leader is a throughput ceiling and a failure point during the election gap; needs fencing tokens to stop a zombie ex-leader from also dispatching.
- **Approach B — Database lease / lock per job:** each due job is claimed with an atomic `UPDATE … WHERE status='due' AND locked_until < now()` or `SELECT … FOR UPDATE SKIP LOCKED`. Whoever wins the row owns it.
  - *Pros:* no separate coordinator; scales dispatch across all replicas; naturally load-balances. *Cons:* DB becomes the contention point; needs lease expiry so a crashed claimer's job is reclaimed.
- **Approach C — Partitioned/sharded schedulers:** shard the schedule space (by job-ID hash or time bucket) so each replica owns a disjoint slice and dispatches independently.
  - *Pros:* horizontal dispatch scaling, no global leader. *Cons:* rebalancing on replica change; a shard owner's downtime delays its slice.
- **Choosing:** few jobs → leader (A); many jobs, want simple infra → DB lease (B); very high schedule volume → sharded (C).

### 14c. Execution — Queue + Worker Pool

- **Decouple dispatch from execution:** the dispatcher enqueues; a pool of workers pulls and runs. Lets you scale execution independently and get retries/DLQ for free.
- **Delivery semantics:**
  - **At-most-once:** ack before running — fast, but a crash loses the job. Rare.
  - **At-least-once:** ack after running — a crash re-runs the job; **the standard**, so jobs must be idempotent.
  - **Exactly-once:** effectively at-least-once + idempotency keys / dedup; "true" exactly-once execution doesn't exist across failures.
- **Lease + visibility timeout:** a pulled job is invisible to others for a lease window; the worker **heartbeats** to extend it for long jobs. If the worker dies, the lease expires and the job returns to the queue.
- **Zombie / stuck-job detection:** jobs whose lease expired without completion or acknowledgment are reaped and retried; cap retries to avoid poison-job loops.

### 14d. Prioritization & Fairness

- **Priority queues:** high-priority work jumps ahead (separate queues per priority, or a scored queue).
- **Weighted fair scheduling:** prevent one tenant/job-type from starving others (per-tenant queues drained proportionally).
- **Rate-limited dispatch:** throttle how fast a job class is released (protect a fragile downstream).
- *Trade-off:* strict priority can starve low-priority work; pure FIFO ignores urgency — most systems blend the two.

### 14e. Recurrence & Misfire Handling

- **One-shot vs recurring:** recurring jobs compute the *next* fire-time after each run.
- **Misfire policy (scheduler was down at fire-time):**
  - **Fire-once-now:** run a single catch-up immediately.
  - **Skip:** ignore missed runs, resume on the next slot.
  - **Backfill all:** run every missed occurrence (dangerous — a day of downtime can stampede 288 five-minute runs at once).
- **Overlap policy:** if the previous run is still going at the next fire-time — *allow concurrent*, *skip*, or *queue* the new run.

### 14f. Multi-Step Workflows (DAGs)

- Model dependent steps as a DAG in an orchestrator (Airflow, Temporal, Dagster, Step Functions) instead of hand-wiring sequencing.
- **Fan-out / fan-in:** split work into parallel branches, then aggregate (e.g., process 1000 partitions, then summarize).
- **Conditional branching & failure propagation:** decide downstream behavior when a branch fails (fail-fast, continue, compensate).
- *Pros:* visibility, retries, and resumability handled by the engine. *Cons:* another stateful dependency; engine throughput limits at extreme scale.

- **Overall pros:** scalable, resilient, observable; clean separation makes each layer independently testable.
- **Overall cons:** requires idempotency, dedup, lease/heartbeat, and zombie detection; misfire/overlap policy is easy to get wrong; the dispatch layer is the correctness-critical piece.

## 15. Timer / Delayed-Execution Pattern

- **Problem:** "Do X in 30 minutes / at this exact future time" (expire a booking, send a reminder, retry after backoff) at the scale of millions of pending timers, efficiently, surviving restarts.

- **Approach A — Timer wheel (in-memory):**
  - Circular buffer of time-slot buckets; each tick fires everything in the current bucket.
  - O(1) insert and expiry — how TCP timeouts and many in-process schedulers work.
  - Hierarchical wheels (seconds/minutes/hours hands) extend range without exploding memory.
  - *Pros:* O(1) insert/expire; handles millions of timers cheaply.
  - *Cons:* not durable by itself; bounded time range per wheel.

- **Approach B — Durable timer service (persisted):**
  - Store each timer with its fire-time; a poller queries `fire_time <= now()` and dispatches.
  - Survives crashes; handles arbitrarily long delays (days/weeks). Can use a delay queue (SQS delay, RabbitMQ TTL+DLX).
  - Engine behind notification reminders and booking expiry.
  - *Pros:* durable, arbitrary delays, simple to reason about.
  - *Cons:* polling overhead and latency granularity; DB load at scale; due-time storms if many timers share an instant.

- **Approach C — Hybrid (durable store + in-memory wheel):**
  - Persist all timers for durability; load the near-future window into a wheel for precise, low-latency firing; refill periodically.
  - *Pros:* durable *and* precise *and* scalable.
  - *Cons:* most complex; must handle the load/refill boundary and dedup on restart.

- **Choosing:**
  - Short, in-process timeouts → wheel (A).
  - Long, durable business delays at moderate precision → persisted store / delay queue (B).
  - Large-scale, precise, durable scheduling → hybrid (C).

## 16. Rate Limiter Pattern

- **Problem:** Cap how many requests a client may make in a window — protect from overload, enforce fairness, meter quotas. The algorithm controls burst handling and precision.

- **Approach A — Token bucket:**
  - Bucket holds up to B tokens, refills at a steady rate; each request spends one; empty → reject/queue.
  - Allows short bursts (spend the bucket) while enforcing a sustained average — the most widely used.
  - *Pros:* permits controlled bursts, smooth sustained rate, simple and memory-light.
  - *Cons:* burst behavior must be tuned (bucket size); slightly less intuitive than fixed windows.

- **Approach B — Window counters:**
  - **Fixed window:** count per calendar interval; trivial, but allows a double-rate burst at the boundary.
  - **Sliding window log:** timestamp every request and count the trailing window; accurate but memory-heavy.
  - **Sliding window counter:** weights the previous window's count — the common production compromise.
  - *Pros:* fixed window is dead simple; sliding window is precise.
  - *Cons:* fixed window has boundary bursts; sliding log is memory-expensive at scale.

- **Approach C — Distributed rate limiting:**
  - **Centralized (Redis):** atomic increment/check on a shared counter — globally accurate, but adds a network hop and a hotspot/dependency.
  - **Local-share:** each node enforces limit/N with no coordination — fast but imprecise and unfair under uneven traffic.
  - **Gossip:** nodes exchange approximate counts periodically.
  - **Cell-based:** confine a client to one cell whose local limiter is authoritative.

- **Choosing:**
  - Single node, bursty → token bucket (A).
  - Exact global limits → centralized Redis (C).
  - Extreme scale, approximate OK → local-share or gossip (C).
  - Most API gateways use token-bucket semantics backed by a centralized store.

## 17. Idempotency Pattern

- **Problem:** Networks retry. A "charge $50" that times out but actually succeeded gets retried → $100 charged. At-least-once delivery is the rule, so operations must be safe to repeat.
- **How it works (idempotency key):**
  - Client attaches a unique key per logical operation (e.g., a UUID).
  - Server records the key + result on first processing; on retry with the same key, it returns the *stored* result instead of re-executing.
  - Stripe's API is the textbook example.
  - Handle the concurrent-retry race with a unique constraint or atomic insert so only one wins; give keys a TTL.
- **What needs it:**
  - Naturally idempotent ops (`SET balance = 50`, `DELETE user X`) need no key.
  - Relative ops (`balance += 50`, "send email") compound on repeat — these need keys.
- **Pros:** makes retries safe, which makes the whole reliability story tractable.
- **Cons:** requires a dedup store, careful concurrent-retry handling, discipline to apply on every non-idempotent mutation.
- **Note:** a prerequisite for at-least-once messaging, the outbox pattern, and saga steps — they all assume idempotent consumers.

## 18. Saga Pattern (Distributed Transactions)

- **Problem:** A business operation spans multiple services with separate databases, so there's no distributed ACID transaction — yet you need all-or-nothing semantics, tolerating step 3 failing after steps 1–2 committed.
- **How it works:**
  - Break the transaction into a sequence of local transactions, each in one service.
  - Every step has a **compensating action** that semantically undoes it (refund, release inventory).
  - On failure, run compensations for completed steps in reverse.
  - Compensation is *semantic* undo (issue a refund), not rollback — so intermediate states are briefly visible (the key difference from ACID).
- **Coordination styles:**
  - **Choreography:** each service listens for events and emits the next — simple for short sagas, but the flow is implicit and hard to follow as it grows.
  - **Orchestration:** a central orchestrator invokes each step and triggers compensations — more visible and manageable for complex flows, at the cost of a coordinator.
- **Pros:** enables long-running cross-service transactions without distributed locks; resilient to partial failure.
- **Cons:** compensation logic per step is real work; eventual consistency with visible intermediate states; hard to debug; some actions can't be cleanly compensated (you can't un-send a physical package).
- **Tip:** design pivot transactions carefully — the point past which the saga must complete rather than roll back.

## 19. Leaderboard / Top-K Pattern

- **Problem:** Maintain a real-time ranking (top scorers, trending posts) with frequent updates and fast "top N" and "what's my rank?" queries — sorting the whole dataset per read is hopeless at scale.
- **How it works:**
  - Use a **sorted set** (Redis `ZSET`): members ordered by score, O(log N) updates, O(log N + M) range reads.
  - Increment score, fetch top 10, query a member's rank — all efficient built-ins.
  - One structure often suffices for a global leaderboard.
- **At larger scale / time windows:**
  - Shard the sorted set or keep per-window structures and merge.
  - Pair exact top-K with approximate heavy-hitter detection (Count-Min Sketch) before precise ranking.
  - For "trending," store timestamped events and apply a decay function rather than raw counts.
- **Pros:** real-time ranks, efficient top-K and rank queries, simple with the right primitive.
- **Cons:** memory-bound for huge member counts; sharding complicates exact global ranks; time-windowed/trending variants add complexity.
- **Lesson:** reach for the right primitive (sorted set) before building ranking machinery by hand.

## 20. Search & Autocomplete Pattern

- **Problem:** Find content by free-text query and suggest completions as the user types, over a large corpus, in milliseconds — a `LIKE '%term%'` scan doesn't scale or rank.
- **Full-text search:**
  - Built on the **inverted index**: term → list of documents, so a query is a fast intersection of postings lists.
  - Engines (Elasticsearch, Solr) keep this index as a secondary read path, synced from the source of truth via CDC or an indexing pipeline.
  - Relevance ranking (BM25, learning-to-rank) orders results; tokenization, stemming, synonyms shape matching.
- **Autocomplete:**
  - Optimized for prefix matching and latency.
  - A **trie** returns completions in time proportional to prefix length.
  - Top suggestions per prefix are often precomputed and cached so a keystroke is one lookup.
  - Suggestion ranking blends popularity, recency, personalization.
- **Pros:** sub-second search and instant suggestions over huge corpora; rich relevance control.
- **Cons:** the index is a second copy to keep in sync (eventual consistency); indexing pipelines and relevance tuning are ongoing work; near-real-time indexing adds complexity.

---

# PART III — RESILIENCE & TRAFFIC PATTERNS

## 21. Circuit Breaker

- **Problem:** Continuing to call a failing/slow dependency wastes resources, ties up threads, and cascades failure upstream until everything collapses.
- **How it works (three states):**
  - **Closed:** calls pass through; failures are counted.
  - **Open:** once failures cross a threshold (count or rate), calls fail fast or fall back without attempting — protecting both sides.
  - **Half-open:** after a cooldown, a trial trickle is allowed; success → close, failure → re-open.
- **Best combined with a fallback:** when open, return a cached value, default, or degraded result instead of a hard error. Tune per dependency and per endpoint.
- **Pros:** prevents cascade failures, gives dependencies room to recover, fails fast instead of hanging.
- **Cons:** needs per-dependency threshold/timeout tuning; too sensitive → needless outages, too lenient → no protection; half-open transitions need careful testing.

## 22. Retry with Backoff and Jitter

- **Problem:** Transient failures usually resolve on a second attempt, but naive synchronized retries hammer a recovering service into staying down (retry storm).
- **How it works:**
  - **Exponential backoff:** double the wait each attempt (1s, 2s, 4s, 8s) to give the dependency room.
  - **Jitter:** randomize each delay so clients that failed together don't retry in synchronized waves; full jitter (random 0..ceiling) is the proven default.
- **Guardrails:**
  - Only retry idempotent operations (or those protected by idempotency keys).
  - Cap the number of attempts.
  - Enforce a **retry budget** so retries stay a small fraction of traffic, not a multiplier on a struggling service.
- **Pros:** cheaply recovers from transient failures, big lift to perceived reliability.
- **Cons:** retrying non-idempotent ops causes duplicates; unbounded/un-jittered retries cause metastable failures where the system can't recover even after the fault clears.
- **Tip:** pair with circuit breakers so you stop retrying a dependency that's genuinely down.

## 23. Bulkhead Isolation

- **Problem:** A failure or resource exhaustion in one part shouldn't sink the whole system (named after a ship's watertight compartments).
- **How it works:**
  - Isolate resource pools by concern.
  - If all downstream calls share one thread pool, a single slow dependency consumes every thread and starves healthy calls.
  - Give each dependency (or tenant, or request class) its own bounded pool of threads/connections/semaphore permits, so exhaustion is contained.
- **Applies at every level:** thread/connection pools in-process, separate process/container groups, and infrastructure-tier cells or swimlanes (where this meets cell-based architecture).
- **Pros:** contains blast radius, prevents cascading failure, enables per-class resource guarantees.
- **Cons:** partitioning reduces sharing efficiency (idle capacity can't help a saturated bulkhead); requires sizing each pool; over-partitioning fragments capacity.
- **The art:** choose partition boundaries that match real failure-isolation needs.

## 24. Load Shedding & Admission Control

- **Problem:** Under overload, serving *every* request serves *all* of them badly — latency climbs, queues grow, and throughput can collapse (congestion collapse). Better to reject some work so the rest succeeds.
- **How it works:**
  - **Load shedding:** drop/reject requests once past healthy capacity — return 503s fast rather than accepting doomed work.
  - **Admission control:** proactively gate requests at the door based on whether the SLO can be met.
  - Shed low-priority/retryable traffic first; protect high-value requests; base decisions on a real saturation signal (queue depth, latency, CPU), not a guessed static threshold.
- **Watch the retry interaction:** shedding a request the client immediately retries just moves load — respond with 429 + Retry-After, and clients must honor it.
- **Pros:** preserves throughput and latency for accepted work, prevents total collapse, enables graceful degradation.
- **Cons:** some users get rejected (a product decision); needs good saturation signals and priority classification; must coordinate with client retry behavior.
- **Why it matters:** keeps a system *partially* up instead of *fully* down — often the difference between a brownout and an outage.

## 25. Cell-Based Architecture

- **Problem:** In a shared system, a bad deploy, poison request, or runaway tenant can take down *everyone* because all users share infrastructure.
- **How it works:**
  - A cell is a complete, independent instance of the stack (own compute, data, dependencies) serving a subset of users.
  - A routing layer maps users to cells (by ID hash, tenant, or geography); cells share no state, so a failure in one is invisible to the others.
  - Deploys roll out cell-by-cell, turning a bad release into a one-cell incident caught before it spreads.
- **Shuffle sharding (the refinement):**
  - Assign each user to a random *combination* of nodes rather than one cell.
  - Even if some nodes are knocked out, the chance any two users share their *entire* node set is tiny, so most users keep at least one working node.
- **Pros:** dramatically reduced blast radius, safe incremental deploys, fault and tenant isolation, predictable per-cell scaling.
- **Cons:** significant operational complexity (many copies), hard cross-cell operations and rebalancing, fragmented capacity, the routing layer becomes critical infrastructure.
- **When to use:** when blast-radius containment justifies the multiplied operational cost — typically at large scale or with strict isolation requirements.

---

*End of cookbook. Pairs with the companion Table of Contents reference — use this for "which pattern and why," and the reference for "how it works underneath."*
