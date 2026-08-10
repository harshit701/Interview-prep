# System Design Interview Prep

## Interview Approach

### What is System Design?

System Design is the blueprint of an application that defines how different components work together to build a scalable, reliable, and efficient system.

### Suppose I ask: "Design WhatsApp." What should you ask me first?

Don't start drawing the architecture immediately.

#### What is the first thing you should clarify with the interviewer?

Most candidates make this mistake:
> **Interviewer:** "Design WhatsApp." **Candidate:** "I'll use Redis, Kafka, Kubernetes..." ❌

That's not how system design interviews work.

**Step 1: Clarify the requirements**

Instead of assuming what "WhatsApp" means, ask what features should be designed. WhatsApp has many features:

- One-to-one chat
- Group chat
- Voice calls
- Video calls
- Status
- File sharing
- End-to-end encryption
- Online status
- Read receipts

The interviewer usually expects you to narrow the scope. For example:
> **Interviewer:** "Design WhatsApp." **You:** "Before I start, I'd like to clarify the requirements. Should we focus only on one-to-one messaging, or should we also include group chats, voice calls, and media sharing?" **Interviewer:** "Let's only design one-to-one messaging."

You've now reduced the scope.

**Step 2: Estimate scale**

Now ask:

- How many daily active users?
- How many messages per day?
- Is this a global application?
- What's the expected read/write ratio?

**Golden rule for every system design interview** — always follow this order:

1. **Clarify Requirements** — what exactly are we building?
2. **Estimate Scale** — how many users, how much traffic?
3. **High-Level Design** — APIs, database, cache, load balancer, message queue
4. **Deep Dive** — scalability, availability, fault tolerance, bottlenecks

This framework works for almost every question, whether the interviewer asks you to design WhatsApp, YouTube, Uber, Instagram, a URL Shortener, or BookMyShow — you always start the same way.

### Functional Requirements vs Non-Functional Requirements

**Functional Requirements** answer: *what should the system do?* They describe the features.

Example — for WhatsApp: send messages, receive messages, create groups, upload images, show online status.
> **Simple definition ⭐⭐⭐⭐⭐** — Functional requirements define what the system should do or what features it should provide to users.

**Non-Functional Requirements** answer: *how well should the system perform?* They describe the quality of the system.

Examples: handle 10 million users, response time under 200ms, 99.99% availability, secure, scalable, reliable. These are not features.
> **Simple definition ⭐⭐⭐⭐⭐** — Non-functional requirements define how the system should perform, including performance, scalability, reliability, availability, and security.

## Scalability

### What is Scalability?

Scalability is the ability of a system to handle increasing traffic or workload without significantly affecting its performance — or, put another way, the system can serve more users by increasing its capacity while maintaining good performance.

#### Types of Scalability

**1. Vertical Scaling (Scale Up)**

Imagine one server with 4 GB RAM and 2 CPUs. When traffic increases, instead of adding another server, you upgrade it to 64 GB RAM and 32 CPUs — the same machine, made more powerful. This is called **Vertical Scaling**.

```
Node.js Server
      │
    2 CPU
      │
   Upgrade
      │
   16 CPU

Still one server.
```

*Advantages:* very easy, no application changes. *Disadvantages:* very expensive, hardware has limits, single point of failure — if this server dies, users get no application at all.

**2. Horizontal Scaling (Scale Out)**

Instead of buying a bigger server, add more servers (Server 1, Server 2, Server 3, Server 4) and share the traffic between them.

```
Instead of:
1000 Requests → One Server

You get:
1000 Requests → Load Balancer → Server 1, Server 2, Server 3, Server 4

Each server handles about 250 requests. Much easier.
```

**Why big companies prefer horizontal scaling**

Imagine Amazon with 50 million users of traffic — can one server handle this? No. Instead:

```
Load Balancer → 100 Servers

If traffic doubles → 200 Servers
```

Simple.

#### Difference between vertical scaling and horizontal scaling

Vertical Scaling increases the capacity of a single server by adding more CPU, RAM, or storage, whereas Horizontal Scaling increases system capacity by adding more servers and distributing traffic across them.

#### Your Node.js API currently handles 100 requests/sec. Tomorrow it starts receiving 2,000 requests/sec. What would you do?

First, I would identify the bottleneck by monitoring CPU, memory, database performance, network usage, and response times. If the application server is the bottleneck, I would horizontally scale the Node.js application by running multiple instances behind a load balancer. If the database becomes the bottleneck, I would optimize queries, add indexes, introduce Redis for caching, and, if needed, use read replicas or database sharding. For asynchronous tasks such as sending emails or notifications, I would offload them to a message broker like Kafka or RabbitMQ. Finally, I would continuously monitor the system and scale individual services based on traffic.

## Consistency Models

### What is Strong Consistency vs Eventual Consistency?

**Strong consistency** — after a write completes, every subsequent read (from any node) sees that write immediately. Simpler to reason about, but requires more coordination between nodes, which costs latency and availability during network issues.

