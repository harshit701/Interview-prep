# REST API Interview Prep

## Fundamentals

### What is REST?

REST (Representational State Transfer) is an architectural style for designing networked APIs. It defines a set of constraints — statelessness, a uniform interface, resource-based URLs, and use of standard HTTP methods — that make APIs predictable, scalable, and easy to consume.

### What makes an API "RESTful"?

An API is RESTful when it follows REST's core constraints:

- **Statelessness** — the server stores no client session state between requests; every request contains all the information needed to process it.
- **Uniform interface** — resources are identified by URLs, and standard HTTP methods (GET, POST, PUT, PATCH, DELETE) act on them consistently.
- **Client-server separation** — frontend and backend evolve independently.
- **Cacheable responses** — responses explicitly indicate whether they can be cached.
- **Resource-based** — URLs represent nouns (`/users`, `/orders/123`), not actions (`/getUser`, `/createOrder`).

### Why does statelessness matter in practice?

If the server stored session state per client, you couldn't scale horizontally without sticky sessions — a request would need to hit the exact server that holds that client's session. With statelessness (e.g., a JWT sent on every request instead of a server-side session), any server instance behind a load balancer can handle any request, which is what makes horizontal scaling straightforward.

## HTTP Methods & Status Codes

### What are the standard HTTP methods and what do they mean?

- **GET** — retrieve a resource, no side effects (safe, idempotent)
- **POST** — create a new resource, or trigger a non-idempotent action
- **PUT** — replace a resource entirely (idempotent)
- **PATCH** — partially update a resource (not necessarily idempotent)
- **DELETE** — remove a resource (idempotent)

### What is Idempotency?

An operation is idempotent if calling it multiple times produces the same result as calling it once. `GET`, `PUT`, and `DELETE` are idempotent by specification. `POST` is not.

```
PUT /users/1  { "name": "Harshit" }
```
Calling this 5 times leaves the user in the exact same state as calling it once. Compare:
```
POST /orders  { "item": "Laptop" }
```
Calling this 5 times creates 5 separate orders — not idempotent.

#### Why does idempotency matter for a real system?

Network failures happen. If a client sends a `POST /payments` request and the response is lost due to a timeout, the client doesn't know if the payment succeeded. If it retries blindly, a non-idempotent endpoint could double-charge the user.

#### How do you make a POST endpoint idempotent?

Use an **idempotency key** — a unique client-generated identifier sent with the request (often a UUID).

```
POST /payments
Idempotency-Key: 8f14e45f-ceea-4f8f-a6ce-...
{ "amount": 500 }
```

The server stores the key alongside the result of the first successful request. If the same key arrives again, the server returns the stored result instead of processing the payment a second time.

```
1. Client generates key once
2. Client sends request with key
3. Server processes request, stores { key → result }
4. Client retries (network issue) with the SAME key
5. Server sees key already exists → returns the original result, does NOT reprocess
```

This is the same pattern used in the AWS Lambda notes for handling duplicate/retried invocations — the mechanism is identical whether the retry comes from a flaky client or an at-least-once delivery system like SQS.

### What are the important HTTP status code categories?

- **1xx** — Informational (rarely used directly by application code)
- **2xx** — Success: `200 OK`, `201 Created`, `204 No Content`
- **3xx** — Redirection: `301 Moved Permanently`, `304 Not Modified`
- **4xx** — Client error: `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests`
- **5xx** — Server error: `500 Internal Server Error`, `503 Service Unavailable`

#### What's the difference between 400 and 422?

`400 Bad Request` means the request itself is malformed — invalid JSON, missing required fields, wrong data types. `422 Unprocessable Entity` means the request is well-formed and parseable, but fails business/semantic validation — e.g., an email field that's syntactically valid JSON but not a valid email address, or a date range where the end date is before the start date.

#### What's the difference between 401 and 403?

`401 Unauthorized` means the request has no valid authentication at all — the server doesn't know who you are. `403 Forbidden` means the server knows who you are, but you don't have permission to perform this action. `401` is an authentication failure; `403` is an authorization failure.

#### When should you use 409?

`409 Conflict` signals the request conflicts with the current state of the resource — e.g., trying to create a user with an email that already exists, or a version mismatch in optimistic concurrency control (updating a record that someone else already modified).

#### Why shouldn't you return 200 with an error in the response body?

