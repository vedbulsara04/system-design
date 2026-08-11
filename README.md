# System-Design Guide
> Created by: @vedbulsara04
---

## *Introduction*

### > What is System Design ?
```txt
System design is the process of planning and structuring the architecture 
of a software system based on user requirements. It defines how different
components will work together to achieve the desired function efficiently.
```
### > How to approach System Design ?
<img src="https://markdownviewer.pages.dev/api/image/xs21rbsOYSju_Qbv7EKn6fXX" width="270">

---

```txt
1. Clarify Requirements: Never jump straight into designing. Ask questions first.
   - What are the core features? (functional requirements)
   - How many users? What's the expected load? (non-functional requirements)
   - Any constraints? (budget, existing tech stack, latency SLAs?)

2. Estimate Scale: Quick estimates for system sizing.
   - How many requests per second (QPS)?
   - How much data stored per day/year?
   - Read-heavy or Write-heavy data?
   = This guides whether you need Caching, Sharding, CDN, etc.

3. Define the API: Before drawing boxes, define what the system does.
   - What endpoints exist? (POST /tweet, GET /feed)
   - What goes in and out of each endpoint?
   = This keeps the design grounded in real user actions. 

4. High-Level Design: 
   - Draw the big picture (clients, servers, DBs, queues, CDNs).
   - Don't over-engineer yet; focus on getting the data flowing end-to-end.

5. Deep Dive: Pick the hardest or most interesting components and go deeper.
   - How does the database schema look?
   - Where do you add a cache?
   - How do you handles hot spots or failures?

6. Identify Trade-Offs: No design is perfect. Acknowledge the trade-offs you made:
   - Why SQL over NoSQL (or vice versa)?
   - What would break at 10x scale?
   - What did you leave out and why?
```

---

## *Performance v/s Scalability*

### > Performance in System-Design
```txt
* Performance is about how fast, efficiently, and reliably a system responds under load. 
  It's one of the most critical non-functional requirements in any system design.

* Core Performance Metrics:
1. Latency: Time taken to process a single request.
2. Throughput: Number of requests a system can handle per unit of time (QPS).
3. Availability: Percentage of time a system is operational.
   [ 99.9% availability = ~8.7 hrs downtime/year ]
4. Error Rate: Percentage of requests that fail out of all requests made to a system.
```

### > Scalability in System-Design
```txt
* Scalability is the system's ability to handle increased load,
  without sacrificing system performance.

* Types of Scalability:
1. Vertical Scaling (Scale-Up): Add more power to the same machine.
   [ more CPU, RAM, Storage ] 
   [ Simple but has a hardware limit and single point of failure ]
2. Horizontal Scaling (Scale-Out): Add more machines to distribute the load.
   [ More complex but virtually unlimited and fault tolerant ]
```