**Eventual consistency** — after a write, reads may return stale data for a short period, but the system guarantees all nodes will converge to the same value eventually (once replication catches up). Higher availability and lower latency, at the cost of temporary staleness.

```
Example: you post something on social media.
Strong consistency  -> every follower sees it immediately, no matter which server they hit.
Eventual consistency -> some followers might see it a few seconds later, but everyone converges eventually.
```

### When would you choose eventual consistency over strong consistency?

For systems where a brief staleness window is acceptable in exchange for availability and performance — social media feeds, view/like counts, product catalogs. For systems where staleness is unacceptable — account balances, inventory counts near zero stock, anything involving money — strong consistency (or careful compensating logic) is worth the cost.

## Availability & Reliability

### What's the difference between Availability and Reliability?

**Availability** is the ability of a system to remain accessible and usable when users need it — can users use the system right now? If yes, that's high availability.

*Example:* Suppose Amazon is running at 10:00 AM — the website opens, search works, orders work, payments work. Amazon is available. Now suppose the Amazon website returns a 500 Internal Server Error and nobody can access it — availability is low.

*Real-life analogy:* think about electricity. If it's available, you can use your TV, laptop, and AC. If it goes off, everything stops. Same idea.

**Reliability** is the ability of a system to perform its intended function correctly and consistently without failures.

*Example:* suppose you transfer ₹5,000 and the bank shows "Transaction Successful," but the recipient receives ₹0. The bank was *available* because you could use it, but it was not *reliable* because it gave the wrong result.

### Suppose you're designing UPI (Google Pay / PhonePe). Which would you prioritize more — high availability or high reliability?

**Scenario 1:** the app is available. You send ₹50,000, but money is deducted twice, the wrong account receives the money, or the transaction status is incorrect. This is a reliability failure, and it's far more serious than downtime.

**Scenario 2:** the app is down for 5 minutes. Nobody can make payments, and users are frustrated — but nobody loses money.

For a financial system, the second scenario is generally less damaging than incorrect financial transactions. That's why banks and payment systems prioritize correctness above all else.

**Interview answer:**
> "Ideally, I would design the system to be both highly available and highly reliable. However, if I had to prioritize one for a payment system like UPI, I would prioritize reliability because financial transactions must be processed correctly. Users may tolerate temporary downtime, but they cannot tolerate incorrect or duplicate transactions or loss of money."

**Different systems have different priorities:**

| System          | Higher Priority |
| --------------- | --------------- |
| Banking / UPI   | ✅ Reliability   |
| WhatsApp        | Availability    |
| Netflix         | Availability    |
| Google Search   | Availability    |
| Stock Trading   | Reliability     |
| Hospital System | Reliability     |

## Performance: Latency & Throughput

> **Latency** = how fast? **Throughput** = how much?

### Latency

Latency is the time taken by the system to process a single request and return a response.

*Example:* you open YouTube and click Play — the video starts in 200ms, so latency is 200ms. Or: you call `GET /users` and the server responds in 150ms — latency is 150 milliseconds. Lower latency is better.

### Throughput

Throughput is the number of requests or operations a system can process in a given period of time.

*Example:* if your API can process 500 requests/second, its throughput is 500 requests/sec. If tomorrow it handles 5,000 requests/sec, it has a much higher throughput.

**Restaurant analogy 🍕**

- *Latency:* you order one pizza — how long until you receive it? 15 minutes. That's latency.
- *Throughput:* how many pizzas can the restaurant make in one hour? 300 pizzas/hour. That's throughput.

**Toll booth analogy**

- *Latency:* one car takes 5 seconds to pass. Latency: 5 seconds.
- *Throughput:* the toll booth allows 720 cars/hour. Throughput: 720 cars/hour.

### How do we improve latency?

- Redis caching
- CDN
- Faster database queries
- Better indexes
- Reduce unnecessary network calls

### How do we improve throughput?

- Horizontal scaling
- Load balancers
- Multiple application servers
- Message queues
- Better resource utilization

## Infrastructure Components

### What is a Load Balancer?

A Load Balancer is a component that distributes incoming client requests across multiple servers so that no single server becomes overloaded.

### What is a Health Check?

A Health Check is a periodic request sent by the Load Balancer to verify that a server is healthy and able to serve requests.

### What is a Reverse Proxy?

A Reverse Proxy is a server that sits between clients and backend servers, receives client requests, and forwards them to the appropriate backend server without exposing the backend directly.

### What is a Content Delivery Network (CDN)?

A CDN (Content Delivery Network) is a network of geographically distributed servers that stores copies of static content closer to users, reducing latency and improving loading speed — it stores your content closer to users so they don't have to fetch it from the main server every time.

#### What does a CDN store?

A CDN is best for static content that doesn't change very often, such as:

- Images
- CSS
- JavaScript
- Fonts
- Videos
- PDFs
- Downloadable files

