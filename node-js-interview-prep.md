# Node.js Interview Prep

## What is the V8 Engine?

V8 is Google's open-source JavaScript engine written in C++. It executes JavaScript code by compiling it into machine code, allowing it to run efficiently on the CPU.

## Why is Node.js fast?

Because it uses the V8 engine, which uses Just-In-Time (JIT) compilation to convert JavaScript into optimized machine code.

## If V8 can execute JavaScript, then why do we need Node.js?

Node.js is an open-source JavaScript runtime built on Google's V8 engine. V8 executes JavaScript code, while Node.js provides additional features like the file system, HTTP server, timers, streams, and other APIs that allow JavaScript to build backend applications and run outside the browser.

## What is a Runtime Environment?

A runtime environment is the software that allows a programming language to run. It provides the execution engine along with APIs and services such as file system access, networking, timers, memory management, and interaction with the operating system.

## What happens internally when we execute `node app.js`?

```js
// app.js
const fs = require("fs");

console.log("Start");

fs.readFile("data.txt", () => {
  console.log("File Read");
});

console.log("End");
```

When a Node.js application starts, the operating system creates a Node.js process. Node initializes the V8 engine, libuv, the Event Loop, the Thread Pool, and the Node APIs. V8 executes the JavaScript code. When asynchronous operations such as file system access or networking are encountered, they are delegated to libuv. Once the operation completes, libuv places the callback into the appropriate queue, and the Event Loop moves it to the Call Stack when it becomes empty.

### Does V8 execute asynchronous operations?

No. V8 only executes JavaScript. Asynchronous operations are managed by the Node.js runtime, primarily through libuv.

### Who creates the Event Loop?

The Event Loop is part of libuv, not the V8 engine.

### Who manages the Thread Pool?

The Thread Pool is managed by libuv.

### Does V8 know about `fs.readFile()`?

No. `fs.readFile()` is provided by the Node.js runtime. V8 executes the JavaScript that calls it, but the actual file I/O is handled by libuv and the operating system.

## What is libuv?

libuv is a C library used by Node.js to handle asynchronous operations. It provides the Event Loop, Thread Pool, and non-blocking I/O, allowing Node.js to perform tasks like file system operations, networking, DNS lookups, and timers without blocking the main JavaScript thread.

### Think of libuv as a manager

```text
JavaScript Developer
        │
        ▼
     Manager (libuv)
        │
        ▼
 Operating System
```

JavaScript says "read this file." libuv (the manager) says "I'll take care of it," the operating system reads the file, and libuv comes back with "It's done — here's the callback." That's exactly what libuv does.

### Responsibilities of libuv

libuv is responsible for:
- Event Loop ✅
- Thread Pool ✅
- File System Operations ✅
- DNS Operations ✅
- Some Crypto Operations ✅
- Child Processes ✅
- Timers ✅
- Networking (TCP/UDP) ✅

Without libuv, Node.js would not be asynchronous.

### Internal Architecture

```text
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

### Why does Node.js need libuv?

Node.js uses libuv to provide asynchronous, non-blocking I/O. It manages the Event Loop, Thread Pool, timers, networking, and communication with the operating system, allowing JavaScript to remain single-threaded while background operations execute efficiently.

## Why is Node.js called single-threaded if it has a Thread Pool?

JavaScript execution happens on a single main thread using the V8 engine. However, Node.js uses libuv's Thread Pool to perform certain asynchronous operations in the background. These worker threads do not execute JavaScript — they only perform background tasks and notify the Event Loop when they're finished.

## What is the Event Loop in Node.js?

The Node.js Event Loop is a mechanism provided by libuv that continuously goes through different phases to process asynchronous operations. It checks for completed tasks such as timers, file system operations, network requests, and other callbacks. When JavaScript finishes executing the current code and the Call Stack is free, the Event Loop moves the appropriate callback to the Call Stack for execution. This allows Node.js to perform non-blocking operations while executing JavaScript on a single main thread.

### Node.js Event Loop phases

**1. Timers Phase**
Executes callbacks whose timer has expired. Handles `setTimeout()` and `setInterval()`.
> A timer becomes eligible to run after its delay has expired — it does not guarantee execution at the exact specified time.

**2. Pending Callbacks Phase**
Executes certain system-level callbacks deferred from the previous Event Loop iteration. Handles some TCP connection errors and certain deferred I/O system callbacks.
> Application developers rarely interact with this phase directly.

**3. Idle / Prepare Phase**
An internal phase used by libuv to prepare for the Poll phase. Handles internal libuv operations only.
> No user code runs here.

**4. Poll Phase** ⭐ (Most Important)
Processes completed I/O operations and waits for new I/O events. Handles `fs.readFile()`, `fs.writeFile()`, database query callbacks, HTTP server request callbacks, network socket events, stream callbacks, and most completed asynchronous I/O.
> This is the busiest and most important phase of the Event Loop.

**5. Check Phase**
Executes callbacks scheduled using `setImmediate()`.

**6. Close Callbacks Phase**
Executes cleanup callbacks for closed resources, such as `socket.on("close")`, stream close callbacks, and other resource cleanup callbacks.

### Special queues (not Event Loop phases)

These aren't phases, but Node.js processes them before moving to the next Event Loop phase:

1. **`process.nextTick()` queue** — executes callbacks immediately after the current JavaScript code finishes. It has the highest priority in Node.js.
   ```js
   process.nextTick(() => {
     console.log("nextTick");
   });
   ```
2. **Promise microtask queue** — executes Promise callbacks after the `process.nextTick()` queue is empty. Handles `.then()`, `.catch()`, `.finally()`, and `async`/`await`.
   ```js
   Promise.resolve().then(() => {
     console.log("Promise");
   });
   ```

### Complete flow

```text
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

