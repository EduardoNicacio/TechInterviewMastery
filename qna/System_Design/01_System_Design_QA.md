# System Design & Distributed Systems – Interview Questions & Answers

## Core Concepts

**Question**: Explain the CAP theorem and the trade-offs between CP and AP systems.

**Answer**: CAP states that a distributed system can guarantee at most two of Consistency, Availability, and Partition Tolerance simultaneously. CP systems sacrifice availability during partitions (e.g., HBase), while AP systems sacrifice consistency for availability (e.g., DynamoDB). In practice, partitions are inevitable, so you choose between CP and AP.

---

**Question**: What is PACELC and how does it extend the CAP theorem?

**Answer**: PACELC adds the trade-off between latency and consistency during normal operation (no partition). It states: if a Partition occurs (P), choose between Availability (A) and Consistency (C); Else (E), choose between Low latency (L) and Consistency (C). This captures that even without partitions, systems like Cassandra prioritize low latency over strong consistency.

---

**Question**: Compare strong, eventual, causal, and read-your-writes consistency models.

**Answer**: Strong consistency guarantees all reads return the latest write. Eventual consistency converges to the same value given no new writes. Causal consistency preserves cause-effect order. Read-your-writes ensures a writer always sees its own writes. Each trades performance for guarantees.

---

**Question**: Distinguish between latency, throughput, and bandwidth in distributed systems.

**Answer**: Latency is the time for a single request to complete (measured in ms). Throughput is the number of requests processed per second. Bandwidth is the maximum data transfer capacity of a channel (bits/sec). Low latency doesn't guarantee high throughput, and vice versa.

---

**Question**: Compare hash partitioning, range partitioning, and consistent hashing.

**Answer**: Hash partitioning distributes data by a hash function, enabling even distribution but poor range queries. Range partitioning splits data by key ranges (good for scans, risk of hotspots). Consistent hashing minimizes remapping when nodes are added/removed, using a ring topology where each node owns a range of hash values.

---

**Question**: Describe single-leader, multi-leader, and leaderless replication.

**Answer**: Single-leader has one node accept writes and replicate to followers (simple, but single point of write failure). Multi-leader allows multiple nodes to accept writes and conflict-resolution is needed. Leaderless (e.g., Dynamo) lets any node accept writes with quorum-based read repair.

---

**Question**: How do quorum reads and writes work in distributed systems?

**Answer**: Quorum requires R replicas to read and W replicas to write such that R + W > N (total replicas). This ensures at least one overlapping node has the latest write, enabling strong consistency. Common choices: W = N/2 + 1, R = N/2 + 1.

---

**Question**: Explain NTP, Lamport clocks, and vector clocks for time synchronization.

**Answer**: NTP synchronizes real clocks across machines via hierarchical time servers. Lamport clocks provide logical ordering using a monotonically increasing counter. Vector clocks extend this by tracking per-node counters, enabling causality detection across distributed processes.

---

**Question**: What are hybrid logical clocks (HLC) and why are they useful?

**Answer**: HLC combines physical (NTP) and logical clocks to provide causality tracking with bounded drift. They remain close to physical time while acting as logical clocks, making them ideal for systems like CockroachDB (which uses HLC) that need both external consistency and distributed ordering.

---

**Question**: List and explain the eight fallacies of distributed computing.

**Answer**: (1) The network is reliable, (2) latency is zero, (3) bandwidth is infinite, (4) the network is secure, (5) topology never changes, (6) there is one administrator, (7) transport cost is zero, (8) the network is homogeneous. Ignoring these leads to brittle systems that fail under real-world conditions.

---

## Consensus & Coordination

**Question**: Summarize Paxos in simple terms.

**Answer**: Paxos achieves consensus across unreliable nodes in two phases: Phase 1 (Prepare/Promise) where the proposer picks a ballot number and acceptors promise not to accept lower ballots, and Phase 2 (Accept/Accepted) where the proposer sends the value and acceptors persist it. A value is chosen when a majority of acceptors accept the same proposal.

