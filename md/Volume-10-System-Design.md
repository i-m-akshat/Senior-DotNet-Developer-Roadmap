# THE COMPLETE .NET DEVELOPER ENCYCLOPEDIA + INTERVIEW MASTERY GUIDE
# VOLUME 10: SYSTEM DESIGN

> **Target Audience:** .NET Developers (~3 years experience) transitioning to Senior / Staff Distributed Systems Engineers.
> **Scope:** High-concurrency architectures, enterprise reliability, distributed consensus, failure modes, data modeling, and end-to-end system designs for tier-1 tech and enterprise product interviews (Infosys lateral technical rounds, product engineering firms, and global enterprise architects).

---

## TABLE OF CONTENTS

1. [System Design Interview Framework (The 5-Step Blueprint)](#1-system-design-interview-framework)
   - 1.1 Why Candidates Fail System Design Rounds
   - 1.2 Step 1: Requirements Clarification & Scope Bounding
   - 1.3 Step 2: Back-of-the-Envelope Capacity Estimations
   - 1.4 Step 3: High-Level Architecture & API Contracts
   - 1.5 Step 4: Component Deep Dive & Concurrency Bottlenecks
   - 1.6 Step 5: Failure Modes, SPOF & Production Resiliency
   - 1.7 Latency Numbers Every Distributed Systems Engineer Must Know
   - 1.8 Powers of Two & Storage Unit Cheat Sheet
2. [Scalability Fundamentals](#2-scalability-fundamentals)
   - 2.1 Vertical Scaling vs Horizontal Scaling (Limits & Economics)
   - 2.2 Stateless Services & Externalized State
   - 2.3 Read Replicas, Replication Lag & Read-Your-Own-Writes Consistency
   - 2.4 Database Sharding & Partitioning Strategies
   - 2.5 Consistent Hashing with Virtual Nodes (C# Implementation)
3. [Caching Strategies & Redis Internals](#3-caching-strategies-and-redis-internals)
   - 3.1 Caching Topologies (Cache-Aside, Read-Through, Write-Through, Write-Behind, Refresh-Ahead)
   - 3.2 Cache Invalidation, Stampede (Thundering Herd) & Solutions
   - 3.3 Redis Deep Dive: Internal Threading Model, Memory Allocations & Eviction Algorithms
   - 3.4 Production Redis Data Types for System Designers
   - 3.5 Distributed Locking with Redis (Redlock & Atomic Lua Scripts in C#)
   - 3.6 HybridCache in .NET 9 (L1 In-Memory + L2 Distributed)
4. [Load Balancing & Reverse Proxies](#4-load-balancing-and-reverse-proxies)
   - 4.1 Layer 4 vs Layer 7 Load Balancing
   - 4.2 Load Balancing Algorithms & Session Persistence
   - 4.3 Reverse Proxies: Nginx, Envoy vs Microsoft YARP (.NET)
   - 4.4 Health Probing (Liveness vs Readiness vs Startup Probes)
   - 4.5 Production YARP Gateway Configuration
5. [CAP & PACELC Theorems in Practice](#5-cap-and-pacelc-theorems)
   - 5.1 Deconstructing the CAP Theorem: Why "Pick 2" is a Myth
   - 5.2 Real-World CP vs AP Systems Analysis
   - 5.3 PACELC Theorem: Accounting for Latency in Normal Operations
   - 5.4 Practical System Architect Decision Tree
6. [Database Selection Matrix](#6-database-selection-matrix)
   - 6.1 Relational Databases (RDBMS: SQL Server, PostgreSQL)
   - 6.2 Document Stores (MongoDB, Azure Cosmos DB)
   - 6.3 Key-Value Stores (Redis, AWS DynamoDB, Aerospike)
   - 6.4 Time-Series Databases (TimescaleDB, InfluxDB)
   - 6.5 Wide-Column Stores (Apache Cassandra, ScyllaDB)
   - 6.6 Unified Database Decision Matrix
7. [Full Design: Scalable URL Shortener (bit.ly Clone)](#7-full-design-url-shortener)
8. [Full Design: Distributed Multi-Channel Notification System](#8-full-design-notification-system)
9. [Full Design: High-Throughput Idempotent Payment System](#9-full-design-payment-system)
10. [Full Design: E-Commerce Order Management System (OMS)](#10-full-design-order-management-system)
11. [Full Design: Real-Time Chat & Collaboration System](#11-full-design-real-time-chat-system)
12. [Full Design: Distributed Object & File Storage (S3 / Drive Clone)](#12-full-design-distributed-file-storage)
13. [Senior Engineer Interview Evaluation Rubric](#13-senior-engineer-interview-rubric)

---

## 1. SYSTEM DESIGN INTERVIEW FRAMEWORK {#1-system-design-interview-framework}

### 1.1 Why Candidates Fail System Design Rounds
In senior developer interviews (e.g., Infosys Specialist Programmer, Senior Systems Engineer, Technical Architect), system design rounds carry the highest rejection rate. Over 70% of candidate failures stem from structural mistakes rather than lack of technical knowledge:

1. **Premature Solutioning:** Jumping directly into sketching databases or drawing microservice boxes before asking a single clarifying question about throughput, traffic spikes, read/write ratios, or consistency guarantees.
2. **The Buzzword Trap:** Blindly placing Kafka, Redis, Elasticsearch, and Kubernetes on the diagram without justifying *why* simpler constructs (e.g., an indexed relational database with an asynchronous worker) fail the requirements.
3. **Monologue Trap:** Treating the interview as a lecture rather than a collaborative technical design session with an engineering lead.
4. **Ignoring Failure Modes:** Designing the "happy path" exclusively, assuming networks never partition, disks never fill up, third-party APIs never timeout, and distributed locks never expire prematurely.
5. **Lack of Back-of-the-Envelope Math:** Inability to calculate storage requirements, network bandwidth, or queries per second (QPS), resulting in wildly over-provisioned or architecturally flawed systems.

---

### 1.2 Step 1: Requirements Clarification & Scope Bounding (Duration: 5–7 mins)
Never assume anything. System design interview prompts are intentionally ambiguous (e.g., *"Design bit.ly"* or *"Design WhatsApp"*). Your immediate responsibility is to establish boundaries.

Divide requirements into **Functional** and **Non-Functional**:

```
                              REQUIREMENTS TAXONOMY
                                        │
           ┌────────────────────────────┴────────────────────────────┐
           ▼                                                         ▼
   FUNCTIONAL REQUIREMENTS                               NON-FUNCTIONAL REQUIREMENTS
 (What the system MUST do)                                (How the system MUST perform)
   • Core feature set (MVP)                                • Latency (P50, P99, P99.9)
   • Data inputs & outputs                                 • Availability (99.9% vs 99.999%)
   • User personas & roles                                 • Consistency (Strong vs Eventual)
   • Out-of-scope boundaries                               • Durability & Data Loss Tolerance
                                                           • Scalability (Peak vs Average load)
                                                           • Security & Regulatory Compliance
```

#### Senior Engineer Questioning Framework
- **Traffic Volume:** *"What is the active user base? Daily Active Users (DAU) or Monthly Active Users (MAU)?"*
- **Read-to-Write Ratio:** *"Is this system read-heavy (like Twitter/bit.ly at 100:1) or write-heavy (like IoT ingestion or metrics at 1:10)?"*
- **Latency Budget:** *"What are the strict latency targets? For example, sub-50ms P99 for redirects, or sub-500ms for bulk notification delivery?"*
- **Consistency vs Availability:** *"In case of network partition, do we prioritize absolute consistency (financial ledgers) or maximum availability with eventual consistency (social media feeds)?"*
- **Data Retention & Lifecycle:** *"How long do we store this data? Indefinitely, 5 years for legal compliance, or 30 days with automated TTL eviction?"*

---

### 1.3 Step 2: Back-of-the-Envelope Capacity Estimations (Duration: 5–8 mins)
Numbers dictate architecture. A system handling 100 requests per second can run on a single ASP.NET Core process backed by Azure SQL. A system handling 500,000 requests per second requires consistent hashing, distributed caches, sharded write buffers, and asynchronous message brokers.

#### The Core Estimation Flow:
1. **Traffic Estimation (QPS & Throughput):**
   $$\text{Daily Requests} = \text{DAU} \times \text{Requests per User per Day}$$
   $$\text{Average QPS} = \frac{\text{Daily Requests}}{86,400 \text{ seconds}}$$
   $$\text{Peak QPS} = \text{Average QPS} \times \text{Peak Multiplier (Typically } 2\times \text{ to } 5\times\text{)}$$

2. **Storage Estimation:**
   $$\text{Daily Storage} = \text{Daily Writes} \times \text{Payload Size per Record}$$
   $$\text{5-Year Storage} = \text{Daily Storage} \times 365 \times 5 \times 1.4 \text{ (Capacity headroom + index overhead)}$$

3. **Bandwidth Estimation:**
   $$\text{Ingress Bandwidth} = \text{Write QPS} \times \text{Average Request Size}$$
   $$\text{Egress Bandwidth} = \text{Read QPS} \times \text{Average Response Size}$$

4. **Memory / Cache Estimation (80/20 Pareto Principle):**
   - 20% of the keys generate 80% of read traffic.
   - Cache memory needed:
     $$\text{RAM Required} = \text{Daily Read Volume} \times 0.20 \times \text{Payload Size}$$

---

### 1.4 Step 3: High-Level Architecture & API Contracts (Duration: 10–12 mins)
Establish the high-level structural blueprint before diving into individual components:
- **API Contracts:** Define RESTful endpoints or gRPC service definitions with precise HTTP verbs, headers, status codes, and JSON schemas.
- **Data Flow:** Trace the path of both Read and Write requests from client to storage.
- **Core Building Blocks:** Place Clients, DNS, CDN, Reverse Proxy / API Gateway, Stateless Application Nodes, Distributed Cache, Message Queues, and Primary/Replica Databases.

```
+--------+      +-----+      +-------------------+      +----------------------+
| Client | ---> | CDN | ---> | Layer 7 Gateway   | ---> | ASP.NET Core API     |
+--------+      +-----+      | (YARP / Envoy)    |      | (Stateless Cluster)  |
                             +-------------------+      +----------+-----------+
                                                                   |
                                         +-------------------------+-------------------------+
                                         |                                                   |
                                         v                                                   v
                              +--------------------+                               +--------------------+
                              | Distributed Cache  |                               | Primary Database   |
                              | (Redis Cluster)    |                               | (Write Master)     |
                              +--------------------+                               +---------+----------+
                                                                                             | (CDC / Repl)
                                                                                             v
                                                                                   +--------------------+
                                                                                   | Read Replicas /    |
                                                                                   | Search Engine      |
                                                                                   +--------------------+
```

---

### 1.5 Step 4: Component Deep Dive & Concurrency Bottlenecks (Duration: 12–15 mins)
This is where senior engineers distinguish themselves from juniors. Pick the 2–3 hardest architectural bottlenecks and demonstrate deep engineering mastery:
- How to prevent race conditions during concurrent stock reservations (Pessimistic vs Optimistic vs Redis Lua).
- How to generate globally unique IDs without database bottlenecks (Snowflake vs UUIDv7 vs KGS).
- How to partition data so that no single "hot partition" degrades the cluster (Virtual nodes, composite partition keys).
- How to handle cross-service transactions without two-phase locking (2PC) bottlenecks (Saga orchestration with transactional outbox).

---

### 1.6 Step 5: Failure Modes, SPOF & Production Resiliency (Duration: 5–8 mins)
System design does not end when the system works; it ends when you prove it survives when components crash:
- **Single Point of Failure (SPOF):** Is any component singular? (e.g., active-passive gateway pairs, multi-AZ database clustering).
- **Cascading Failures:** Use Circuit Breakers (Polly in .NET) to fail fast and stop queue backpressures from poisoning upstream callers.
- **Replication Lag:** Handling stale reads when replica nodes lag behind master nodes.
- **Data Loss Prevention:** Write-Ahead Logging (WAL), distributed quorum writes ($W + R > N$).
- **Disaster Recovery (RTO & RPO):**
  - **Recovery Time Objective (RTO):** Maximum acceptable duration of downtime.
  - **Recovery Point Objective (RPO):** Maximum acceptable data loss period (e.g., zero loss for payments, 5 mins for chat history).

---

### 1.7 Latency Numbers Every Distributed Systems Engineer Must Know
These numbers, originally compiled by Jeff Dean, govern every trade-off in distributed system design:

| Operation | Latency (Human Time Scale) | Engineering Implication |
|---|---|---|
| L1 cache reference | 0.5 ns (0.5 s) | Optimize memory layouts, cache line alignment. |
| Branch mispredict | 5 ns (5 s) | Keep code paths branchless where hyper-critical. |
| L2 cache reference | 7 ns (7 s) | 14x slower than L1. |
| Mutex lock/unlock | 25 ns (25 s) | Atomic operations (`Interlocked`) beat kernel locks. |
| Main memory reference (RAM) | 100 ns (1.5 min) | In-memory lookup is blazing fast. |
| Compress 1KB with Zstandard | 2,000 ns (33 min) | Compressing payloads saves massive network I/O. |
| Read 1 MB sequentially from memory | 3,000 ns (50 min) | Memory streaming bandwidth is massive (~30 GB/s). |
| Round trip within same datacenter | 500,000 ns = 0.5 ms (5.7 days) | Intra-datacenter RPCs are cheap, but not zero-cost. |
| Read 1 MB sequentially from NVMe SSD | 1,000,000 ns = 1 ms (11.5 days) | Sequential disk I/O rivals network operations. |
| Read 1 MB sequentially from HDD | 20,000,000 ns = 20 ms (7.6 months) | Spinning disks are completely dead for real-time random I/O. |
| Send packet US East to US West | 30,000,000 ns = 30 ms (11.4 months) | Cross-region traffic dictates strict multi-region caching. |
| Round-trip trans-Atlantic (US to Europe) | 150,000,000 ns = 150 ms (5 years) | Speed of light in fiber cannot be optimized; cache at edge! |

---

### 1.8 Powers of Two & Storage Unit Cheat Sheet
Mental math speed during system design interviews builds instant credibility:

| Power ($2^n$) | Approximate Value | Standard Unit | Daily Equivalent Context |
|---|---|---|---|
| $2^{10}$ | 1 Thousand (1,024) | 1 KB | 1 typical user profile JSON object |
| $2^{20}$ | 1 Million (1,048,576) | 1 MB | 1 minute of standard MP3 audio |
| $2^{30}$ | 1 Billion (1,073,741,824) | 1 GB | $10^6$ short URL mapping records |
| $2^{40}$ | 1 Trillion ($10^{12}$) | 1 TB | Daily log volume of mid-scale system |
| $2^{50}$ | 1 Quadrillion ($10^{15}$) | 1 PB | Monthly video ingestion at TikTok/YouTube |

#### Time Constants:
- 1 Day = $24 \times 60 \times 60 = 86,400 \text{ seconds} \approx 10^5 \text{ seconds}$ (Rule of thumb: Divide daily requests by $10^5$ for quick rough average QPS).
- 1 Million requests per day $\approx 12 \text{ requests/second}$.
- 1 Billion requests per month $\approx 400 \text{ requests/second}$.

---

## 2. SCALABILITY FUNDAMENTALS {#2-scalability-fundamentals}

### 2.1 Vertical Scaling vs Horizontal Scaling

```
VERTICAL SCALING (Scale Up)                     HORIZONTAL SCALING (Scale Out)
     +-----------------+                             +------+ +------+ +------+
     | 128 Core CPU    |                             | Node | | Node | | Node |
     | 2 TB RAM Server |                             | 4 vCPU | 4 vCPU | 4 vCPU |
     | $20,000/month   |                             | 16GB | | 16GB | | 16GB |
     +-----------------+                             +------+ +------+ +------+
  Single Point of Failure                         Fault-tolerant, elastic pool
```

| Dimension | Vertical Scaling (Scale Up) | Horizontal Scaling (Scale Out) |
|---|---|---|
| **Mechanism** | Add more CPU, RAM, NVMe to a single node. | Add more identical commodity instances to a cluster. |
| **Architectural Change** | None. Zero code changes required. | High. Requires stateless services, load balancers, distributed state. |
| **Hard Physical Limit** | Hard ceiling bounded by motherboards and thermal dissipation. | Theoretically boundless. Scale from 2 to 20,000 nodes. |
| **Resilience & SPOF** | Catastrophic. Single hardware fault terminates entire service. | High. Any single node failure is automatically routed around. |
| **Cost Curve** | Hyper-exponential at the high end (specialized hardware). | Linear cost progression using commodity cloud instances. |

---

### 2.2 Stateless Services & Externalized State
To scale an ASP.NET Core web tier horizontally behind a load balancer, instances must be strictly **stateless**. If Node A crashes, Node B must process subsequent requests from the same user without discrepancy.

```
       POOR DESIGN (Stateful)                              PRODUCTION DESIGN (Stateless)
+------------------------------------+             +------------------------------------+
| ASP.NET Core Instance A            |             | ASP.NET Core Instance A (Stateless)|
| - In-Memory Session Cache          |             +-----------------+------------------+
| - User Token State                 |                               |
+------------------------------------+                               v
  (If Instance A dies, session lost)                  +--------------------------------+
                                                      | Externalized Distributed State |
+------------------------------------+                | - Redis Cluster (Tokens/Sess)  |
| ASP.NET Core Instance B            |                | - PostgreSQL (Business Data)   |
| - Can't read Instance A's session  |                +--------------------------------+
+------------------------------------+                               ^
                                                                     |
                                                   +-----------------+------------------+
                                                   | ASP.NET Core Instance B (Stateless)|
                                                   +------------------------------------+
```

#### Tenets of Stateless .NET Architecture:
1. **JWT / OIDC Authentication:** Cryptographically sign user identity into the JWT token (`claimsPrincipal`). Stateless nodes validate the signature using the shared public key without hitting a session store on every request.
2. **External Session Stores:** If session persistence is non-negotiable (e.g., incomplete wizard forms, shopping carts), store state in a Redis cluster using `AddStackExchangeRedisCache`.
3. **No Local File System Reliance:** File uploads must stream directly to cloud object storage (S3/Azure Blob Storage) rather than saving to local container storage `C:\Uploads` or `/var/data`.

---

### 2.3 Read Replicas, Replication Lag & Read-Your-Own-Writes Consistency
Most applications are read-heavy (e.g., 90% reads, 10% writes). Single databases choke under read pressure. We introduce **Master-Replica Replication**.

```
                   +------------------------+
                   |  Write Client (POST)   |
                   +-----------+------------+
                               |
                               v
                   +------------------------+
                   | Primary Database       | <------+
                   | (Accepts all Writes)   |        |
                   +-----------+------------+        | Read-Your-Own-Writes
                               |                     | Fallback (Sticky routing
               Asynchronous    | Replication         | for 10 seconds post-write)
               Stream (WAL)    | Lag (50ms - 2s)     |
                               v                     |
                   +------------------------+        |
                   | Read Replica Database  |        |
                   | (Serves SELECT queries)| -------+
                   +-----------+------------+
                               ^
                               |
                   +-----------+------------+
                   |  Read Client (GET)     |
                   +------------------------+
```

#### The Replication Lag Trap:
In asynchronous replication, the Primary acknowledges a write immediately after writing to its local Write-Ahead Log (WAL), before replicas replicate the change. If a user publishes a post and immediately refreshes their feed, the read lands on an asynchronous replica that hasn't received the WAL entry yet. The user screams: *"My post disappeared!"*

#### Architectural Solutions for Read-Your-Own-Writes:
1. **User-Action Stickiness (Time-Window Routing):** If a user performs a write, route all subsequent read requests from that specific user to the Primary database for $T$ seconds (e.g., 5 seconds), before resuming replica routing.
2. **Replication Watermark / LSN Tracking:** Primary returns the Log Sequence Number (LSN) or transaction version header in the write response. Read queries supply this LSN; if the replica has not caught up to that LSN, the read either waits (with timeout) or forwards to the Primary.

---

### 2.4 Database Sharding & Partitioning Strategies
When a relational table surpasses tens of millions of rows or hundreds of gigabytes, single-instance indexing and disk throughput degrade. **Sharding** breaks large tables into smaller, independent databases across physical hardware.

```
                                  CUSTOMER ID ROUTING
                                           │
                ┌──────────────────────────┼──────────────────────────┐
                ▼                          ▼                          ▼
     +---------------------+    +---------------------+    +---------------------+
     | Shard 0             |    | Shard 1             |    | Shard 2             |
     | CustomerId % 3 == 0 |    | CustomerId % 3 == 1 |    | CustomerId % 3 == 2 |
     | Rows: 50,000,000    |    | Rows: 50,000,000    |    | Rows: 50,000,000    |
     +---------------------+    +---------------------+    +---------------------+
```

#### Sharding Methodologies:
1. **Hash-Based Sharding:**
   $$\text{ShardId} = \text{Hash}(\text{ShardKey}) \pmod N$$
   - *Pros:* Uniform data distribution; avoids hotspotting.
   - *Cons:* Adding or removing a shard requires resharding nearly 100% of all keys unless consistent hashing is used.
2. **Range-Based Sharding:**
   - Sharding by continuous ranges (e.g., CustomerId 1–1,000,000 to Shard 1; 1,000,001–2,000,000 to Shard 2).
   - *Pros:* Range queries within a single shard are localized and fast.
   - *Cons:* Massive write hotspots (all new monotonically increasing IDs hammer the newest shard).
3. **Directory-Based (Lookup) Sharding:**
   - A central lookup service / mapping table stores which shard holds each entity ID.
   - *Pros:* Unmatched flexibility. Moving a customer to another shard is a single metadata update.
   - *Cons:* The lookup table is an operational SPOF and introduces extra query latency.

#### Production Costs of Sharding:
- **Cross-Shard Joins:** Virtually impossible; requires in-memory application-layer joins.
- **Cross-Shard Transactions:** Requires distributed 2PC or Saga patterns.
- **Resharding Complexity:** Doubling shards in production requires zero-downtime dual-writing and background sync engines.

---

### 2.5 Consistent Hashing with Virtual Nodes
In traditional modulo hashing ($\text{Hash}(\text{Key}) \pmod N$), adding or removing a single node invalidates almost every key-to-node mapping. If you scale from 9 to 10 cache servers, approximately $\frac{9}{10}$ (90%) of keys are remapped, instantly causing a catastrophic **cache stampede** that incinerates your primary database.

**Consistent Hashing** bounds key relocation so that when a node is added or removed, only $\frac{K}{N}$ keys are moved on average (where $K$ is total keys and $N$ is total nodes).

```
                         THE CONSISTENT HASH RING
                                    0
                              .-----*-----.
                           .-'             '-.
                         .'                   '.
                     Node C                    Node A
                   (vnode C1)                (vnode A1)
                    /                             \
                   |        * Key: User_1092       |
                   |        (Routes clockwise      |
                   |         to Node A)            |
                    \                             /
                     '.                         .'
                   (vnode B1)                (vnode A2)
                       '-.                   .-'
                          '-----*-----*-----'
                              Node B  (vnode C2)
```

#### Virtual Nodes (VNodes) Solve Non-Uniformity:
With few physical nodes, keys cluster unevenly on the ring. By mapping each physical node to multiple virtual positions (e.g., 250 virtual tokens per physical node), data distribution variance approaches $< 1\%$.

#### Production C# Implementation of Consistent Hash Ring:

```csharp
using System.Security.Cryptography;
using System.Text;

public class ConsistentHashRing<TNode>
{
    private readonly SortedDictionary<uint, TNode> _ring = new();
    private readonly int _virtualNodeReplicas;

    public ConsistentHashRing(int virtualNodeReplicas = 200)
    {
        _virtualNodeReplicas = virtualNodeReplicas;
    }

    public void AddNode(TNode node)
    {
        for (int i = 0; i < _virtualNodeReplicas; i++)
        {
            string vNodeKey = $"{node}:{i}";
            uint hash = ComputeHash(vNodeKey);
            _ring[hash] = node;
        }
    }

    public void RemoveNode(TNode node)
    {
        for (int i = 0; i < _virtualNodeReplicas; i++)
        {
            string vNodeKey = $"{node}:{i}";
            uint hash = ComputeHash(vNodeKey);
            _ring.Remove(hash);
        }
    }

    public TNode GetNode(string routingKey)
    {
        if (_ring.Count == 0)
            throw new InvalidOperationException("Hash ring is empty.");

        uint keyHash = ComputeHash(routingKey);

        // Find the first virtual node with hash >= keyHash (clockwise search)
        foreach (var entry in _ring)
        {
            if (entry.Key >= keyHash)
                return entry.Value;
        }

        // Wrap around to the first node on the ring
        return _ring.First().Value;
    }

    private static uint ComputeHash(string input)
    {
        byte[] bytes = MD5.HashData(Encoding.UTF8.GetBytes(input));
        // Take the first 4 bytes to form a 32-bit unsigned integer ring point
        return BitConverter.ToUInt32(bytes, 0);
    }
}
```

---

## 3. CACHING STRATEGIES & REDIS INTERNALS {#3-caching-strategies-and-redis-internals}

### 3.1 Caching Topologies

```
1. CACHE-ASIDE (Lazy Load)         2. WRITE-THROUGH                 3. WRITE-BEHIND (Write-Back)
   App -> Read Cache?                 App -> Write Cache               App -> Write Cache (ACK)
   If MISS -> Read DB                 Cache -> Write DB (Sync)         Cache -> Async Queue -> DB
   App -> Write Cache                 App <- Return                    App <- Immediate Return
```

| Strategy | Read Mechanism | Write Mechanism | Pros | Cons | Best Use Case |
|---|---|---|---|---|---|
| **Cache-Aside** | Application reads cache. On miss, reads DB, then populates cache. | Application writes directly to DB, then invalidates or updates cache. | Only requested data is cached. Node crash does not lose data. | Cache miss penalty (2 roundtrips). Risk of stale data during concurrent updates. | General read-heavy web apps (product catalogs, user profiles). |
| **Write-Through** | Application queries cache. Cache acts as single interface. | Application writes to cache; cache synchronously writes to DB before acknowledging. | High consistency between cache and DB. No stale reads. | Write latency penalty (sum of cache write + DB write). Redundant caching of rarely read items. | Financial account balances, authorization rights. |
| **Write-Behind (Write-Back)** | Application queries cache. | Application writes to cache; cache returns immediately, then asynchronously flushes to DB. | Ultra-low write latency. Absorb massive write spikes (write coalescing). | High data loss risk if cache crashes before dirty buffer flushes to persistent DB. | Click tracking, analytics counters, real-time gaming state. |
| **Refresh-Ahead** | Cache automatically reloads frequently accessed keys prior to TTL expiry. | Standard write-through or cache-aside. | Eliminates read latency on hot keys; prevents cold cache misses. | Hard to predict access patterns accurately; wasted DB load on false predictions. | Hot news feeds, stock ticker homepages. |

---

### 3.2 Cache Invalidation, Stampede & Solutions
> *"There are only two hard things in Computer Science: cache invalidation and naming things."* — Phil Karlton

#### The Cache Stampede (Thundering Herd) Problem:
When a high-traffic key (e.g., home page feed with 50,000 reads/sec) expires its TTL, 50,000 concurrent threads detect a cache miss simultaneously. All 50,000 threads blast the primary database with identical queries, saturating connection pools and driving CPU to 100%, causing cascading system outage.

```
                               CACHE STAMPEDE MITIGATION
                         +-----------------------------------+
                         | 10,000 Requests for Expired Key   |
                         +-----------------+-----------------+
                                           |
                                           v
                         +-----------------------------------+
                         | Distributed Mutex (Redis SETNX)   |
                         +--------+-----------------+--------+
                                  |                 |
                   Winner (1 req) |                 | 9,999 Waiting Requests
                                  v                 v
                         +-----------------+  +--------------------------------+
                         | Query Database  |  | Spin / Sleep 50ms & Retry      |
                         | & Repopulate    |  | Cache (Read fresh cache value) |
                         +-----------------+  +--------------------------------+
```

#### Production Solutions:
1. **Distributed Mutex Locking (`SETNX`):** Only the single thread that acquires the distributed lock queries the database. All other callers sleep for 25–50ms and re-check the cache.
2. **Probabilistic Early Expiration (XFetch Algorithm):**
   Instead of hard TTL, calculate an early refresh probabilistically:
   $$\text{ShouldRefresh} = (\Delta \times \beta \times \ln(\text{random}())) > (\text{TTL} - \text{currentTime})$$
   As the expiration time nears, the probability of an incoming read proactively fetching fresh data asynchronously rises, ensuring the cache is refreshed *before* it officially expires.
3. **Soft TTL with Background Refresh:** Store a JSON payload containing `{ data, softExpiry, hardExpiry }`. When `softExpiry` passes, the current caller receives stale data immediately, while a background `Task.Run` fires to refresh the DB and update the cache.

---

### 3.3 Redis Deep Dive: Internal Threading Model & Memory Eviction
Many candidates incorrectly answer: *"Redis is purely single-threaded."* As a senior engineer, state the exact architecture:

- **The Main Execution Thread:** Redis runs commands (GET, SET, ZADD, Lua scripts) on a **single-threaded event loop** backed by non-blocking multiplexing I/O primitives (`epoll` on Linux, `kqueue` on BSD). Because operations run in memory without context switches or thread contention, execution takes sub-microsecond time.
- **Background Worker Threads (Redis 4.0+):** Asynchronous deletion of heavy keys (`UNLINK` command, non-blocking cache eviction) runs on secondary OS threads to prevent blocking the main event loop.
- **Threaded I/O (Redis 6.0+):** Socket reading, packet parsing, and client response serialization run across multi-threaded I/O workers, while the core memory state machine remains strictly single-threaded and lock-free.

#### Memory Eviction Algorithms (`maxmemory-policy`):
When Redis reaches its memory ceiling, it drops keys according to its configured policy:
1. `volatile-lru`: Evict the least recently used keys among those with an explicit TTL.
2. `allkeys-lru`: Evict the least recently used keys across all stored data (standard for pure caches).
3. `allkeys-lfu`: Evict keys accessed least frequently (better for retaining permanent hot items).
4. `volatile-ttl`: Evict keys with the shortest remaining time-to-live.
5. `noeviction`: Return an Out-Of-Memory (OOM) error on any incoming write operation.

---

### 3.4 Production Redis Data Types for System Designers

```
REDIS DATA STRUCTURES IN SYSTEM DESIGN
├── Strings      -> Caching JSON, atomic counters (INCRBY), distributed locks (SET NX PX)
├── Hashes       -> User objects, product catalog fields (HGET, HSET, HINCRBYFLOAT)
├── Lists        -> FIFO job queues (LPUSH / RPOPLPUSH), timeline activity feeds
├── Sets         -> Unique followers, IP blacklists, tag matching (SADD, SINTER, SISMEMBER)
├── Sorted Sets  -> Leaderboards, rate limiters, priority queues, time-window sliding logs (ZADD, ZREVRANGEBYSCORE)
├── Bitmaps      -> Daily active user tracking (SETBIT, BITCOUNT) - 100M users mapped in 12 MB RAM
├── HyperLogLog  -> Cardinality estimation of billions of unique pageviews (PFADD, PFCOUNT) with 0.81% error in 12 KB
└── Streams      -> Distributed append-only message log, consumer groups, event streaming (XADD, XREADGROUP)
```

---

### 3.5 Distributed Locking with Redis (Atomic Lua Script in C#)
Using a plain `SETNX` without atomic release tokens results in dangerous race conditions: Process A acquires the lock, experiences a long garbage collection (GC) pause, the TTL expires, Process B acquires the lock, and then Process A wakes up and blindly deletes Process B's lock!

#### Production C# Distributed Lock Manager:

```csharp
using StackExchange.Redis;

public sealed class RedisDistributedLock
{
    private readonly IDatabase _db;
    private const string ReleaseLockLuaScript = @"
        if redis.call('get', KEYS[1]) == ARGV[1] then
            return redis.call('del', KEYS[1])
        else
            return 0
        end";

    public RedisDistributedLock(IConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    public async Task<bool> ExecuteWithLockAsync(
        string resourceKey, 
        TimeSpan lockTimeout, 
        Func<Task> criticalAction)
    {
        string lockToken = Guid.NewGuid().ToString("N");
        string lockKey = $"lock:{resourceKey}";

        // SET key token NX PX timeout_ms
        bool acquired = await _db.StringSetAsync(
            lockKey, 
            lockToken, 
            lockTimeout, 
            When.NotExists);

        if (!acquired)
            return false;

        try
        {
            await criticalAction();
            return true;
        }
        finally
        {
            // Atomically verify the token matches before releasing
            await _db.ScriptEvaluateAsync(
                ReleaseLockLuaScript,
                new RedisKey[] { lockKey },
                new RedisValue[] { lockToken });
        }
    }
}
```

---

## 4. LOAD BALANCING & REVERSE PROXIES {#4-load-balancing-and-reverse-proxies}

### 4.1 Layer 4 vs Layer 7 Load Balancing

```
LAYER 4 LOAD BALANCING (Transport / TCP/UDP)
Client [198.51.100.1:443] ──────> L4 Balancer (IP/Port NAT) ──────> Worker Node [10.0.1.5:443]
                                  (Zero payload inspection;
                                   Terminates no TLS; Blazing fast)

LAYER 7 LOAD BALANCING (Application / HTTP/gRPC)
Client ──[TLS Terminate]──> L7 Proxy (Inspect Headers, Path, JWT) ──[HTTP/2]──> Catalog Svc (/catalog)
                                                                 ──[gRPC]────> Order Svc (/orders)
```

| Feature | Layer 4 (Transport: AWS NLB, IPVS) | Layer 7 (Application: YARP, Nginx, Envoy, AWS ALB) |
|---|---|---|
| **Protocol Level** | TCP, UDP, SCTP. | HTTP/1.1, HTTP/2, HTTP/3 (QUIC), gRPC, WebSockets. |
| **Inspection Capability** | IP address, TCP port numbers only. | URL path, cookies, request headers, payload body, HTTP method. |
| **TLS Termination** | Passes raw encrypted bytes or offloads at TCP level. | Full TLS termination; inspects decrypted traffic; re-encrypts upstream. |
| **Routing Capability** | IP hash, round robin, least TCP connections. | Path-based routing (`/api/v1/orders`), header routing, blue-green canary routing. |
| **Performance / Overhead**| Microsecond latency; hundreds of thousands of RPS per node. | Millisecond latency overhead due to buffer parsing and TLS decryptions. |

---

### 4.2 Load Balancing Algorithms
1. **Round Robin & Weighted Round Robin:** Distributes requests sequentially. Weights allow beefier hardware to process proportionally higher traffic.
2. **Least Connections:** Routes new requests to the instance with the lowest number of active concurrently serviced requests. Ideal for long-running connections (e.g., file processing, database exports).
3. **IP / Consistent Hash:** Hashes the client's source IP address or a cookie header. Guarantees that requests from the same client route to the identical upstream server (useful for caching locality, but problematic behind corporate NAT gateways).
4. **Least Response Time:** Dynamic routing targeting the server combining lowest active connections with fastest recent latency.

---

### 4.3 Reverse Proxies: Nginx, Envoy vs Microsoft YARP (.NET)
In modern .NET enterprise ecosystems, **YARP (Yet Another Reverse Proxy)** has emerged as a premier L7 reverse proxy. Built natively on ASP.NET Core primitives (`Kestrel`, `SocketsHttpHandler`), YARP matches or exceeds Nginx performance for .NET workloads, while offering 100% C# extensibility:

```
                                YARP TOPOLOGY
                             +------------------+
                             | Internet Clients |
                             +--------+---------+
                                      |
                                      v
                             +------------------+
                             |   Kestrel Engine |
                             +--------+---------+
                                      |
                +---------------------+---------------------+
                | YARP Custom Middleware Pipeline           |
                | - Distributed Token Bucket Rate Limiter   |
                | - Custom JWT Signature & Scope Validator  |
                | - OpenTelemetry Distributed Trace Injector|
                +---------------------+---------------------+
                                      |
                      +---------------+---------------+
                      |                               |
                      v                               v
             +-----------------+             +-----------------+
             | Order Service   |             | Payment Service |
             | (.NET 8 WebAPI) |             | (.NET 8 WebAPI) |
             +-----------------+             +-----------------+
```

#### Production YARP Configuration (`appsettings.json`):

```json
{
  "ReverseProxy": {
    "Routes": {
      "order-route": {
        "ClusterId": "order-cluster",
        "Match": {
          "Path": "/api/v1/orders/{**catch-all}"
        },
        "Transforms": [
          { "PathPattern": "/api/v1/orders/{**catch-all}" },
          { "RequestHeader": "X-Forwarded-Gateway", "Set": "YARP-Gateway-01" }
        ]
      }
    },
    "Clusters": {
      "order-cluster": {
        "LoadBalancingPolicy": "RoundRobin",
        "HealthCheck": {
          "Active": {
            "Enabled": true,
            "Interval": "00:00:10",
            "Timeout": "00:00:02",
            "Policy": "ConsecutiveFailures",
            "Path": "/health/ready"
          }
        },
        "Destinations": {
          "order-worker-1": { "Address": "http://10.0.4.11:8080" },
          "order-worker-2": { "Address": "http://10.0.4.12:8080" }
        }
      }
    }
  }
}
```

---

## 5. CAP & PACELC THEOREMS IN PRACTICE {#5-cap-and-pacelc-theorems}

### 5.1 Deconstructing CAP: Why "Pick 2" is a Myth
Formulated by Eric Brewer and proved by Seth Gilbert and Nancy Lynch, the CAP theorem states that a distributed data store can simultaneously provide at most two out of three guarantees:
- **Consistency ($C$):** Every read receives the most recent write or an error (Linearizability).
- **Availability ($A$):** Every non-failing node returns a non-error response, without guarantee that it contains the most recent write.
- **Partition Tolerance ($P$):** The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network.

```
                           THE CAP TRADEOFF REALITY
                  You CANNOT choose "CA" without Partitions!
                     Network Partitions (P) are an immutable
                           physical reality of hardware.
                                       │
                    ┌──────────────────┴──────────────────┐
                    ▼                                     ▼
        CHOOSE CP (Consistency)                  CHOOSE AP (Availability)
  Reject / Block writes on minority        Accept writes on all partitions.
  partition to guarantee zero dirty data.  Risk stale reads and split-brain conflicts;
  (Banking, Ledgers, Consensus)            reconcile asynchronously later.
  Examples: etcd, ZooKeeper, CockroachDB   Examples: Cassandra, DynamoDB, Couchbase
```

> **Interview Pro-Tip:** State unequivocally: *"You cannot choose CA. Network cables get cut, switches fail, and cloud hypervisors drop packets. Partition Tolerance ($P$) is mandatory in distributed networks. Therefore, your only real choice is: When a network partition occurs, do you choose Consistency ($CP$) or Availability ($AP$)?"*

---

### 5.2 PACELC Theorem: Accounting for Normal Operations
The CAP theorem only describes behavior **during rare network partitions**. Daniel Abadi formulated the **PACELC theorem** to describe distributed trade-offs during the 99.99% of time when the network is healthy:

$$\mathbf{If\ P\ (Partition):}\ \text{Choose between } \mathbf{A} \text{ (Availability) and } \mathbf{C} \text{ (Consistency);}$$
$$\mathbf{Else\ (Normal):}\ \text{Choose between } \mathbf{L} \text{ (Latency) and } \mathbf{C} \text{ (Consistency).}$$

#### PACELC System Classification:

| System | PACELC Classification | Partition Behavior | Normal Behavior | Engineering Reasoning |
|---|---|---|---|---|
| **Cassandra / DynamoDB** | **PA / EL** | Chooses Availability (accepts writes anywhere). | Chooses Latency (returns local read without waiting for cross-node replication). | Maximum throughput and sub-5ms response times across the globe. |
| **MongoDB (Default)** | **PC / EC** | Chooses Consistency (primary stops accepting writes if it loses majority). | Chooses Consistency (routes reads/writes to primary by default). | Avoids document anomalies and dirty reads. |
| **PostgreSQL (Sync Repl)** | **PC / EC** | Chooses Consistency (blocks writes if synchronous replica does not ACK). | Chooses Consistency (waits for replica ACK, paying latency penalty on every commit). | Absolute data integrity for enterprise accounting. |
| **Azure Cosmos DB** | **Configurable** | Tunable from Strong ($PC/EC$) to Eventual ($PA/EL$) with Bounded Staleness and Session Consistency options. | Configurable per account and read request. | Allows tailoring latency and consistency SLA to specific microservice bounded contexts. |

---

## 6. DATABASE SELECTION MATRIX {#6-database-selection-matrix}

Choosing the wrong database engine is an architectural failure that can sink an enterprise project. A senior engineer matches the storage engine's physical disk layout and memory architecture to the application's query access patterns.

```
+----------------------------------------------------------------------------------------------------+
|                                    DATABASE STORAGE PARADIGMS                                      |
+-------------------+--------------------+--------------------+--------------------+-----------------+
| Relational (RDBMS)| Document Store     | Key-Value Store    | Time-Series (TSDB) | Wide-Column     |
| (PostgreSQL, SQL) | (MongoDB, Cosmos)  | (Redis, DynamoDB)  | (Timescale, Influx)| (Cassandra)     |
+-------------------+--------------------+--------------------+--------------------+-----------------+
| B-Tree on disk    | B-Tree / WiredTiger| Hash Table in RAM  | LSM-Tree + Chunking| LSM-Tree on disk|
| Strict ACID       | Flexible JSON      | Sub-ms Key Lookups | High Ingest Rates  | Massive Writes  |
| Complex Joins     | Hierarchical data  | Ephemeral / Caching| Time-bucket rollup | Write-anywhere  |
+-------------------+--------------------+--------------------+--------------------+-----------------+
```

### 6.1 Database Archetypes & Internal Storage Engines
1. **Relational Databases (PostgreSQL / Microsoft SQL Server):**
   - *Storage Engine:* Balanced Trees (B+Trees) on disk pages.
   - *Access Pattern:* High-complexity queries, multi-table relational joins, strict ACID transaction boundaries.
   - *Weakness:* Horizontal sharding is complex; write throughput bounded by single-master write locks.
2. **Document Stores (MongoDB / Couchbase):**
   - *Storage Engine:* B-Trees (WiredTiger) storing BSON/JSON binary payloads.
   - *Access Pattern:* Entities that are read and written as cohesive, self-contained aggregates (e.g., user profiles, catalogs with varying attributes).
   - *Weakness:* Cross-document transactions are costly; unbounded document growth degrades disk page performance.
3. **Key-Value Stores (Redis / AWS DynamoDB):**
   - *Storage Engine:* In-memory hash tables with optional disk persistence (RDB/AOF), or partition hash rings.
   - *Access Pattern:* O(1) reads and writes keyed by a unique string identifier.
   - *Weakness:* Inability to perform arbitrary attribute queries or complex filtering without secondary indexing penalties.
4. **Time-Series Databases (TimescaleDB / InfluxDB):**
   - *Storage Engine:* Append-only partitioned chunks structured chronologically; heavy columnar compression (Gorilla / Delta-of-delta).
   - *Access Pattern:* High-frequency, append-only timestamped events (telemetry, stock ticks, IoT metrics); aggregations over moving time windows.
   - *Weakness:* Updating or deleting historic individual records is computationally expensive.
5. **Wide-Column Stores (Apache Cassandra / ScyllaDB):**
   - *Storage Engine:* Log-Structured Merge-trees (LSM-Trees): Memtable (RAM) $\to$ CommitLog (disk append) $\to$ SSTables (immutable sorted files on disk) $\to$ Compaction.
   - *Access Pattern:* High-throughput writes across distributed masterless clusters.
   - *Weakness:* No relational joins, no ad-hoc querying. Queries must strictly match the pre-defined partition key and clustering column sequence.

---

### 6.2 Unified Database Decision Matrix

| Requirement | Recommended Database | Second Best | Why? |
|---|---|---|---|
| **Financial Ledger / Double-Entry** | PostgreSQL / SQL Server | CockroachDB | Strict ACID guarantees, serializable isolation, deterministic constraints. |
| **High-Throughput Global Click Ingestion** | Cassandra / ScyllaDB | ClickHouse | LSM-tree sequential append writes; masterless multi-datacenter replication. |
| **Product Catalog with Dynamic Specs** | MongoDB / Azure Cosmos DB | PostgreSQL (JSONB) | Polymorphic schemas allow electronics, apparel, and groceries in a single collection. |
| **Real-Time Leaderboard** | Redis (Sorted Sets) | Memcached | In-memory skiplist provides $O(\log N)$ score insertion and instant rank queries. |
| **IoT Sensor Telemetry (1M points/sec)** | TimescaleDB / InfluxDB | Cassandra | Time-partitioned chunks, automated downsampling, delta-of-delta timestamp compression. |
| **Global User Session Store** | Redis Cluster | DynamoDB | Sub-millisecond latency, configurable key TTLs, lightweight memory footprints. |


---

## 7. FULL DESIGN: SCALABLE URL SHORTENER (BIT.LY CLONE) {#7-full-design-url-shortener}

A URL Shortener generates compact alias links for long URLs (e.g., transforming `https://www.enterprise-corp.com/products/catalog/electronics/2026/sku-89218903` into `https://sho.rt/aZ81kQ`). When users navigate to the alias, the service redirects them to the original address.

### 7.1 Requirements & Scope Bounding

#### Functional Requirements:
1. **URL Shortening:** Given an arbitrary long URL, generate a unique, compact alias (7 characters).
2. **URL Redirection:** When an alias is visited, redirect the client to the original long URL with sub-50ms P99 latency.
3. **Custom Vanity Aliases:** Allow users to request custom short links (e.g., `sho.rt/blackfriday`).
4. **URL Expiration:** Links should optionally expire after a user-specified TTL (default: 5 years).
5. **Telemetry & Analytics:** Track click count, referring domain, user device, and geographical country.

#### Non-Functional Requirements:
1. **Ultra-High Availability:** 99.999% uptime. A broken redirect breaks user navigation across the web.
2. **Read-to-Write Skew:** Extremely read-heavy ($100:1$ ratio).
3. **Low Latency:** Redirection must execute in $< 20\text{ms}$ at the application layer.
4. **Unpredictability:** Short tokens must be non-sequential to prevent scrapers from enumerating customer URLs.

---

### 7.2 Capacity Estimations & Back-of-the-Envelope Math

#### Traffic & Throughput:
- **New Short URLs (Writes):** 500 Million per year.
  $$\text{Daily Writes} = \frac{500,000,000}{365} \approx 1.37 \text{ Million/day}$$
  $$\text{Average Write QPS} = \frac{1,370,000}{86,400} \approx 16 \text{ requests/sec}$$
  $$\text{Peak Write QPS} = 16 \times 5 = 80 \text{ requests/sec}$$

- **Redirect Requests (Reads at 100:1 ratio):**
  $$\text{Daily Reads} = 1.37 \text{ Million} \times 100 = 137 \text{ Million/day}$$
  $$\text{Average Read QPS} = \frac{137,000,000}{86,400} \approx 1,585 \text{ requests/sec}$$
  $$\text{Peak Read QPS} = 1,585 \times 3 \approx 4,800 \text{ requests/sec}$$

#### Storage Requirements (5-Year Horizon):
- Average long URL size: 500 bytes.
- Short token size: 7 bytes.
- Metadata (created_at, user_id, expires_at): 43 bytes.
- Total storage per record: $\approx 550 \text{ bytes}$.
$$\text{Total Records (5 years)} = 500 \text{ Million} \times 5 = 2.5 \text{ Billion records}$$
$$\text{Raw Storage} = 2.5 \times 10^9 \times 550 \text{ bytes} \approx 1.375 \text{ TB}$$
$$\text{Total Storage (with indexes + 30\% replication headroom)} \approx 2.5 \text{ TB}$$

#### Cache Memory Requirements (RAM):
- Follow Pareto's 80/20 rule: 20% of daily redirects account for 80% of read traffic.
$$\text{Daily Read Volume} = 137 \text{ Million} \times 550 \text{ bytes} \approx 75.35 \text{ GB}$$
$$\text{Memory Required (20\%)} = 75.35 \text{ GB} \times 0.20 \approx 15 \text{ GB RAM}$$
*A standard two-node Redis replica cluster with 32 GB RAM can effortlessly hold all hot URLs.*

---

### 7.3 High-Level Architecture Diagram

```
                                      ARCHITECTURE: URL SHORTENER
                                      
  [Write Client]                [Read Client / Browser]
        │                                  │
        │ POST /api/v1/urls                │ GET /{short_code}
        ▼                                  ▼
+──────────────────────────────────────────────────────────+
|                    Global Anycast DNS                    |
+──────────────────────────────────────────────────────────+
                             │
                             v
+──────────────────────────────────────────────────────────+
|           Edge CDN & Cloudflare Reverse Proxy            |
|       (DDoS Mitigation, SSL Offload, Edge Caching)       |
+──────────────────────────────────────────────────────────+
                             │
                             v
+──────────────────────────────────────────────────────────+
|            Layer 7 Load Balancer / YARP Gateway          |
+────────────────────────────┬─────────────────────────────+
                             │
             ┌───────────────┴───────────────┐
             ▼                               ▼
+─────────────────────────+     +─────────────────────────+
| URL Shortening Service  |     | URL Redirection Service |
| (Stateless ASP.NET 8)   |     | (Stateless ASP.NET 8)   |
+────────────┬────────────+     +────────────┬────────────+
             │                               │
    Fetch    │                               ├────────────────────────┐
    Token    v                               v                        v
+─────────────────────+             +─────────────────+      +─────────────────+
| Key Generation Svc  |             |  Bloom Filter   |      |  Redis Cluster  |
| (KGS Worker Pool)   |             | (Reject 404s)   |      |  (L2 Cache)     |
+──────────┬──────────+             +─────────────────+      +────────┬────────+
           │                                                          │
           v                                                          │ Cache Miss
+─────────────────────+                                               v
| KGS Token Database  |                                      +─────────────────+
| (Pre-allocated keys)|                                      | PostgreSQL DB   |
+─────────────────────+                                      | (Sharded Store) |
                                                             +────────┬────────+
                                                                      │
                                                                      v
                                                             +─────────────────+
                                                             | Kafka Click Log |
                                                             | (Telemetry Svc) |
                                                             +─────────────────+
```

---

### 7.4 Data Flow Step-by-Step

#### Write Path (URL Creation):
1. **Client Request:** Client sends `POST /api/v1/urls` with `{ "original_url": "https://example.com/long-page", "custom_alias": null }`.
2. **Validation:** Gateway validates URL formatting and runs anti-phishing/malware checks via domain safety APIs.
3. **Token Acquisition:**
   - If `custom_alias` is supplied: Query Redis/DB to verify availability. If taken, throw `409 Conflict`.
   - If standard generation: Fetch an unused 7-character pre-generated token from the local in-memory buffer supplied by the **Key Generation Service (KGS)**.
4. **Persistence:** Insert mapping `(short_token, original_url, user_id, expires_at)` into PostgreSQL.
5. **Cache Warming:** Concurrently write the new mapping into the Redis Cluster with configured TTL.
6. **Bloom Filter Registration:** Add the new `short_token` to the Redis Bloom Filter.
7. **Response:** Return `HTTP 201 Created` with payload `{ "short_url": "https://sho.rt/aZ81kQ" }`.

#### Read Path (Redirection):
1. **Incoming Request:** User clicks `https://sho.rt/aZ81kQ`. Browser issues `GET /aZ81kQ`.
2. **Bloom Filter Gatekeeping:** Redirection service checks the Bloom Filter. If Bloom Filter returns `false`, the token definitely does not exist $\to$ instantly return `HTTP 404 Not Found` without touching Redis or DB.
3. **Cache Lookup:** Redirection service queries Redis: `GET url:aZ81kQ`.
   - **Cache Hit:** Retrieve `original_url` in $< 2\text{ms}$.
   - **Cache Miss:** Query PostgreSQL read replica. Upon retrieval, asynchronously populate Redis.
4. **Telemetry Dispatch:** Redirection service pushes click metadata (timestamp, user-agent, IP, referer) to a Kafka topic `url-clicks`.
5. **Redirect Header:** Return `HTTP 302 Found` with `Location: https://example.com/long-page`.

---

### 7.5 Database Schema Design

#### Relational Schema (PostgreSQL):

```sql
CREATE TABLE urls (
    short_token     VARCHAR(7) PRIMARY KEY,
    original_url    TEXT NOT NULL,
    user_id         UUID NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at      TIMESTAMPTZ NULL,
    is_active       BOOLEAN NOT NULL DEFAULT TRUE
);

-- Index for expiration cleanup jobs
CREATE INDEX ix_urls_expires_at ON urls (expires_at) 
WHERE expires_at IS NOT NULL;

-- Dedicated analytics aggregate table (populated by Kafka consumer stream)
CREATE TABLE url_analytics_daily (
    short_token     VARCHAR(7) NOT NULL,
    click_date      DATE NOT NULL,
    country_code    CHAR(2) NOT NULL DEFAULT 'ZZ',
    click_count     BIGINT NOT NULL DEFAULT 0,
    PRIMARY KEY (short_token, click_date, country_code)
);
```

#### Sharding Strategy:
When the table exceeds 100M rows, shard horizontally across multiple PostgreSQL nodes using a **Hash-based sharding key on `short_token`**:
$$\text{ShardId} = \text{Murmur3}(\text{short\_token}) \pmod{\text{NumberOfShards}}$$
Because all read lookups query strictly by `short_token`, every query routes directly to a single shard, eliminating cross-shard joins.

---

### 7.6 Component Deep Dive

#### 7.6.1 Encoding Strategy: Base62 vs Hash Truncation
Why not use MD5 or SHA-256?
- $\text{MD5}(\text{URL})$ yields a 128-bit hex string (32 characters).
- Taking the first 7 characters of MD5 guarantees severe **hash collisions** at high volumes. Checking the database on every collision and appending a salt adds disk round-trips and race conditions.

**The Base62 Solution:**
Use characters `[0-9, a-z, A-Z]` ($10 + 26 + 26 = 62\text{ unique characters}$).
$$62^7 = 3,521,614,606,208 \text{ combinations (3.52 Trillion)}$$
At 500 million URLs per year, a 7-character Base62 space will not exhaust for over **7,000 years**.

```csharp
public static class Base62Encoder
{
    private const string CharacterSet = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    public static string Encode(long numericId)
    {
        if (numericId == 0) return CharacterSet[0].ToString();

        var sb = new StringBuilder();
        while (numericId > 0)
        {
            int remainder = (int)(numericId % 62);
            sb.Append(CharacterSet[remainder]);
            numericId /= 62;
        }

        // Pad to ensure 7 characters
        while (sb.Length < 7)
        {
            sb.Append('0');
        }

        char[] charArray = sb.ToString().ToCharArray();
        Array.Reverse(charArray);
        return new string(charArray);
    }
}
```

#### 7.6.2 Key Generation Service (KGS)
To guarantee zero collisions and avoid runtime hashing or auto-increment DB locks, build a dedicated **Key Generation Service (KGS)**.

```
+─────────────────────────────────────────────────────────────+
|                     KGS Architecture                        |
|                                                             |
|  +──────────────────────+        +───────────────────────+  |
|  | Token Generator Tool | -----> | Token Database        |  |
|  | (Offline Pre-calc)   |        | (Stores unused tokens)|  |
|  +──────────────────────+        +──────────┬────────────+  |
|                                             |               |
|                      Loads block of 100,000 | tokens        |
|                                             v               |
|                                  +───────────────────────+  |
|                                  | KGS Service Node      |  |
|                                  | Buffer 1: Active      |  |
|                                  | Buffer 2: Standby     |  |
|                                  +──────────┬────────────+  |
|                                             |               |
|               Dispenses via gRPC in < 1ms   v               |
|                                  +───────────────────────+  |
|                                  | ASP.NET Web API       |  |
|                                  +───────────────────────+  |
+─────────────────────────────────────────────────────────────+
```

1. **Pre-computation:** An offline utility creates unique sequential integers from 1 to 3.5 Trillion, encodes them into Base62, randomly shuffles them, and stores them in a `token_store` table marked `is_allocated = FALSE`.
2. **Dual-Buffer In-Memory Allocation:**
   The KGS service node loads a block of 100,000 tokens into memory and marks them `is_allocated = TRUE` in the database.
3. **Fault Tolerance:** If a KGS node crashes, the unassigned tokens currently in its memory buffer are lost. **This is completely acceptable.** In a 3.5-trillion space, losing 100,000 keys ($0.000002\%$) has zero impact on capacity.
4. **Concurrency:** Dispensing tokens from the memory buffer uses lock-free atomic increments (`Interlocked.Increment`), handling hundreds of thousands of requests per second per node.

#### 7.6.3 301 vs 302 HTTP Redirects
Interviewers *always* grill candidates on this distinction:

| Redirect Code | HTTP Standard | Browser Behavior | Telemetry Impact | Recommendation |
|---|---|---|---|---|
| **301 Moved Permanently** | Permanent redirection. | Browser permanently caches the target URL in local browser storage. | **Catastrophic for Analytics.** Future clicks never reach the server; click counts and geolocation metrics stop working. | Use only if infrastructure cost is critical and you do not care about analytics. |
| **302 Found (Temporary)** | Temporary redirection. | Browser does NOT cache target URL. Every redirect hits the server. | **Full Analytics Preserved.** Every click can be logged, validated, rate-limited, and inspected. | **Industry Standard (bit.ly, t.co).** |

---

### 7.7 Scalability & Caching Strategy
1. **Edge Caching via CDN:** Standard CDN caches responses using `Cache-Control: private, max-age=0, no-cache` to ensure every click routes to the server for analytics while leveraging CDN POPs for TLS termination.
2. **Redis Eviction:** Configure Redis cluster with `allkeys-lru`. Frequently clicked links stay in memory; dead links are pruned automatically.
3. **Bloom Filters:** Place a Redis Bloom Filter in front of the cache. If malicious bots blast your cluster with random non-existent URLs (`sho.rt/fake99`), the Bloom Filter drops them before they query the primary database.

---

### 7.8 Production Trade-offs & Failure Scenarios
- **Scenario A: KGS Database Outage:**
  - *Mitigation:* Web API servers maintain a local reserve buffer of 5,000 tokens. The system can continue creating URLs for several minutes while the KGS DB fails over to its replica.
- **Scenario B: Cache Node Eviction Storm (Stampede on Hot Influencer Link):**
  - *Mitigation:* Implement distributed locking on cache miss. The first worker thread acquires the lock, fetches from Postgres, and sets Redis. Remaining threads wait 20ms and hit the cache.
- **Scenario C: Malicious Phishing Redirects:**
  - *Mitigation:* Asynchronous Kafka worker scans `original_url` against Google Safe Browsing and Microsoft Defender APIs. If flagged, sets `is_active = FALSE` in DB and purges the key from Redis, displaying a warning landing page to visitors.

---

## 8. FULL DESIGN: DISTRIBUTED MULTI-CHANNEL NOTIFICATION SYSTEM {#8-full-design-notification-system}

Enterprise notification platforms deliver billions of messages daily across **Push Notifications (APNs, FCM)**, **SMS (Twilio, Infobip)**, and **Email (SendGrid, AWS SES)**. They must guarantee high deliverability, prevent spamming, respect user preferences, and survive upstream third-party outages.

### 8.1 Requirements & Scope Bounding

#### Functional Requirements:
1. **Multi-Channel Dispatch:** Support Mobile Push (iOS/Android), SMS, and Email.
2. **Priority Delivery:** Support High-Priority (OTP, Fraud alerts, transactional receipts: $< 2\text{s}$ SLA) and Low-Priority (Promotional campaigns, newsletters: $< 4\text{h}$ SLA).
3. **User Preferences & Quiet Hours:** Honor user opt-outs, channel preferences, and time-zone-based quiet hours (e.g., no promotional SMS between 9 PM and 8 AM).
4. **Deduplication:** Prevent duplicate messages if an upstream producer retries requests.
5. **Template Management:** Dynamically render templates with localization and personalization tags.

#### Non-Functional Requirements:
1. **Scale:** Handle 100 Million notifications daily.
2. **Peak Burst Tolerance:** Absorb marketing campaign bursts up to 25,000 requests/sec without crashing.
3. **Reliability & Zero Loss:** High-priority transactional notifications must never be dropped.
4. **Third-Party Fault Isolation:** If Twilio is down, email and push notifications must continue unimpeded.

---

### 8.2 Capacity Estimations

- **Total Volume:** 100 Million notifications per day.
  $$\text{Average Ingestion QPS} = \frac{100,000,000}{86,400} \approx 1,160 \text{ requests/sec}$$
  $$\text{Peak Ingestion QPS (Burst Campaign)} = 1,160 \times 10 \approx 11,600 \text{ requests/sec}$$
- **Channel Breakdown:**
  - Mobile Push: 60% (60M/day $\approx$ 700 QPS avg)
  - Email: 30% (30M/day $\approx$ 350 QPS avg)
  - SMS: 10% (10M/day $\approx$ 115 QPS avg)
- **Payload Size:** Average JSON notification event $\approx 1 \text{ KB}$.
  $$\text{Daily Storage} = 100 \times 10^6 \times 1 \text{ KB} \approx 100 \text{ GB/day}$$
  $$\text{3-Year Log Retention} = 100 \text{ GB} \times 365 \times 3 \approx 109.5 \text{ TB}$$

---

### 8.3 High-Level Architecture Diagram

```
                                 ARCHITECTURE: NOTIFICATION PLATFORM
                                 
+--------------------+    +--------------------+    +--------------------+
| Order Microservice |    | Auth Microservice  |    | Marketing Campaign |
| (High Priority)    |    | (High Priority OTP)|    | (Bulk Low Priority)|
+---------+----------+    +---------+----------+    +---------+----------+
          │                         │                         │
          └─────────────────────────┼─────────────────────────┘
                                    │
                                    v
          +───────────────────────────────────────────────────+
          |           API Gateway & Rate Limiter              |
          +─────────────────────────┬─────────────────────────+
                                    │
                                    v
          +───────────────────────────────────────────────────+
          |         Notification Ingestion Service            |
          |       - Idempotency Deduplication (Redis)         |
          |       - User Preference & Quiet Hour Filter       |
          |       - Template Engine Resolver                  |
          +─────────────────────────┬─────────────────────────+
                                    │
               ┌────────────────────┴────────────────────┐
               │ Routes by Priority & Channel            │
               ▼                                         ▼
+──────────────────────────────+          +──────────────────────────────+
| KAFKA TOPIC: High-Priority   |          | KAFKA TOPIC: Bulk-Priority   |
| (Partitions: sms, push, email|          | (Partitions: sms, push, email|
+──────────────┬───────────────+          +──────────────┬───────────────+
               │                                         │
               └────────────────────┬────────────────────┘
                                    │
       ┌────────────────────────────┼────────────────────────────┐
       ▼                            ▼                            ▼
+──────────────────────+   +──────────────────────+   +──────────────────────+
| Push Dispatch Worker |   | SMS Dispatch Worker  |   | Email Dispatch Worker|
| (Scales on Push lag) |   | (Scales on SMS lag)  |   | (Scales on Email lag)|
+──────────┬───────────+   +──────────┬───────────+   +──────────┬───────────+
           │                          │                          │
           v                          v                          v
    +--------------+           +--------------+           +--------------+
    | Apple APNs / |           | Twilio /     |           | SendGrid /   |
    | Google FCM   |           | Infobip      |           | AWS SES      |
    +--------------+           +------+-------+           +--------------+
                                      │ Failure / Timeout
                                      v
                               +--------------+
                               |  Retry DLQ   |
                               | (Exp Backoff)|
                               +--------------+
```

---

### 8.4 Data Flow Step-by-Step

1. **Ingestion:** Producer issues `POST /api/v1/notifications` with payload:
   ```json
   {
     "idempotency_key": "order_rec_9821_attempt_1",
     "user_id": "usr_4829104",
     "channel": "SMS",
     "priority": "HIGH",
     "template_id": "OTP_LOGIN",
     "parameters": { "code": "849201" }
   }
   ```
2. **Deduplication Check:** Ingestion service queries Redis: `SET notification:order_rec_9821_attempt_1 "PENDING" NX EX 86400`. If key exists, drop as duplicate and return `HTTP 200 OK` (Idempotent success).
3. **Preference Validation:** Ingestion service reads user preferences from Redis Cache (or fallback Postgres read replica):
   - Has the user unsubscribed from SMS?
   - Is it currently 11:30 PM local time for the user and priority is `LOW`? If quiet hours apply, schedule event in delayed execution queue or drop.
4. **Template Compilation:** Fetch template from memory cache, interpolate `parameters`, and assemble finalized message body.
5. **Topic Enqueuing:** Publish to dedicated Kafka topic partitioned by channel: `notifications.high.sms`.
6. **Worker Processing:** The SMS Dispatch Worker group consumes from `notifications.high.sms`:
   - Checks external provider rate limits (e.g., Twilio account limit of 100 SMS/sec) using Redis Token Bucket.
   - Dispatches payload via HTTP/2 POST to Twilio API.
7. **Delivery Receipt & Dead Letter Queue (DLQ):**
   - If Twilio returns `200 OK`: Write success record to Cassandra/Postgres log.
   - If Twilio returns `429 Too Many Requests` or `500 Internal Error`: Worker retries 3 times with exponential backoff and jitter. If retries exhaust, route to `notifications.sms.dlq` for operator investigation.

---

### 8.5 Database Schema Design

#### Relational Core (PostgreSQL):

```sql
-- User Channel Preferences & Quiet Hours
CREATE TABLE user_notification_preferences (
    user_id             UUID NOT NULL,
    channel             VARCHAR(20) NOT NULL, -- 'PUSH', 'SMS', 'EMAIL'
    is_enabled          BOOLEAN NOT NULL DEFAULT TRUE,
    quiet_hours_start   TIME NULL,            -- e.g. '22:00:00'
    quiet_hours_end     TIME NULL,            -- e.g. '07:00:00'
    timezone            VARCHAR(50) NOT NULL DEFAULT 'UTC',
    PRIMARY KEY (user_id, channel)
);

-- Template Store
CREATE TABLE notification_templates (
    template_id         VARCHAR(50) PRIMARY KEY,
    channel             VARCHAR(20) NOT NULL,
    language_code       CHAR(2) NOT NULL DEFAULT 'en',
    subject_template    VARCHAR(255) NULL,
    body_template       TEXT NOT NULL,
    version             INT NOT NULL DEFAULT 1,
    is_active           BOOLEAN NOT NULL DEFAULT TRUE
);
```

#### High-Volume Audit Log (Apache Cassandra / ScyllaDB):
Relational databases choke on 100M log inserts per day. Store immutable delivery tracking logs in Cassandra:

```sql
CREATE KEYSPACE notification_logs WITH replication = {
    'class': 'NetworkTopologyStrategy', 
    'us-east': 3
};

CREATE TABLE notification_logs.deliveries (
    user_id           uuid,
    notification_id   timeuuid,
    channel           text,
    priority          text,
    status            text, -- 'DISPATCHED', 'DELIVERED', 'FAILED', 'BOUNCED'
    provider          text, -- 'TWILIO', 'SENDGRID', 'FCM'
    error_message     text,
    dispatched_at     timestamp,
    PRIMARY KEY ((user_id), notification_id)
) WITH CLUSTERING ORDER BY (notification_id DESC);
```
*Partitioning by `user_id` allows instant retrieval of a user's recent notification history on a single cluster node.*

---

### 8.6 Component Deep Dive

#### 8.6.1 Priority Ingestion & Queue Isolation
A critical failure mode of naive notification systems is mixing bulk marketing campaigns with transactional OTP codes. If marketing launches a 5,000,000-user blast, single-queue architectures cause login OTP codes to sit behind 4 million marketing emails!

**Architectural Isolation:**
- **Separate Queues/Topics:** Physically partition brokers into `queue.high_priority` and `queue.bulk`.
- **Worker Allocation:** Allocate 80% of consumer instances to high-priority topics with aggressive autoscaling. Bulk consumers operate at a capped, steady rate to avoid saturating downstream email providers.

#### 8.6.2 Token Bucket Rate Limiting per Channel in .NET

```csharp
public class DistributedTokenBucketRateLimiter
{
    private readonly IDatabase _redis;
    private const string TokenBucketScript = @"
        local key = KEYS[1]
        local limit = tonumber(ARGV[1])
        local current = tonumber(redis.call('get', key) or '0')
        if current + 1 > limit then
            return 0
        else
            redis.call('incrby', key, 1)
            if current == 0 then
                redis.call('expire', key, tonumber(ARGV[2]))
            end
            return 1
        end";

    public DistributedTokenBucketRateLimiter(IConnectionMultiplexer redis)
    {
        _redis = redis.GetDatabase();
    }

    public async Task<bool> AllowRequestAsync(string rateLimitKey, int maxPerSecond)
    {
        long currentSecond = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
        string key = $"ratelimit:{rateLimitKey}:{currentSecond}";

        var result = (int)await _redis.ScriptEvaluateAsync(
            TokenBucketScript,
            new RedisKey[] { key },
            new RedisValue[] { maxPerSecond, 1 });

        return result == 1;
    }
}
```

---

### 8.7 Scalability & Caching Strategy
1. **User Preference Cache:** Cache preference profiles in Redis under key `pref:{user_id}` with a 24-hour TTL. Invalidate on profile update via pub/sub.
2. **Template Caching:** Cache compiled Liquid/Handlebars templates in application memory (`IMemoryCache`). Templates rarely change; reload via configuration hot-reload or distributed cache bus.
3. **Batching:** Workers dispatch push notifications to APNs/FCM in bulk batches of 500 tokens per HTTP/2 request to maximize network pipe efficiency.

---

### 8.8 Production Trade-offs & Failure Scenarios
- **Scenario A: Primary SMS Provider (Twilio) Experiences Outage:**
  - *Mitigation:* Implement the **Circuit Breaker Pattern (Polly)** combined with **Multi-Vendor Fallback**. When Twilio error rate crosses 25%, trip the circuit breaker and dynamically reroute SMS traffic to secondary provider (Infobip / AWS SNS).
- **Scenario B: Upstream Campaign Producer Spams Millions of Identical Messages:**
  - *Mitigation:* Sliding window rate limiter enforces per-user receiving quotas (e.g., maximum 3 marketing emails per 24-hour rolling window). Excess messages are silently dropped with a `QUOTA_EXCEEDED` audit entry.
- **Scenario C: Worker Process Crashes Mid-Transmission:**
  - *Mitigation:* Kafka consumer offsets are committed **only after** receiving an acknowledgement from the provider API. If worker dies, partition rebalances to a live worker, ensuring at-least-once delivery. Idempotency keys prevent double sending.


---

## 9. FULL DESIGN: HIGH-THROUGHPUT IDEMPOTENT PAYMENT SYSTEM {#9-full-design-payment-system}

A payment platform processes money transfers between buyers, merchants, and banking networks. Unlike social networks where eventual consistency is acceptable, financial payment systems require **strict consistency, zero data loss, exact-once processing semantics (idempotency), and regulatory compliance (PCI-DSS)**.

### 9.1 Requirements & Scope Bounding

#### Functional Requirements:
1. **Two-Phase Payment Flow:** Support **Authorize** (hold funds on customer account) and **Capture** (settle funds upon order fulfillment) workflows.
2. **Strict Idempotency:** Guarantee that identical client requests (caused by network retries or double-clicks) never double-charge a customer.
3. **Double-Entry Ledger:** Every financial movement must be recorded with balanced Debit and Credit entries; money cannot be created or destroyed.
4. **Reconciliation Engine:** Asynchronously match internal payment records against external payment gateway settlement files (Stripe/Adyen/JPMorgan) and bank statements.
5. **PCI-DSS Compliance:** The system must never store or process raw Primary Account Numbers (PAN) or CVVs directly on application servers.

#### Non-Functional Requirements:
1. **Zero Data Loss ($RPO = 0$):** Financial transactions written to the ledger must survive infrastructure failures without loss.
2. **High Availability:** 99.999% availability for payment authorization endpoints.
3. **Throughput:** Support 5,000 TPS baseline, peaking at 15,000 TPS during global sales events.
4. **Sub-Second Latency:** P99 payment authorization $< 800\text{ms}$ (bounded by upstream card network latency).

---

### 9.2 Capacity Estimations

- **Transaction Volume:** 100 Million payment transactions per day.
  $$\text{Average TPS} = \frac{100,000,000}{86,400} \approx 1,160 \text{ TPS}$$
  $$\text{Peak TPS (Flash Sale / Black Friday)} = 1,160 \times 10 \approx 11,600 \text{ TPS}$$
- **Data Footprint:**
  - Double-entry ledger creates 2 ledger entries per transaction (Debit customer account, Credit merchant holding account).
  - Average ledger row size: 300 bytes.
  $$\text{Daily Ledger Data} = 100 \times 10^6 \times 2 \times 300 \text{ bytes} \approx 60 \text{ GB/day}$$
  $$\text{5-Year Ledger Retention (Regulatory Mandate)} = 60 \text{ GB} \times 365 \times 5 \approx 109.5 \text{ TB}$$
- **Cache Memory:**
  - Active transaction idempotency keys cached for 24 hours:
  $$\text{RAM for Idempotency Keys} = 100 \times 10^6 \times 128 \text{ bytes} \approx 12.8 \text{ GB RAM}$$

---

### 9.3 High-Level Architecture Diagram

```
                                  ARCHITECTURE: PAYMENT SYSTEM
                                  
 [Client App / Checkout]
         │ 1. Collects raw card details
         │ 2. Sends directly to Token Vault (Stripe Elements / Vault SDK)
         ▼
+─────────────────────────────────+
| External Tokenization Vault     | ──> Returns ephemeral card token: "tok_visa_8921a"
+─────────────────────────────────+
         │
         │ 3. POST /api/v1/payments/authorize
         │    Header: Idempotency-Key: "uuid-9a8b-7c6d"
         ▼
+─────────────────────────────────────────────────────────────────+
|               API Gateway / YARP (SSL Offload & WAF)            |
+────────────────────────────────┬────────────────────────────────+
                                 │
                                 v
+─────────────────────────────────────────────────────────────────+
|                    Payment Orchestrator Service                 |
|  +───────────────────────────────────────────────────────────+  |
|  | Idempotency Filter (Redis Key Check & Distributed Lock)   |  |
|  +───────────────────────────────────────────────────────────+  |
|  | Payment State Machine (CREATED -> AUTHORIZED -> CAPTURED) |  |
|  +───────────────────────────────────────────────────────────+  |
+───────────────┬───────────────────────────────┬─────────────────+
                │                               │
       Database │ Sync                          │ Async Outbox
       Commit   │ Transaction                   │ Events
                v                               v
+───────────────────────────────+   +─────────────────────────────+
| PostgreSQL Primary Database   |   | Kafka Topic: payment-events |
| - Transactions Table          |   +──────────────┬──────────────+
| - Double-Entry Ledger Table   |                  │
| - Idempotency Keys Table      |                  v
+───────────────────────────────+   +─────────────────────────────+
                │                   | Reconciliation Engine       |
                │                   | (Matches SFTP Settlement    |
                v                   |  Batch Files against Ledger)|
+───────────────────────────────+   +─────────────────────────────+
| Upstream Payment Gateway      |
| (Stripe / Adyen / Chase / PSP)|
+-------------------------------+
```

---

### 9.4 Data Flow Step-by-Step

#### Phase 1: Payment Authorization Flow:
1. **Browser Tokenization:** The customer enters card details into an isolated iframe hosted by the PCI-certified token vault. The vault returns a non-sensitive token (`tok_visa_8921a`).
2. **Client Submission:** Checkout service issues `POST /api/v1/payments/authorize` with token, amount ($150.00), order ID, and `Idempotency-Key: c9b2f481-9871-4a21`.
3. **Idempotency Guard:**
   - The Payment Orchestrator attempts to atomically acquire a lock in Redis: `SETNX lock:pay:c9b2f481 "PROCESSING" EX 60`.
   - If key already exists in PostgreSQL with status `COMPLETED`, return the cached authorization response immediately.
4. **State Machine Transition:** Insert record in PostgreSQL `payment_transactions` with status `AUTHORIZING`.
5. **Gateway Invocation:** Payment Orchestrator calls upstream Payment Service Provider (PSP) API (e.g., Stripe/Adyen) via gRPC/HTTPS with card token and internal payment reference ID.
6. **Synchronous Settlement Hold:** PSP contacts Card Network (Visa/Mastercard) and issuing bank. Issuing bank holds $150 on customer card and returns `auth_code: AUTH_774921`.
7. **Atomic Ledger Commit:**
   Within a single PostgreSQL serializable transaction:
   - Update `payment_transactions` status to `AUTHORIZED`.
   - Insert Authorize record into `idempotency_keys` table.
   - Insert transactional outbox message `PaymentAuthorizedEvent`.
8. **Lock Release:** Release Redis distributed lock and return `HTTP 200 OK` with `{ "status": "AUTHORIZED", "payment_id": "pay_01829" }`.

#### Phase 2: Payment Capture Flow (Triggered when Warehouse ships goods):
1. Order Fulfillment service emits `OrderDispatchedEvent`.
2. Payment Orchestrator triggers `POST /api/v1/payments/{payment_id}/capture`.
3. Calls PSP to capture previously held authorization.
4. Issuing bank settles funds to merchant acquiring bank.
5. Double-entry ledger records Debit to Settlement Clearing Account and Credit to Merchant Revenue Account.

---

### 9.5 Database Schema & Double-Entry Ledger Design

#### Double-Entry Accounting Rule:
$$\sum \text{Debits} = \sum \text{Credits}$$
Money never appears out of nowhere. Every transaction must reference two accounts: a Source and a Destination.

```sql
-- Core Payment Transaction State Table
CREATE TABLE payment_transactions (
    payment_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id            VARCHAR(64) NOT NULL,
    customer_id         UUID NOT NULL,
    amount              NUMERIC(18, 4) NOT NULL,
    currency            CHAR(3) NOT NULL, -- 'USD', 'EUR', 'GBP'
    status              VARCHAR(30) NOT NULL, -- 'CREATED', 'AUTHORIZED', 'CAPTURED', 'FAILED', 'REFUNDED'
    psp_reference       VARCHAR(100) NULL,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    version             INT NOT NULL DEFAULT 1
);

CREATE UNIQUE INDEX uq_payment_order_id ON payment_transactions (order_id);

-- Idempotency Guard Table
CREATE TABLE idempotency_records (
    idempotency_key     VARCHAR(128) PRIMARY KEY,
    request_hash        CHAR(64) NOT NULL, -- SHA-256 of request payload
    payment_id          UUID REFERENCES payment_transactions(payment_id),
    response_code       INT NOT NULL,
    response_body       JSONB NOT NULL,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Financial Double-Entry Ledger Accounts
CREATE TABLE ledger_accounts (
    account_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_number      VARCHAR(50) UNIQUE NOT NULL,
    account_type        VARCHAR(30) NOT NULL, -- 'ASSET', 'LIABILITY', 'EQUITY', 'REVENUE', 'EXPENSE'
    currency            CHAR(3) NOT NULL,
    balance             NUMERIC(18, 4) NOT NULL DEFAULT 0.0000,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Immutable Ledger Entries (Append-Only)
CREATE TABLE ledger_entries (
    entry_id            BIGSERIAL PRIMARY KEY,
    payment_id          UUID NOT NULL REFERENCES payment_transactions(payment_id),
    account_id          UUID NOT NULL REFERENCES ledger_accounts(account_id),
    entry_type          VARCHAR(6) NOT NULL, -- 'DEBIT' or 'CREDIT'
    amount              NUMERIC(18, 4) NOT NULL CHECK (amount > 0),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX ix_ledger_payment_id ON ledger_entries (payment_id);
```

---

### 9.6 Component Deep Dive

#### 9.6.1 Idempotency Key Handling in .NET
To guarantee exactly-once processing, build an enterprise-grade ASP.NET Core Action Filter that intercepts requests carrying `Idempotency-Key` headers:

```csharp
public class IdempotentPaymentAttribute : Attribute, IAsyncActionFilter
{
    private readonly IDatabase _redis;
    private readonly IPaymentDbContext _dbContext;

    public IdempotentPaymentAttribute(IConnectionMultiplexer redis, IPaymentDbContext dbContext)
    {
        _redis = redis.GetDatabase();
        _dbContext = dbContext;
    }

    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        if (!context.HttpContext.Request.Headers.TryGetValue("Idempotency-Key", out var rawKey) ||
            string.IsNullOrWhiteSpace(rawKey))
        {
            context.Result = new BadRequestObjectResult(new { error = "Missing Idempotency-Key header." });
            return;
        }

        string idempotencyKey = rawKey.ToString();
        string lockKey = $"idemp_lock:{idempotencyKey}";

        // 1. Check if request was already completed previously
        var existingRecord = await _dbContext.IdempotencyRecords
            .AsNoTracking()
            .FirstOrDefaultAsync(r => r.IdempotencyKey == idempotencyKey);

        if (existingRecord != null)
        {
            // Return identical cached response directly without re-executing payment
            context.Result = new ContentResult
            {
                StatusCode = existingRecord.ResponseCode,
                ContentType = "application/json",
                Content = existingRecord.ResponseBody
            };
            return;
        }

        // 2. Acquire Redis distributed lock to prevent concurrent double-clicks
        bool acquired = await _redis.StringSetAsync(lockKey, "LOCKED", TimeSpan.FromSeconds(30), When.NotExists);
        if (!acquired)
        {
            // Another thread is actively processing this exact payment attempt right now
            context.Result = new ConflictObjectResult(new { error = "Payment request is already processing." });
            return;
        }

        try
        {
            var executedContext = await next();

            if (executedContext.Result is ObjectResult objectResult && objectResult.StatusCode is >= 200 and < 300)
            {
                // 3. Persist success response for future retries
                var record = new IdempotencyRecord
                {
                    IdempotencyKey = idempotencyKey,
                    ResponseCode = objectResult.StatusCode.Value,
                    ResponseBody = JsonSerializer.Serialize(objectResult.Value),
                    CreatedAt = DateTime.UtcNow
                };
                _dbContext.IdempotencyRecords.Add(record);
                await _dbContext.SaveChangesAsync();
            }
        }
        finally
        {
            await _redis.KeyDeleteAsync(lockKey);
        }
    }
}
```

#### 9.6.2 The Distributed Saga Pattern for Payments
A payment often spans multiple bounded contexts: **Payment Orchestrator**, **Customer Loyalty Points**, **Inventory**, and **Fraud Engine**. Because distributed two-phase locking (2PC) does not scale across microservices, use an **Orchestrated Saga**:

```
                       PAYMENT SAGA ORCHESTRATION
                       
                    +─────────────────────────+
                    | Payment Saga Coordinator|
                    +────────────┬────────────+
                                 │
           ┌─────────────────────┼─────────────────────┐
           │ 1. Hold Funds       │ 2. Deduct Points    │ 3. Reserve Stock
           ▼                     ▼                     ▼
    +──────────────+      +──────────────+      +──────────────+
    | Card Gateway |      | Loyalty Svc  |      | Inventory Svc|
    +──────────────+      +──────────────+      +──────┬───────+
           │                     │                     │
           │ Success             │ Success             │ FAILS (Out of Stock!)
           └─────────────────────┼─────────────────────┘
                                 │
                                 v
               +───────────────────────────────────+
               | COMPENSATING TRANSACTIONS FIRED   |
               | - Release Card Hold (Void Auth)   |
               | - Refund Loyalty Points           |
               +───────────────────────────────────+
```

---

### 9.7 Scalability & Caching Strategy
1. **PostgreSQL Connection Pooling with PgBouncer:** Payment databases experience massive concurrent spikes. Placing **PgBouncer** in transaction pooling mode prevents PostgreSQL connection exhaustion.
2. **Zero In-Memory Caching of Balances:** Never cache customer account balances or ledger totals in Redis. Balances must be queried strictly from the relational database using pessimistic locking (`SELECT ... FOR UPDATE`) or calculated from ledger sums to prevent dirty reads.
3. **Partitioning Ledger Entries:** Range-partition `ledger_entries` by `created_at` (monthly partitions). Old partitions are detached and archived to cold S3 Parquet tables for long-term historical analysis.

---

### 9.8 Production Trade-offs & Failure Scenarios
- **Scenario A: Network Timeout During Gateway Call (Did the card get charged?):**
  - *Problem:* Payment service sends authorization request to Stripe. Network cable drops before response returns. Did the customer get charged?
  - *Resolution:* Never assume failure! The orchestrator triggers an asynchronous **Payment Status Inquiry Job** using the shared reference ID. It polls Stripe's `/v1/charges/{ref_id}` endpoint. If Stripe confirms the charge, mark internal transaction as `AUTHORIZED`. If Stripe confirms no charge, retry or mark `FAILED`.
- **Scenario B: Nightly Reconciliation Discrepancy (Ghost Transaction):**
  - *Mechanism:* Every night at 02:00 UTC, PSP delivers a signed settlement CSV file via secure SFTP. The Reconciliation Engine compares internal ledger transactions against the PSP settlement file. If a transaction appears in the bank statement but not in our database (Ghost Charge), the engine automatically flags the record, creates an investigation ticket, and initiates an automated refund.
- **Scenario C: Database Primary Node Hard Crash Mid-Transaction:**
  - *Mitigation:* Deploy PostgreSQL with synchronous replication across three Availability Zones (Quorum Commit). A commit is acknowledged to the application only after being written to the WAL of at least two AZs, guaranteeing zero financial data loss.

---

## 10. FULL DESIGN: E-COMMERCE ORDER MANAGEMENT SYSTEM (OMS) {#10-full-design-order-management-system}

An Order Management System (OMS) orchestrates the lifecycle of customer orders from checkout to delivery. During high-concurrency flash sales (e.g., 50,000 units of a flagship smartphone sold in 10 seconds), the OMS must prevent **overselling**, maintain **deterministic order state**, and guarantee **eventual consistency across decoupled microservices** using the Transactional Outbox pattern.

### 10.1 Requirements & Scope Bounding

#### Functional Requirements:
1. **High-Speed Inventory Reservation:** Reserve stock atomically during checkout. Expire and release reservations if payment is not completed within 15 minutes.
2. **Order Lifecycle State Machine:** Maintain deterministic transitions: `CREATED` $\to$ `INVENTORY_RESERVED` $\to$ `PAYMENT_PENDING` $\to$ `PAID` $\to$ `PROCESSING` $\to$ `SHIPPED` $\to$ `DELIVERED`.
3. **Transactional Outbox Pattern:** Atomically persist order state changes and publish domain events to Apache Kafka without distributed 2PC transactions.
4. **Order History Audit Trail:** Provide an append-only event-sourced log of every action taken on an order for customer support.

#### Non-Functional Requirements:
1. **Concurrency & Zero Overselling:** Strictly prevent selling more items than available in physical warehouse inventory.
2. **Throughput:** Sustain 50,000 reservation requests per second during peak flash sales.
3. **Auditability:** Complete historical traceability of all state transitions.

---

### 10.2 Capacity Estimations

- **Flash Sale Concurrency:** 100,000 active shoppers competing for 10,000 available items.
  $$\text{Peak Write QPS (Checkout)} \approx 25,000 \text{ requests/sec}$$
- **Daily Orders:** 5 Million orders created daily on average.
- **Order Payload Size:** Complete order aggregate (items, shipping address, discounts) $\approx 2 \text{ KB}$.
  $$\text{Daily Order Storage} = 5 \times 10^6 \times 2 \text{ KB} \approx 10 \text{ GB/day}$$
  $$\text{5-Year Order Storage} = 10 \text{ GB} \times 365 \times 5 \approx 18.25 \text{ TB}$$

---

### 10.3 High-Level Architecture Diagram

```
                                  ARCHITECTURE: ORDER MANAGEMENT
                                  
  [Shopper App]
        │ POST /api/v1/orders/checkout
        ▼
+─────────────────────────────────────────────────────────────+
|               API Gateway / Rate Limiter                    |
+──────────────────────────────┬──────────────────────────────+
                               │
                               v
+─────────────────────────────────────────────────────────────+
|                    Order Ingestion API                      |
+──────────────────────────────┬──────────────────────────────+
                               │
        1. Atomic Lua Script   v
+─────────────────────────────────────────────────────────────+
|             Redis Flash-Inventory Cluster                   |
|  - Key: "stock:sku_9012"                                    |
|  - Value: 5000 (Atomically decremented via Lua)             |
|  - Key: "reservation:order_123" -> Expires in 15 mins       |
+──────────────────────────────┬──────────────────────────────+
                               │
        2. Stock Reserved OK   v
+─────────────────────────────────────────────────────────────+
|                  Order Processing Service                   |
|                                                             |
|   BEGIN TRANSACTION;                                        |
|     INSERT INTO orders (id, status, ...);                   |
|     INSERT INTO outbox_messages (event_type, payload, ...); |
|   COMMIT;                                                   |
+──────────────────────────────┬──────────────────────────────+
                               │
                               v
+─────────────────────────────────────────────────────────────+
|                 PostgreSQL Primary Database                 |
+──────────────────────────────┬──────────────────────────────+
                               │
        3. Transaction Log CDC │ (Debezium / Kafka Connect)
                               v
+─────────────────────────────────────────────────────────────+
|                 Apache Kafka Event Backbone                 |
|             Topic: "order-lifecycle-events"                 |
+──────────────┬──────────────────────────────┬───────────────+
               │                              │
               v                              v
+──────────────────────────────+  +───────────────────────────+
| Payment Orchestrator Worker  |  | Inventory Sync Worker     |
| (Consumes OrderCreatedEvent; |  | (Reconciles Postgres      |
|  initiates payment capture)  |  |  warehouse tables)        |
+──────────────────────────────+  +───────────────────────────+
```

---

### 10.4 Data Flow Step-by-Step

1. **Checkout Submission:** Shopper submits cart: `{ "customer_id": "u1", "sku": "IPHONE_16", "quantity": 1 }`.
2. **Atomic Inventory Reservation (Redis):**
   - The Order Ingestion API executes an atomic Lua script against Redis.
   - If stock $\ge 1$, Redis decrements available inventory and creates an ephemeral reservation key with a 15-minute TTL.
   - If stock $< 1$, the API immediately returns `HTTP 409 Conflict: Sold Out`.
3. **Local Database Transaction (Outbox Pattern):**
   - In a single local PostgreSQL database transaction:
     a. Insert order record with status `PAYMENT_PENDING`.
     b. Insert event `OrderCreated` into `outbox_messages` table.
   - Database commits in $< 5\text{ms}$.
4. **Event Streaming via Change Data Capture (CDC):**
   - **Debezium** tails the PostgreSQL Write-Ahead Log (WAL) and streams the `OrderCreated` event into Kafka topic `orders.events`.
5. **Payment Processing:**
   - The Payment Service consumes `OrderCreated` and attempts to charge the customer's card.
   - **Payment Succeeded:** Emits `PaymentCapturedEvent`. OMS updates status to `PAID`.
   - **Payment Failed / Timed Out:** Emits `PaymentFailedEvent`. OMS updates status to `CANCELLED` and executes compensating Lua script on Redis to return reserved stock back to the pool.

---

### 10.5 Database Schema & Outbox Design

```sql
-- Core Orders Table
CREATE TABLE orders (
    order_id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id         UUID NOT NULL,
    total_amount        NUMERIC(12, 2) NOT NULL,
    currency            CHAR(3) NOT NULL DEFAULT 'USD',
    status              VARCHAR(30) NOT NULL, -- 'CREATED', 'PAYMENT_PENDING', 'PAID', 'CANCELLED', 'SHIPPED'
    shipping_address    JSONB NOT NULL,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    version             INT NOT NULL DEFAULT 1
);

CREATE INDEX ix_orders_customer_id ON orders (customer_id);

-- Order Items
CREATE TABLE order_items (
    item_id             BIGSERIAL PRIMARY KEY,
    order_id            UUID NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    sku                 VARCHAR(50) NOT NULL,
    unit_price          NUMERIC(12, 2) NOT NULL,
    quantity            INT NOT NULL CHECK (quantity > 0)
);

-- Transactional Outbox Table
CREATE TABLE outbox_messages (
    message_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type      VARCHAR(50) NOT NULL, -- 'Order'
    aggregate_id        VARCHAR(64) NOT NULL, -- order_id
    event_type          VARCHAR(100) NOT NULL, -- 'OrderCreated', 'OrderCancelled'
    payload             JSONB NOT NULL,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at        TIMESTAMPTZ NULL
);

CREATE INDEX ix_outbox_unprocessed ON outbox_messages (created_at) 
WHERE processed_at IS NULL;
```

---

### 10.6 Component Deep Dive

#### 10.6.1 Flash Sale Inventory Reservation (Redis Lua Script)
Relational `SELECT FOR UPDATE` locks rows and kills DB connection pools under 50,000 QPS. Execute inventory deduction in Redis atomically using Lua:

```csharp
public class FlashInventoryService
{
    private readonly IDatabase _redis;
    private const string ReserveStockScript = @"
        local stockKey = KEYS[1]
        local orderReserveKey = KEYS[2]
        local quantity = tonumber(ARGV[1])
        local ttlSeconds = tonumber(ARGV[2])

        local currentStock = tonumber(redis.call('get', stockKey) or '0')
        if currentStock >= quantity then
            redis.call('decrby', stockKey, quantity)
            redis.call('set', orderReserveKey, quantity, 'EX', ttlSeconds)
            return 1 -- Success
        else
            return 0 -- Insufficient stock
        end";

    public FlashInventoryService(IConnectionMultiplexer redis)
    {
        _redis = redis.GetDatabase();
    }

    public async Task<bool> TryReserveInventoryAsync(string sku, string orderId, int quantity, int timeoutSeconds = 900)
    {
        string stockKey = $"stock:{sku}";
        string reserveKey = $"reserve:{sku}:{orderId}";

        var result = (int)await _redis.ScriptEvaluateAsync(
            ReserveStockScript,
            new RedisKey[] { stockKey, reserveKey },
            new RedisValue[] { quantity, timeoutSeconds });

        return result == 1;
    }
}
```

#### 10.6.2 Polling Outbox Publisher in C# (.NET BackgroundService)
If Change Data Capture (Debezium) is unavailable, implement a reliable polling publisher using PostgreSQL's `FOR UPDATE SKIP LOCKED` primitive to prevent lock contention across multiple worker replicas:

```csharp
public class OutboxPublisherWorker : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly IProducer<string, string> _kafkaProducer;

    public OutboxPublisherWorker(IServiceProvider serviceProvider, IProducer<string, string> kafkaProducer)
    {
        _serviceProvider = serviceProvider;
        _kafkaProducer = kafkaProducer;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _serviceProvider.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<OrderDbContext>();

            // Lock and fetch 100 unprocessed messages, skipping rows locked by peer workers
            var messages = await db.OutboxMessages
                .FromSqlRaw(@"
                    SELECT * FROM outbox_messages 
                    WHERE processed_at IS NULL 
                    ORDER BY created_at ASC 
                    LIMIT 100 
                    FOR UPDATE SKIP LOCKED")
                .ToListAsync(stoppingToken);

            if (messages.Count == 0)
            {
                await Task.Delay(200, stoppingToken);
                continue;
            }

            foreach (var message in messages)
            {
                await _kafkaProducer.ProduceAsync("orders.events", new Message<string, string>
                {
                    Key = message.AggregateId,
                    Value = message.Payload
                }, stoppingToken);

                message.ProcessedAt = DateTime.UtcNow;
            }

            await db.SaveChangesAsync(stoppingToken);
        }
    }
}
```

---

### 10.7 Scalability & Caching Strategy
1. **Redis Master-Replica with Sentinel:** Inventory reservations run strictly against the Redis Master node (in-memory atomic Lua operations).
2. **Order Read Query Caching:** Orders in `PROCESSING`, `SHIPPED`, or `DELIVERED` status are immutable. Cache them aggressively in Redis with a 7-day TTL; invalidate only on customer return/refund events.
3. **Database Sharding on `customer_id`:** Shard order databases across PostgreSQL nodes using `customer_id`. This guarantees all orders for a specific user reside on a single shard, making "My Orders" listing queries sub-millisecond without cross-shard joins.

---

### 10.8 Production Trade-offs & Failure Scenarios
- **Scenario A: Customer Abandons Cart (Reserved Stock Left Hanging):**
  - *Mitigation:* The 15-minute TTL on the Redis reservation key expires. A background reconciliation worker or Redis Key Expiration Keyspace Notification triggers a compensatory `INCRBY stock:sku quantity` to return the reserved units back to available inventory.
- **Scenario B: Kafka Broker Outage:**
  - *Mitigation:* Because the Transactional Outbox stores events inside PostgreSQL tables as part of the ACID checkout transaction, **zero events are lost**. When Kafka recovers, the `OutboxPublisherWorker` resumes publishing from the exact point it paused.


---

## 11. FULL DESIGN: REAL-TIME CHAT & COLLABORATION SYSTEM {#11-full-design-real-time-chat-system}

Modern real-time communication platforms (Slack, WhatsApp, Microsoft Teams) support billions of instant messages daily across 1:1 direct conversations and high-concurrency group channels. The system must maintain persistent duplex network connections, provide sub-100ms message delivery, track online presence, handle offline push notifications, and persist message histories at planetary scale.

### 11.1 Requirements & Scope Bounding

#### Functional Requirements:
1. **1:1 and Group Messaging:** Low-latency bi-directional messaging with support for group channels up to 1,000 members.
2. **Online Presence Engine:** Real-time visibility into user status (`ONLINE`, `OFFLINE`, `AWAY`, `LAST_SEEN`).
3. **Delivery & Read Receipts:** Sent ($\checkmark$), Delivered ($\checkmark\checkmark$), and Read ($\checkmark\checkmark$ blue) receipts.
4. **Offline Message Queuing:** If a recipient is offline, buffer messages and dispatch push notifications; deliver queued messages instantly upon reconnection.
5. **Message History Synchronization:** Fast, paginated retrieval of conversation history across multi-device clients.

#### Non-Functional Requirements:
1. **Scale:** 50 Million Daily Active Users (DAU), 10 Million peak concurrent persistent connections.
2. **Latency:** P99 message delivery latency $< 100\text{ms}$ between online users.
3. **Message Ordering:** Strict temporal ordering within a single conversation channel.
4. **Durability:** Zero dropped messages once acknowledged by the ingestion gateway.

---

### 11.2 Capacity Estimations

- **Active User Connections:**
  - 10 Million peak concurrent WebSocket connections.
  - Memory per open WebSocket connection in optimized Kestrel server $\approx 10 \text{ KB}$.
  $$\text{RAM for 10M Concurrent WebSockets} = 10 \times 10^6 \times 10 \text{ KB} \approx 100 \text{ GB RAM}$$
  *(Can be easily distributed across 10 to 15 gateway servers with 16 GB RAM each).*

- **Message Throughput:**
  - 50M DAU $\times$ 40 messages/day = 2 Billion messages/day.
  $$\text{Average QPS} = \frac{2,000,000,000}{86,400} \approx 23,150 \text{ messages/sec}$$
  $$\text{Peak QPS (New Year / Crisis Events)} \approx 23,150 \times 3 \approx 70,000 \text{ messages/sec}$$

- **Storage Footprint (5-Year Horizon):**
  - Average text message payload (metadata + text) $\approx 200 \text{ bytes}$.
  $$\text{Daily Message Storage} = 2 \times 10^9 \times 200 \text{ bytes} \approx 400 \text{ GB/day}$$
  $$\text{5-Year Storage} = 400 \text{ GB} \times 365 \times 5 \approx 730 \text{ TB}$$

---

### 11.3 High-Level Architecture Diagram

```
                                 ARCHITECTURE: REAL-TIME CHAT
                                 
   [Client Alice]                                            [Client Bob]
         │                                                         ▲
         │ WSS (WebSocket)                                         │ WSS (WebSocket)
         ▼                                                         │
+──────────────────────────────────+                     +──────────────────────────────────+
| Chat Gateway Cluster (Node A)    |                     | Chat Gateway Cluster (Node B)    |
| (ASP.NET Core WebSockets/SignalR)|                     | (ASP.NET Core WebSockets/SignalR)|
+─────────────────┬────────────────+                     +─────────────────▲────────────────+
                  │                                                        │
                  │ 1. Forward Message                                     │ 5. Push to Bob's
                  ▼                                                        │    Socket
+───────────────────────────────────────────────────+                      │
|                Message Router Service             |                      │
+────────┬─────────────────────────┬────────────────+                      │
         │                         │                                       │
         │ 2. Lookup               │ 3. Enqueue                            │
         v    Connection           v                                       │
+─────────────────+      +─────────────────────+                           │
| Redis Session   |      | Kafka Topic:        |                           │
| & Presence Store|      | "chat-messages"     |                           │
| (User -> NodeB) |      +──────────┬──────────+                           │
+─────────────────+                 │                                      │
                                    v                                      │
                         +─────────────────────+                           │
                         | Group Fanout Worker | ──────────────────────────┘
                         +──────────┬──────────+
                                    │
                                    v 4. Async Persistence
                         +─────────────────────+
                         | Apache Cassandra /  |
                         | ScyllaDB Message DB |
                         +─────────────────────+
```

---

### 11.4 Data Flow Step-by-Step

#### 1:1 Direct Message Flow:
1. **Connection Establishment:** Client Alice connects to `wss://chat.domain.com/ws`. The L4 load balancer routes her to **Chat Gateway Node A**. Node A registers Alice's connection in the **Redis Session Store**: `HSET session:users usr_alice "gateway_node_a"`.
2. **Sending a Message:** Alice sends JSON packet over WebSocket:
   ```json
   {
     "client_msg_id": "c-9812-a1",
     "conversation_id": "conv_alice_bob",
     "recipient_id": "usr_bob",
     "text": "Hey Bob, review this architecture PR."
   }
   ```
3. **Ordering & ID Generation:** Node A generates a monotonically increasing, time-sortable 64-bit ID using **Twitter Snowflake** algorithm.
4. **Recipient Session Lookup:** Message Router queries Redis: `HGET session:users usr_bob`.
   - **Case 1: Bob is Online (connected to Node B):** The router publishes the message to Redis Pub/Sub channel `channel:gateway_node_b`. Gateway Node B picks up the message and pushes it directly down Bob's open WebSocket connection.
   - **Case 2: Bob is Offline:** Message Router publishes a notification event to Kafka `notifications.push`, triggering an Apple Push Notification (APNs) / FCM message to wake up Bob's mobile device.
5. **Persistence:** The message payload is asynchronously written to the **Apache Cassandra** cluster.
6. **Delivery Acknowledgment:** Node A sends an immediate ACK back to Alice: `{ "client_msg_id": "c-9812-a1", "server_msg_id": 182947192847192, "status": "SENT" }`.

---

### 11.5 Database Schema Design

#### Message Persistence Store (Apache Cassandra / ScyllaDB):
Relational databases collapse when handling 70,000 writes/sec and billions of chat messages. Wide-column stores are built for sequential write-heavy append logs:

```sql
CREATE KEYSPACE enterprise_chat WITH replication = {
    'class': 'NetworkTopologyStrategy', 
    'us-east': 3, 
    'eu-west': 3
};

CREATE TABLE enterprise_chat.messages (
    conversation_id   uuid,
    message_id        bigint, -- Snowflake ID (encodes millisecond timestamp)
    sender_id         uuid,
    message_type      text,   -- 'TEXT', 'IMAGE', 'FILE'
    content           text,
    attachment_url    text,
    created_at        timestamp,
    PRIMARY KEY ((conversation_id), message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```
- **Partition Key (`conversation_id`):** Guarantees that all messages within a single chat channel live on the same physical Cassandra partition.
- **Clustering Key (`message_id DESC`):** Messages are physically sorted on disk in reverse chronological order. Fetching the last 50 messages (`SELECT * FROM messages WHERE conversation_id = ? LIMIT 50`) is a single sequential disk scan with zero in-memory sorting.

---

### 11.6 Component Deep Dive

#### 11.6.1 Real-Time Online Presence Engine
Tracking online presence for 50M users cannot use persistent database writes. Use **Redis Key TTL Heartbeats**:

```
                                PRESENCE HEARTBEAT FLOW
                                
   [Mobile Client] ────(Ping every 30s)────> [Presence Service]
                                                     │
                                                     │ SET presence:usr_bob "ONLINE" EX 60
                                                     v
                                            +──────────────────+
                                            |  Redis Cluster   |
                                            +──────────────────+
                                                     │
                                                     │ If key expires (no ping for 60s)
                                                     v
                                            Trigger OFFLINE event
```

1. **Heartbeat Protocol:** Client sends a lightweight ping packet over WebSocket every 30 seconds.
2. **Redis TTL Storage:** Presence Service executes `SET presence:{user_id} "ONLINE" EX 60`.
3. **Status Query:** To display Bob's status to Alice, Alice's client queries `GET presence:{bob_id}`. If the key exists, Bob is `ONLINE`. If expired, read Bob's `last_seen` timestamp from Cassandra.
4. **Presence Fanout Optimization:** Do **not** broadcast presence changes to all friends globally! Use **Fanout-on-Read**: Alice only requests the presence status of friends currently visible on her active screen viewport.

#### 11.6.2 High-Throughput Group Chat Fanout (Fanout-on-Write)
In small groups ($< 500$ members), **Fanout-on-Write** ensures minimal read latency:
1. Sender publishes message to group `grp_engineering`.
2. Fanout service looks up all 100 members of `grp_engineering`.
3. For each active member, checks their active gateway node and pushes the message down their individual WebSocket.

```csharp
public class ChatWebSocketHandler
{
    private readonly ConcurrentDictionary<string, WebSocket> _sockets = new();
    private readonly IConnectionMultiplexer _redis;

    public ChatWebSocketHandler(IConnectionMultiplexer redis)
    {
        _redis = redis;
    }

    public async Task HandleConnectionAsync(HttpContext context, WebSocket webSocket, string userId)
    {
        _sockets[userId] = webSocket;
        var sub = _redis.GetSubscriber();

        // Subscribe to a private Redis Pub/Sub channel dedicated to this user
        var channel = RedisChannel.Literal($"user:{userId}");
        await sub.SubscribeAsync(channel, async (_, message) =>
        {
            if (_sockets.TryGetValue(userId, out var socket) && socket.State == WebSocketState.Open)
            {
                byte[] payload = Encoding.UTF8.GetBytes(message.ToString());
                await socket.SendAsync(
                    new ArraySegment<byte>(payload), 
                    WebSocketMessageType.Text, 
                    true, 
                    CancellationToken.None);
            }
        });

        // Keep connection open until client disconnects
        byte[] buffer = new byte[1024 * 4];
        while (webSocket.State == WebSocketState.Open)
        {
            var result = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);
            if (result.MessageType == WebSocketMessageType.Close)
            {
                break;
            }
        }

        _sockets.TryRemove(userId, out _);
        await sub.UnsubscribeAsync(channel);
        await webSocket.CloseAsync(WebSocketCloseStatus.NormalClosure, "Closed", CancellationToken.None);
    }
}
```

---

### 11.7 Scalability & Caching Strategy
1. **L4 Load Balancing (AWS NLB):** WebSockets are long-lived TCP connections. Use Layer 4 load balancers with TCP connection termination and cross-zone balancing to distribute connection load evenly.
2. **Recent Message Caching:** Cache the most recent 100 messages of active conversations in a Redis **List** (`LPUSH` / `LTRIM` to 100). 95% of user chat reads hit the active conversation cache without touching Cassandra.
3. **Read Receipts Watermarking:** Do not emit a database update per message read! Instead, maintain a single watermark integer: `last_read_message_id`. If Bob reads 50 messages, client sends `{ "conversation_id": "c1", "last_read_message_id": 1092819 }`. All messages with $\text{id} \le 1092819$ are marked read in a single update.

---

### 11.8 Production Trade-offs & Failure Scenarios
- **Scenario A: Chat Gateway Server Crashes (Dropping 100,000 WebSocket Sockets):**
  - *Mitigation:* Clients implement exponential backoff with jitter on reconnect. When reconnecting, the L4 balancer distributes callers across surviving gateway nodes. Reconnecting clients submit their `last_received_message_id`, and the new gateway fetches missing messages from Redis/Cassandra.
- **Scenario B: Out-of-Order Message Delivery (Mobile Network Switching):**
  - *Mitigation:* Never rely on client timestamps. The gateway generates 64-bit Snowflake IDs upon server receipt. Clients render chat feeds strictly ordered by `message_id`, guaranteeing deterministic chronological order across all participant devices.

---

## 12. FULL DESIGN: DISTRIBUTED OBJECT & FILE STORAGE (S3 / GOOGLE DRIVE CLONE) {#12-full-design-distributed-file-storage}

A distributed object storage system stores and retrieves unstructured binary data (video, images, disk images, backups) of arbitrary sizes (from 1 KB to 5 TB). The system must separate **Metadata Management** from **Binary Chunk Storage**, provide **Content-Addressable Deduplication**, and allow direct client-to-storage streaming via **Presigned URLs**.

### 12.1 Requirements & Scope Bounding

#### Functional Requirements:
1. **File Upload & Download:** Support uploading and downloading files up to 50 GB.
2. **Chunking & Resumable Uploads:** Automatically break large files into chunks (e.g., 8 MB); support resuming interrupted uploads without restarting from byte zero.
3. **Content Deduplication:** Compute cryptographic chunk hashes (SHA-256) so identical file blocks uploaded by different users are stored only once physically.
4. **Presigned URL Authorization:** Generate time-limited cryptographically signed URLs enabling direct client streaming without routing gigabytes through backend API servers.
5. **Hierarchical Directory & Versioning:** Support folder structures and multiple file revisions.

#### Non-Functional Requirements:
1. **Scale:** 100 Million registered users, storing 15 Petabytes (PB) of data.
2. **Durability:** 99.999999999% (11 Nines) durability using Reed-Solomon Erasure Coding.
3. **Availability:** 99.99% service availability.
4. **Strong Consistency for Metadata:** Renaming or deleting a file must be immediately visible to all user sessions.

---

### 12.2 Capacity Estimations

- **Storage Footprint:**
  - 100 Million users $\times$ 150 MB average stored files = 15 Petabytes (PB).
- **Chunking Math:**
  - Standard chunk size: 8 Megabytes (MB).
  $$\text{Total Chunks Stored} = \frac{15 \times 10^{15} \text{ bytes}}{8 \times 10^6 \text{ bytes}} \approx 1.875 \text{ Billion chunks}$$
- **Metadata Footprint:**
  - Average chunk metadata row: 120 bytes.
  $$\text{Chunk Metadata Storage} = 1.875 \times 10^9 \times 120 \text{ bytes} \approx 225 \text{ GB}$$
  *(Metadata easily fits into a managed PostgreSQL or CockroachDB cluster).*
- **Throughput & Bandwidth:**
  - 10 Million file uploads/downloads per day.
  $$\text{Average Data Transfer Rate} = \frac{10 \times 10^6 \times 5 \text{ MB}}{86,400} \approx 578 \text{ MB/second} \approx 4.6 \text{ Gbps}$$

---

### 12.3 High-Level Architecture Diagram

```
                             ARCHITECTURE: DISTRIBUTED OBJECT STORE
                             
       [Client Application]
          │
          │ 1. POST /api/v1/files/initiate-upload
          │    (Sends file name, size, chunk hashes)
          ▼
+──────────────────────────────────────────────────────────+
|                    Metadata Service                      |
|                  (ASP.NET Core Web API)                  |
+─────────────────────────────┬────────────────────────────+
                              │
       2. Verifies Chunks in  │ 3. Generates Presigned URLs
          Metadata DB         │    for MISSING chunks only
                              v
+──────────────────────────────────────────────────────────+
|                   Metadata Database                      |
|          (PostgreSQL / CockroachDB Cluster)              |
|   - files, file_versions, chunks, chunk_references       |
+──────────────────────────────────────────────────────────+
                              │
          ┌───────────────────┘
          │ 4. Client streams missing 8MB chunks directly!
          ▼
+──────────────────────────────────────────────────────────+
|             Edge Storage Gateways / S3 Proxies           |
+─────────────────────────────┬────────────────────────────+
                              │
                              v
+──────────────────────────────────────────────────────────+
|               Storage Nodes (Raw Disk Pool)              |
|        (MinIO / Ceph / Reed-Solomon Erasure Coded)       |
+──────────────────────────────────────────────────────────+
```

---

### 12.4 Data Flow Step-by-Step

#### Multipart Resumable Upload with Chunk Deduplication:
1. **Client Chunking & Hashing:**
   - Client slices a 40 MB video file into 5 chunks of 8 MB each.
   - Client computes SHA-256 checksum for each chunk: `[Hash1, Hash2, Hash3, Hash4, Hash5]`.
2. **Initiate Upload Request:**
   - Client sends `POST /api/v1/files/initiate-upload` with file name, directory path, total size, and the array of 5 chunk hashes.
3. **Deduplication Lookup:**
   - The Metadata Service queries the `chunks` table: Which of these 5 hashes already exist in storage?
   - Result: `Hash1`, `Hash2`, and `Hash3` already exist (e.g., uploaded previously by other users).
   - Only `Hash4` and `Hash5` are new!
4. **Presigned Upload URL Generation:**
   - The Metadata Service increments the `reference_count` for `Hash1`, `Hash2`, and `Hash3` immediately.
   - The service generates time-limited **Presigned S3/Storage PUT URLs** for `Hash4` and `Hash5` only.
   - Server returns response: `{ "missing_chunks": [ { "part": 4, "url": "https://storage/..." }, { "part": 5, "url": "..." } ] }`.
5. **Direct Binary Streaming:**
   - The client uploads chunks 4 and 5 directly to the Storage Nodes via HTTP/2 PUT using the presigned URLs.
   - **Crucial Optimization:** Application servers never touch the 40 MB payload; network streaming is completely offloaded to dedicated storage infrastructure.
6. **Upload Finalization:**
   - Client calls `POST /api/v1/files/complete-upload`.
   - The Metadata Service creates the `file_version` and links all 5 chunk pointers to the new file entity.

---

### 12.5 Database Schema Design

```sql
-- Hierarchical Files & Folders
CREATE TABLE files (
    file_id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id             UUID NOT NULL,
    parent_folder_id    UUID NULL REFERENCES files(file_id),
    name                VARCHAR(255) NOT NULL,
    is_directory        BOOLEAN NOT NULL DEFAULT FALSE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    is_deleted          BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX ix_files_parent ON files (user_id, parent_folder_id);

-- File Versions (Allows rollback and history tracking)
CREATE TABLE file_versions (
    version_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    file_id             UUID NOT NULL REFERENCES files(file_id) ON DELETE CASCADE,
    version_number      INT NOT NULL DEFAULT 1,
    file_size_bytes     BIGINT NOT NULL,
    file_hash           CHAR(64) NOT NULL, -- SHA-256 of entire file
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Physical Content-Addressable Storage Chunks (Deduplicated)
CREATE TABLE storage_chunks (
    chunk_hash          CHAR(64) PRIMARY KEY, -- SHA-256 content key
    byte_size           INT NOT NULL,
    storage_path        VARCHAR(512) NOT NULL, -- URI in Ceph/MinIO/Blob
    reference_count     INT NOT NULL DEFAULT 1,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Mapping Table: Reconstructs a File Version from Chunks
CREATE TABLE version_chunks (
    version_id          UUID NOT NULL REFERENCES file_versions(version_id),
    chunk_hash          CHAR(64) NOT NULL REFERENCES storage_chunks(chunk_hash),
    chunk_order_index   INT NOT NULL,
    PRIMARY KEY (version_id, chunk_order_index)
);
```

---

### 12.6 Component Deep Dive

#### 12.6.1 Generating Presigned Streaming URLs in .NET
Offloading binary I/O from Kestrel worker processes is essential for multi-gigabit throughput:

```csharp
using Amazon.S3;
using Amazon.S3.Model;

public class StorageGatewayService
{
    private readonly IAmazonS3 _s3Client;
    private const string BucketName = "enterprise-user-storage";

    public StorageGatewayService(IAmazonS3 s3Client)
    {
        _s3Client = s3Client;
    }

    public string GeneratePresignedUploadUrl(string chunkHash, TimeSpan duration)
    {
        var request = new GetPreSignedUrlRequest
        {
            BucketName = BucketName,
            Key = $"chunks/{chunkHash}",
            Verb = HttpVerb.PUT,
            Expires = DateTime.UtcNow.Add(duration),
            ContentType = "application/octet-stream"
        };

        // Generates cryptographically HMAC-SHA256 signed URL valid for specified window
        return _s3Client.GetPreSignedURL(request);
    }
}
```

#### 12.6.2 High Durability: Erasure Coding vs 3-Way Replication
Storing 3 complete replicas of 15 PB requires 45 PB of physical raw disk—a staggering infrastructure expense.

**Reed-Solomon Erasure Coding (e.g., $8 + 4$ Configuration):**
- Data is split into $K = 8$ data fragments and encoded into $M = 4$ parity fragments (total 12 fragments).
- The 12 fragments are distributed across 12 different physical server racks and availability zones.
- **Fault Tolerance:** Any 4 fragments can be completely destroyed or lost simultaneously without data loss; the original file can be mathematically reconstructed from *any* 8 surviving fragments.
- **Storage Overhead:**
  $$\text{Storage Overhead} = \frac{K + M}{K} = \frac{8 + 4}{8} = 1.5\times \text{ (Only 50\% overhead!)}$$
  *Saves tens of millions of dollars compared to traditional 3-way replication ($3.0\times$ overhead).*

---

### 12.7 Scalability & Caching Strategy
1. **Metadata In-Memory Caching:** Cache directory tree listings in Redis using Hashes. Navigating folders does not query PostgreSQL.
2. **CDN Edge Caching for Popular Files:** Frequently accessed public files are cached at Cloudflare/CloudFront edge nodes with HTTP headers `Cache-Control: public, max-age=86400`.
3. **Chunk Garbage Collection:** When files are deleted, decrement `reference_count` in `storage_chunks`. An asynchronous background worker scans for chunks with `reference_count == 0` and purges raw disk blocks from physical storage.

---

### 12.8 Production Trade-offs & Failure Scenarios
- **Scenario A: Client Network Drops at Chunk 99 of 100:**
  - *Mitigation:* The client queries `GET /api/v1/files/{file_id}/upload-status`. The server returns the list of chunks successfully uploaded. The client uploads only chunk 100 and finalizes the file, saving 99% of bandwidth.
- **Scenario B: Bit Rot & Silent Disk Corruption:**
  - *Mitigation:* A background **Data Scrubbing Worker** continuously reads random chunks from disk, re-computes their SHA-256 checksums, and compares against the canonical database hash. If corruption is detected, Erasure Coding automatically reconstructs the corrupted block from parity and writes it to healthy sectors.

---

## 13. SENIOR ENGINEER INTERVIEW EVALUATION RUBRIC {#13-senior-engineer-interview-rubric}

When interviewing for Senior, Specialist Programmer, or Lead Architect roles, hiring managers evaluate your responses across five core competency dimensions:

```
+───────────────────────────────────────────────────────────────────────────────────────────+
|                          SYSTEM DESIGN EVALUATION DIMENSIONS                              |
+──────────────────────────┬────────────────────────────────────────────────────────────────+
| 1. Scope & Boundaries    | Does candidate clarify requirements, state assumptions, and    |
|                          | bound the scope before drawing boxes?                           |
+──────────────────────────┼────────────────────────────────────────────────────────────────+
| 2. Quantitative Rigor    | Can candidate perform back-of-the-envelope math for QPS, RAM,  |
|                          | storage, and bandwidth without prompting?                      |
+──────────────────────────┼────────────────────────────────────────────────────────────────+
| 3. Distributed Mastery   | Does candidate demonstrate deep understanding of CAP, PACELC,  |
|                          | consistent hashing, idempotency, and sharding trade-offs?      |
+──────────────────────────┼────────────────────────────────────────────────────────────────+
| 4. Failure Engineering   | Does candidate design for hardware failure, network partitions,|
|                          | stampedes, split-brain, and data reconciliation?               |
+──────────────────────────┼────────────────────────────────────────────────────────────────+
| 5. Technical Leadership  | Does candidate lead the conversation, justify trade-offs, and  |
|                          | articulate clear rationale rather than reciting buzzwords?     |
+──────────────────────────┴────────────────────────────────────────────────────────────────+
```

### System Design Rapid Architecture Cheat Sheet

| Design Scenario | Primary DB | Cache / Hot Store | Messaging / Async | Key Distributed Pattern |
|---|---|---|---|---|
| **URL Shortener** | PostgreSQL (Sharded) | Redis (LRU + Bloom Filter) | Kafka (Clicks) | Base62 + Key Generation Service (KGS) |
| **Notification System** | PostgreSQL / Cassandra | Redis (Token Bucket + Idemp) | Kafka (Priority Topics) | Deduplication & Multi-Vendor Circuit Breaker |
| **Payment System** | PostgreSQL (Quorum Write) | Redis (Distributed Locks) | Kafka (CDC Outbox) | Orchestrated Saga + Double-Entry Bookkeeping |
| **Order Management** | PostgreSQL | Redis (Flash Lua Scripts) | Kafka (Debezium CDC) | Transactional Outbox + Atomic Decrement |
| **Real-Time Chat** | Cassandra / ScyllaDB | Redis (Presence & Pub/Sub) | Kafka (Fanout Worker) | WebSockets (WSS) + Snowflake Monotonic IDs |
| **Distributed Storage** | PostgreSQL (CockroachDB) | Redis (Folder Tree) | Kafka (Thumbnail / AV) | Reed-Solomon Erasure Coding + Presigned URLs |

---
*End of Volume 10: System Design. Continue to Volume 11: Project Interview Mastery.*
