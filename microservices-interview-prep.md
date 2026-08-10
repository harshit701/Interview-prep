# Microservices Interview Prep

## Fundamentals

### What is a Monolithic Application?

Before understanding microservices, you need to know what a monolith is.

Imagine we're building an app called **Iqubec**, where everything lives inside one application:

```
Iqubec
├── Authentication
├── Users
├── Transactions
├── Budget
├── Investments
├── Notifications
├── Reports
└── Payments
```

When you run it, it's one application, deployed to one server, backed by one database. Everything is deployed together.

**Advantages**

- Easy to build
- Easy to debug
- Easy to deploy (initially)

**Problems**

Suppose only the Notification module has a bug — can we deploy just Notification? ❌ No, we must deploy the entire application.

Suppose Transactions receives 10,000 requests/sec while Notifications receives 100 requests/sec — can we scale only Transactions? ❌ No, we must scale the entire application, which wastes CPU and memory.

### What are Microservices?

Microservices is an architectural style where an application is divided into small, independent services. Each service is responsible for one specific business capability and can be developed, deployed, and scaled independently.

Instead of one application (Iqubec), we build separate services:

- Authentication Service
- User Service
- Transaction Service
- Budget Service
- Investment Service
- Notification Service
- Payment Service

```
                 Client
                   │
              API Gateway
   ┌──────┬────────┬────────┐
 Auth   Users  Transactions  Budget
                     │
                PostgreSQL

 Notification
      │
    Redis

 Payment
      │
Payment Gateway
```

Each service:

- Has its own codebase
- Can be deployed independently
- Can have its own database (often)
- Can scale independently

### Should each microservice have its own database?

Generally yes — this is the "Database per Service" pattern, and it's one of the defining traits that separates real microservices from a monolith split into multiple deployables that still share one database.

**Why it matters:** if two services share a database, a schema change in one can silently break the other, and services become coupled at the data layer even though they're deployed independently — undermining the whole point of splitting them apart.

**The cost:** you lose the ability to do a simple SQL `JOIN` across data owned by different services. Combining data from multiple services now requires either an API call between them, or a pattern like API Composition or CQRS (see below).

### What is the API Composition pattern?

When a client needs data that spans multiple services (e.g., an order summary combining Order Service + User Service + Payment Service data), a composing service (often the API Gateway, or a dedicated backend-for-frontend layer) calls each relevant service and merges the results before returning to the client.

```
Client -> Composer -> Order Service
                    -> User Service
                    -> Payment Service
        <- merged response
```

**Downside:** this can be slow if you have many services to call, and it's still doing the "join" at the application layer rather than the database layer — for very read-heavy composed views, some teams instead maintain a separate denormalized read-model (CQRS) that's kept in sync via events.

### Advantages of Microservices

- ✅ Independent deployment
- ✅ Independent scaling
- ✅ Better fault isolation
- ✅ Smaller codebase
- ✅ Easier maintenance
- ✅ Freedom to use different technologies per service

### Disadvantages of Microservices

- ❌ More infrastructure
- ❌ Network communication overhead
- ❌ Distributed debugging
- ❌ More monitoring
- ❌ Data consistency challenges

## Service Discovery

### What is Service Discovery, and why do microservices need it?

In a monolith, components call each other via direct in-process function calls. In microservices, Service A needs to know Service B's network location (IP/port) to call it — but in a dynamically scaled environment (containers being created/destroyed, instances scaling up/down), IPs change constantly. Service Discovery solves this by maintaining a live registry of which service instances exist and where they currently are.

**Client-side discovery:** the calling service queries a registry (e.g., Consul, Eureka) directly to find an available instance, then calls it.

**Server-side discovery:** the calling service calls a fixed address (e.g., a load balancer or the API Gateway), which itself looks up the registry and routes the request — this is more common in Kubernetes-based deployments, where the built-in service networking layer handles this transparently.

## Service-to-Service Communication

### How should the Transaction Service tell the Notification Service to send a notification?

#### 1. Synchronous Communication (HTTP)

```
User
 │
 ▼
Transaction Service
 │
 ▼ HTTP Request
Notification Service
 │
 ▼ Response
Transaction Service finishes
```

**Flow:**

1. Transaction is created
2. Transaction Service calls Notification Service
3. Notification Service sends the email
4. Notification Service returns a response
5. Transaction Service completes

**Problem:** if the Notification Service is down, the HTTP request fails, and the transaction may fail even though it was created successfully. This is called **tight coupling**.

#### 2. Asynchronous Communication (Kafka)

Instead of calling the Notification Service directly:

```
User
 │
 ▼
Transaction Service
 │
 ▼
Kafka
 │
 ▼
Notification Service
 │
 ▼
Send Email
```

**Flow:**