## What is a Stream?

A Stream is an object in Node.js that reads or writes data in small chunks instead of loading the entire data into memory. This makes data processing more memory-efficient and is especially useful for large files and real-time data.

### How do we send these chunks to the write stream?

Say you have:

```js
const readStream = fs.createReadStream("movie.mp4");
const writeStream = fs.createWriteStream("copy.mp4");
```

The read stream reads data as a sequence of chunks. You listen for each chunk and write it manually:

```js
readStream.on("data", (chunk) => {
  writeStream.write(chunk);
});

readStream.on("end", () => {
  writeStream.end();
});
```

Full example:

```js
const fs = require("fs");

const readStream = fs.createReadStream("movie.mp4");
const writeStream = fs.createWriteStream("copy.mp4");

readStream.on("data", (chunk) => {
  writeStream.write(chunk);
});

readStream.on("end", () => {
  writeStream.end();
});
```

This works, but Node.js offers a shortcut so you don't have to write this boilerplate every time:

```js
readStream.pipe(writeStream);
```

That one line automatically:
- Reads each chunk
- Writes it to the destination
- Ends the write stream when reading is complete
- Handles the flow efficiently

### What is `pipe()`?

`pipe()` is a method used to connect a readable stream to a writable stream. It automatically transfers data from the source to the destination in chunks without loading the entire data into memory.

### What is Backpressure?

Imagine a readable stream producing data at 100 MB/sec while the writable stream can only consume 20 MB/sec:

| Time | Produced | Written | Remaining in memory |
|------|----------|---------|----------------------|
| 1s   | 100 MB   | 20 MB   | 80 MB                |
| 2s   | 200 MB   | 40 MB   | 160 MB               |
| 10s  | 1000 MB  | 200 MB  | 800 MB               |

Memory keeps growing because the writer can't consume data as fast as the reader produces it — this is called **backpressure**.

Backpressure is a situation where a readable stream produces data faster than a writable stream can consume it, which can cause data to accumulate in memory. Node.js handles backpressure automatically when using `pipe()` by pausing the readable stream until the writable stream is ready to receive more data.

### Where is each chunk stored before Node.js processes or sends it?

```js
const readStream = fs.createReadStream("movie.mp4");

readStream.on("data", (chunk) => {
  console.log(chunk);
});
```

Whenever Node.js reads data from a file, a socket, an HTTP request, or a TCP connection, it receives raw bytes — and those bytes are stored in a **Buffer**.

### What is a Buffer?

A Buffer is an object in Node.js that temporarily stores binary data in memory before it is processed or transferred.

### What is the difference between a Buffer and a Stream?

A Stream is an object in Node.js that reads, writes, or transforms data in small chunks instead of loading the entire data into memory. A Buffer is an object that temporarily stores raw binary data in memory before it is processed or transferred. In simple terms, a stream *moves* the data, while a buffer temporarily *holds* the data.

### Can a Stream work without a Buffer?

No. Streams transfer data, but the actual data is temporarily stored in Buffers. When a readable stream reads a chunk from a file or network, that chunk is stored in a Buffer before being processed or written to the destination.

## What is a Callback?

A callback is a function that is passed as an argument to another function so that it can be executed later, usually after a specific task is completed.

```js
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

## What is Callback Hell?

Callback Hell is a situation where multiple asynchronous operations depend on each other, causing callbacks to be nested inside other callbacks. This results in deeply indented code that is difficult to read, maintain, and debug.

## What is the Error-First Callback pattern?

An Error-First Callback is a Node.js convention where the first argument of a callback is reserved for an error. If the operation succeeds, the error argument is `null`; if it fails, it contains an `Error` object. This allows developers to handle errors before processing the result.

## What is `uncaughtException`?

`uncaughtException` is a Node.js process event that is emitted when a synchronous exception is not caught anywhere in the application.

## What is `unhandledRejection`?

`unhandledRejection` is a Node.js process event that is emitted when a Promise is rejected but no `.catch()` handler is attached to it.

### What do you do when an uncaught exception occurs in production?

First, I log the error using a logging framework or monitoring service. Then I gracefully close open resources such as database connections, Redis connections, message queues, and the HTTP server. After cleanup, I terminate the process using `process.exit(1)` because the application's state may be inconsistent. Finally, I rely on a process manager such as PM2, Docker, or Kubernetes to automatically restart the application.

```text
Log → Clean up → Exit → Restart
```

## What is JWT (JSON Web Token)?

JWT (JSON Web Token) is a token generated by the server after a user successfully logs in. The client sends this token with every request so the server can identify and verify the user without storing session information.

## What does Stateless mean?

Stateless means the server does not remember the user. Instead, the client sends the JWT with every request, and the server verifies the token to identify the user.

## What is the difference between an Access Token and a Refresh Token?

### Access Token

An Access Token is a short-lived JWT used to authenticate and authorize a user for accessing protected APIs. It is sent with every request and expires quickly to reduce security risks.

### Refresh Token

A Refresh Token is a long-lived token used to obtain a new Access Token after it expires, allowing the user to stay logged in without entering credentials again.

## Authentication (Who are you?)

Authentication is the process of verifying a user's identity, usually by checking credentials such as an email and password or validating a JWT.

## Authorization (What can you do?)

Authorization is the process of determining what an authenticated user is allowed to access or perform, based on roles or permissions.

## What is CORS (Cross-Origin Resource Sharing)?

CORS allows or blocks requests from different domains to protect users from unauthorized access.

## What is Rate Limiting?

Rate Limiting restricts the number of requests a client can make within a specific time period to prevent abuse and protect the server.

```js
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

## What is Clustering in Node.js?

Clustering creates multiple Node.js processes so the application can use all CPU cores, improving performance and scalability.