---

**Question**: How does Raft handle leader election and log replication?

**Answer**: Raft divides time into terms; each server starts as a follower. If a follower hears no heartbeat, it becomes a candidate, requests votes, and becomes leader if it wins a majority. The leader accepts client requests, appends entries to its log, and replicates them to followers, committing once a majority acknowledges.

---

**Question**: What is the Zab protocol and how does it compare to Raft?

**Answer**: Zab (ZooKeeper Atomic Broadcast) is used by Apache ZooKeeper for crash-recovery consensus. Like Raft, it elects a leader and replicates state changes. Unlike Raft's randomized timeouts, Zab uses a more deterministic discovery phase. Both provide linearizable writes but differ in epoch management.

---

**Question**: Describe the Gossip protocol and its use cases.

**Answer**: Gossip (epidemic) protocols periodically exchange state with random peers. Information spreads exponentially, providing eventual consistency without centralized coordination. Used in Cassandra, Dynamo, and Redis Cluster for failure detection and metadata propagation.

---

**Question**: How does the SWIM membership protocol work?

**Answer**: SWIM (Scalable Weakly Consistent Infection-style Membership) combines gossip with suspicion. Each node periodically pings a random member; if no ack, it initiates an indirect probe via a third node. A node is marked as suspicious before final removal to avoid false positives.

---

**Question**: How do ZooKeeper and etcd provide distributed coordination?

**Answer**: Both provide a hierarchical key-value store with atomic compare-and-swap operations, watches, and leases. ZooKeeper uses Zab; etcd uses Raft. Applications use them for leader election, service discovery, and configuration management via ephemeral znodes (ZooKeeper) or leases (etcd).

---

**Question**: What was Google's Chubby lock service and its design principles?

**Answer**: Chubby is a coarse-grained distributed lock service using Paxos for consensus. Clients maintain a lease to a lock cell; locks are associated with a sequencer for ordering. It favors consistency over availability for lock operations (CP system) and inspired ZooKeeper and etcd.

---

**Question**: Compare two-phase commit (2PC) and three-phase commit (3PC).

**Answer**: 2PC has a coordinator that asks all participants to prepare, then commits (blocking if coordinator fails). 3PC adds a pre-commit phase and timeout, reducing blocking but at the cost of extra round trips. 2PC is simpler but less resilient; 3PC avoids blocking in more failure scenarios.

---

## Storage Patterns

**Question**: Compare LSM Trees and B-Trees for write-heavy workloads.

**Answer**: LSM Trees (LevelDB, Cassandra) buffer writes in memory, flush to immutable SSTables, and merge in background. They excel at writes by sequential I/O. B-Trees (InnoDB) update data in-place with random I/O, offering faster reads but slower writes due to page splits.

---

**Question**: How do SSTables and compaction work in LSM-based stores?

**Answer**: SSTables (Sorted String Table) are immutable, sorted key-value files. Compaction merges multiple SSTables into one, discarding tombstones and redundant entries. Two common strategies: size-tiered (merge same-size files) and leveled (merge across levels) — both reclaim space and bound read amplification.

---

**Question**: What is a Write-Ahead Log (WAL) and why is it critical?

**Answer**: WAL records every mutation to a persistent log before applying it to the main data structure. On crash recovery, the log is replayed to restore durability. Used in PostgreSQL and Cassandra to ensure no data loss without flushing main storage on every write.

---

**Question**: Describe common database sharding strategies.

**Answer**: Key-based sharding uses a hash of the shard key (even distribution, poor range queries). Range sharding splits by key ranges (good for scans, hotspot risk). Directory-based uses a lookup service mapping keys to shards (flexible, adds a hop). Geo-sharding partitions by user location for latency.

---

**Question**: Explain CQRS (Command Query Responsibility Segregation) in depth.

**Answer**: CQRS separates read (query) and write (command) models, allowing different data stores, schemas, and scaling strategies for each. Commands use an optimized write store; queries use denormalized read replicas, views, or search indexes. Often paired with Event Sourcing for full audit trails.

