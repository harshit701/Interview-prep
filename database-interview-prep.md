# Database Interview Prep

## ACID & Transactions

### What is a Transaction?

A database transaction is a group of one or more database operations that are treated as a single unit of work. Either all operations succeed, or all operations fail.

### What does ACID stand for?

- **Atomicity** — a transaction's operations either all succeed or all fail together; there's no partial completion.
- **Consistency** — a transaction moves the database from one valid state to another, respecting all constraints (foreign keys, unique constraints, etc).
- **Isolation** — concurrent transactions don't interfere with each other's intermediate states.
- **Durability** — once a transaction commits, the change survives even a crash immediately after (written to durable storage).

```
BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;
```
If the second `UPDATE` fails, Atomicity guarantees the first one is rolled back too — you never end up with money deducted from one account but not credited to the other.

### What are the Isolation Levels, and what problem does each solve?

Isolation levels control how much concurrent transactions can "see" of each other's uncommitted or in-progress changes. From weakest to strongest:

- **Read Uncommitted** — a transaction can see another transaction's uncommitted changes. Allows **dirty reads** (reading data that might get rolled back).
- **Read Committed** — a transaction only sees committed data. Prevents dirty reads, but allows **non-repeatable reads** (reading the same row twice in one transaction can return different values if another transaction committed a change in between).
- **Repeatable Read** — guarantees the same row read twice returns the same value within a transaction. Prevents non-repeatable reads, but can still allow **phantom reads** (a repeated range query can return new rows that were inserted by another transaction).
- **Serializable** — the strongest level; transactions behave as if executed one at a time, sequentially. Prevents all of the above, at the cost of the most locking/performance overhead.

```
Dirty Read:          Transaction A reads uncommitted data from Transaction B, which then rolls back.
Non-repeatable Read: Transaction A reads a row twice, gets different values because B committed a change in between.
Phantom Read:        Transaction A runs the same range query twice, gets a different SET of rows because B inserted a new matching row.
```

**PostgreSQL's default is Read Committed.** Most applications don't need `Serializable` — it's reserved for cases where correctness under heavy concurrent writes (e.g., financial ledgers, seat booking systems) is critical enough to justify the performance cost.

### Optimistic vs Pessimistic Locking

**Pessimistic locking** — lock the row before reading it, preventing any other transaction from touching it until you're done.
```
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- other transactions trying to update this row will block until this one commits/rolls back
```
Good when conflicts are frequent and you'd rather block than retry.

**Optimistic locking** — don't lock; instead, track a version number, and check it hasn't changed before committing.
```
UPDATE accounts SET balance = 900, version = version + 1
WHERE id = 1 AND version = 5;
-- if another transaction already changed the version, this UPDATE affects 0 rows -> conflict detected
```
Good when conflicts are rare — avoids the overhead of holding locks, but requires the application to handle the "someone else updated this first" case (usually by retrying).

## Indexing & Query Optimization

### What is Indexing in a database?

An index is a database data structure that improves query performance by allowing the database to quickly locate rows without scanning the entire table.

#### Should we create indexes on every column?

No. Indexes improve read performance but add overhead to write operations because the index also needs to be updated. We should create indexes on columns frequently used in `WHERE` clauses, `JOIN` conditions, and sorting operations.

#### You have a query with multiple WHERE conditions. How will you design an index?

```
SELECT *
FROM transactions
WHERE user_id = 10
AND created_at > '2026-01-01'
ORDER BY created_at DESC;

CREATE INDEX idx_transactions_user_created
ON transactions(user_id, created_at DESC);
```

### What is a composite index?

A composite index is a single index that stores multiple columns together instead of indexing only one column.

### What are the types of SQL Joins?

```
INNER JOIN  - only rows that match in both tables
LEFT JOIN   - all rows from the left table, matched rows from the right (NULL if no match)
RIGHT JOIN  - all rows from the right table, matched rows from the left (NULL if no match)
FULL JOIN   - all rows from both tables, NULL where there's no match on either side
```

```
SELECT users.name, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id;
-- only users who have at least one order appear

SELECT users.name, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
-- ALL users appear, orders.amount is NULL for users with no orders
```

**Real-world use case for LEFT JOIN:** "show all users, and their order count if they have one" — you want every user in the result even if `orders.amount` is `NULL`, which `INNER JOIN` would silently exclude.

### What is Query Optimization?

Query optimization is the process of improving database queries so they execute faster while consuming fewer database resources.

#### Why do we need Query Optimization?

Imagine your API endpoint is:

```
GET /dashboard
```

which calls:

```
SELECT * FROM transactions;
```

Your `transactions` table has 10 million rows. The database returns all 10 million records, but your frontend only needs the 20 most recent transactions.

**Problems this causes:**

- More memory usage
- More network transfer
- Slower response
- Higher database load

#### Common Query Optimization Techniques

**1. Avoid `SELECT *`**

Bad — loads every column, even ones you don't need:

```
-- ❌ Bad
SELECT * FROM users;
```

