# Module 08: Systems Components Deep Dive

> The essential components of modern software systems — what they do, how they actually work under the hood, when to use them, and what breaks.

This is a **growing reference**. Each component follows a consistent structure. Add new components and deepen existing ones as you learn.

---

## How to Use This Module

Each component has:
- **What it solves** — the problem it exists to address
- **Core concepts** — the mental model you need
- **How it actually works** — under-the-hood mechanics
- **Failure modes** — what breaks and why
- **When to use / not use** — decision framework
- **Key principles** — the timeless ideas underneath
- **Go deeper** — what to study next

**Suggested approach:** Read one component per week at Depth 1 (conceptual). Go to Depth 2 (operational) for things you actually work with. Let AI handle Depth 3 (implementation details) when you need them.

---

## Layer 1: Data Storage

### 1.1 Relational Databases (PostgreSQL / MySQL)

**What it solves:** Storing structured data with guaranteed consistency and complex query capability.

**Core concepts:**
- **Tables, rows, columns** — data organized in a schema
- **ACID transactions** — Atomicity (all or nothing), Consistency (rules always hold), Isolation (concurrent transactions don't interfere), Durability (committed data survives crashes)
- **Indexes** — data structures that speed up reads at the cost of slower writes
- **Normalization** — eliminating data duplication across tables
- **SQL** — declarative language: you say WHAT you want, the database decides HOW

**How it actually works:**

```
Client sends query
    ↓
Parser → converts SQL to syntax tree
    ↓
Planner/Optimizer → decides HOW to execute
    │   "Should I use the index or scan the whole table?"
    │   "Should I join A→B or B→A?"
    ↓
Executor → runs the plan
    ↓
Storage Engine → reads/writes actual data on disk
    │
    ├── Pages: data stored in fixed-size blocks (typically 8KB)
    ├── B-tree indexes: sorted tree structure for fast lookups
    ├── WAL (Write-Ahead Log): writes go to log FIRST, then to data files
    │   (this is how durability works — if crash happens, replay the log)
    └── Buffer Pool: frequently accessed pages cached in memory
```

**The WAL is the key insight:** Every write goes to a sequential log first. This is fast (sequential I/O). Then it's lazily applied to the actual data files. If the system crashes between these steps, the WAL can be replayed to recover. This is how databases guarantee durability without being slow.

**Indexes — how they actually work:**
```
B-tree index on "email" column:

                    [M]
                   /   \
              [D, H]   [R, W]
             / | \     / | \
           [A-C][E-G][I-L][N-Q][S-V][X-Z]
                              ↓
                         Points to actual row on disk

Lookup "user@example.com" → start at root → go right (U > M) →
go middle (U is between R and W) → find the leaf → follow pointer to disk

Time: O(log n) — finding 1 row in 1 billion takes ~30 steps, not 1 billion
```

**MVCC (Multi-Version Concurrency Control):**
PostgreSQL doesn't lock rows for reading. Instead, each transaction sees a "snapshot" of the database at the time it started. Multiple versions of each row exist simultaneously. This is why:
- Readers don't block writers
- Writers don't block readers
- But writers CAN block writers (if modifying the same row)

**Failure modes:**
- **Connection exhaustion** — too many clients, not enough connections. Fix: connection pooling (PgBouncer)
- **Slow queries** — missing indexes, bad query plans. Fix: EXPLAIN ANALYZE, add indexes
- **Lock contention** — multiple transactions fighting over the same rows. Fix: smaller transactions, better access patterns
- **Disk full** — WAL files grow, old data isn't vacuumed. Fix: monitoring, autovacuum tuning
- **Replication lag** — replica falls behind primary. Fix: monitor lag, don't read-after-write from replica

**When to use:**
- Default choice for most applications
- When you need transactions (financial data, inventory, anything where consistency matters)
- When you need complex queries (joins, aggregations, subqueries)
- When your data is naturally relational (users have orders, orders have items)

**When NOT to use:**
- Write-heavy workloads with millions of writes/second (consider time-series DB)
- Unstructured/highly variable data (consider document DB)
- Simple key-value lookups at extreme scale (consider Redis or DynamoDB)

**Key principles:**
1. **Schema is a feature, not a constraint.** It prevents bad data from entering the system.
2. **Indexes are a trade-off.** Every index speeds up reads but slows down writes and uses disk space. Don't index everything.
3. **The query planner is smarter than you think.** Write clear SQL. Let the planner optimize. Don't try to outsmart it unless you have evidence.
4. **Transactions should be as short as possible.** Long transactions hold locks and block other operations.
5. **Connection pooling is mandatory in production.** Opening a database connection is expensive (~100ms). Reuse them.

---

### 1.2 Document Databases (MongoDB / DynamoDB)

**What it solves:** Storing semi-structured data that doesn't fit neatly into rows and columns, with flexible schemas and horizontal scaling.

**Core concepts:**
- **Documents** — self-contained data objects (usually JSON-like). A user document contains EVERYTHING about that user, not spread across tables.
- **Collections** — groups of documents (analogous to tables, but no enforced schema)
- **Denormalization** — deliberately duplicating data so each document is self-contained. Opposite of relational normalization.
- **Eventual consistency** — reads might not see the latest write immediately (in distributed setups)

**How it actually works (DynamoDB as example):**

```
Data is partitioned across many machines by a "partition key"

Write: hash(partition_key) → determines which machine stores this item
Read:  hash(partition_key) → go directly to that machine

┌──────────┐  ┌──────────┐  ┌──────────┐
│Partition A│  │Partition B│  │Partition C│
│ user_001  │  │ user_002  │  │ user_003  │
│ user_004  │  │ user_005  │  │ user_006  │
└──────────┘  └──────────┘  └──────────┘

Each partition is replicated 3x for durability
```

**The key insight: access patterns must be known upfront.** In SQL, you can query any combination of fields. In DynamoDB, you can only efficiently query by partition key (and optionally sort key). Everything else requires a scan (slow) or a secondary index.

**When to use:**
- Known, simple access patterns (get user by ID, get orders by user)
- Need horizontal scaling beyond what a single SQL server handles
- Data is naturally document-shaped (user profiles, product catalogs, content)
- Schema varies between items (some products have color, some have size, some have both)

**When NOT to use:**
- Complex queries across different entity types (use SQL)
- Need strong consistency and transactions across multiple items
- Don't know your access patterns yet (SQL is more forgiving)
- Small-to-medium scale (SQL is simpler and sufficient)

**Key principles:**
1. **Model for access patterns, not for data relationships.** In SQL, you model the data. In document DBs, you model the queries.
2. **Denormalization is the point.** If you're normalizing a document database, you probably want SQL.
3. **Partition key choice is the most important decision.** Bad partition key = hot partitions = performance problems. Choose a key with high cardinality and even distribution.

---

### 1.3 Key-Value Stores (Redis)

**What it solves:** Extremely fast reads and writes for simple data structures, primarily in memory.

**Core concepts:**
- **In-memory storage** — data lives in RAM, not disk. This is why it's fast (~0.1ms vs ~1-10ms for disk).
- **Data structures** — not just key→string. Redis supports strings, lists, sets, sorted sets, hashes, streams, bitmaps, HyperLogLog.
- **TTL (Time-to-Live)** — keys can auto-expire. This is what makes it perfect for caching.
- **Single-threaded event loop** — one thread handles all commands sequentially. No locking needed because there's no concurrency within Redis itself.

**How it actually works:**

```
Client sends: SET user:123 '{"name":"Alice"}' EX 3600

Redis main thread:
  1. Parse command
  2. Store in hash table (in RAM): key → value
  3. Set expiry timer: 3600 seconds from now
  4. Respond OK

All in ~microseconds because it's pure memory operations.

Client sends: GET user:123
  1. Hash table lookup
  2. Check if expired → if yes, delete and return nil
  3. Return value

Persistence (optional):
  - RDB: periodic snapshots of entire dataset to disk
  - AOF: append every write command to a log file
  - Both are async — writes are fast, persistence is background
```

**Why single-threaded works:** Redis operations are so fast (microseconds) that a single thread can handle 100,000+ operations per second. The bottleneck is network, not CPU.

**Common use cases:**
- **Cache** — cache database query results, API responses, session data
- **Session store** — user sessions with auto-expiry
- **Rate limiting** — count requests per user per time window
- **Leaderboard** — sorted sets give O(log n) rank queries
- **Pub/Sub** — lightweight message broadcasting
- **Distributed lock** — coordinate between multiple servers

**Failure modes:**
- **Memory full** — Redis can't store more data. Fix: set maxmemory policy (evict LRU keys)
- **Data loss on crash** — if using RDB, you lose data since last snapshot. Fix: use AOF for durability (or accept the risk for cache use-cases)
- **Key namespace collisions** — multiple features using the same key pattern. Fix: prefix keys (cache:user:123, session:user:123)
- **Hot keys** — one key getting disproportionate traffic. Fix: read replicas, key sharding

**Key principles:**
1. **Redis is not a database replacement.** It's a complement. Cache the hot path; database is the source of truth.
2. **Every cached value must be recreatable.** If Redis dies, your app should still work (just slower).
3. **Set TTLs on everything.** Stale cache is worse than no cache. Default to expiring data.
4. **Memory is finite and expensive.** Be deliberate about what you cache. Don't cache everything "just in case."

---

### 1.4 Search Engines (Elasticsearch / OpenSearch)

**What it solves:** Full-text search, log aggregation, and analytics over large volumes of semi-structured data.

**Core concepts:**
- **Inverted index** — instead of "document → words," it stores "word → documents." This is why search is fast.
- **Analyzers** — text processing pipeline: tokenization, lowercasing, stemming ("running" → "run")
- **Shards** — data split across multiple nodes for parallelism
- **Replicas** — copies of shards for fault tolerance and read throughput

**How an inverted index works:**

```
Documents:
  doc1: "The quick brown fox"
  doc2: "The quick blue car"
  doc3: "A slow brown dog"

Inverted index:
  "quick"  → [doc1, doc2]
  "brown"  → [doc1, doc3]
  "fox"    → [doc1]
  "blue"   → [doc2]
  "car"    → [doc2]
  "slow"   → [doc3]
  "dog"    → [doc3]

Search "quick brown":
  "quick" → [doc1, doc2]
  "brown" → [doc1, doc3]
  Intersection → [doc1] (contains both words)
  Score: doc1 has both terms, rank highest
```

**When to use:** Full-text search, log analysis (ELK stack), e-commerce product search, autocomplete/suggestions.

**When NOT to use:** Primary data store (it's not ACID), simple exact-match lookups (use a database), small datasets (overhead not worth it).

**Key principles:**
1. **Elasticsearch is a search engine, not a database.** Always have a source of truth elsewhere.
2. **Mapping (schema) decisions are hard to change later.** Think about your fields upfront.
3. **Relevance scoring is an art.** Default scoring works for most cases. Tuning it is a deep rabbit hole.

---

### 1.5 Time-Series Databases (InfluxDB / TimescaleDB / Prometheus)

**What it solves:** Efficiently storing and querying data points that are indexed by time — metrics, IoT sensor data, financial ticks, monitoring data.

**Core concepts:**
- **Time as primary index** — data is naturally ordered by when it happened
- **Write-heavy workload** — millions of data points per second, rarely updated
- **Downsampling** — aggregating old data (minute-level → hour-level → day-level) to save space
- **Retention policies** — auto-delete data older than X days

**How it works differently from SQL:**

```
Regular database:
  INSERT into metrics VALUES (now(), 'cpu', 72.5)
  → Writes to B-tree index, updates WAL, maintains MVCC versions
  → Designed for random access patterns

Time-series database:
  INSERT cpu,host=server01 value=72.5 1616000000
  → Appends to compressed, time-ordered columnar storage
  → No index updates needed (time IS the index)
  → Can compress heavily (time values are sequential, metric values are similar)
  → 10-100x more efficient for this specific pattern
```

**When to use:** Monitoring/metrics, IoT data, financial data, any "append-mostly, query-by-time-range" workload.

**When NOT to use:** General application data, data that needs updates/deletes, relational data.

---

### 1.6 Graph Databases (Neo4j)

**What it solves:** Storing and querying highly connected data where relationships are as important as the entities themselves.

**Core concepts:**
- **Nodes** — entities (people, products, pages)
- **Edges** — relationships between nodes (FOLLOWS, PURCHASED, LINKS_TO)
- **Traversal** — following edges from node to node. "Friends of friends who like jazz" is a natural query.

**How it works differently:**

```
Relational (finding friends of friends):
  SELECT * FROM users u3
  JOIN friendships f2 ON u3.id = f2.friend_id
  JOIN friendships f1 ON f2.user_id = f1.friend_id
  WHERE f1.user_id = 123
  → Multiple JOIN operations, gets exponentially slower with depth

Graph:
  MATCH (me)-[:FRIENDS]->(friend)-[:FRIENDS]->(fof)
  WHERE me.id = 123
  RETURN fof
  → Follows pointers directly, constant time per hop regardless of total data size
```

**When to use:** Social networks, recommendation engines, fraud detection, knowledge graphs, network topology.

**When NOT to use:** Most applications. If your data is naturally tabular, use SQL. Graph DBs shine only when relationship traversal is the core use case.

**Key principles:**
1. **If you're doing lots of JOINs across 4+ tables, consider a graph.** That's the pain signal.
2. **Graph databases are terrible for aggregations.** "Average purchase price across all users" is better in SQL.

---

## Layer 2: Messaging & Streaming

### 2.1 Message Queues (RabbitMQ / SQS / NATS)

**What it solves:** Decoupling producers from consumers. Service A doesn't need to know (or wait for) Service B to process a task.

**Core concepts:**
- **Producer** — sends messages to a queue
- **Queue** — holds messages until consumed
- **Consumer** — reads and processes messages
- **Acknowledgment** — consumer tells queue "I processed this successfully." If not acknowledged, queue retries.
- **Dead letter queue** — where messages go after too many failed processing attempts

**How it actually works:**

```
Producer → Queue → Consumer

  Order Service                    Email Service
      │                                ↑
      │  send_email({                  │  receive & process
      │    to: "user@...",             │
      │    template: "receipt"         │
      │  })                            │
      └──────→ [Queue] ───────────────→┘
               Messages sit here
               until consumed

If Email Service is down:
  → Messages accumulate in the queue
  → When Email Service comes back, it processes the backlog
  → Order Service was never affected (it doesn't wait)
```

**Key patterns:**
- **Work queue** — multiple consumers share the load. Each message is processed by ONE consumer.
- **Pub/Sub** — message goes to ALL subscribers. Used for event broadcasting.
- **Request/Reply** — synchronous communication over async infrastructure. (Usually a sign you should use HTTP instead.)

**Failure modes:**
- **Queue backup** — consumers can't keep up with producers. Fix: scale consumers, or fix slow consumer.
- **Message loss** — queue crashes before persisting. Fix: use durable queues with persistence.
- **Poison message** — a message that always fails processing. Fix: dead letter queue + alerting.
- **Duplicate processing** — consumer processes a message, crashes before acknowledging. Queue redelivers. Fix: make consumers idempotent.

**Key principles:**
1. **Consumers MUST be idempotent.** Messages can be delivered more than once. Processing the same message twice should produce the same result.
2. **Queues are not databases.** Don't store data in queues long-term. Process and move on.
3. **If you need strict ordering, life gets harder.** Most queues don't guarantee order (or only guarantee it within a partition).

---

### 2.2 Event Streaming (Apache Kafka)

**What it solves:** Durable, ordered, replayable stream of events at massive scale. Unlike message queues, messages are NOT deleted after consumption — they're retained for a configurable period.

**Core concepts:**
- **Topics** — named channels of events (e.g., "user-events", "order-events")
- **Partitions** — a topic is split into partitions for parallelism. Each partition is an ordered, immutable log.
- **Producers** — write events to topics
- **Consumers** — read events from topics
- **Consumer groups** — a group of consumers that divide partitions among themselves. Each partition is read by exactly one consumer in the group.
- **Offsets** — each message in a partition has a sequential ID. Consumers track their position by offset.

**How it actually works:**

```
Topic: "orders" with 3 partitions

Partition 0: [msg0][msg1][msg2][msg3][msg4] → newest
Partition 1: [msg0][msg1][msg2][msg3]       → newest
Partition 2: [msg0][msg1][msg2][msg3][msg4][msg5] → newest

Producer writes order event:
  1. Key: user_id=456
  2. hash(456) mod 3 = partition 1
  3. Append to end of partition 1
  4. Gets offset 4 in that partition

Consumer Group "payment-service" (2 consumers):
  Consumer A reads: Partition 0, Partition 1
  Consumer B reads: Partition 2
  Each tracks its own offset per partition

Consumer Group "analytics-service" (1 consumer):
  Consumer A reads: ALL partitions
  Completely independent from payment-service group
  Can be behind or ahead — doesn't matter
```

**The log is the key insight:**

```
A Kafka partition is a commit log:

  [0] [1] [2] [3] [4] [5] [6] [7] [8]
                    ↑              ↑
              payment-service    newest write
              last read here
              (offset 4)

  - New messages append to the right (sequential I/O = fast)
  - Old messages are NOT deleted when read (unlike queues)
  - Messages are retained for a configured period (e.g., 7 days)
  - Any consumer can re-read old messages by resetting its offset
  - This is what makes Kafka "replayable"
```

**Why this matters:**
- **Replay:** New service joins? It can read the entire history from offset 0.
- **Multiple consumers:** Unlike a queue, reading a message doesn't remove it. N services can independently consume the same topic.
- **Ordering:** Messages within a partition are strictly ordered. Same key always goes to the same partition → ordering per key.

**Failure modes:**
- **Consumer lag** — consumers falling behind producers. Fix: scale consumers (max = number of partitions), fix slow processing.
- **Partition hot spots** — one partition gets more traffic than others. Fix: better key distribution.
- **Broker failure** — Kafka server goes down. Fix: replication factor ≥ 3 (data exists on multiple brokers).
- **Message too large** — default max is 1MB. Fix: store large payloads elsewhere, send reference in Kafka.

**When to use:**
- Event-driven architectures (multiple services react to same events)
- Event sourcing (the event log IS the source of truth)
- High-throughput data pipelines (logs, metrics, clickstream)
- When you need replayability (reprocessing historical data)

**When NOT to use:**
- Simple task queues (use SQS/RabbitMQ — much simpler)
- Low volume messaging (Kafka's operational overhead isn't worth it for small scale)
- Request/reply patterns (use HTTP)

**Key principles:**
1. **Kafka is infrastructure, not a library.** Running Kafka is non-trivial. Use managed services (Confluent, AWS MSK) unless you have a dedicated team.
2. **Partition count is hard to change later.** Think about your parallelism needs upfront.
3. **Key design determines ordering and data locality.** Same key = same partition = ordered. Choose keys carefully.
4. **Idempotent consumers are mandatory.** Kafka guarantees at-least-once delivery. Your consumer will see duplicates.

---

### 2.3 Stream Processing (Apache Flink / Kafka Streams)

**What it solves:** Processing continuous streams of data in real-time — aggregations, transformations, pattern detection, joins across streams.

**Core concepts:**
- **Stream** — unbounded sequence of events (data keeps coming forever)
- **Window** — bounded chunk of a stream for aggregation. "Count events per 5-minute window."
- **State** — data the processor remembers between events. "Running total of orders per user."
- **Checkpoint** — periodic snapshot of all state, so processing can recover from failure without losing progress.
- **Watermark** — mechanism for handling late-arriving data. "I believe I've received all events up to time T."
- **Exactly-once processing** — despite failures, each event is processed exactly once (Flink's key guarantee).

**How it actually works (Flink):**

```
Source (Kafka topic) → Operators → Sink (database, another topic, etc.)

Example: "Count orders per product per minute"

Kafka: [order1, order2, order3, order4, ...]
         ↓
   Map: extract (product_id, timestamp)
         ↓
   KeyBy: group by product_id
         ↓
   Window: tumbling 1-minute windows
         ↓
   Aggregate: count per window
         ↓
   Sink: write to dashboard database

State management:
  - Each operator can maintain state (e.g., running count per product)
  - State is stored in a state backend (memory, RocksDB)
  - Checkpoints periodically snapshot ALL state across ALL operators
  - If a node crashes, Flink restores state from last checkpoint
    and replays events from Kafka (using stored offsets)
```

**Window types:**
```
Tumbling (non-overlapping):
  |----W1----|----W2----|----W3----|
  0         60        120       180  (seconds)

Sliding (overlapping):
  |----W1----|
       |----W2----|
            |----W3----|
  Every 30 seconds, compute over last 60 seconds

Session (gap-based):
  |--W1--|   |----W2----|  |--W3--|
  Events     gap > threshold starts new window
```

**When to use:** Real-time analytics, fraud detection, monitoring/alerting, ETL pipelines, IoT event processing.

**When NOT to use:** Simple event routing (use Kafka consumers directly), batch processing of historical data (consider Spark), low-volume processing (overkill).

**Key principles:**
1. **Think in streams, not batches.** The data is continuous. Your processing should be too.
2. **State is the hard part.** Stateless stream processing is trivial. The complexity comes from managing state correctly across failures.
3. **Late data is inevitable.** Events arrive out of order. Your watermark and window strategy determines how you handle this.

---

## Layer 3: Compute & Orchestration

### 3.1 Containers (Docker)

**What it solves:** "It works on my machine" → "It works on any machine." Packages code + dependencies + runtime into a portable unit.

**Core concepts:**
- **Image** — a snapshot of a filesystem + metadata. Immutable. Like a class definition.
- **Container** — a running instance of an image. Like an object instance.
- **Layer** — images are built in layers. Each Dockerfile instruction creates a layer. Unchanged layers are cached and shared.
- **Registry** — where images are stored and distributed (Docker Hub, ECR, GCR)

**How it actually works (Linux):**

```
Docker doesn't use VMs. It uses Linux kernel features:

1. Namespaces — isolation
   - PID namespace: container sees only its own processes
   - Network namespace: container has its own IP, ports
   - Mount namespace: container has its own filesystem
   - User namespace: container has its own root user (not host root)

2. Cgroups — resource limits
   - Limit CPU: "this container can use 2 cores max"
   - Limit memory: "this container can use 512MB max"
   - Limit I/O: "this container can read 100MB/s max"

3. Union filesystem — layered storage
   - Base layer: OS files (ubuntu, alpine)
   - Layer 2: runtime (node, python)
   - Layer 3: dependencies (npm install)
   - Layer 4: your code (COPY . .)
   - Each layer is read-only except the top (container layer)
   - Shared base layers save disk space

Container is NOT a VM:
  VM: Full OS kernel + userspace → heavy (GB), slow to start (minutes)
  Container: Shared host kernel + isolated userspace → light (MB), fast to start (seconds)
```

**Dockerfile principles:**
```dockerfile
# Layer caching: put things that change LEAST at the top
FROM node:20-alpine          # Changes almost never
WORKDIR /app
COPY package*.json ./        # Changes when deps change
RUN npm ci                   # Cached if package.json unchanged
COPY . .                     # Changes every build (your code)
RUN npm run build

# Multi-stage: build in one image, run in another (smaller final image)
FROM node:20-alpine
COPY --from=0 /app/dist ./dist
COPY --from=0 /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

**Key principles:**
1. **One process per container.** Don't run your app + database + nginx in one container.
2. **Images should be immutable and reproducible.** Same Dockerfile + same context = same image. Always.
3. **Use specific base image tags.** `node:20.11-alpine`, not `node:latest`. Reproducibility matters.
4. **Don't store data inside containers.** They're ephemeral. Use volumes for persistent data.

---

### 3.2 Container Orchestration (Kubernetes)

**What it solves:** Running, scaling, and managing hundreds/thousands of containers across multiple machines.

**Core concepts:**
- **Pod** — smallest deployable unit. Usually one container, sometimes co-located sidecars.
- **Deployment** — declares "I want 3 replicas of this pod running." Kubernetes makes it so.
- **Service** — stable network endpoint that load-balances across pods. Pods come and go; the service IP stays constant.
- **Ingress** — routes external traffic to internal services (like a reverse proxy).
- **ConfigMap / Secret** — configuration and credentials, injected into pods at runtime.
- **Namespace** — logical isolation within a cluster (dev, staging, prod).

**How it actually works:**

```
You: kubectl apply -f deployment.yaml
  "I want 3 replicas of my-app:v2.1"

Control Plane:
  ┌─────────────────────────────────────────┐
  │ API Server: receives your request       │
  │ Scheduler: picks which nodes to use     │
  │ Controller Manager: maintains desired   │
  │   state ("3 replicas" means 3 running)  │
  │ etcd: stores all cluster state          │
  └─────────────────────────────────────────┘
        ↓ schedules pods onto nodes

Worker Nodes:
  ┌────────────┐  ┌────────────┐  ┌────────────┐
  │ Node 1     │  │ Node 2     │  │ Node 3     │
  │ kubelet    │  │ kubelet    │  │ kubelet    │
  │ [pod A]    │  │ [pod B]    │  │ [pod C]    │
  └────────────┘  └────────────┘  └────────────┘

If Node 2 dies:
  Controller Manager detects pod B is gone
  Scheduler picks Node 1 or 3 for a replacement
  New pod starts automatically
  Service routes traffic to remaining healthy pods during recovery
```

**The reconciliation loop (most important concept):**
```
Desired state: 3 replicas
Current state: 2 replicas (one crashed)
Action: start 1 more replica

This loop runs CONTINUOUSLY. Kubernetes doesn't execute commands —
it converges toward declared desired state. You say WHAT you want,
Kubernetes figures out HOW to get there and maintains it.
```

**When to use:** Multiple services, need auto-scaling, need self-healing, need rolling deployments, team/org scale.

**When NOT to use:** Single service, small team, simple deployment. A $5 VPS with Docker Compose is fine for most early-stage projects. Kubernetes is operational complexity you earn, not a starting point.

**Key principles:**
1. **Declare desired state, don't script actions.** "3 replicas" not "start a container."
2. **Everything is cattle, not pets.** Pods are disposable. Never SSH into a pod to fix something — fix the image/config and redeploy.
3. **Start with managed Kubernetes** (EKS, GKE, AKS). Running your own control plane is a full-time job.

---

### 3.3 Serverless (AWS Lambda / Cloudflare Workers)

**What it solves:** Run code without managing servers. Pay per execution, scale to zero, scale to millions.

**Core concepts:**
- **Function** — a unit of code that handles one event
- **Cold start** — first invocation takes longer (runtime initialization). Subsequent calls are warm.
- **Event source** — what triggers the function (HTTP request, queue message, file upload, schedule)
- **Execution model** — stateless. Each invocation is independent. No local filesystem persistence.

**How it actually works:**

```
Request arrives → API Gateway → Lambda

Cold start path:
  1. Provision a micro-VM (Firecracker) → ~100-500ms
  2. Load your code + dependencies
  3. Initialize runtime (Node.js, Python, etc.)
  4. Execute your handler function
  5. Return response
  6. Container stays warm for ~5-15 minutes

Warm path:
  1. Route to existing warm container → ~1-5ms
  2. Execute your handler function
  3. Return response
```

**When to use:** Event-driven workloads, intermittent traffic, webhooks, scheduled tasks, API endpoints with variable load.

**When NOT to use:** Long-running processes (15-minute timeout), latency-sensitive (cold starts), high-throughput steady-state (more expensive than containers at scale), stateful processing.

**Key principles:**
1. **Functions should be small and focused.** One function, one job.
2. **Cold starts are real.** Keep dependencies minimal. Use provisioned concurrency for latency-sensitive paths.
3. **Cost model flips at scale.** Serverless is cheaper at low/variable traffic. Containers are cheaper at high, steady traffic. Know your crossover point.

---

## Layer 4: Networking & Communication

### 4.1 Load Balancers (ALB / nginx / HAProxy)

**What it solves:** Distributing traffic across multiple servers so no single server is overwhelmed.

**Core concepts:**
- **Health checks** — load balancer pings each server periodically. Unhealthy servers get no traffic.
- **Algorithms:**
  - Round-robin: each server takes turns
  - Least connections: send to the server with fewest active connections
  - Weighted: some servers get more traffic than others
  - IP hash: same client IP always goes to same server (session affinity)

**How it actually works:**

```
Client → Load Balancer → Server Pool

   Client
     │
     ↓
 ┌──────────┐    ┌──────────┐
 │   Load   │───→│ Server 1 │ ✓ healthy
 │ Balancer │───→│ Server 2 │ ✓ healthy
 │          │──╳→│ Server 3 │ ✗ health check failed, no traffic
 └──────────┘    └──────────┘

Layer 4 (TCP): Routes based on IP/port. Fast, dumb.
Layer 7 (HTTP): Routes based on URL path, headers, cookies. Slower, smart.
  - /api/* → API servers
  - /static/* → CDN/file servers
  - Header "X-Canary: true" → canary deployment servers
```

**Key principles:**
1. **Stateless servers are easy to load balance.** If a server stores session data locally, you need sticky sessions (which limits flexibility). Store sessions in Redis instead.
2. **Health checks must be meaningful.** Not just "server is reachable" but "server can handle requests" (check DB connection, etc.).

---

### 4.2 API Patterns (REST / GraphQL / gRPC)

**What it solves:** Defining how services communicate with each other and with clients.

**REST:**
```
GET    /users/123      → Read user 123
POST   /users          → Create a user
PATCH  /users/123      → Update user 123
DELETE /users/123      → Delete user 123

Principles:
- Resources identified by URLs
- Standard HTTP methods for operations
- Stateless — each request contains all needed information
- Cacheable — GET responses can be cached by intermediaries

Strength: Simple, well-understood, great tooling, cacheable
Weakness: Over-fetching (GET /users returns 50 fields, you need 3)
          Under-fetching (need user + orders = 2 requests)
```

**GraphQL:**
```
POST /graphql
{
  user(id: 123) {
    name
    email
    orders(last: 5) {
      total
      status
    }
  }
}

Client asks for EXACTLY the data it needs. One request, no over/under-fetching.

Strength: Flexible queries, great for complex frontends, self-documenting schema
Weakness: Caching is hard (everything is POST), complexity, N+1 query problem on backend
```

**gRPC:**
```
Protocol Buffers define the contract:
  message User {
    int32 id = 1;
    string name = 2;
    string email = 3;
  }
  service UserService {
    rpc GetUser(GetUserRequest) returns (User);
  }

Binary protocol over HTTP/2. Much faster than JSON/REST.

Strength: Performance, strong typing, streaming support, code generation
Weakness: Not human-readable, harder to debug, poor browser support
```

**When to use each:**
| Pattern | Best For |
|---------|----------|
| REST | Public APIs, simple CRUD, browser clients, when caching matters |
| GraphQL | Complex frontends needing flexible data fetching, mobile apps |
| gRPC | Service-to-service communication, high-performance, streaming |

---

### 4.3 CDN (Content Delivery Network)

**What it solves:** Serving static content (images, JS, CSS, videos) from servers geographically close to the user.

**How it works:**
```
Without CDN:
  User in Tokyo → Server in Virginia → 200ms latency

With CDN:
  User in Tokyo → CDN edge in Tokyo → 20ms latency
                  (serves cached copy)

Cache miss:
  User in Tokyo → CDN edge in Tokyo → Origin server in Virginia
                  (caches the response for next user)
```

**Key principles:**
1. **Cache-Control headers drive CDN behavior.** `max-age=31536000` for immutable assets (hashed filenames). `no-cache` for HTML.
2. **Cache invalidation is the hard part.** When you deploy new code, the CDN might still serve old files. Use content-hashing in filenames (`app.a3f8b2.js`).

---

### 4.4 DNS (Domain Name System)

**What it solves:** Translating human-readable domain names to IP addresses.

**How it works:**

```
Browser: "What's the IP for api.myapp.com?"

1. Browser cache → not found
2. OS cache → not found
3. Resolver (your ISP or 8.8.8.8) → not found
4. Root nameserver → "ask .com nameserver"
5. .com nameserver → "ask myapp.com nameserver"
6. myapp.com nameserver → "api.myapp.com = 52.1.2.3, TTL 300s"
7. Response cached at every level for 300 seconds

Record types:
  A:     domain → IPv4 address
  AAAA:  domain → IPv6 address
  CNAME: domain → another domain (alias)
  MX:    domain → mail server
  TXT:   domain → arbitrary text (used for verification, SPF, etc.)
```

**Why you care as a developer:**
- **TTL (Time-to-Live)** determines how long DNS is cached. Low TTL = faster propagation of changes, but more DNS queries. High TTL = faster lookups, but slow to change.
- **DNS is often the cause of "it's not working" after deployment.** Changed the IP? DNS might be cached for hours.
- **DNS-based load balancing** — return different IPs for the same domain to distribute traffic.

---

## Layer 5: Observability

### 5.1 The Three Pillars

**Logs** — discrete events with text descriptions
```
2026-03-25T10:15:23Z INFO  [user-service] User 123 logged in from 1.2.3.4
2026-03-25T10:15:24Z ERROR [order-service] Failed to process order 456: database timeout
```
- **What happened?** Specific events.
- **Tools:** ELK Stack (Elasticsearch + Logstash + Kibana), Loki, CloudWatch Logs

**Metrics** — numerical measurements over time
```
http_requests_total{method="GET", path="/users", status="200"} 15234
http_request_duration_seconds{quantile="0.99"} 0.543
system_memory_usage_bytes 4294967296
```
- **How is the system performing?** Trends, aggregates, alerting thresholds.
- **Tools:** Prometheus + Grafana, Datadog, CloudWatch Metrics

**Traces** — the journey of a single request across services
```
Trace ID: abc-123
  ├── [user-service] GET /users/123          12ms
  │   ├── [cache] Redis GET user:123         0.5ms (cache miss)
  │   └── [database] SELECT * FROM users     8ms
  └── [order-service] GET /orders?user=123   45ms
      └── [database] SELECT * FROM orders    40ms  ← slow!
```
- **Why is this specific request slow?** End-to-end visibility.
- **Tools:** Jaeger, Zipkin, OpenTelemetry, Datadog APM

**How they work together:**
```
Alert fires: "p99 latency > 500ms" (metric)
  → Look at dashboard: spike started at 10:15 (metric)
  → Find slow traces from that time period (trace)
  → Trace shows database query taking 400ms (trace)
  → Check database logs: "connection pool exhausted" (log)
  → Root cause found in 5 minutes instead of 50
```

**Key principles:**
1. **Metrics for alerting, traces for debugging, logs for details.** Each serves a different purpose.
2. **Structured logging is mandatory.** JSON logs with consistent fields (timestamp, service, request_id, level) are searchable. Unstructured text logs are not.
3. **Instrument first, optimize later.** You can't improve what you can't measure.
4. **Correlate across pillars.** Use a request_id / trace_id that appears in logs, metrics tags, and traces. This connects all three for any single request.

---

## Layer 6: Security

### 6.1 Authentication vs. Authorization

```
Authentication (AuthN): "WHO are you?"
  → Proving identity (username + password, OAuth token, API key)

Authorization (AuthZ): "WHAT are you allowed to do?"
  → Checking permissions (can this user delete this resource?)

Common mistake: checking authentication but not authorization.
"User is logged in" ≠ "User can access this specific resource"
```

### 6.2 Common Auth Patterns

**Session-based:**
```
Login → server creates session → stores in DB/Redis → sends session_id cookie
Each request → cookie sent automatically → server looks up session → knows who you are

Pros: Simple, server controls sessions (easy to invalidate)
Cons: Server-side storage, doesn't scale as easily, not great for APIs
```

**JWT (JSON Web Token):**
```
Login → server creates signed token → sends to client
Token contains: { user_id: 123, role: "admin", exp: 1616000000 }
Each request → client sends token in header → server VERIFIES SIGNATURE → trusts contents

Pros: Stateless (no server-side storage), works across services
Cons: Can't easily invalidate (until expiry), token size grows with claims
```

**OAuth 2.0 / OIDC:**
```
"Login with Google":
  1. Your app redirects user to Google
  2. User authenticates with Google
  3. Google redirects back with an authorization code
  4. Your server exchanges code for tokens (with Google directly)
  5. Your server gets user info from Google's API
  6. Your server creates its own session/JWT

You never see the user's Google password. Google handles authentication.
Your app handles authorization based on who Google says the user is.
```

### 6.3 OWASP Top 10 (Condensed)

| Vulnerability | What It Is | Prevention |
|--------------|------------|------------|
| **Injection** (SQL, NoSQL, OS) | Untrusted input executed as code | Parameterized queries, ORMs, input validation |
| **Broken Auth** | Weak passwords, missing MFA, session issues | Rate limiting, MFA, secure session handling |
| **Sensitive Data Exposure** | Unencrypted data at rest/in transit | HTTPS everywhere, encrypt PII, don't log secrets |
| **XXE** | Malicious XML input | Disable external entity processing |
| **Broken Access Control** | User accesses resources they shouldn't | Check authorization on EVERY request, deny by default |
| **Security Misconfiguration** | Default passwords, verbose errors, open ports | Hardened defaults, automated security scanning |
| **XSS** | Malicious script injected into web page | Output encoding, CSP headers, sanitize HTML |
| **Insecure Deserialization** | Malicious serialized objects | Don't deserialize untrusted data, use JSON |
| **Vulnerable Dependencies** | Known CVEs in libraries | `npm audit`, Dependabot, Snyk |
| **Insufficient Logging** | Attacks go undetected | Log auth failures, access control failures, input validation failures |

**Key principles:**
1. **Never trust user input.** Validate, sanitize, and parameterize everything from outside your system.
2. **Least privilege.** Every component should have the minimum permissions it needs. Database users, API keys, IAM roles — all scoped tightly.
3. **Defense in depth.** Don't rely on a single security measure. Firewall + auth + authorization + input validation + encryption + logging.
4. **Secrets never go in code.** Use environment variables, secret managers (Vault, AWS Secrets Manager), never commit secrets to git.

---

## Layer 7: CI/CD & Deployment

### 7.1 CI/CD Pipeline

**Continuous Integration (CI):** Automatically build and test every code change.
**Continuous Deployment (CD):** Automatically deploy passing changes to production.

```
Developer pushes code
  ↓
CI Pipeline:
  1. Checkout code
  2. Install dependencies
  3. Lint / type check
  4. Run unit tests
  5. Run integration tests
  6. Build artifact (Docker image, binary, etc.)
  7. Security scan
  ↓ (all pass?)
CD Pipeline:
  8. Deploy to staging
  9. Run smoke tests against staging
  10. Deploy to production (canary → full rollout)
  11. Monitor for errors
  ↓ (errors spike?)
  12. Auto-rollback
```

### 7.2 Deployment Strategies

```
Blue-Green:
  ┌──────┐     ┌──────┐
  │ Blue │ ←── │ LB   │  (current: Blue serves traffic)
  │ v1   │     │      │
  └──────┘     └──────┘
  ┌──────┐
  │Green │  (new: Green deployed, tested, ready)
  │ v2   │
  └──────┘

  Cutover: switch LB to Green. Instant. Rollback: switch back to Blue.

Canary:
  Traffic: 95% → v1, 5% → v2
  Monitor v2 metrics for errors/latency
  Gradually: 10%, 25%, 50%, 100%
  If problems → route 100% back to v1

Rolling:
  Server 1: v1 → v2 (update, verify)
  Server 2: v1 → v2 (update, verify)
  Server 3: v1 → v2 (update, verify)
  At any point, some servers run v1, some v2
```

**Key principles:**
1. **Every deploy must be reversible.** If you can't roll back in under 5 minutes, your deployment process is broken.
2. **Database migrations must be backward compatible.** Deploy new code that works with old AND new schema. Then migrate schema. Then remove old code paths. Never break the running version.
3. **Feature flags separate deployment from release.** Deploy code anytime. Enable features for users when ready. This decouples the "is the code live?" question from "can users see this?"

---

## Layer 8: Reliability & Scaling Patterns

### 8.1 Caching Strategies

```
Cache-Aside (most common):
  1. Read: check cache → if miss, read DB, write to cache
  2. Write: write to DB, invalidate cache
  Problem: cache can be stale between write and invalidation

Write-Through:
  1. Write goes to cache AND database simultaneously
  2. Read always hits cache
  Problem: every write is slower (two writes)

Write-Behind:
  1. Write goes to cache only
  2. Cache asynchronously writes to database later
  Problem: data loss if cache crashes before writing to DB
```

### 8.2 Rate Limiting

```
Algorithms:
  Token Bucket: bucket fills with tokens at fixed rate. Each request costs 1 token. Empty bucket = rejected.
  Sliding Window: count requests in last N seconds. If > limit, reject.
  Fixed Window: count requests in current minute. Resets each minute. (Bursty at window edges.)

Implementation:
  Redis INCR + EXPIRE for distributed rate limiting:
    key: "ratelimit:user123:2026-03-25T10:15"
    INCR → returns current count
    if count > limit → reject
    EXPIRE key 60 → auto-cleanup
```

### 8.3 Circuit Breaker

```
When a downstream service is failing, stop calling it temporarily.

States:
  Closed (normal):    requests flow through
  Open (tripped):     requests immediately fail (don't call downstream)
  Half-Open (testing): allow a few requests through to test recovery

  Closed ──[failures > threshold]──→ Open
  Open ──[timeout expires]──→ Half-Open
  Half-Open ──[test request succeeds]──→ Closed
  Half-Open ──[test request fails]──→ Open

Why: prevents cascade failures.
If Service A calls Service B which is down:
  Without circuit breaker: Service A's threads all block waiting for B → A also goes down
  With circuit breaker: Service A fails fast → stays healthy → users get partial functionality
```

### 8.4 Horizontal vs. Vertical Scaling

```
Vertical: bigger machine (more CPU, RAM)
  Pro: Simple. No code changes.
  Con: Has a ceiling. Single point of failure. Expensive.

Horizontal: more machines
  Pro: No ceiling. Fault tolerant.
  Con: Requires stateless design. Adds complexity (load balancing, data consistency).

Rule of thumb: Scale vertically until it hurts, then scale horizontally.
Don't distribute prematurely — it adds enormous complexity.
```

### 8.5 The CAP Theorem

```
In a distributed system, you can have at most 2 of 3:
  C - Consistency: every read sees the latest write
  A - Availability: every request gets a response
  P - Partition tolerance: system works despite network failures

In practice, P is not optional (networks fail), so you choose:
  CP: Consistent but may be unavailable during partition (e.g., traditional databases)
  AP: Available but may return stale data during partition (e.g., DynamoDB, Cassandra)

Real-world nuance: it's not binary. Most systems tune the trade-off per operation.
  - "User balance" → CP (must be accurate)
  - "User profile photo" → AP (stale for a few seconds is fine)
```

---

## Growth Tracker

### Components Covered

| Layer | Component | Depth 1 (Concept) | Depth 2 (Operational) | Notes |
|-------|-----------|:------------------:|:---------------------:|-------|
| Data | Relational DB | [ ] | [ ] | |
| Data | Document DB | [ ] | [ ] | |
| Data | Key-Value (Redis) | [ ] | [ ] | |
| Data | Search Engine | [ ] | [ ] | |
| Data | Time-Series DB | [ ] | [ ] | |
| Data | Graph DB | [ ] | [ ] | |
| Messaging | Message Queues | [ ] | [ ] | |
| Messaging | Kafka | [ ] | [ ] | |
| Messaging | Stream Processing | [ ] | [ ] | |
| Compute | Docker | [ ] | [ ] | |
| Compute | Kubernetes | [ ] | [ ] | |
| Compute | Serverless | [ ] | [ ] | |
| Network | Load Balancers | [ ] | [ ] | |
| Network | API Patterns | [ ] | [ ] | |
| Network | CDN | [ ] | [ ] | |
| Network | DNS | [ ] | [ ] | |
| Observe | Logs/Metrics/Traces | [ ] | [ ] | |
| Security | Auth patterns | [ ] | [ ] | |
| Security | OWASP Top 10 | [ ] | [ ] | |
| CI/CD | Pipelines | [ ] | [ ] | |
| CI/CD | Deploy strategies | [ ] | [ ] | |
| Reliability | Caching | [ ] | [ ] | |
| Reliability | Rate Limiting | [ ] | [ ] | |
| Reliability | Circuit Breaker | [ ] | [ ] | |
| Reliability | Scaling | [ ] | [ ] | |
| Reliability | CAP Theorem | [ ] | [ ] | |

### Components to Add (Future Weeks)

- [ ] Object storage (S3)
- [ ] Data warehouses (BigQuery, Snowflake, Redshift)
- [ ] Service mesh (Istio, Linkerd)
- [ ] API Gateway patterns
- [ ] WebSockets / Server-Sent Events / real-time patterns
- [ ] Consensus algorithms (Raft, Paxos)
- [ ] Distributed transactions (Saga pattern, 2PC)
- [ ] Bloom filters, consistent hashing, CRDTs
- [ ] Infrastructure as Code (Terraform, Pulumi)
- [ ] Secret management (Vault, AWS Secrets Manager)
- [ ] Identity providers (Okta, Auth0, Keycloak)
- [ ] Data pipelines (Airflow, dbt)
- [ ] Feature flags (LaunchDarkly, Unleash)
- [ ] Chaos engineering (principles and tools)

---

*This module grows weekly. Check off components as you study them. Add depth notes. The goal is not to memorize — it's to build a mental model of how modern systems are composed.*

→ **Back to [README](../README.md) | Previous: [Module 07](07-engineering-edge-training.md)**
