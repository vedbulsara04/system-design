# System-Design Guide

> Created by: [@vedbulsara04](https://github.com/vedbulsara04)

---

## *Introduction*

### ` What is System Design ? `

System design is the process of planning and structuring the architecture of a software system based on user requirements. 
It defines how different components will work together to achieve the desired function efficiently.

### ` How to approach System Design `
<img src="media/system_design_approach.png" width="270">

---

1. **Clarify Requirements**: Never jump straight into designing. Ask questions first.
   - What are the core features? (functional requirements)
   - How many users? What's the expected load? (non-functional requirements)
   - Any constraints? (budget, existing tech stack, latency SLAs?)

2. **Estimate Scale**: Quick estimates for system sizing.
   - How many requests per second (QPS)?
   - How much data stored per day/year?
   - Read-heavy or Write-heavy data?
   
   > This guides whether you need Caching, Sharding, CDN, etc.

3. **Define the API**: Before drawing boxes, define what the system does.
   - What endpoints exist? (POST /tweet, GET /feed)
   - What goes in and out of each endpoint?
   
   > This keeps the design grounded in real user actions. 

4. **High-Level Design**: 
   - Draw the big picture (clients, servers, DBs, queues, CDNs).
   - Don't over-engineer yet; focus on getting the data flowing end-to-end.

5. **Deep Dive**: Pick the hardest or most interesting components and go deeper.
   - How does the database schema look?
   - Where do you add a cache?
   - How do you handles hot spots or failures?

6. **Identify Trade-Offs**: No design is perfect. Acknowledge the trade-offs you made:
   - Why SQL over NoSQL (or vice versa)?
   - What would break at 10x scale?
   - What did you leave out and why?

---

## *Performance v/s Scalability*

### ` Performance in System-Design `
Performance is about how fast, efficiently, and reliably a system responds under load. 
It's one of the most critical non-functional requirements in any system design.

**Core Performance Metrics**:
1. *Latency*: Time taken to process a single request.
2. *Throughput*: Number of requests a system can handle per unit of time (QPS).
3. *Availability*: Percentage of time a system is operational.
   
   > 99.9% availability = ~8.7 hrs downtime/year
4. *Error Rate*: Percentage of requests that fail out of all requests made to a system.


### ` Scalability in System-Design `

Scalability is the system's ability to handle increased load, without sacrificing system performance.

**Types of Scalability**:
1. *Vertical Scaling (Scale-Up)*: Add more power to the same machine.
   
   > more CPU, RAM, Storage
   
   > Simple but has a hardware limit and single point of failure
2. *Horizontal Scaling (Scale-Out)*: Add more machines to distribute the load.
   
   > More complex but virtually unlimited and fault tolerant

---

## *Latency v/s Throughput*

### ` Latency in System-Design `

Latency is the time it takes to complete a single operation, from request to response.

**Types of Latency**:
1. *Network Latency*: Time for data to travel across the network.
2. *Disk I/O Latency*: Time to read/write from storage.
3. *Processing Latency*: Time the CPU spends computing.
4. *End-to-end Latency*: Total perceived delay by the user.

**How to reduce Latency**:
- ***Caching***: serve data from memory(Redis,CDN) instead of disk or remote services.
- ***Connection Pooling***: avoid TCP handshake overhead on every request.
- ***Geographically distributed nodes***: bring data closer to users.
- ***Async I/O or non-blocking operations***: don't wait idle; do other work.
- ***Reduce hops***: fewer services in the call chain means less accumulated delay.

### ` Throughput in System-Design `

Throughput is the number of operations a system can handle per unit of time, often measured in requests per second or QPS.

**How to increase throughput**:
- ***Horizontal Scaling***: add more machines to share the load.
- ***Load Balancing***: distribute requests evenly across instances.
- ***Batching***: process multiple operations together.
- ***Concurrency***: handle many requests in parallel (threads, async, event loops)
- ***Efficient data formats***: Protocol Buffers over JSON reduces serialization cost.

---

## *Availability & Consistency*

### ` Availability in System-Design `

Availability means every request receives a response, not necessarily the most up-to date, but the system stays operational and never returns an error due to node failure.

**How to achieve high availability**:

- ***Replication***: duplicate data across multiple nodes so no single node is a point of failure.
- ***Failover***: automatically switch to a standby node when the primary fails.
- ***Health checks & Load Balancers***: route traffic away from unhealthy or overloaded nodes.
- ***Redundancy at every layer***: servers, databases, network links, data centers.
- ***Graceful Degradation***: return partial or cached data rather than erroring out.

### ` Consistency in System-Design `

Consistency means every read reflects the most recent write; all nodes in the system see the same data at the same time

### ` The CAP Theorem `

Proposed by Eric Brewer, it states that distributed systems can guarantee only 2 of these 3 properties simultaneously:

- ***Consistency***: every node returns the same, most recent data.
- ***Availability***: every request gets a response.
- ***Partition Tolerance***: system keeps working even if nodes cant talk to each other.

#### **CP: Consistency + Partition Tolerance**

"I'd rather return an error than return stale data"
- When a partition occurs, the system refuses to respond until it can guarantee the data is consistent.
- Nodes may reject reads/writes to avoid serving outdated state.
- Prioritizes correctness over uptime.

> Usage: Bank balances, Inventory counts, etc; anywhere wrong data causes real damage.

#### **AP: Availability + Partition Tolerance**

"I'd rather return something than return nothing"
- When a partition occurs, the system keeps responding even if the nodes have stale data.
- Nodes diverge temporarily and reconcile later (eventual consistency)
- Prioritizes uptime over correctness.

> Usage: Social feeds, product catalogue, user profile, view counts, etc; anywhere a slightly stale response is acceptable.

### ` Consistency Patterns `

It defines what value a read returns after a write, across nodes that may be geographically distributed, partially failed or temporarily partitioned.

**The Core Question**

After a client writes data to a system:
> "What will the next read return and from which node ?
- The 3 patterns answer this differently.

#### **1. Weak Consistency**

After a write, the system makes no guarantee that subsequent reads will reflect it. The write propogates on a best-effort basis: some nodes may see it immediately, others never and the order of propagation is undefined.

> Usage: Real-time, latency-sensitive data where loss or staleness is acceptable.

#### **2. Eventual Consistency**

After a write, a system guarantees that all replicas will converge into the same state, provided no new conflicting writes occur. There is no bound on *when* convergence happens (in practice, typically milliseconds to seconds under normal operation).

> Usage: Write throughput and availability are primary constraints; slight staleness on reads is acceptable to the application.

#### **3. Strong Consistency**

After a write completes, every subsequent read from any node in the system reflects that write, regardless of which replica handles that read. The system behaves as if there is a single copy of the data.

> Usage: Incorrect or stale data causes real, unrecoverable harm. ( financial transactions, inventory deductions, distributed locks, authentication tokens, configuration state )

### ` Availability Patterns `

They are architectural strategies to ensure a system remains operational under node failure. The 2 primary mechanisms are: Fail-Over and Replication.

#### **1. Fail-Over**

It is the mechanism of automatically switching traffic to a standby node when the active node fails. A heartbeat signal is continuously sent between active and passive nodes to detect failure.
   
   - **Active-Passive Fail-Over**: One active node handles all traffic. The second node (passive/standby) sits idle, receiving replicated state but serving no requests.
  
      * *On Failure*:
         - Heartbeat from active node stops.
         - Passive node detects failure, promotes itself to active.
         - Traffic is rerouted to newly promoted node.

   - **Active-Active Fail-Over**: Both nodes are active simultaneously, handling traffic in parallel. A load balancer distributes requests across both.
      
      * *On Failure*:
         - Load balancer detects the failed node via health checks.
         - All traffic is rerouted to the surviving node.
         - No promotion required, the other node is already active.

#### **2. Replication**

It is the mechanism of copying data across multiple nodes to ensure durability and read scalability.
   
   **There are two primary toplogies:**

   - **Master-Slave Replication**: One node (master) handles all writes. Changes are replicated asynchronously (or synchronously) to one or more slave nodes, which serve reads.
  
      ```
      Client Writes -> Master -> Replicates -> Slave 1 & Slave 2

      Client Reads -> Slave 1 / Slave 2
      ```

      * *On Master Failure*: 
         - A slave is manually or automatically promoted to master.
         - Until promotion completes, writes are unavailable.
   
   - **Master-Master Replication**: Both nodes (masters) accept reads and writes simultaneously. Changes made on either node are replicated to the other.
   
      ```
      Client Writes -> Master 1 <-> Replicates <-> Master 2 <- Client Writes

      Client Reads -> Master 1 / Master 2
      ```

      * *On Failure*: 
         - Surviving master continues serving both reads and writes uninterrupted.
         - No promotion needed.

### ` Availability in Numbers ` 

Availability is referred as the percentage of time a system is operational over a given period. Commonly referred to as "nines", each additional nine reduces allowable downtime by ~10x.

#### **Availability Tiers**

| Availability | Nines | Downtime / Year | Downtime / Month | Downtime / Week |
| --- | --- | --- | --- | --- |
| 90% | One 9 | 36.5 days | 72 hours | 16.8 hours |
| 99% | Two 9s | 3.65 days | 7.2 hours | 1.68 hours |
| 99.9% | Three 9s | 8.76 hours | 43.8 minutes | 10.1 minutes |
| 99.99% | Four 9s | 52.6 minutes | 4.38 minutes | 1.01 minutes |
| 99.999% | Five 9s | 5.26 minutes | 26.3 seconds | 6.05 seconds |
| 99.9999% | Six 9s | 31.5 seconds | 2.63 seconds | 0.6 seconds |

#### **Industry standard SLA targets** 

- Most web services target Three to Four 9s.
- Telecom and financial systems target Five 9s.
- Six 9s is rare and extremely expensive to engineer.

#### **Availability in Sequence v/s Parallel**

When a request passes through multiple components, the topology of those components determines the overall system availability.

- ***Components in Sequence***: A request must pass through all components to succeed. If any one fails, the entire request fails.

   ```
   Request -> [Component A] -> [Component B] -> [Component C] -> Response
   ```

   Formula:
   ```
   Total_Availability = Availability_A x Availability_B x Availability_C
   ```
   Example:
   ```
   A = 99.9%,  B = 99.9%,  C = 99.9%

   Total_Availability = 0.999 × 0.999 × 0.999 = 0.997 = 99.7%
   ```

- ***Components in Parallel***: Multiple instances of a component run simultaneously. A request succeeds if at least one instance is available. All must fail simultaneously for the system to go down.

   ```
                ┌─ [Component A1] ─┐
   Request  --> ├─ [Component A2] ─┼ --> Response
                └─ [Component A3] ─┘
   ```

   Formula:
   ```
   Total_Availability = 1 - (1 - Availability_A)^n

   Where 'n' = number of parallel instances
   ```

   Example:
   ```
   A = 99.9%,  n = 2 parallel instances

   Total_Availability = 1 - (1 - 0.999)² = 1 - (0.001)² = 1 - 0.000001 = 99.9999%   
   ```

- ***Practical Implication***:

   A system's critical path, the sequential chain of components a request must traverse, is the primary availability bottleneck. The standard architectural respones is:

   - **Minimize** the no. of sequential dependencies in the critical path.
   - **Replicate** each component in the critical path in parallel (redundancy).
   - **Isolate failures**: a failing non-critical component should not bring down the critical path.

---

## *Background Jobs*

Background jobs are processing units that execute outside the request-response cycle asynchronously, without blocking the client. They handle workloads where immediate response is not required or where processing time exceeds acceptable request latency.

**When to use background jobs**:

- Processing time exceeds acceptable synchronous latency. (ex: video encoding, PDF generation, ML inference)
- Work can be deferred without impacting user experience. (ex: sending emails, pushing notifications)
- Workload is periodic and not user-initiated. (ex: billing cycles, report generation)
- Decoupling producer throughput from consumer throughput is required. (ex: smoothing traffic spikes via queue buffering)


### ` Trigger Mechanisms `

#### **1. Event-Driven**

Jobs are triggered by a specific event occuring in the system, like a message arriving on a queue, a record being written to a database, a file being uploaded to object storage.

   **Mechanism**:
  - Producer emits an event (message, signal, record).
  - Event broker (queue or stream) holds the event.
  - Consumer (background worker) picks it up and processes it asynchronously.

   ```
   Producer -> [Event Queue / Stream] -> Worker Process
                 (Kafka, RabbitMQ,           (processes independently
                  SQS, Redis Streams)          of original request)
   ```

   **Examples**:
   Order placed -> trigger invoice generation; File uploaded -> trigger transcoding; User registered -> trigger welcome email.

#### **2. Schedule-Driven**

Jobs are triggered by time, i.e. a fixed schedule (cron expression) or a regular interval regardless of system events.

   **Mechanism**:
   - A scheduler (cron daemon, job scheduler) fires the job at the configured time.
   - Job runs against the current state of the system at that moment.

   ```
   Cron Expression:  0 2 * * *   (every day at 02:00)
      |
      ▼
   Job Executor -> [Process] -> Write results to DB / storage
   ```

   **Examples**: Nightly database aggregations, weekly report generation, daily cache warming, periodic data cleanup / TTL expiry.

   ***Returning Results***:

   Background jobs execute asynchronously i.e. the client does not wait. Several patterns handle result delivery:

   | Pattern | 	Mechanism | 	Use Case |
| --- | --- | --- |
| Polling | Client periodically queries a status endpoint until job is complete | Simple; works with any client |
| Webhook / Callback | Job posts result to a pre-registered client URL on completion | Push-based; efficient; client must expose endpoint |
| WebSocket / SSE | Persistent connection; server pushes result to client when ready | Real-time UX without polling overhead |
| Shared storage | Job writes result to DB or object store; client reads when ready | Decoupled; works across services |
| Message queue | Job publishes result event; client consumes from its own queue | Fully async; used in service-to-service flows |

---

## *Domain Name System*

DNS is the distributed hierarchical naming system that translates human-readable domain names into IP addresses. It is the first step in virtually every network request and operates as a globally distributed, eventually consistent database.

### ` Core Function `

```
Client requests "www.example.com"
         │
         ▼
DNS Resolution -> Returns -> 93.184.216.34 (IPv4) or 2606:2800:220:1:248:1893:25c8:1946 (IPv6)
         │
         ▼
Client opens TCP connection to resolved IP

```

Without DNS, every client would need to know the IP address of every server, DNS decouples the human-facing name from the underlying infrastructure IP, allowing IPs to change without affecting clients.

### ` DNS Hierarchy `

DNS is organized as an inverted tree, with resolution delegated downard through levels:

```
                        . (Root)
                        │
          ┌─────────────┼──────────────┐
         .com          .org           .net        <- Top-Level Domains (TLD)
          │
     example.com                                  <- Second-Level Domain
          │
    www.example.com                               <- Subdomain
```

### ` Resolution Process `

DNS resolution involves up to four server types, each with a distinct role:

| Server | Role |
| --- | --- |
| DNS Resolver (Recursive Resolver) | Client-facing; performs the full lookup on behalf of the client; caches results |
| Root Name Server | Directs resolver to the correct TLD server |
| TLD Name Server | Directs resolver to the authoritative name server for the domain |
| Authoritative Name Server | Holds the actual DNS records; returns the final answer |

### ` TTL - Time To Live `

Every DNS record carries a TTL value in seconds - the duration for which resolvers and clients may cache the record before re-querying.

```
example.com.   300   IN   A   93.184.216.34
                │
                └──  TTL = 300 seconds (5 minutes)
```

#### **TTL tradeoffs**

| TTL | Effect |
| --- | --- |
| High TTL (hours/days) | Fewer DNS queries; lower DNS infrastructure load; stale records persist longer after changes |
| Low TTL (seconds/minutes) | Changes propagate faster; higher query volume to authoritative servers |


### ` DNS Caching Layers `

A DNS record is cached at multiple layers before expiry:

```
Browser cache          →  shortest TTL, per-tab or per-session
OS resolver cache      →  /etc/hosts checked first; then OS DNS cache
ISP / Corporate DNS    →  shared cache across many clients
Recursive Resolver     →  upstream cache (8.8.8.8, 1.1.1.1)
```

> `/etc/hosts` file is checked before any DNS query, static entries here always take precedence. Used in local development and container networking.

### ` DNS and Eventual Consistency `

DNS is a canonical example of an eventually consistent system:

- Changes to DNS records propagate asynchronously across the global resolver network.
- Propagation time is bounded by TTL - resolvers hold cached records until TTL expires.
- During propagation, different clients may resolve the same domain to different IPs.
- No global synchronization - by design, for scalability.


## **Content Delivery Network (CDN)**

A CDN is a globally distributed network of edge servers (Points of Presence - PoPs) that cache and serve content from locations geographically closer to the end user. The primary purpose is to reduce latency by eliminating the round-trip to the origin server for cacheable content, and to reduce load on the origin infrastructure.

<img src="media/cdn.png">

> source: [Cloudflare - How does a CDN work?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/#how-does-a-cdn-work)


---

### ` Core Mechanism `

- **Without CDN:**
      
  ```
  User (Mumbai) -> [Internet] -> Origin Server (US-East) -> Response travels back
                                 Round trip: ~200ms
  ```

- **With CDN:**

  ```
  User (Mumbai) -> CDN Edge (Mumbai PoP) -> Response served locally
                   Cache hit: ~5-10ms
  ```

On a cache miss, the edge node fetches from origin, caches the response and serves it - subsequent requests for the same content are served from cache until TTL expires.

### ` What CDNs Serve `

- **Static assets** - HTML, CSS, JavaScript, images, fonts.
- **Video and audio streams** - adaptive bitrate streaming. (HLS, DASH)
- **Software downloads** - binaries, packages.
- **API responses** - for cacheable, non-personalized endpoints.
- **Dynamic content** - via edge computing. (Cloudflare Workers, Lambda@Edge)

### ` Pull CDNs `

In a pull CDN, content is fetched from the origin server on demand - the CDN pulls content lazily when a user first requests it. No pre-population of the cache is required.

**Mechanism**: 
   
1. User requests asset -> hits CDN edge node.
   
2. Edge node has no cached copy (cache miss).
   
3. Edge node fetches asset from origin server.

4. Asset is cached at edge with configured TTL.

5. Edge serves asset to user.

6. All subsequent requests -> cache hit -> served from edge until TTL expires.

**Best suited for**: 

- Large content libraries where pre-loading all assets is impractical.
- Frequently updated content where TTL controls freshness.
- Standard web assets - images, JS, CSS.

**Examples**: Cloudflare (defualt mode), AWS CloudFront, Fastly.

### ` Push CDNs `

In a Push CDN, content is explicitly uploaded to the CDN by the operator before any user requests it. The CDN does not fetch from origin - it serves only what has been pushed.

**Mechanism**:

1. Operator uploads assets to CDN storage. (via API or deployment pipeline)
   
2. CDN distributes assets to all edge nodes proactively.

3. User requests asset → edge node always has it. (no origin fetch)

4. Operator is responsible for pushing updates and managing expiry.

**Best suited for**:

- Small, well-defined content sets that change infrequently.
- Assets requiring guaranteed availability regardless of origin health.
- Large file distribution - software releases, game patches, firmware updates.
- Content that can be precomputed and uploaded ahead of demand.

**Examples**: Akamai NetStorage, AWS CloudFront with S3 origin pre-loaded, traditional media distribution networks.

## **Load Balancers**

A load balancer is a component that distributes incoming traffic across a pool of backend servers to maximize throughput, minimze legacy, and prevent any single server from becoming a bottleneck or single point of failure.

### ` Load Balancer v/s Reverse Proxy `

| Property | Load Balancer | Reverse Proxy |
| --- | --- | --- |
| Primary function | Distribute traffic across multiple backend instances | Sit in front of one or more servers; mediate all client request |
| Requires multiple backends? | Yes - meaningless with one backend | No - useful even with a single backend |
| Additional features | Health checking, session persistence, scaling | SSL termination, caching, compression, authentication, WAP |
| Scope | Traffic distribution | Request mediation and transformation |
| Relationship | A load balancer is a specialized reverse proxy | A reverse proxy is not necessarily a load balancer |