---

**Question**: How does Event Sourcing work and when should you use it?

**Answer**: Event Sourcing stores each state change as an immutable, append-only event. The current state is rebuilt by replaying events. It provides a full audit log and enables temporal queries (past states). Use when audit compliance, complex state reconstruction, or event-driven integrations are needed.

---

**Question**: What are materialized views and how do they improve performance?

**Answer**: Materialized views are pre-computed query results stored as tables. They avoid expensive joins/aggregations at read time by refreshing periodically or on change. Common in OLAP databases (PostgreSQL, BigQuery) for dashboards and reporting where query speed is critical.

---

**Question**: Compare data lake, data warehouse, and lakehouse architectures.

**Answer**: Data warehouses store structured, pre-processed data optimized for SQL analytics (Snowflake, Redshift). Data lakes store raw data (structured/semi-structured/unstructured) in cheap object storage (S3, ADLS). Lakehouses (Databricks, Iceberg) combine both: ACID transactions and schema enforcement on data lake storage.

---

**Question**: How would you design a time-series database?

**Answer**: Optimize for append-heavy writes and range scans by time. Use LSM Trees for write throughput, partition by time range, and downsample older data. Index by (metric, timestamp). Examples: InfluxDB uses a TSM engine; TimescaleDB uses hypertables; Prometheus uses a custom TSDB with compaction.

---

**Question**: Describe the architecture of a blob storage system like S3 or Azure Blob.

**Answer**: Blobs are stored in flat namespaces (buckets/containers) with a key mapping to a storage node via a distributed partition layer. A metadata service tracks blob locations, while data is replicated across availability zones for durability. Large blobs are split into chunks stored across many nodes.

---

## Communication

**Question**: How does gRPC work and what are its key features?

**Answer**: gRPC uses HTTP/2 for multiplexed streams, Protocol Buffers for efficient serialization, and supports four call types: unary, server-streaming, client-streaming, and bidirectional streaming. It generates client/server stubs from `.proto` files and is ideal for inter-service communication in microservices.

---

**Question**: Compare REST and GraphQL trade-offs.

**Answer**: REST exposes fixed endpoints with clear caching semantics; GraphQL lets clients request exactly the fields they need via a single endpoint. REST is simpler to cache; GraphQL reduces over-fetching but requires a resolver layer and has complex query cost management. Both coexist in practice.

---

**Question**: How do Kafka and RabbitMQ differ as message queues?

**Answer**: Kafka is a distributed log optimized for high-throughput, replayable event streaming with persistent offsets. RabbitMQ is a traditional broker with sophisticated routing (exchanges, bindings) and push-based delivery. Kafka suits event sourcing and stream processing; RabbitMQ fits transactional messaging and RPC.

---

**Question**: Explain the pub/sub pattern and its common use cases.

**Answer**: Publishers emit messages to a topic/broker; subscribers receive messages without knowing each other. This decouples producers and consumers, enabling fan-out (one-to-many) delivery. Used in notification systems, event-driven architectures, and real-time data pipelines.

---

**Question**: Compare request-reply with event-driven communication.

**Answer**: Request-reply is synchronous: a client sends a request and waits for a response (tight coupling, simple). Event-driven is asynchronous: the producer emits events and consumers react independently (loose coupling, better scalability). Event-driven suits cross-service workflows; request-reply fits simple CRUD APIs.

---

**Question**: Compare long-polling, SSE, and WebSockets for real-time data.

**Answer**: Long-polling holds an HTTP request open until data arrives (simple, high latency). SSE (Server-Sent Events) is a unidirectional stream over HTTP, auto-reconnects, ideal for push notifications. WebSockets provide full-duplex bidirectional communication over a persistent TCP connection, lowest latency.

---

**Question**: Why is idempotency critical in distributed messaging?

**Answer**: Idempotency ensures processing a message multiple times produces the same result. With at-least-once delivery, duplicates are inevitable. Implement via idempotency keys (stored with dedup windows) or using upsert semantics in the consumer to prevent double-charges or duplicate records.

