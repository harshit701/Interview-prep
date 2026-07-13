# System Design Interview Prep

## What is System Design?

System Design is the blueprint of an application that defines how different components work together to build a scalable, reliable, and efficient system.

### Suppose I ask: "Design WhatsApp." What should you ask me first?

Don't start drawing the architecture immediately.

#### What is the first thing you should clarify with the interviewer?

Most candidates make this mistake:

> **Interviewer:** "Design WhatsApp."
> **Candidate:** "I'll use Redis, Kafka, Kubernetes..." ❌

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

> **Interviewer:** "Design WhatsApp."
> **You:** "Before I start, I'd like to clarify the requirements. Should we focus only on one-to-one messaging, or should we also include group chats, voice calls, and media sharing?"
> **Interviewer:** "Let's only design one-to-one messaging."

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

## Functional Requirements vs Non-Functional Requirements

### Functional Requirements

These answer: **what should the system do?** They describe the features.

Example — for WhatsApp:
- Send messages
- Receive messages
- Create groups
- Upload images
- Show online status

**Simple definition ⭐⭐⭐⭐⭐**
> Functional requirements define what the system should do or what features it should provide to users.

### Non-Functional Requirements

These answer: **how well should the system perform?** They describe the quality of the system.

Examples:
- Handle 10 million users
- Response time under 200ms
- 99.99% availability
- Secure
- Scalable
- Reliable

These are not features.

**Simple definition ⭐⭐⭐⭐⭐**
> Non-functional requirements define how the system should perform, including performance, scalability, reliability, availability, and security.

## What is Scalability?

Scalability is the ability of a system to handle increasing traffic or workload without significantly affecting its performance — or, put another way, the system can serve more users by increasing its capacity while maintaining good performance.

### Types of Scalability

There are two types.

#### 1. Vertical Scaling (Scale Up)

Imagine one server with 4 GB RAM and 2 CPUs. When traffic increases, instead of adding another server, you upgrade it to 64 GB RAM and 32 CPUs — the same machine, made more powerful. This is called **Vertical Scaling**.

```text
Node.js Server
      │
    2 CPU
      │
   Upgrade
      │
   16 CPU

Still one server.
```

**Advantages**
- Very easy
- No application changes

**Disadvantages**
- Very expensive
- Hardware has limits
- Single point of failure — if this server dies, users get no application at all

#### 2. Horizontal Scaling (Scale Out)

Instead of buying a bigger server, add more servers (Server 1, Server 2, Server 3, Server 4) and share the traffic between them.

```text
Instead of:
1000 Requests → One Server

You get:
1000 Requests → Load Balancer → Server 1, Server 2, Server 3, Server 4

Each server handles about 250 requests. Much easier.
```

#### Why big companies prefer horizontal scaling

Imagine Amazon with 50 million users of traffic — can one server handle this? No. Instead:

```text
Load Balancer → 100 Servers

If traffic doubles → 200 Servers
```

Simple.

### Difference between vertical scaling and horizontal scaling

Vertical Scaling increases the capacity of a single server by adding more CPU, RAM, or storage, whereas Horizontal Scaling increases system capacity by adding more servers and distributing traffic across them.

#### Your Node.js API currently handles 100 requests/sec. Tomorrow it starts receiving 2,000 requests/sec. What would you do?

First, I would identify the bottleneck by monitoring CPU, memory, database performance, network usage, and response times. If the application server is the bottleneck, I would horizontally scale the Node.js application by running multiple instances behind a load balancer. If the database becomes the bottleneck, I would optimize queries, add indexes, introduce Redis for caching, and, if needed, use read replicas or database sharding. For asynchronous tasks such as sending emails or notifications, I would offload them to a message broker like Kafka or RabbitMQ. Finally, I would continuously monitor the system and scale individual services based on traffic.

## What's the difference between Availability and Reliability?

### Availability

Availability is the ability of a system to remain accessible and usable when users need it — can users use the system right now? If yes, that's high availability.

**Example:** Suppose Amazon is running at 10:00 AM — the website opens, search works, orders work, payments work. Amazon is available. Now suppose the Amazon website returns a 500 Internal Server Error and nobody can access it — availability is low.

**Real-life analogy:** think about electricity. If it's available, you can use your TV, laptop, and AC. If it goes off, everything stops. Same idea.

### Reliability

Reliability is the ability of a system to perform its intended function correctly and consistently without failures.

**Example:** suppose you transfer ₹5,000 and the bank shows "Transaction Successful," but the recipient receives ₹0. The bank was *available* because you could use it, but it was not *reliable* because it gave the wrong result.

### Suppose you're designing UPI (Google Pay / PhonePe). Which would you prioritize more — high availability or high reliability?

**Scenario 1:** the app is available. You send ₹50,000, but money is deducted twice, the wrong account receives the money, or the transaction status is incorrect. This is a reliability failure, and it's far more serious than downtime.

**Scenario 2:** the app is down for 5 minutes. Nobody can make payments, and users are frustrated — but nobody loses money.

For a financial system, the second scenario is generally less damaging than incorrect financial transactions. That's why banks and payment systems prioritize correctness above all else.

**Interview answer:**
> "Ideally, I would design the system to be both highly available and highly reliable. However, if I had to prioritize one for a payment system like UPI, I would prioritize reliability because financial transactions must be processed correctly. Users may tolerate temporary downtime, but they cannot tolerate incorrect or duplicate transactions or loss of money."

**Different systems have different priorities:**

| System | Higher Priority |
|---|---|
| Banking / UPI | ✅ Reliability |
| WhatsApp | Availability |
| Netflix | Availability |
| Google Search | Availability |
| Stock Trading | Reliability |
| Hospital System | Reliability |

## Latency vs Throughput

> **Latency** = how fast?
> **Throughput** = how much?

### Latency

Latency is the time taken by the system to process a single request and return a response.

**Example:** you open YouTube and click Play — the video starts in 200ms, so latency is 200ms. Or: you call `GET /users` and the server responds in 150ms — latency is 150 milliseconds. Lower latency is better.

### Throughput

Throughput is the number of requests or operations a system can process in a given period of time.

**Example:** if your API can process 500 requests/second, its throughput is 500 requests/sec. If tomorrow it handles 5,000 requests/sec, it has a much higher throughput.

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

## What is a Load Balancer?

A Load Balancer is a component that distributes incoming client requests across multiple servers so that no single server becomes overloaded.

## What is a Health Check?

A Health Check is a periodic request sent by the Load Balancer to verify that a server is healthy and able to serve requests.

## What is a Reverse Proxy?

A Reverse Proxy is a server that sits between clients and backend servers, receives client requests, and forwards them to the appropriate backend server without exposing the backend directly.

## What is a Content Delivery Network (CDN)?

A CDN (Content Delivery Network) is a network of geographically distributed servers that stores copies of static content closer to users, reducing latency and improving loading speed — it stores your content closer to users so they don't have to fetch it from the main server every time.

### What does a CDN store?

A CDN is best for static content that doesn't change very often, such as:

- Images
- CSS
- JavaScript
- Fonts
- Videos
- PDFs
- Downloadable files

## What is the Cache-Aside Pattern (Lazy Loading)?

In the Cache-Aside pattern, the application first checks the cache. If the data is not found, it fetches it from the database, stores it in the cache, and then returns it to the client.
