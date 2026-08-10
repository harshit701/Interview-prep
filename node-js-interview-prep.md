# Node.js Interview Prep

## Node.js Runtime & Architecture

### What is the V8 Engine?

V8 is Google's open-source JavaScript engine written in C++. It executes JavaScript code by compiling it into machine code, allowing it to run efficiently on the CPU.

### Why is Node.js fast?

Because it uses the V8 engine, which uses Just-In-Time (JIT) compilation to convert JavaScript into optimized machine code.

### If V8 can execute JavaScript, then why do we need Node.js?

Node.js is an open-source JavaScript runtime built on Google's V8 engine. V8 executes JavaScript code, while Node.js provides additional features like the file system, HTTP server, timers, streams, and other APIs that allow JavaScript to build backend applications and run outside the browser.

### What is a Runtime Environment?

A runtime environment is the software that allows a programming language to run. It provides the execution engine along with APIs and services such as file system access, networking, timers, memory management, and interaction with the operating system.

When JavaScript runs in Node.js, it gets capabilities like the File System, an HTTP Server, TCP Connections, Streams, libuv, and the Event Loop:

```
Node.js Runtime
├── V8 Engine
├── libuv
├── File System APIs
├── HTTP Module
├── Event Loop
└── Process APIs
```

> This is different from a browser runtime, which instead provides Web APIs and the DOM — see the **JavaScript Interview Prep** file.

### What happens internally when we execute `node app.js`?

```
// app.js
const fs = require("fs");

console.log("Start");

fs.readFile("data.txt", () => {
  console.log("File Read");
});

console.log("End");
```

When a Node.js application starts, the operating system creates a Node.js process. Node initializes the V8 engine, libuv, the Event Loop, the Thread Pool, and the Node APIs. V8 executes the JavaScript code. When asynchronous operations such as file system access or networking are encountered, they are delegated to libuv. Once the operation completes, libuv places the callback into the appropriate queue, and the Event Loop moves it to the Call Stack when it becomes empty.

**Does V8 execute asynchronous operations?** No. V8 only executes JavaScript. Asynchronous operations are managed by the Node.js runtime, primarily through libuv.

**Who creates the Event Loop?** The Event Loop is part of libuv, not the V8 engine.

**Who manages the Thread Pool?** The Thread Pool is managed by libuv.

**Does V8 know about `fs.readFile()`?** No. `fs.readFile()` is provided by the Node.js runtime. V8 executes the JavaScript that calls it, but the actual file I/O is handled by libuv and the operating system.

### What is libuv?

libuv is a C library used by Node.js to handle asynchronous operations. It provides the Event Loop, Thread Pool, and non-blocking I/O, allowing Node.js to perform tasks like file system operations, networking, DNS lookups, and timers without blocking the main JavaScript thread.

**Why does Node.js need libuv?** JavaScript is single-threaded. If JavaScript itself tried to read a huge file with `fs.readFile(...)`, it would freeze the application until the file finished reading — that's bad. Instead, Node.js asks libuv to do the work. libuv provides asynchronous, non-blocking I/O — it manages the Event Loop, Thread Pool, timers, networking, and communication with the operating system, allowing JavaScript to remain single-threaded while background operations execute efficiently.

**Is libuv only for asynchronous operations?** Mostly yes — it provides asynchronous capabilities for Node.js. Without libuv, Node.js would behave much more like a synchronous program.

**Think of libuv as a manager 📋**

```
JavaScript Developer
        │
        ▼
     Manager (libuv)
        │
        ▼
 Operating System
```

JavaScript says "read this file." libuv (the manager) says "I'll take care of it," the operating system reads the file, and libuv comes back with "It's done — here's the callback." That's exactly what libuv does. JavaScript never reads the file itself — libuv does.

Another example — who waits for the 2 seconds in `setTimeout(() => {...}, 2000)`? Not JavaScript — libuv manages the timer, and signals the Event Loop once it's finished.

**Responsibilities of libuv:**