```
// Bad
HTTP 200 OK
{ "success": false, "error": "User not found" }
```

This breaks HTTP semantics — clients, proxies, monitoring tools, and caching layers all read the status code to determine success/failure. Returning `200` for an actual failure means error-rate monitoring, retries, and client error handling all silently misbehave, because everything downstream trusts the status code first.

```
// Correct
HTTP 404 Not Found
{ "error": "User not found" }
```

## Pagination, Filtering, Sorting

### What is Pagination, and why is it necessary?

Pagination splits a large result set into smaller chunks so a single request doesn't try to return millions of rows at once — protecting both server memory and client rendering performance.

### Offset-based vs Cursor-based Pagination

**Offset-based:**
```
GET /transactions?limit=20&offset=40
```
Simple to implement, but has two real problems at scale:
- **Performance** — `OFFSET 100000` still requires the database to scan and discard the first 100,000 rows before returning the next page; this gets slower as the offset grows.
- **Consistency under concurrent writes** — if new rows are inserted while a user is paging through results, offsets shift, and the user can see duplicate or skipped rows between pages.

**Cursor-based:**
```
GET /transactions?limit=20&after=eyJpZCI6MTIzfQ==
```
The cursor encodes a pointer to the last item seen (often the last row's ID or timestamp). The next page's query becomes `WHERE id > last_seen_id ORDER BY id LIMIT 20` — this is a direct indexed lookup regardless of how deep into the results you are, and it's immune to the insert-shifting problem because it's anchored to a specific row, not a numeric position.

#### When would you still use offset-based pagination?

When the dataset is small-to-medium, doesn't change frequently, and you need features cursor-based pagination doesn't support well — like jumping directly to "page 7" or showing total page count. Cursor-based pagination is the better default for large, frequently-written datasets like a transaction feed.

### Filtering and Sorting

```
GET /products?category=electronics&minPrice=100&sort=-createdAt
```

Filters are typically passed as query parameters, validated against an allow-list of filterable fields (never pass raw user input directly into a query — that's a SQL/NoSQL injection risk). Sort direction is often indicated with a prefix (`-createdAt` for descending, `createdAt` for ascending) or an explicit `sortOrder` parameter.

## Versioning

### Why does an API need versioning?

Once clients depend on an API's response shape, changing that shape breaks them. Versioning lets you evolve the API (add fields, change behavior, deprecate old fields) without breaking existing consumers.

### What are the common versioning approaches?

**1. URI versioning** (most common, most explicit):
```
GET /v1/users
GET /v2/users
```
Simple, visible in logs and browser history, but "pollutes" the URL and can make it awkward to represent that `/v1/users/1` and `/v2/users/1` are the same resource.

**2. Header versioning:**
```
GET /users
Accept: application/vnd.myapi.v2+json
```
Keeps URLs clean/stable, but is less discoverable — you can't tell the version just by looking at the URL, which makes debugging and sharing links harder.

**3. Query parameter versioning:**
```
GET /users?version=2
```
Easy to implement, but easy to forget/omit accidentally, and not considered a strong REST practice by most teams.

#### Which would you choose, and why?

URI versioning is the most common and pragmatic default for most APIs — it's explicit, cacheable per-version, and easy for consumers to understand without reading documentation. Header versioning is preferred when you want a cleaner URL scheme and have disciplined API consumers (e.g., internal service-to-service APIs).

## Rate Limiting (Design, not implementation)

> The actual rate limiter code (token bucket window logic) is implemented in the **Node.js Interview Prep** file. This section covers the design-level decisions.

### Where should rate limiting be enforced — gateway or application layer?

**Gateway-level** (API Gateway, load balancer, Nginx) — stops abusive traffic before it even reaches your application servers, protecting backend resources. Better for coarse-grained, IP-based or API-key-based limits.

**Application-level** — needed when limits depend on business logic the gateway doesn't know about (e.g., "this specific user's plan allows 100 requests/day" — tied to a database lookup), or per-endpoint limits that vary by route.

Most production systems use both: a coarse gateway-level limit as a first line of defense, and finer application-level limits for business-specific rules.

### What are the common rate limiting algorithms?

- **Fixed window** — count requests in a fixed time block (e.g., per-minute). Simple, but allows a burst at the boundary between two windows (e.g., 100 requests in the last second of one window + 100 in the first second of the next = 200 in ~1 second).
- **Sliding window** — smooths this out by considering a rolling time range instead of fixed blocks.
- **Token bucket** — a bucket refills tokens at a fixed rate; each request consumes a token. Allows short bursts up to the bucket size while enforcing a steady average rate — this is the algorithm implemented in the Node.js file's rate limiter example.

### How would you rate limit in a multi-instance (horizontally scaled) deployment?

An in-memory `Map`-based rate limiter (like the one in the Node.js file) only works correctly on a single instance — each server would track its own separate counts, effectively multiplying the allowed rate by the number of instances. For a scaled deployment, rate limit state needs to live in a shared store like **Redis**, so all instances check and increment the same counter.

## Validation & Error Handling

### Where should request validation happen?

As early as possible — in middleware, before the request reaches business logic. This is the "fail-fast" principle: reject invalid input immediately with a clear error, rather than letting it propagate deeper into the system where the failure becomes harder to trace.

```
app.post('/users', validateUserSchema, createUser);
```

Common libraries: **Zod** or **Joi** for schema-based validation.

```
const userSchema = z.object({
  email: z.string().email(),
  age: z.number().min(18),
});
```

### What does a good, consistent error response contract look like?

```
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": [
      { "field": "email", "issue": "required" }
    ]
  }
}
```

Consistency matters — every error across every endpoint should follow the same shape, so client code can handle errors generically instead of writing custom parsing logic per endpoint.

### How do you handle partial failures in a batch endpoint?

```
POST /users/bulk
[{ "email": "a@x.com" }, { "email": "invalid" }, { "email": "c@x.com" }]
```

If 2 of 3 succeed and 1 fails, returning a single status code for the whole batch loses information. Use **`207 Multi-Status`** with a per-item result array:

```
{
  "results": [
    { "status": 201, "id": 1 },
    { "status": 422, "error": "Invalid email" },
    { "status": 201, "id": 2 }
  ]
}
```

## Caching

### What HTTP mechanisms support caching?

- **`Cache-Control`** header — directives like `max-age=3600`, `no-cache`, `private`/`public` tell clients and intermediate caches (CDNs, proxies) how long a response can be reused.
- **`ETag`** — a hash/version identifier for a resource. The client sends it back on the next request via `If-None-Match`; if unchanged, the server responds `304 Not Modified` with no body, saving bandwidth.

```
GET /users/1
Response: ETag: "abc123"

// Next request
GET /users/1
If-None-Match: "abc123"
Response: 304 Not Modified (no body)
```

### When is caching the wrong choice for an API response?

For personalized or financial data that must always be current — account balances, real-time inventory, security-sensitive data. Caching stale financial data could show a user an incorrect balance, which is worse than a slightly slower response. Static or rarely-changing data (product catalogs, configuration) is a much better caching candidate.

## Designing for Multiple Clients

### How do you design an API that serves both a mobile app and a partner/third-party integration with different auth needs?

- Mobile app (first-party, trusted): typically uses short-lived JWTs obtained via a login flow, tied to a specific user session.
- Partner integration (third-party): typically uses API keys or OAuth2 client-credentials flow, tied to the partner organization rather than an individual user, often with coarser scopes/permissions.

Both can hit the same underlying REST API, but the authentication middleware needs to support multiple strategies and resolve them to a consistent internal identity/permission model, so the rest of the application logic doesn't need to know or care which auth method was used.

## Interview-Ready Summary Answers

### "Walk me through how you'd design a REST API for X."

1. Identify resources (nouns) and their relationships.
2. Map CRUD operations to HTTP methods (`GET`/`POST`/`PUT`/`PATCH`/`DELETE`).
3. Decide pagination strategy based on expected data volume and write frequency.
4. Define a consistent error response contract up front.
5. Decide auth strategy per client type.
6. Add rate limiting appropriate to abuse risk and business tiering.
7. Version from day one (URI versioning is the safe default) even if you don't expect to need v2 soon.

### "How would you design a paginated, filterable, sortable endpoint for a large transactions table?"

Cursor-based pagination (given it's a frequently-written, potentially very large table — the exact FinPilot transactions use case), allow-listed filter fields validated server-side, indexed sort columns (an unindexed `ORDER BY` on a large table is a common hidden performance bug), and a hard cap on `limit` to prevent a client from requesting an unbounded page size.