### Caching

Caching is a core system-design building block for improving both latency and throughput. The Cache-Aside pattern — the most commonly expected answer in interviews — is covered in depth (with Redis specifics) in the **Database Interview Prep** file.

### What is the Circuit Breaker pattern, and why is it needed?

When Service A calls Service B, and B is failing or extremely slow, A retrying repeatedly (or waiting on a long timeout for every request) wastes resources and can cascade the failure back to A's own callers. A circuit breaker wraps calls to B and tracks failure rate:

```
Closed (normal)   -> requests pass through normally
       │ (failure rate exceeds threshold)
       ▼
Open              -> requests fail immediately WITHOUT calling B, for a cooldown period
       │ (cooldown expires)
       ▼
Half-Open         -> allow a small number of test requests through
       │
       ├── succeed -> back to Closed
       └── fail    -> back to Open
```

This prevents a struggling downstream service from being hammered with requests it can't handle, and lets A fail fast (return a fallback/cached response, or a clear error) instead of hanging on every request waiting for B to time out.

## Message Queues

### Kafka vs RabbitMQ — when would you choose which?

**Kafka** — a distributed log/streaming platform. Messages are retained for a configurable period (not deleted immediately after consumption), and multiple independent consumers can each read the full stream at their own pace. Built for very high throughput and event-sourcing/replay use cases.

**RabbitMQ** — a traditional message broker/queue. Messages are typically removed once acknowledged by a consumer. Better suited for classic task-queue patterns (a job should be processed exactly once by exactly one worker) and offers more flexible routing (exchanges, topic-based routing) out of the box.

```
Choose Kafka when:
  - You need multiple independent services to consume the SAME event stream
  - You need replay/audit capability (event sourcing)
  - Very high throughput (millions of events/sec)

Choose RabbitMQ when:
  - Classic task queue (one job, one worker)
  - Complex routing logic between producers and consumers
  - Lower operational complexity is a priority
```

### What is a Dead Letter Queue (DLQ)?

A DLQ is a separate queue where messages are routed after they fail processing repeatedly (exceeding a retry limit), instead of being retried forever or silently dropped. This lets you investigate and reprocess failed messages without blocking the main queue.

## Common System Design Walkthroughs

### Design a URL Shortener

**Requirements clarification:** custom aliases needed? Expiration? Analytics (click tracking)? Expected scale (reads vs writes ratio — typically read-heavy, since one shortened URL gets clicked many times)?

**High-level design:**
```
Client -> API Gateway -> Shortener Service -> Database
                              │
                         Cache (Redis) for hot URLs
```

**Core logic:** generate a short, unique key for each long URL — either a counter-based approach (base62-encode an auto-incrementing ID) or a hash-based approach (hash the URL, take the first N characters, handle collisions). Store the mapping `shortKey -> longURL`.

**Read path (the hot path, since reads >> writes):** `GET /:shortKey` — look up in cache first, fall back to DB on cache miss, redirect with `301`/`302`.

**Scale considerations:** since it's read-heavy, aggressive caching (Redis) for popular URLs matters more than write optimization. Database can be sharded by short-key prefix if it grows large enough to need it.

### Design a Rate Limiter (as a distributed system, not just the algorithm)

**Requirements:** per-user or per-IP limits? What should happen when the limit is exceeded (reject vs queue)? Does it need to work correctly across multiple application server instances?

**Design:** since multiple app instances can't rely on local in-memory counters (each instance would track separately, effectively multiplying the real limit), rate limit state needs to live in a shared, fast store — **Redis**, using atomic increment operations (`INCR` with a TTL) so concurrent requests from different instances don't race.

```
Request arrives -> App instance checks Redis: INCR user:123:count, set TTL if new key
                 -> if count > limit: reject with 429
                 -> else: proceed
```

**Where to enforce it:** at the API Gateway level for coarse global protection, plus application-level checks for business-specific limits (tied to a user's subscription tier, for example).

### Design a Notification Service

**Requirements:** channels needed (email, SMS, push)? Delivery guarantees (at-least-once vs best-effort)? Retry behavior on failure?

**High-level design:**
```
Event Source (e.g., "OrderPlaced") -> Message Queue (Kafka/RabbitMQ) -> Notification Service
                                                                              │
                                                              ┌───────────────┼───────────────┐
                                                            Email          SMS            Push
                                                            Provider     Provider       Provider
```

The producing service (e.g., Transaction Service) publishes an event and moves on — it doesn't wait for the notification to actually send. The Notification Service consumes events asynchronously, decides which channel(s) to use, and calls the relevant third-party provider (SendGrid, Twilio, FCM).

**Reliability:** failed sends go through retry with exponential backoff; persistent failures route to a Dead Letter Queue for investigation rather than being silently dropped. This is the same reliability pattern already covered in the AWS Lambda notes (retries, DLQs, idempotency) — the pattern is provider-agnostic.