- Event Loop ✅
- Thread Pool ✅
- File System Operations ✅
- DNS Operations ✅
- Some Crypto Operations ✅
- Child Processes ✅
- Timers ✅
- Networking (TCP/UDP) ✅

Without libuv, Node.js would not be asynchronous.

**Internal architecture:**

```
JavaScript Code
       │
       ▼
    V8 Engine
       │
       ▼
     libuv
 ┌───────────────┐
 │ Event Loop    │
 │ Thread Pool   │
 │ Timers        │
 │ Networking    │
 │ File System   │
 └───────────────┘
       │
       ▼
Operating System
```

**Is libuv the Event Loop?** This is a common interview question. The answer is **no** — libuv _implements_ the Event Loop for Node.js:

```
Node.js
│
├── V8 Engine
└── libuv
      │
      ├── Event Loop
      ├── Thread Pool
      ├── Timers
      └── Async I/O
```

The Event Loop is one part of libuv.

**Interview answer — "What is libuv?"**

> libuv is a C library used by Node.js to provide asynchronous I/O operations. It is responsible for implementing the Event Loop and managing features like timers, file system operations, networking, and a thread pool for tasks that cannot be handled asynchronously by the operating system. It allows Node.js to perform non-blocking operations while JavaScript continues executing other code.

### What is a Thread Pool?

A Thread Pool is a group of worker threads managed by libuv that perform certain time-consuming operations in the background, allowing the JavaScript thread to continue executing other code.

### Why is Node.js called single-threaded if it has a Thread Pool?

JavaScript execution happens on a single main thread using the V8 engine. However, Node.js uses libuv's Thread Pool to perform certain asynchronous operations in the background. These worker threads do not execute JavaScript — they only perform background tasks and notify the Event Loop when they're finished.

### Blocking vs. Non-Blocking I/O

Blocking I/O stops the execution of the program until the current operation completes, whereas non-blocking I/O starts the operation and immediately allows the application to continue executing other tasks. Node.js achieves non-blocking I/O using libuv, the operating system, and the Event Loop, enabling it to handle many concurrent requests efficiently.

## The Node.js Event Loop

### What is the Event Loop in Node.js?

The Node.js Event Loop is a mechanism provided by libuv that continuously goes through different phases to process asynchronous operations. It checks for completed tasks such as timers, file system operations, network requests, and other callbacks. When JavaScript finishes executing the current code and the Call Stack is free, the Event Loop moves the appropriate callback to the Call Stack for execution. This allows Node.js to perform non-blocking operations while executing JavaScript on a single main thread.

### Node.js Event Loop phases

**1. Timers Phase** Executes callbacks whose timer has expired. Handles `setTimeout()` and `setInterval()`.

> A timer becomes eligible to run after its delay has expired — it does not guarantee execution at the exact specified time.

**2. Pending Callbacks Phase** Executes certain system-level callbacks deferred from the previous Event Loop iteration. Handles some TCP connection errors and certain deferred I/O system callbacks.

> Application developers rarely interact with this phase directly.

**3. Idle / Prepare Phase** An internal phase used by libuv to prepare for the Poll phase. Handles internal libuv operations only.

> No user code runs here.

**4. Poll Phase** ⭐ (Most Important)
Processes completed I/O operations and waits for new I/O events. Handles `fs.readFile()`, `fs.writeFile()`, database query callbacks, HTTP server request callbacks, network socket events, stream callbacks, and most completed asynchronous I/O.

> This is the busiest and most important phase of the Event Loop.

**5. Check Phase** Executes callbacks scheduled using `setImmediate()`.

**6. Close Callbacks Phase** Executes cleanup callbacks for closed resources, such as `socket.on("close")`, stream close callbacks, and other resource cleanup callbacks.

### Special queues (not Event Loop phases)

These aren't phases, but Node.js processes them before moving to the next Event Loop phase:

1. **`process.nextTick()` queue** — executes callbacks immediately after the current JavaScript code finishes. It has the highest priority in Node.js.

```
process.nextTick(() => {
  console.log("nextTick");
});
```