---

**Question**: What is backpressure and how do you handle it?

**Answer**: Backpressure occurs when a consumer is slower than a producer. Strategies include: bounded queues (block producer when full), sliding window (drop old messages), rate limiting (throttle producer), or reactive streams (demand-based flow control via protocols like RSocket or Kafka consumer pause/resume).

---

## Resilience & Reliability

**Question**: Describe the Circuit Breaker pattern.

**Answer**: A circuit breaker wraps a remote call in three states: Closed (normal, calls pass through), Open (fail fast when error threshold exceeded), Half-Open (allow test calls after timeout). Implementations (Hystrix, Polly) track failure counts and prevent cascading failures by short-circuiting degraded dependencies.

---

**Question**: What is the Bulkhead pattern and when is it used?

**Answer**: Bulkhead isolates resources into fixed pools so failure in one pool doesn't affect others. Each service client, thread pool, or connection pool gets its own partition. Example: separate thread pools for read and write operations so a write storm doesn't block reads.

---

**Question**: Explain retry with exponential backoff and jitter.

**Answer**: Exponential backoff increases wait time between retries (e.g., 1s, 2s, 4s, 8s). Jitter adds randomness to prevent thundering herd when many clients retry simultaneously. Formula: `min(cap, base * 2^attempt) + random(0, jitter)`.

---

**Question**: Distinguish between connect, read, and write timeouts.

**Answer**: Connect timeout limits how long to establish a TCP connection. Read timeout limits waiting for a response after sending a request. Write timeout limits sending request data. Setting appropriate timeouts prevents resource exhaustion; each timeout targets a different phase of the call.

---

**Question**: What are fallback patterns in distributed systems?

**Answer**: Fallbacks provide alternative behavior when a primary operation fails. Examples: return cached data, serve a degraded response, redirect to a secondary service, or return a default value. They maintain partial functionality rather than failing completely. Often combined with circuit breakers.

---

**Question**: How does graceful degradation maintain system usability under load?

**Answer**: The system disables non-essential features while keeping core functionality available. Examples: showing static content when recommendations are down, disabling comments during high traffic, or reducing image quality when CDN fails. Users experience limited but acceptable behavior.

---

**Question**: What are the core principles of chaos engineering?

**Answer**: Chaos engineering proactively injects failures into production to uncover weaknesses. Principles: (1) define a steady state baseline, (2) hypothesize the system will remain steady, (3) introduce realistic failures (latency, crashes, network partitions), (4) measure deviation, (5) automate experiments. Tools: Chaos Monkey, Gremlin.

---

**Question**: Differentiate between liveness and readiness probes.

**Answer**: A liveness probe checks if the application is alive (restart on failure). A readiness probe checks if the app can serve traffic (stop routing requests on failure). Liveness handles deadlocks; readiness handles temporary overload. Both are used in Kubernetes for automated recovery and load balancing.

---

**Question**: Compare token bucket and sliding window rate-limiting algorithms.

**Answer**: Token bucket accumulates tokens at a fixed rate; a request consumes one token. Bursts are allowed up to bucket capacity. Sliding window divides time into buckets, counting requests in the current window. Token bucket allows short bursts; sliding window provides smoother, memory-efficient limits.

---

**Question**: What are supervisory patterns inspired by Erlang/OTP?

**Answer**: Supervisors monitor child processes and apply a restart strategy: one-for-one (restart only the failed child), one-for-all (restart all children), rest-for-one (restart failed child and those started after). This builds self-healing systems where failures are isolated and recovered automatically.

---

## Observability

**Question**: How does distributed tracing work with OpenTelemetry?

**Answer**: Each request is assigned a trace ID propagated via context headers. Spans represent units of work (e.g., HTTP call, DB query) tagged with timing, service name, and metadata. OpenTelemetry provides SDKs and exporters (Jaeger, Zipkin) for collecting, sampling, and visualizing traces across services.

---

**Question**: Explain the RED method for services and USE method for infrastructure.