A `users` table might have columns like `id`, `name`, `email`, `password_hash`, `created_at`, `updated_at`, `profile_image`, `address`, etc. If you only need a few:

```
-- ✅ Better
SELECT id, name, email FROM users;
```

**Benefits:** less data transferred, less memory usage, faster response.

**2. Use Proper Indexes**

Slow, with no index (full table scan):

```
SELECT * FROM users WHERE email = 'abc@test.com';
```

Optimized:

```
CREATE INDEX idx_users_email ON users(email);
-- Now this becomes an index lookup instead of a full scan
```

**3. Avoid the N+1 Query Problem ⭐⭐⭐⭐⭐**

This is a very common backend interview question.

Imagine you need a list of users along with their orders.

Bad approach:

```
// 1 query for all users
const users = await db.query("SELECT * FROM users");

// Then, inside a loop — 1 query PER user
for (const user of users) {
  await db.query("SELECT * FROM orders WHERE user_id = ?", [user.id]);
}
```

If there are 100 users, that's 1 query for users + 100 queries for orders = **101 queries**. This is the N+1 problem.

Better approach — use a JOIN:

```
SELECT
  users.id,
  users.name,
  orders.amount
FROM users
JOIN orders ON users.id = orders.user_id;
```

Now it's a single query. ✅

**4. Pagination**

Never do this for large tables:

```
SELECT * FROM transactions;
```

Instead, paginate:

```
-- Page 1
SELECT * FROM transactions LIMIT 20 OFFSET 0;

-- Page 2
SELECT * FROM transactions LIMIT 20 OFFSET 20;
```

**5. Use `EXPLAIN ANALYZE` ⭐⭐⭐⭐⭐**

Before optimizing a query, inspect its execution plan:

```
EXPLAIN ANALYZE
SELECT * FROM transactions WHERE user_id = 10;
```

This tells you:

- Whether an index was used
- How many rows were scanned
- Execution time
- The full query plan

**6. Avoid Unnecessary Joins**

```
-- ❌ Bad — joins everything even if you only need the user's name
SELECT *
FROM users
JOIN orders ON ...
JOIN payments ON ...
JOIN addresses ON ...
JOIN logs ON ...;
```

If you only need the user name, don't join every related table.

## Normalization

### What is Database Normalization?

Normalization is the process of organizing a database schema to reduce data redundancy and prevent update anomalies, by splitting data into related tables rather than repeating it.

```
-- Unnormalized: repeats customer info on every order row
Orders: id, customer_name, customer_email, product, amount

-- Normalized: customer info lives once, referenced by ID
Customers: id, name, email
Orders: id, customer_id, product, amount
```

If a customer changes their email, the unnormalized version requires updating every order row they've ever placed — a real bug source (some rows updated, some missed). The normalized version requires updating exactly one row.

### When would you deliberately denormalize?

For read-heavy systems where join performance matters more than write-time consistency risk — e.g., a reporting/analytics table that pre-joins and flattens data for fast dashboard queries, accepting that it needs to be refreshed/synced rather than always perfectly live. Denormalization trades write complexity and redundancy for read speed.

## Connection Management

### What is Connection Pooling?

Connection pooling is a technique where a set of reusable database connections are created and maintained so that applications can reuse existing connections instead of creating a new connection for every request.

#### Why not create a new DB connection for every request?

Creating a database connection is expensive. Connection pooling allows reuse of existing connections, reducing latency and controlling the number of active connections.

#### What happens if all connections in the pool are busy?

New requests wait until a connection becomes available, or a timeout occurs based on pool configuration.

#### How do you handle database connections in a high-traffic Node.js application?

I use connection pooling to maintain a controlled number of reusable database connections. Each request borrows a connection from the pool, executes the query, and returns it. This avoids the overhead of creating connections repeatedly and prevents exhausting the database's connection limits.

## Caching with Redis

### What is Redis?

Redis is an in-memory key-value data store used for caching, session management, real-time data processing, and reducing database load.

#### Cache-Aside Pattern (Most Common) ⭐⭐⭐⭐⭐

This is the pattern interviewers expect.

```
Request
  │
  ▼
Check Redis ── Data exists? ── Yes ──▶ Return cached data
  │
  No
  │
  ▼
Query Database
  │
  ▼
Store result in Redis
  │
  ▼
Return response
```

In the Cache-Aside pattern, the application first checks the cache. If the data is not found, it fetches it from the database, stores it in the cache, and then returns it to the client.
> This same pattern is also referenced from a system-design angle (load balancers, CDNs, etc.) in the **System Design Interview Prep** file, but the mechanics are identical.

#### What is Cache Invalidation?

Cache invalidation is the process of removing or updating cached data whenever the original data changes, so users always receive the latest information instead of stale data.

## Data Architecture & Reliability

### What is Database Replication?

Database replication is the process of copying data from one database server to one or more database servers to improve scalability, availability, and reliability.

### What is the CAP Theorem?