2. **Promise microtask queue** — executes Promise callbacks after the `process.nextTick()` queue is empty. Handles `.then()`, `.catch()`, `.finally()`, and `async`/`await`.

```
Promise.resolve().then(() => {
  console.log("Promise");
});
```

### Complete flow

```
        Event Loop

┌────────────────────┐
│ 1. Timers          │
│ setTimeout()       │
│ setInterval()      │
└────────────────────┘
          │
          ▼
process.nextTick()
          │
          ▼
Promise Microtasks
          │
          ▼
┌────────────────────┐
│ 2. Pending          │
│ Callbacks           │
└────────────────────┘
          │
          ▼
process.nextTick()
          │
          ▼
Promise Microtasks
          │
          ▼
┌────────────────────┐
│ 3. Idle/Prepare     │
│ (Internal)          │
└────────────────────┘
          │
          ▼
process.nextTick()
          │
          ▼
Promise Microtasks
          │
          ▼
┌────────────────────┐
│ 4. Poll             │
│ File System         │
│ Network             │
│ Database             │
│ Streams              │
└────────────────────┘
          │
          ▼
process.nextTick()
          │
          ▼
Promise Microtasks
          │
          ▼
┌────────────────────┐
│ 5. Check             │
│ setImmediate()       │
└────────────────────┘
          │
          ▼
process.nextTick()
          │
          ▼
Promise Microtasks
          │
          ▼
┌────────────────────┐
│ 6. Close Callbacks   │
│ socket.close()       │
└────────────────────┘
          │
          ▼
process.nextTick()
          │
          ▼
Promise Microtasks
          │
          ▼
Back to Timers Phase
```

## Streams & Buffers

### What is a Stream?

A Stream is an object in Node.js that reads or writes data in small chunks instead of loading the entire data into memory. This makes data processing more memory-efficient and is especially useful for large files and real-time data.

**How do we send these chunks to the write stream?**

Say you have:

```
const readStream = fs.createReadStream("movie.mp4");
const writeStream = fs.createWriteStream("copy.mp4");
```

The read stream reads data as a sequence of chunks. You listen for each chunk and write it manually:

```
readStream.on("data", (chunk) => {
  writeStream.write(chunk);
});

readStream.on("end", () => {
  writeStream.end();
});
```

This works, but Node.js offers a shortcut so you don't have to write this boilerplate every time:

```
readStream.pipe(writeStream);
```

That one line automatically:

- Reads each chunk
- Writes it to the destination
- Ends the write stream when reading is complete
- Handles the flow efficiently

**What is `pipe()`?** `pipe()` is a method used to connect a readable stream to a writable stream. It automatically transfers data from the source to the destination in chunks without loading the entire data into memory.

### What is Backpressure?

Imagine a readable stream producing data at 100 MB/sec while the writable stream can only consume 20 MB/sec:

| Time | Produced | Written | Remaining in memory |
| ---- | -------- | ------- | ------------------- |
| 1s   | 100 MB   | 20 MB   | 80 MB               |
| 2s   | 200 MB   | 40 MB   | 160 MB              |
| 10s  | 1000 MB  | 200 MB  | 800 MB              |

Memory keeps growing because the writer can't consume data as fast as the reader produces it — this is called **backpressure**.

Backpressure is a situation where a readable stream produces data faster than a writable stream can consume it, which can cause data to accumulate in memory. Node.js handles backpressure automatically when using `pipe()` by pausing the readable stream until the writable stream is ready to receive more data.

### What is a Buffer?

Whenever Node.js reads data from a file, a socket, an HTTP request, or a TCP connection, it receives raw bytes — and those bytes are stored in a Buffer. A Buffer is an object in Node.js that temporarily stores binary data in memory before it is processed or transferred.

```
const readStream = fs.createReadStream("movie.mp4");

readStream.on("data", (chunk) => {
  console.log(chunk); // each chunk is stored in a Buffer
});
```

