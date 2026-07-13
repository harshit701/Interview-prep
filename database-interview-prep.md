# Database Interview Prep

## What is Indexing in a database?

An index is a database data structure that improves query performance by allowing the database to quickly locate rows without scanning the entire table.

### Should we create indexes on every column?

No. Indexes improve read performance but add overhead to write operations because the index also needs to be updated. We should create indexes on columns frequently used in `WHERE` clauses, `JOIN` conditions, and sorting operations.

### You have a query with multiple WHERE conditions. How will you design an index?

```sql
SELECT *
FROM transactions
WHERE user_id = 10
AND created_at > '2026-01-01'
ORDER BY created_at DESC;

CREATE INDEX idx_transactions_user_created
ON transactions(user_id, created_at DESC);
```

## What is a composite index?

A composite index is a single index that stores multiple columns together instead of indexing only one column.

## What is Query Optimization?

Query optimization is the process of improving database queries so they execute faster while consuming fewer database resources.

### Why do we need Query Optimization?

Imagine your API endpoint is:

```
GET /dashboard
```

which calls:

```sql
SELECT * FROM transactions;
```

Your `transactions` table has 10 million rows. The database returns all 10 million records, but your frontend only needs the 20 most recent transactions.

**Problems this causes:**

- More memory usage
- More network transfer
- Slower response
- Higher database load

### Common Query Optimization Techniques

#### 1. Avoid `SELECT *`

Bad — loads every column, even ones you don't need:

```sql
-- ❌ Bad
SELECT * FROM users;
```

A `users` table might have columns like `id`, `name`, `email`, `password_hash`, `created_at`, `updated_at`, `profile_image`, `address`, etc. If you only need a few:

```sql
-- ✅ Better
SELECT id, name, email FROM users;
```

**Benefits:** less data transferred, less memory usage, faster response.

#### 2. Use Proper Indexes

Slow, with no index (full table scan):

```sql
SELECT * FROM users WHERE email = 'abc@test.com';
```

Optimized:

```sql
CREATE INDEX idx_users_email ON users(email);
-- Now this becomes an index lookup instead of a full scan
```

#### 3. Avoid the N+1 Query Problem ⭐⭐⭐⭐⭐

This is a very common backend interview question.

Imagine you need a list of users along with their orders.

**Bad approach:**

```js
// 1 query for all users
const users = await db.query("SELECT * FROM users");

// Then, inside a loop — 1 query PER user
for (const user of users) {
  await db.query("SELECT * FROM orders WHERE user_id = ?", [user.id]);
}
```

If there are 100 users, that's 1 query for users + 100 queries for orders = **101 queries**. This is the N+1 problem.

**Better approach — use a JOIN:**

```sql
SELECT
  users.id,
  users.name,
  orders.amount
FROM users
JOIN orders ON users.id = orders.user_id;
```

Now it's a single query. ✅

#### 4. Pagination

Never do this for large tables:

```sql
SELECT * FROM transactions;
```

Instead, paginate:

```sql
-- Page 1
SELECT * FROM transactions LIMIT 20 OFFSET 0;

-- Page 2
SELECT * FROM transactions LIMIT 20 OFFSET 20;
```

#### 5. Use `EXPLAIN ANALYZE` ⭐⭐⭐⭐⭐

Before optimizing a query, inspect its execution plan:

```sql
EXPLAIN ANALYZE
SELECT * FROM transactions WHERE user_id = 10;
```

This tells you:

- Whether an index was used
- How many rows were scanned
- Execution time
- The full query plan

#### 6. Avoid Unnecessary Joins

```sql
-- ❌ Bad — joins everything even if you only need the user's name
SELECT *
FROM users
JOIN orders ON ...
JOIN payments ON ...
JOIN addresses ON ...
JOIN logs ON ...;
```

If you only need the user name, don't join every related table.

## What is Connection Pooling?

Connection pooling is a technique where a set of reusable database connections are created and maintained so that applications can reuse existing connections instead of creating a new connection for every request.

### Why not create a new DB connection for every request?

Creating a database connection is expensive. Connection pooling allows reuse of existing connections, reducing latency and controlling the number of active connections.

### What happens if all connections in the pool are busy?

New requests wait until a connection becomes available, or a timeout occurs based on pool configuration.

### How do you handle database connections in a high-traffic Node.js application?

I use connection pooling to maintain a controlled number of reusable database connections. Each request borrows a connection from the pool, executes the query, and returns it. This avoids the overhead of creating connections repeatedly and prevents exhausting the database's connection limits.

## What is Redis?

Redis is an in-memory key-value data store used for caching, session management, real-time data processing, and reducing database load.

### Cache-Aside Pattern (Most Common) ⭐⭐⭐⭐⭐

This is the pattern interviewers expect.

```text
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

### What is Cache Invalidation?

Cache invalidation is the process of removing or updating cached data whenever the original data changes, so users always receive the latest information instead of stale data.

## What is Database Replication?

Database replication is the process of copying data from one database server to one or more database servers to improve scalability, availability, and reliability.

## What is a Transaction?

A database transaction is a group of one or more database operations that are treated as a single unit of work. Either all operations succeed, or all operations fail.

## What is Sharding?

Sharding is the process of splitting a large database into smaller, independent databases called shards, where each shard stores a subset of the data.

## What is SQL?

SQL databases store data in tables with rows and columns and follow a fixed schema.

**Examples:** PostgreSQL, MySQL, SQL Server, Oracle

## What is NoSQL?

NoSQL databases store data in flexible formats such as documents, key-value pairs, graphs, or columns and do not require a fixed schema.

**Examples:** MongoDB, Cassandra, DynamoDB, CouchDB

## When should you use SQL?

Choose SQL when:

- Building financial systems
- Building banking systems
- Handling e-commerce orders
- Handling payment systems
- Managing inventory
- Data has strong relationships
- ACID transactions are required

## When should you use NoSQL?

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

```text
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

```js
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

```js
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