1. Transaction Service creates the transaction
2. It publishes a `TransactionCreated` event
3. Kafka stores the event
4. Notification Service consumes the event whenever it's available
5. Notification Service sends the email

Notice: the Transaction Service doesn't wait — it finishes immediately.

#### Why is Kafka better here?

Suppose the Notification Service crashes.

**With HTTP:** Transaction → Notification ❌ → the whole request fails.

**With Kafka:** Transaction → Publish Event → Kafka stores event ✅ → Notification Service is down → it comes back later → consumes the event → sends the email.

The transaction still succeeds; the notification is simply delayed.

#### When should you use HTTP?

Use HTTP when:

- You need an immediate response
- One service depends on another to complete the request

Examples: Authentication Service validating a JWT, Payment Service verifying a coupon, User Service fetching profile information.

#### When should you use Kafka/RabbitMQ?

Use a message broker when:

- The task can happen later
- The services should be loosely coupled
- Reliability is important
- You're building an event-driven system

Examples: sending emails, push notifications, SMS, analytics, audit logs, order processing, inventory updates.

#### When would you choose HTTP over Kafka? (Interview answer)

I use HTTP for synchronous communication when I need an immediate response or the result is required before continuing the request. I use Kafka or RabbitMQ for asynchronous communication when tasks can be processed later, such as sending notifications, updating analytics, or processing background jobs. This improves scalability, fault tolerance, and loose coupling between services.

## Distributed Transactions & the Saga Pattern

### How do you handle a transaction that spans multiple services (each with its own database)?

Since each service has its own database, you can't use a single ACID transaction across all of them the way you could in a monolith with one shared database. The common solution is the **Saga pattern** — a sequence of local transactions, each in a different service, coordinated through events, with explicit **compensating actions** to undo previous steps if a later step fails.

```
Example: placing an order that involves Order Service, Payment Service, Inventory Service

1. Order Service creates order (local transaction) -> publishes "OrderCreated"
2. Payment Service charges the card (local transaction) -> publishes "PaymentSucceeded"
3. Inventory Service reserves stock (local transaction) -> publishes "StockReserved"

If step 3 fails (out of stock):
  -> Inventory Service publishes "StockReservationFailed"
  -> Payment Service consumes this, REFUNDS the charge (compensating action)
  -> Order Service consumes this, marks the order as CANCELLED (compensating action)
```

Each service only ever manages its own local transaction; correctness across the whole flow comes from every step having a corresponding compensating action if something downstream fails.

### Choreography-based Saga vs Orchestration-based Saga

**Choreography** — no central coordinator; each service listens for events and decides what to do next, publishing its own events in response (the example above). Simple for a small number of steps, but can become hard to trace/debug as the number of services involved grows — there's no single place that shows the whole flow.

**Orchestration** — a central orchestrator service explicitly tells each participating service what to do, step by step, and handles compensating actions if a step fails. Easier to understand and debug (the whole flow lives in one place), but introduces a coordinating component that itself needs to be reliable.

## Architecture Components

### What is an API Gateway?

An API Gateway is a single entry point for all client requests in a microservices architecture. It receives requests from clients, performs common tasks such as authentication, rate limiting, and routing, and forwards the requests to the appropriate microservice.

## Event-Driven Concepts

### What is an Event?

An event is something that has happened in the system.

### What is Event-Driven Architecture (EDA)?

Event-Driven Architecture is an architectural pattern where services communicate by publishing and consuming events through a message broker like Kafka or RabbitMQ. It enables loose coupling, scalability, and asynchronous processing.

## Observability in Microservices

### Why is distributed tracing necessary in a microservices architecture?

In a monolith, a stack trace shows you the full call path for a single request. In microservices, a single user-facing request might touch 5+ services — without a way to correlate logs across all of them, debugging "why was this request slow" or "where did this error actually originate" becomes extremely difficult.

**Distributed tracing** solves this by attaching a unique **trace ID** to a request at the entry point (API Gateway), and propagating it through every downstream service call (usually via an HTTP header). Each service logs using that same trace ID, and tools like Jaeger or Zipkin can then reconstruct the full request path across all services, showing exactly how much time was spent in each hop.

```
Request -> API Gateway (trace-id: abc123)
              │
              ▼
         Order Service (trace-id: abc123) -- 50ms
              │
              ▼
         Payment Service (trace-id: abc123) -- 200ms  <- bottleneck visible here
```

### What is a Service Mesh?

A service mesh (e.g., Istio, Linkerd) is an infrastructure layer that handles service-to-service communication concerns — retries, timeouts, circuit breaking, mutual TLS, load balancing, and observability — outside of the application code, typically via a sidecar proxy deployed alongside each service instance. This lets teams add these cross-cutting concerns uniformly across all services without every service individually implementing its own retry/circuit-breaker logic.