**What is the difference between a Buffer and a Stream?** A Stream is an object in Node.js that reads, writes, or transforms data in small chunks instead of loading the entire data into memory. A Buffer is an object that temporarily stores raw binary data in memory before it is processed or transferred. In simple terms, a stream _moves_ the data, while a buffer temporarily _holds_ the data.

**Can a Stream work without a Buffer?** No. Streams transfer data, but the actual data is temporarily stored in Buffers. When a readable stream reads a chunk from a file or network, that chunk is stored in a Buffer before being processed or written to the destination.

## Memory Leaks & Diagnosis

### How do you check memory usage in a running Node process?

```
console.log(process.memoryUsage());
// {
//   rss: 42545152,        // total memory allocated for the process
//   heapTotal: 20971520,  // total size of the allocated heap
//   heapUsed: 14721624,   // memory actually in use
//   external: 1234567,    // memory used by C++ objects bound to JS (Buffers etc.)
// }
```

`heapUsed` is the number to watch over time. A normal pattern is a sawtooth — memory rises as objects are allocated, GC runs, memory drops. A **leak** looks like the sawtooth's floor steadily rising under steady (non-increasing) load — GC runs, frees what it can, but the baseline never returns to where it started.

### What commonly causes memory leaks in a Node service?

- **Unbounded module-level caches/arrays** — a `const cache = {}` or array declared outside a request handler that keeps growing with every request and is never evicted.
- **Uncleared `setInterval`** — the timer's callback (and everything it closes over) stays alive until `clearInterval` is explicitly called.
- **Event listeners never removed** — an `EventEmitter` (e.g. a DB pool, a pub/sub bus) with listeners attached via `.on()` but never `.off()`'d keeps whatever the listener closure references alive indefinitely.
- **Closures holding references longer than needed** — a returned function that closes over a large object it doesn't even use.

**Not a leak:** data that's local to a function/request handler with nothing outside referencing it after the function returns — even if the data was large, it's eligible for GC the moment the function ends.

### What is a heap snapshot, and how do you use it to find a leak?

A heap snapshot is a complete picture of everything alive in memory at a moment in time, including what's referencing what.

```
node --inspect index.js
# open chrome://inspect, click "inspect", go to Memory tab
```

Or programmatically:

```
const v8 = require('v8');
const fs = require('fs');
const snapshotStream = v8.getHeapSnapshot();
snapshotStream.pipe(fs.createWriteStream(`heap-${Date.now()}.heapsnapshot`));
```

**Workflow:** take snapshot 1 (baseline), apply load/time, take snapshot 2, compare. DevTools shows the delta by constructor/type — if `Array` instances grew from 1,000 to 45,000 between snapshots under proportional traffic, that object type is your leak. Click into an instance and check "Retainers" to trace exactly which variable/closure/listener is holding the reference that's preventing collection.

Tools: `clinic.js` (`clinic heapprofiler`) gives a higher-level flamegraph view; `heapdump` package is a simpler alternative to the built-in `v8` module.

## Events & EventEmitter

### What is EventEmitter?

EventEmitter is a built-in Node.js class that implements the publish-subscribe pattern. It allows objects to emit events and other parts of the application to listen for those events. This helps decouple components and enables event-driven programming.

**Basic example:**

```
const EventEmitter = require("events");

const emitter = new EventEmitter();

emitter.on("greet", () => {
  console.log("Hello Harshit");
});

emitter.emit("greet");
// Output: Hello Harshit
```

**How it works:**

- `emitter.on(...)` registers a listener — think **subscribe**.
- `emitter.emit(...)` triggers the event — think **publish**.

**Multiple listeners:**

```
const emitter = new EventEmitter();

emitter.on("orderPlaced", () => console.log("Send Email"));
emitter.on("orderPlaced", () => console.log("Update Inventory"));
emitter.on("orderPlaced", () => console.log("Generate Invoice"));

emitter.emit("orderPlaced");
// One event, three listeners:
// Send Email
// Update Inventory
// Generate Invoice
```

**Passing data:**

```
emitter.on("userCreated", (user) => {
  console.log(user.name);
});

emitter.emit("userCreated", { name: "Harshit" });
// Output: Harshit
```