In a distributed database system, you can only guarantee two of the following three at the same time:

- **Consistency** — every read receives the most recent write (or an error).
- **Availability** — every request receives a response (not necessarily the latest data).
- **Partition Tolerance** — the system continues operating despite network failures between nodes.

Since network partitions are a real possibility in any distributed system, Partition Tolerance is generally non-negotiable — so in practice, the real-world choice is between **Consistency** and **Availability** when a partition occurs.

```
CP system (e.g., traditional relational DB with synchronous replication):
  Prioritizes correctness -> may refuse requests during a partition rather than risk stale data.

AP system (e.g., many NoSQL databases like Cassandra, DynamoDB by default):
  Prioritizes staying responsive -> may serve slightly stale data during a partition.
```

### Master-Slave vs Master-Master Replication

**Master-Slave (Primary-Replica)** — one node accepts writes, others replicate from it and serve reads. Simple, avoids write conflicts, but the master is a single point of failure for writes, and replicas can lag behind (**replication lag**), meaning a read right after a write might not reflect it yet.

**Master-Master (Multi-Primary)** — multiple nodes accept writes, replicating to each other. Higher write availability, but introduces conflict resolution complexity when the same data is written differently on two masters at nearly the same time.

Most systems default to master-slave/primary-replica for its simplicity, reaching for master-master only when write throughput or write-availability requirements genuinely demand it.

### What is a Transaction? (Sharding context)

Covered above under ACID — see also Sharding below for how large datasets are split across servers.

### What is Sharding?

Sharding is the process of splitting a large database into smaller, independent databases called shards, where each shard stores a subset of the data.

## ORMs

### What are the trade-offs of using an ORM (e.g., Prisma) vs raw SQL?

**Pros:** type safety (especially with Prisma + TypeScript), faster development for standard CRUD, protection from SQL injection by default, easier to reason about across a large codebase.

**Cons:** can generate inefficient queries if used carelessly (the N+1 problem is the classic ORM footgun), an abstraction layer that can obscure what's actually happening at the database level, and complex queries (deep joins, window functions, CTEs) are sometimes easier to write as raw SQL than to express through an ORM's query builder.

**Practical stance for an interview answer:** use the ORM for standard data access, but be comfortable dropping to raw SQL (most ORMs, including Prisma, support raw queries) for complex reporting queries or when you've confirmed via `EXPLAIN ANALYZE` that the ORM-generated query isn't performant enough.

## SQL vs NoSQL

### What is SQL?

SQL databases store data in tables with rows and columns and follow a fixed schema.

**Examples:** PostgreSQL, MySQL, SQL Server, Oracle

### What is NoSQL?

NoSQL databases store data in flexible formats such as documents, key-value pairs, graphs, or columns and do not require a fixed schema.

**Examples:** MongoDB, Cassandra, DynamoDB, CouchDB

### When should you use SQL?

Choose SQL when:

- Building financial systems
- Building banking systems
- Handling e-commerce orders
- Handling payment systems
- Managing inventory
- Data has strong relationships
- ACID transactions are required

### When should you use NoSQL?

Choose NoSQL when:

- Schema changes frequently
- You're dealing with large amounts of unstructured data
- Building social media feeds
- Building product catalogs
- Storing chat messages
- Storing logs
- Doing analytics

### When would you choose which database?

#### When to choose PostgreSQL (SQL)

Choose PostgreSQL when:

**1. Data has strong relationships**

```
Users
 │
Orders
 │
Payments
 │
Invoices
```

Everything is connected.

**2. Transactions are critical**

Examples: banking, UPI, e-commerce payments, stock trading, finance. You cannot afford inconsistent data.

**3. Complex queries**

Example: *"Show all users who purchased Product A in the last 30 days and spent more than ₹50,000."* SQL excels at these kinds of queries.

**4. Reporting**

Examples: monthly reports, financial reports, analytics dashboards.

**5. ACID properties matter**

If losing or corrupting data is unacceptable.

**Examples — choose PostgreSQL for:**

- FinPilot
- Banking system
- Payroll system
- Hospital management
- Airline booking
- Amazon orders

#### When to choose MongoDB (NoSQL)

Choose MongoDB when:

**1. Schema changes frequently**

```
// Today
{ "name": "Harshit" }

// Tomorrow
{
  "name": "Harshit",
  "skills": ["Node.js"]
}

// Next week
{
  "name": "Harshit",
  "skills": [],
  "socialLinks": {}
}
```

No migrations needed.

**2. Data is document-based**

Example — an Instagram post:

```
{
  "caption": "...",
  "comments": [],
  "likes": [],
  "hashtags": []
}
```

Everything belongs to one document.

**3. Massive horizontal scaling**

Applications with millions of users or billions of documents.

**4. Rapid development**

Requirements change frequently — MongoDB is very flexible.

**Examples — choose MongoDB for:**

- Instagram posts
- Facebook posts
- Product catalogs
- CMS
- Blogs
- Logs
- IoT data