**Answer**: RED (Rate, Errors, Duration) tracks request rate, error count, and latency distribution per service — ideal for microservices monitoring. USE (Utilization, Saturation, Errors) tracks resource usage — ideal for infrastructure (CPU, memory, disk, network). Both provide actionable SRE dashboards.

---

**Question**: What makes structured logging better than plain-text logging?

**Answer**: Structured logging outputs key-value pairs (JSON) instead of free-form text, enabling machine parsing, filtering, and search in tools like Elasticsearch or Loki. Fields like `@timestamp`, `level`, `correlation_id`, and `service.name` allow aggregating logs across distributed systems.

---

**Question**: How do correlation IDs work across microservices?

**Answer**: A correlation ID is generated at the entry point and propagated via HTTP headers (e.g., `X-Correlation-ID`) or message metadata. Each service includes it in logs and traces, allowing end-to-end request tracking across service boundaries. This simplifies debugging of multi-service transactions.

---

**Question**: Define SLI, SLO, and SLA and how they relate.

**Answer**: SLI (Service Level Indicator) measures a specific metric (e.g., latency p99). SLO (Service Level Objective) sets a target (e.g., p99 < 200ms over 30 days). SLA (Service Level Agreement) is the contractual commitment, often with penalties. SLOs drive engineering decisions; SLIs measure them.

---

**Question**: When should you use logs vs metrics vs traces?

**Answer**: Logs capture detailed, unstructured events (useful for debugging). Metrics are numeric, time-series aggregates (useful for alerting and dashboards). Traces show request flow across services (useful for latency analysis). Use all three together: traces for root cause, metrics for detection, logs for details.

---

## Scaling & Performance

**Question**: Compare horizontal vs vertical scaling.

**Answer**: Vertical scaling adds resources (CPU, RAM) to a single node (simple but has hardware limits). Horizontal scaling adds more nodes (complex but theoretically infinite, with fault tolerance). Modern systems favor horizontal scaling for resilience, distributing load via load balancers and sharding.

---

**Question**: Describe the caching layers from CDN to in-memory to distributed cache.

**Answer**: CDN caches static assets (images, JS, CSS) at edge locations close to users. In-memory cache (in-process) stores hot data for single-node speed. Distributed cache (Redis, Memcached) provides shared, cross-node caching with persistence and eviction policies. Multi-layer caching reduces origin load stepwise.

---

**Question**: List and compare cache invalidation strategies.

**Answer**: TTL-based: entries expire after a fixed time (simple, stale data possible). Write-through: cache is updated on every write (strong consistency, high write latency). Write-behind: batch updates to cache (high write throughput, risk of loss). Cache-aside (lazy loading): application checks cache, populates on miss.

---

**Question**: How does a Content Delivery Network (CDN) improve performance?

**Answer**: CDNs replicate content across geographically distributed edge servers. Users fetch from the nearest edge, reducing latency and offloading origin servers. CDNs also handle DDoS protection and connection optimization (TLS termination, HTTP/2). Examples: CloudFront, Cloudflare, Akamai.

---

**Question**: How do read replicas improve query performance in CQRS-style reads?

**Answer**: Read replicas are asynchronous copies of the primary database that serve read-only traffic. They offload expensive queries (aggregations, reporting) from the write master. Combined with CQRS, the read model is denormalized and materialized for fast queries, while writes go to a separate optimized store.

---

**Question**: What is connection pooling and why is it essential?

**Answer**: Connection pooling reuses a set of pre-established connections to a database or service instead of opening/closing per request. This avoids TCP handshake and TLS negotiation overhead. Pools have limits (min/max size, idle timeout, connection lifetime) to prevent resource exhaustion.

---

**Question**: Describe auto-scaling strategies in cloud environments.

**Answer**: Reactive auto-scaling triggers scale-up/down based on metrics (CPU > 70%, request queue depth). Predictive auto-scaling uses historical patterns (e.g., scale before known traffic spikes). Scheduled scaling adjusts for predictable events (Black Friday). Target tracking maintains a metric at a desired level.