**Removing listeners:**

```
emitter.off("login", listener); // removes one listener
emitter.removeAllListeners();   // removes every listener
```

## Callbacks & Error Handling

### What is a Callback?

A callback is a function that is passed as an argument to another function so that it can be executed later, usually after a specific task is completed.

```
function greet(name, callback) {
  console.log("Hello " + name);
  callback();
}

function sayBye() {
  console.log("Bye");
}

greet("test", sayBye);
// Output:
// Hello test
// Bye
```

### What is Callback Hell?

Callback Hell is a situation where multiple asynchronous operations depend on each other, causing callbacks to be nested inside other callbacks. This results in deeply indented code that is difficult to read, maintain, and debug.

### What is the Error-First Callback pattern?

An Error-First Callback is a Node.js convention where the first argument of a callback is reserved for an error. If the operation succeeds, the error argument is `null`; if it fails, it contains an `Error` object. This allows developers to handle errors before processing the result.

### What is `uncaughtException`?

`uncaughtException` is a Node.js process event that is emitted when a synchronous exception is not caught anywhere in the application.

### What is `unhandledRejection`?

`unhandledRejection` is a Node.js process event that is emitted when a Promise is rejected but no `.catch()` handler is attached to it.

### What do you do when an uncaught exception occurs in production?

First, I log the error using a logging framework or monitoring service. Then I gracefully close open resources such as database connections, Redis connections, message queues, and the HTTP server. After cleanup, I terminate the process using `process.exit(1)` because the application's state may be inconsistent. Finally, I rely on a process manager such as PM2, Docker, or Kubernetes to automatically restart the application.

```
Log → Clean up → Exit → Restart
```

## Express & Middleware

### What is Middleware?

Middleware is a function that executes during the request-response lifecycle. It sits between the incoming request and the route handler. Middleware can execute code, modify the request or response, terminate the request, or pass control to the next middleware using `next()`. It is commonly used for authentication, logging, validation, CORS, and error handling.

### How does Express middleware execution actually work?

Each middleware receives `(req, res, next)` and either handles the request, modifies it, or calls `next()` to pass control to the next middleware in the chain. If `next()` is never called, the request hangs.

```
app.use((req, res, next) => {
  console.log('Request received');
  next(); // without this, the request never proceeds further
});
```

### What is error-handling middleware?

Middleware with **4 parameters** `(err, req, res, next)` — Express identifies it as an error handler specifically by this signature. It must be registered last, after all other routes/middleware.

```
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong' });
});
```

Calling `next(err)` anywhere in the chain skips all remaining normal middleware and jumps straight to the nearest error-handling middleware.

### How would you structure a Node/Express app for testability?

Layer it: controller (handles req/res) → service (business logic) → repository (data access). This lets you unit test the service layer without spinning up Express or a real database, by injecting mock repositories.

## Authentication & Security

### What is JWT (JSON Web Token)?

JWT (JSON Web Token) is a token generated by the server after a user successfully logs in. The client sends this token with every request so the server can identify and verify the user without storing session information.

### What does Stateless mean?

Stateless means the server does not remember the user. Instead, the client sends the JWT with every request, and the server verifies the token to identify the user.

### What is the difference between an Access Token and a Refresh Token?

**Access Token** — a short-lived JWT used to authenticate and authorize a user for accessing protected APIs. It is sent with every request and expires quickly to reduce security risks.

**Refresh Token** — a long-lived token used to obtain a new Access Token after it expires, allowing the user to stay logged in without entering credentials again.

### Authentication vs. Authorization

**Authentication** (who are you?) is the process of verifying a user's identity, usually by checking credentials such as an email and password or validating a JWT.

**Authorization** (what can you do?) is the process of determining what an authenticated user is allowed to access or perform, based on roles or permissions.

### What is CORS (Cross-Origin Resource Sharing)?

CORS allows or blocks requests from different domains to protect users from unauthorized access.

### What is Rate Limiting?

Rate Limiting restricts the number of requests a client can make within a specific time period to prevent abuse and protect the server.

