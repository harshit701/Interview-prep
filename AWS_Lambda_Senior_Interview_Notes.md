# AWS Lambda Interview Notes (Senior)

## 1. What is AWS Lambda?

AWS Lambda is a serverless compute service that runs code in response to
events. AWS provisions, scales and manages the infrastructure. You pay
only for requests and execution duration.

## 2. Lambda Lifecycle

-   Invocation received
-   Cold start (if no execution environment exists)
-   Initialize runtime and dependencies
-   Execute handler
-   Return response
-   Warm execution environment may be reused

## 3. Cold Starts

Causes: - First invocation - Traffic spikes - New deployment - Idle
environment recycled

Reduce by: - Smaller deployment package - Lazy loading - Reusing
SDK/database clients - Optimizing initialization - Right-sizing memory -
Provisioned Concurrency - Splitting large functions - Moving heavy
workloads to ECS Fargate when appropriate

## 4. Scaling & Concurrency

-   Lambda automatically scales by creating execution environments.
-   Account concurrency limits apply.
-   Reserved Concurrency guarantees capacity for one function.
-   Provisioned Concurrency keeps environments warm.
-   If concurrency is exhausted:
    -   Synchronous requests are throttled (429/TooManyRequests).
    -   Asynchronous sources retry according to source behavior.
    -   SQS retains messages until workers become available.

## 5. Invocation Types

-   Synchronous: API Gateway, Function URL
-   Asynchronous: S3, SNS, EventBridge
-   Poll-based: SQS, Kinesis, DynamoDB Streams

## 6. Cost Optimization

-   Measure first using CloudWatch.
-   Reduce execution duration.
-   Right-size memory after benchmarking.
-   Reuse clients across warm invocations.
-   Prefer event-driven architecture over polling.
-   Avoid unnecessary retries.
-   Use Provisioned Concurrency only when justified.
-   Consider ECS Fargate for long-running compute-heavy workloads.

## 7. Troubleshooting

Check: - CloudWatch Logs - CloudWatch Metrics - Duration - Errors -
Throttles - Concurrent Executions - Memory usage - Timeout - Downstream
dependencies

## 8. Reliability

-   Retries
-   Dead Letter Queues
-   Destinations
-   Idempotency
-   Exponential backoff
-   Partial batch failure support for supported event sources

## 9. Idempotency

Purpose: Prevent duplicate processing when the same request is retried.

Flow: 1. Client generates an idempotency key once. 2. Client reuses the
same key for retries. 3. Backend stores the key and response/status. 4.
Duplicate requests return the original result.

Important: The client does NOT regenerate the same UUID; it stores and
reuses it for the same logical operation.

## 10. Database Down Scenario

-   Distinguish reads vs writes.
-   Serve cached reads where possible.
-   Queue writes using SQS.
-   Retry with exponential backoff.
-   Use DLQ.
-   Monitor with CloudWatch.
-   Protect downstream systems with circuit breaker pattern.
-   Use HA databases (e.g., Multi-AZ).

## 11. Handling 100,000 Requests

Architecture: Client -\> API Gateway -\> Lambda -\> SQS -\> Worker
Lambda -\> Database

Benefits: - Buffers spikes - Protects database - Prevents data loss -
Allows controlled processing

## 12. Cold Start Investigation

Do not assume. Verify using CloudWatch. Investigate: - Traffic -
Deployments - Package size - Initialization - Memory - VPC usage -
Dependency loading

## 13. Large Package (100--200 MB)

Options: - Provisioned Concurrency - Lazy loading - Split Lambda -
Increase memory after benchmarking - Store large assets in S3 if
appropriate - Use ECS Fargate if workload is fundamentally
container-oriented - Lambda Layers help reuse code but are not a
cold-start solution.

## 14. Lambda vs ECS Fargate

Choose Lambda for: - Event-driven - Short-lived execution - Automatic
scaling - Sporadic traffic

Choose ECS Fargate for: - Long-running services - Persistent
connections - Large dependencies - Heavy CPU/memory workloads -
Container-first applications

## 15. Common Senior Interview Answers

### How do you troubleshoot Lambda?

Start with CloudWatch metrics and logs, identify whether latency comes
from initialization, execution, downstream services, throttling or
memory pressure, then optimize based on evidence rather than
assumptions.

### How do you reduce Lambda cost?

Measure invocation count, duration, memory usage and retries. Optimize
slow code, benchmark memory, reuse clients, remove polling, and choose
the right compute service.

### How do you handle database outages?

Serve cached reads, queue writes using SQS, retry with exponential
backoff, use DLQs, monitor health, and protect dependencies with circuit
breakers.

### How do you handle traffic spikes?

Use API Gateway, Lambda auto scaling, SQS buffering, idempotency, rate
limiting, CloudWatch monitoring and load testing.

### When do you choose ECS Fargate?

When workloads are long-running, require persistent connections, large
dependencies or continuous processing. Use Lambda for short-lived
event-driven workloads.

## 16. Lesser-Known Advanced Topics

-   Lambda Extensions
-   Function URLs
-   Response Streaming
-   Container Image support
-   Ephemeral /tmp storage
-   Code Signing
-   SnapStart (Java only)
-   Canary/Blue-Green deployments using aliases

## Key Interview Tips

-   Measure before optimizing.
-   Think about downstream bottlenecks, not only Lambda.
-   Explain trade-offs, not just features.
-   Base architecture decisions on workload, latency, reliability and
    cost.