---

**Question**: How do you manage database connections under high concurrency?

**Answer**: Use a bounded connection pool with wait queue (reject when full). In servers like PgBouncer (transaction pooling) or ProxySQL, multiplex connections from multiple app instances. Connection pooling frameworks (HikariCP, PgBouncer) enforce timeouts and leak detection. Monitor for connection starvation in production.

---

## Design a... Case Studies

**Question**: Design a URL shortener (like TinyURL).

**Answer**: The system generates a unique short key via base62 encoding of a distributed counter (Snowflake) or pre-generated key pool. Keys map to original URLs in a fast KV store (Redis + DB). Redirection returns a 302/301 to the original URL. Add TTL support, analytics tracking per short code, and CDN caching.

---

**Question**: Design a chat system like WhatsApp.

**Answer**: Users connect via WebSocket to a chat server pool. Messages are persisted in a time-ordered log (Cassandra or ScyllaDB) per conversation. Presence status uses Redis pub/sub. For multi-device sync, messages have sequence numbers and are pulled from the log on reconnect. Use server-assigned monotonic sequence numbers per conversation for message ordering.

---

**Question**: Design a rate limiter (like for an API gateway).

**Answer**: Use a sliding window counter stored in Redis sorted sets per user/IP. Each request adds a timestamp; reject if count exceeds limit. For distributed consistency, use Lua scripts for atomic increment and cleanup. Expose headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`.

---

**Question**: Design a notification system (email, SMS, push).

**Answer**: A notification service accepts templated requests via a REST API, enqueues them in Kafka partitioned by user ID. Worker pools consume batches, join with user preferences (channels, opt-outs), and dispatch via provider adapters (SendGrid, Twilio, Firebase). Use dedup windows and retry with exponential backoff per provider.

---

**Question**: Design a news feed like Twitter.

**Answer**: Use fanout-on-write for regular users (pre-compute feeds into Redis lists) and fanout-on-read for high-profile users (query timeline from a time-ordered database since pre-computing feeds for millions of followers is prohibitive). A feed service merges followed users' posts, applies ranking (recency, engagement), and paginates. Push new posts via WebSocket.

---

**Question**: Design a distributed key-value store like DynamoDB.

**Answer**: Data is partitioned using consistent hashing with virtual nodes. Replication factor N with hinted handoff for availability during failures. Uses quorum (R + W > N) for reads/writes. Vector clocks handle conflicts; read repair resolves stale values during reads. A gossip protocol propagates membership changes.

---

**Question**: Design a file storage system like Amazon S3.

**Answer**: Blobs are split into chunks (~8 MB), each chunk replicated across availability zones (3x by default). A metadata store (Paxos/Raft-based) maps bucket+key to chunk IDs and locations. Chunks are stored on a distributed filesystem with erasure coding for durability. Uploads go through a load-balanced gateway layer.

---

**Question**: Design a real-time leaderboard for a game.

**Answer**: Scores are submitted to a service that updates a Redis sorted set (`ZINCRBY`) per game. The leaderboard query `ZREVRANGE` with pagination returns top players in O(log N). For persistence, snapshot to a DB periodically. Use sharding by game ID and a cache layer for hot leaderboards.

---

**Question**: Design a web crawler for a search engine.

**Answer**: The crawler starts with seed URLs from a frontier queue (Redis or Kafka). A URL frontier manager deduplicates URLs using a Bloom filter and assigns them to worker nodes. Workers fetch, parse (HTML/links), and extract content, emitting new URLs back to the frontier. Results are stored in a document store (Elasticsearch) for indexing.

---

**Question**: Design a video streaming service like Netflix.

**Answer**: Videos are transcoded into multiple bitrates (HLS/DASH) and stored in a CDN (multiple tiers). The client fetches a manifest with available renditions and adaptively switches based on bandwidth. A recommendation service uses collaborative filtering and content-based scoring. A content delivery service handles DRM, ABR logic, and geo-restrictions.