```
const rateLimiter = new Map();

function rateLimiterFn(clientIp, options = { windowMS: 60000, maxRequest: 5 }) {
  const now = Date.now();

  if (!rateLimiter.has(clientIp)) {
    rateLimiter.set(clientIp, {
      windowStart: now,
      count: 1,
    });

    return { allowed: true };
  }

  const clientData = rateLimiter.get(clientIp);

  if (now - clientData.windowStart > options.windowMS) {
    clientData.windowStart = now;
    clientData.count = 1;
    return { allowed: true };
  }

  clientData.count++;

  if (clientData.count > options.maxRequest) {
    return {
      allowed: false,
      statusCode: 429,
      message: "Too Many Requests",
    };
  }

  return { allowed: true };
}

// Quick terminal test (limit: 2 requests)
const testOptions = { windowMS: 5000, maxRequest: 2 };

console.log(rateLimiterFn("127.0.0.1", testOptions)); // { allowed: true }  (Hit 1)
console.log(rateLimiterFn("127.0.0.1", testOptions)); // { allowed: true }  (Hit 2)
console.log(rateLimiterFn("127.0.0.1", testOptions)); // { allowed: false, statusCode: 429, ... } (Hit 3 - Blocked!)
```

## Database Connection Handling

### Why does connection pooling matter?

Opening a new DB connection per request is expensive (TCP handshake, auth) and doesn't scale — a connection pool maintains a set of reusable connections, and requests borrow/return them.

If concurrent requests exceed the pool size, additional requests **queue** waiting for a free connection — this shows up as latency in the app layer, but the actual bottleneck is pool size vs concurrency, not the query itself. Diagnosing "slow API" issues should include checking pool utilization metrics before assuming the DB query itself is slow.

### What is the N+1 query problem?

Fetching a list, then making one additional query per item to get related data — instead of one query with a join or a single batched query.

```
// N+1 - one query for users, then one query PER user for their orders
const users = await db.user.findMany();
for (const user of users) {
  user.orders = await db.order.findMany({ where: { userId: user.id } });
}
```

Fix: use eager loading/joins (`include` in Prisma) or a single batched query, so the total query count doesn't scale with result size.

## Scaling

### What is Clustering in Node.js?

Clustering creates multiple Node.js processes so the application can use all CPU cores, improving performance and scalability.

### How does the `cluster` module actually work?

`cluster` forks multiple Node.js processes (workers), each with its own event loop and V8 instance, allowing an app to use multiple CPU cores — since a single Node process only uses one core by default.

```
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) cluster.fork();
} else {
  require('./server'); // each worker runs the actual server
}
```

The primary process distributes incoming connections across workers (round-robin on most platforms). Workers don't share memory — they're separate processes, so any shared state (session data, in-memory caches) needs an external store like Redis, not a module-level JS variable.

### Cluster vs Worker Threads vs Child Process — when do you use which?

Cluster

Use case: Scale an HTTP server across CPU cores — the primary process forks multiple worker processes, each running a full copy of your server, distributing incoming connections between them.

Memory: Separate processes. No shared memory — each worker has its own V8 instance and heap. Any shared state (sessions, caches) needs an external store like Redis.

Overhead: Higher — each worker is a full Node.js process.

Worker Threads

Use case: CPU-intensive JavaScript computation (image processing, heavy calculations, encryption) that would otherwise block the single event loop thread.

Memory: Can share memory directly via SharedArrayBuffer, unlike Cluster or Child Process.

Overhead: Lower — threads, not full processes, so cheaper to spin up.

Child Process

Use case: Run a separate program or script (not necessarily Node/JS — could be a shell command, a Python script, etc.), or isolate a piece of work so a crash doesn't take down the main process.

Memory: Separate process, no shared memory.

Overhead: Higher — full process per child.

Worker threads are the right choice for something like heavy synchronous computation (image processing, complex calculations) that would otherwise block the single event loop thread. Cluster is the right choice for scaling a stateless HTTP server to use all CPU cores.
