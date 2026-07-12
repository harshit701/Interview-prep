## What is a Monolithic Application?

```
Before understanding Microservices, you need to know what a Monolith is.

Imagine we're building Iqubec.

Everything is inside one application.

Iqubec

├── Authentication
├── Users
├── Transactions
├── Budget
├── Investments
├── Notifications
├── Reports
└── Payments

When you run it:

One Application

↓

One Server

↓

One Database

Everything is deployed together.

Advantages
Easy to build
Easy to debug
Easy to deploy (initially)
Problems

Suppose only the Notification module has a bug.

Can we deploy only Notification?

❌ No.

We must deploy the entire application.

Suppose Transactions receives:

10,000 requests/sec

Notifications receive:

100 requests/sec

Can we scale only Transactions?

❌ No.

We must scale the entire application.

That wastes CPU and memory.
```

## What are Microservices?

Microservices is an architectural style where an application is divided into small, independent services. Each service is responsible for one specific business capability and can be developed, deployed, and scaled independently.

```
Instead of one application:

Iqubec

We build:

Authentication Service

User Service

Transaction Service

Budget Service

Investment Service

Notification Service

Payment Service
```

```
                Client

                  |

             API Gateway

      /      |      |      \

 Auth   Users   Transactions   Budget

                 |

            PostgreSQL

 Notification

                 |

              Redis

 Payment

                 |

         Payment Gateway

Each service:

Has its own codebase
Can be deployed independently
Can have its own database (often)
Can scale independently
```

## Advantages of Microservices

✅ Independent deployment

✅ Independent scaling

✅ Better fault isolation

✅ Smaller codebase

✅ Easier maintenance

✅ Different technologies if needed

## Disadvantages of Microservices

❌ More infrastructure

❌ Network communication

❌ Distributed debugging

❌ More monitoring

❌ Data consistency challenges

## How should the Transaction Service tell the Notification Service to send a notification? OR What is Service-to-Service Communication.

#### 1. Synchronous Communication (HTTP)

```

User

↓

Transaction Service

↓

HTTP Request

↓

Notification Service

↓

Response

↓

Transaction Service finishes
Flow
Transaction created
Transaction Service calls Notification Service
Notification Service sends email
Returns response
Transaction Service completes
Problem

If Notification Service is down:

Transaction Service

↓

HTTP Request

↓

Notification Service ❌

↓

Request fails

Now your transaction may fail even though the transaction itself was created successfully.

This is called tight coupling.

```

#### 2. Asynchronous Communication (Kafka)

```


Instead of calling Notification Service directly:

User

↓

Transaction Service

↓

Kafka

↓

Notification Service

↓

Send Email
Flow
Transaction Service creates transaction.
It publishes an event:
TransactionCreated
Kafka stores the event.
Notification Service consumes the event whenever it's available.
Notification Service sends the email.

Notice something important:

The Transaction Service doesn't wait.

It finishes immediately.
```

### Why is Kafka Better Here?

```
Suppose Notification Service crashes.

With HTTP:

Transaction

↓

Notification ❌

↓

Whole request fails

With Kafka:

Transaction

↓

Publish Event

↓

Kafka stores event ✅

↓

Notification Service is down

↓

Notification Service comes back later

↓

Consumes event

↓

Sends email

The transaction succeeds, and the notification is simply delayed.
```

#### When Should You Use HTTP?

```


Use HTTP when:

You need an immediate response.
One service depends on another to complete the request.

Examples:

Authentication Service validates a JWT.
Payment Service verifies a coupon.
User Service fetches profile information.
```

#### When Should You Use Kafka/RabbitMQ?

```


Use a message broker when:

The task can happen later.
The services should be loosely coupled.
Reliability is important.
You're building an event-driven system.

Examples:

Sending emails
Push notifications
SMS
Analytics
Audit logs
Order processing
Inventory updates
```

### When would you choose HTTP over Kafka?

I use HTTP for synchronous communication when I need an immediate response or the result is required before continuing the request. I use Kafka or RabbitMQ for asynchronous communication when tasks can be processed later, such as sending notifications, updating analytics, or processing background jobs. This improves scalability, fault tolerance, and loose coupling between services

## What is an Event?

An event is something that has happened in the system.

## What is Event-Driven Architecture (EDA)?

Event-Driven Architecture is an architectural pattern where services communicate by publishing and consuming events through a message broker like Kafka or RabbitMQ. It enables loose coupling, scalability, and asynchronous processing.
