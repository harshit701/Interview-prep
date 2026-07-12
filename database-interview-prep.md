## What is Indexing in database?

An index is a database data structure that improves query performance by allowing the database to quickly locate rows without scanning the entire table.

### Should we create indexes on every column?

No. Indexes improve read performance but add overhead to write operations because the index also needs to be updated. We should create indexes on columns frequently used in WHERE clauses, JOIN conditions, and sorting operations.

### You have a query with multiple WHERE conditions. How will you design an index?

```
SELECT *
FROM transactions
WHERE user_id = 10
AND created_at > '2026-01-01'
ORDER BY created_at DESC;
```

CREATE INDEX idx_transactions_user_created
ON transactions(user_id, created_at DESC);

## What is composite index?

A composite index is a single index that stores multiple columns together instead of indexing only one column.

## What is Query Optimization?

Query optimization is the process of improving database queries so they execute faster while consuming fewer database resources.

```
Why do we need Query Optimization?

Imagine:

Your API:

GET /dashboard

calls:

SELECT *
FROM transactions;

Your table:

transactions
--------------
10 million rows

The database returns:

10 million records

but your frontend only needs:

20 recent transactions

Problems:

More memory usage
More network transfer
Slower response
Higher database load
```

```
Common Query Optimization Techniques
1. Avoid SELECT *

❌ Bad:

SELECT *
FROM users;

Why?

Because it loads every column.

Example:

id
name
email
password_hash
created_at
updated_at
profile_image
address
...

Maybe you only need:

SELECT id, name, email
FROM users;

Benefits:

Less data transferred
Less memory usage
Faster response

2. Use Proper Indexes

Example:

Slow:

SELECT *
FROM users
WHERE email='abc@test.com';

If no index:

Full table scan

Optimization:

CREATE INDEX idx_users_email
ON users(email);

Now:

Index lookup

3. Avoid N+1 Query Problem ⭐⭐⭐⭐⭐

This is a very common backend interview question.

Imagine:

You need:

Users
+
Their Orders

Bad approach:

First query:

SELECT *
FROM users;

Result:

100 users

Then inside a loop:

for(user of users){
   SELECT *
   FROM orders
   WHERE user_id=user.id
}

Database calls:

1 query for users

+

100 queries for orders

Total:

101 queries ❌

This is N+1 problem.

Better Approach

Use JOIN:

SELECT
users.id,
users.name,
orders.amount

FROM users

JOIN orders
ON users.id = orders.user_id;

Now:

1 query ✅


4. Pagination

Never:

SELECT *
FROM transactions;

For millions of rows.

Instead:

SELECT *
FROM transactions
LIMIT 20
OFFSET 0;

Example:

Page 1:

LIMIT 20 OFFSET 0

Page 2:

LIMIT 20 OFFSET 20

5. Use EXPLAIN ANALYZE ⭐⭐⭐⭐⭐

Before optimizing:

SELECT *
FROM transactions
WHERE user_id=10;

Run:

EXPLAIN ANALYZE
SELECT *
FROM transactions
WHERE user_id=10;

It tells:

Did it use index?
How many rows scanned?
Execution time
Query plan

6. Avoid Unnecessary Joins

Bad:

SELECT *
FROM users
JOIN orders
JOIN payments
JOIN addresses
JOIN logs;

If you only need user name:

Don't join everything.
```

## What is Connection Pooling?

Connection pooling is a technique where a set of reusable database connections are created and maintained so that applications can reuse existing connections instead of creating a new connection for every request.

### Why not create a new DB connection for every request?

Creating a database connection is expensive. Connection pooling allows reuse of existing connections, reducing latency and controlling the number of active connections.

### What happens if all connections in the pool are busy?

New requests wait until a connection becomes available or timeout occurs based on pool configuration.

### How do you handle database connections in a high traffic Node.js application?

I use connection pooling to maintain a controlled number of reusable database connections. Each request borrows a connection from the pool, executes the query, and returns it back. This avoids the overhead of creating connections repeatedly and prevents exhausting database connection limits.

## What is Redis?

Redis is an in-memory key-value data store used for caching, session management, real-time data processing, and reducing database load.

```
### Cache-Aside Pattern (Most Common) ⭐⭐⭐⭐⭐

This is the pattern interviewers expect.

Flow:

Request
   |
   ↓
Check Redis
   |
   |
   +---- Data exists?
   |          |
   |          Yes
   |          |
   |       Return data
   |
   |
   No
   |
   ↓
Query Database
   |
   ↓
Store result in Redis
   |
   ↓
Return response
```

### What is Cache Invalidation?

Cache invalidation is the process of removing or updating cached data whenever the original data changes so users always receive the latest information instead of stale data.

## What is Database Replication?

Database replication is the process of copying data from one database server to one or more database servers to improve scalability, availability, and reliability.

## What is Transactions?

A database transaction is a group of one or more database operations that are treated as a single unit of work. Either all operations succeed, or all operations fail.

## What is Sharding?

Sharding is the process of splitting a large database into smaller independent databases called shards, where each shard stores a subset of the data.
