# Microservices Interview Prep

## What is a Monolithic Application?

Before understanding microservices, you need to know what a monolith is.

Imagine we're building an app called **Iqubec**, where everything lives inside one application:

```text
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

## What are Microservices?

Microservices is an architectural style where an application is divided into small, independent services. Each service is responsible for one specific business capability and can be developed, deployed, and scaled independently.

Instead of one application (Iqubec), we build separate services:

- Authentication Service
- User Service
- Transaction Service
- Budget Service
- Investment Service
- Notification Service
- Payment Service

```text
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

## Advantages of Microservices

- ✅ Independent deployment
- ✅ Independent scaling
- ✅ Better fault isolation
- ✅ Smaller codebase
- ✅ Easier maintenance
- ✅ Freedom to use different technologies per service

## Disadvantages of Microservices

- ❌ More infrastructure
- ❌ Network communication overhead
- ❌ Distributed debugging
- ❌ More monitoring
- ❌ Data consistency challenges

## How should the Transaction Service tell the Notification Service to send a notification? (Service-to-Service Communication)

### 1. Synchronous Communication (HTTP)

```text
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

### 2. Asynchronous Communication (Kafka)

Instead of calling the Notification Service directly:

```text
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

### Why is Kafka better here?

Suppose the Notification Service crashes.

**With HTTP:** Transaction → Notification ❌ → the whole request fails.

**With Kafka:** Transaction → Publish Event → Kafka stores event ✅ → Notification Service is down → it comes back later → consumes the event → sends the email.

The transaction still succeeds; the notification is simply delayed.

### When should you use HTTP?

Use HTTP when:

- You need an immediate response
- One service depends on another to complete the request

Examples: Authentication Service validating a JWT, Payment Service verifying a coupon, User Service fetching profile information.

### When should you use Kafka/RabbitMQ?

Use a message broker when:

- The task can happen later
- The services should be loosely coupled
- Reliability is important
- You're building an event-driven system

Examples: sending emails, push notifications, SMS, analytics, audit logs, order processing, inventory updates.

### When would you choose HTTP over Kafka? (Interview answer)

I use HTTP for synchronous communication when I need an immediate response or the result is required before continuing the request. I use Kafka or RabbitMQ for asynchronous communication when tasks can be processed later, such as sending notifications, updating analytics, or processing background jobs. This improves scalability, fault tolerance, and loose coupling between services.

## What is an Event?

An event is something that has happened in the system.

## What is Event-Driven Architecture (EDA)?

Event-Driven Architecture is an architectural pattern where services communicate by publishing and consuming events through a message broker like Kafka or RabbitMQ. It enables loose coupling, scalability, and asynchronous processing.

## What is an API Gateway?

An API Gateway is a single entry point for all client requests in a microservices architecture. It receives requests from clients, performs common tasks such as authentication, rate limiting, and routing, and forwards the requests to the appropriate microservice.
