## What is the V8 Engine?

V8 is Google's open-source JavaScript engine written in C++. It executes JavaScript code by compiling it into machine code, allowing it to run efficiently on the CPU.

## Why is Node.js fast?

Because it uses the V8 engine, which uses Just-In-Time (JIT) compilation to convert JavaScript into optimized machine code.

## If V8 can execute JavaScript, then why do we need Node.js?

Node.js is an open-source JavaScript runtime built on Google's V8 engine. V8 executes JavaScript code, while Node.js provides additional features like the file system, HTTP server, timers, streams, and other APIs that allow JavaScript to build backend applications and run outside the browser.

## What is a Runtime Environment?

A runtime environment is the software that allows a programming language to run. It provides the execution engine along with APIs and services such as file system access, networking, timers, memory management, and interaction with the operating system.

## What happens internally when we execute node app.js?

```
// app.js

const fs = require("fs");

console.log("Start");

fs.readFile("data.txt", () => {
    console.log("File Read");
});

console.log("End");
```

When a Node.js application starts, the operating system creates a Node.js process. Node initializes the V8 engine, libuv, the Event Loop, the Thread Pool, and Node APIs. V8 executes the JavaScript code. When asynchronous operations such as file system access or networking are encountered, they are delegated to libuv. Once the operation completes, libuv places the callback into the appropriate queue, and the Event Loop moves it to the Call Stack when it becomes empty.

### Does V8 execute asynchronous operations?

No. V8 only executes JavaScript. Asynchronous operations are managed by the Node.js runtime, primarily through libuv.

### Who creates the Event Loop?

The Event Loop is part of libuv, not the V8 engine.

### Who manages the Thread Pool?

The Thread Pool is managed by libuv.

### Does V8 know about fs.readFile()?

No. fs.readFile() is provided by the Node.js runtime. V8 executes the JavaScript that calls it, but the actual file I/O is handled by libuv and the operating system.

## What is libuv?

libuv is a C library used by Node.js to handle asynchronous operations. It provides the Event Loop, Thread Pool, and non-blocking I/O, allowing Node.js to perform tasks like file system operations, networking, DNS lookups, and timers without blocking the main JavaScript thread.

#### Think of libuv as a Manager

```
Imagine a company.

JavaScript Developer
│
▼
Manager
│
▼
Operating System

JavaScript says:

Read this file.

The manager (libuv) says:

I'll take care of it.

The Operating System reads the file.

The manager comes back and says:

It's done. Here's the callback.

That's exactly what libuv does.
```

#### Responsibilities of libuv

```
libuv is responsible for:

Event Loop ✅
Thread Pool ✅
File System Operations ✅
DNS Operations ✅
Some Crypto Operations ✅
Child Processes ✅
Timers ✅
Networking (TCP/UDP) ✅

Notice something. Without libuv, Node.js would not be asynchronous.
```

#### Internal Architecture

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

#### Why does Node.js need libuv?

Node.js uses libuv to provide asynchronous, non-blocking I/O. It manages the Event Loop, Thread Pool, timers, networking, and communication with the operating system, allowing JavaScript to remain single-threaded while background operations execute efficiently.

## Why is Node.js called Single Threaded if it has a Thread Pool?

JavaScript execution happens on a single main thread using the V8 engine. However, Node.js uses libuv's Thread Pool to perform certain asynchronous operations in the background. These worker threads do not execute JavaScript—they only perform background tasks and notify the Event Loop when they're finished.

## What is Event Loop in Node JS?

The Node.js Event Loop is a mechanism provided by libuv that continuously goes through different phases to process asynchronous operations. It checks for completed tasks such as timers, file system operations, network requests, and other callbacks. When JavaScript finishes executing the current code and the Call Stack is free, the Event Loop moves the appropriate callback to the Call Stack for execution. This allows Node.js to perform non-blocking operations while executing JavaScript on a single main thread.

### Node.js Event Loop Phases

#### 1. Timers Phase

Purpose

Executes callbacks whose timer has expired.

Handles
setTimeout()
setInterval()

Note: A timer becomes eligible to run after its delay has expired. It does not guarantee execution at the exact specified time.

#### 2. Pending Callbacks Phase

Purpose

Executes certain system-level callbacks that were deferred from the previous Event Loop iteration.

Handles
Some TCP connection errors
Certain deferred I/O system callbacks

Note: Application developers rarely interact with this phase directly.

#### 3. Idle / Prepare Phase

Purpose

An internal phase used by libuv to prepare for the Poll phase.

Handles
Internal libuv operations

Note: No user code runs here.

#### 4. Poll Phase ⭐ (Most Important)

Purpose

Processes completed I/O operations and waits for new I/O events.

Handles
fs.readFile()
fs.writeFile()
Database query callbacks
HTTP server request callbacks
Network socket events
Stream callbacks
Most completed asynchronous I/O operations

Note: This is the busiest and most important phase of the Event Loop.

#### 5. Check Phase

Purpose

Executes callbacks scheduled using setImmediate().

Handles
setImmediate()

#### 6. Close Callbacks Phase

Purpose

Executes cleanup callbacks for closed resources.

Handles
socket.on("close")
Stream close callbacks
Other resource cleanup callbacks

### Special Queues (Not Event Loop Phases)

These are not phases, but Node.js processes them before moving to the next Event Loop phase.

```
1. process.nextTick() Queue
   Purpose

Executes callbacks immediately after the current JavaScript code finishes.

Priority

Highest priority in Node.js.

Example:

process.nextTick(() => {
console.log("nextTick");
});


2. Promise Microtask Queue
Purpose

Executes Promise callbacks after the process.nextTick() queue is empty.

Handles
Promise.then()
Promise.catch()
Promise.finally()
async/await

Example:

Promise.resolve().then(() => {
console.log("Promise");
});
```

### Complete Flow

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
        │ 2. Pending         │
        │ Callbacks          │
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
        │ 3. Idle/Prepare    │
        │ (Internal)         │
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
        │ 4. Poll            │
        │ File System        │
        │ Network            │
        │ Database           │
        │ Streams            │
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
        │ 5. Check           │
        │ setImmediate()     │
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
        │ 6. Close Callbacks │
        │ socket.close()     │
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

## What is Stream?

A Stream is an object in Node.js that reads or writes data in small chunks instead of loading the entire data into memory. This makes data processing more memory-efficient and is especially useful for large files and real-time data.

### How do we send these chunks to the write stream?

#### Imagine What Happens

You have:

const readStream = fs.createReadStream("movie.mp4");
const writeStream = fs.createWriteStream("copy.mp4");

```
The read stream starts reading:

Chunk 1
Chunk 2
Chunk 3
Chunk 4
...

Question:

How do we send these chunks to the write stream?

We need to listen for every chunk.

readStream.on("data", (chunk) => {
    writeStream.write(chunk);
});

When the reading finishes:

readStream.on("end", () => {
    writeStream.end();
});

So the complete code becomes:

const fs = require("fs");

const readStream = fs.createReadStream("movie.mp4");
const writeStream = fs.createWriteStream("copy.mp4");

readStream.on("data", (chunk) => {
    writeStream.write(chunk);
});

readStream.on("end", () => {
    writeStream.end();
});

This works perfectly.
```

But Node.js says...

"Why write all this boilerplate every time?"

So Node.js gives us:

```
readStream.pipe(writeStream);
```

That's it.

It automatically:

Reads each chunk
Writes it to the destination
Ends the write stream when reading is complete
Handles the flow efficiently

### what is pipe()?

pipe() is a method used to connect a readable stream to a writable stream. It automatically transfers data from the source to the destination in chunks without loading the entire data into memory.

```
Imagine this situation

You have:

Readable Stream

Reading speed:

100 MB/sec

And

Writable Stream

Writing speed:

20 MB/sec

So every second:

Reader produces 100 MB
Writer can only consume 20 MB
```

The data usually doesn't get lost. Instead, the data starts building up in memory because the writable stream can't keep up.

```
Let's visualize it

Suppose:

Readable Stream → 100 MB/sec
Writable Stream → 20 MB/sec
After 1 second
Produced : 100 MB
Written : 20 MB

Remaining in memory : 80 MB
After 2 seconds
Produced : 200 MB
Written : 40 MB

Remaining in memory : 160 MB
After 10 seconds
Produced : 1000 MB
Written : 200 MB

Remaining in memory : 800 MB

Memory keeps growing because the writer can't consume data as fast as the reader produces it.

This is called Backpressure.
```

### What is Backpressure?

Backpressure is a situation where a readable stream produces data faster than a writable stream can consume it. This can cause data to accumulate in memory. Node.js handles backpressure automatically when using pipe() by pausing the readable stream until the writable stream is ready to receive more data.

### Where do you think each chunk is stored before Node.js processes or sends it?

```
const readStream = fs.createReadStream("movie.mp4");

readStream.on("data", (chunk) => {
    console.log(chunk);
});

The chunk variable contains something.
```

```
Whenever Node.js reads data from:

a file
a socket
an HTTP request
a TCP connection

it receives bytes.

Those bytes are stored in a Buffer.
```

### What is a Buffer?

A Buffer is an object in Node.js that temporarily stores binary data in memory before it is processed or transferred.

### What is the difference between a Buffer and a Stream?

A Stream is an object in Node.js that reads, writes, or transforms data in small chunks instead of loading the entire data into memory. A Buffer is an object that temporarily stores raw binary data in memory before it is processed or transferred. In simple terms, a stream moves the data, while a buffer temporarily holds the data.

### Can a Stream work without a Buffer?

No. Streams transfer data, but the actual data is temporarily stored in Buffers. When a readable stream reads a chunk from a file or network, that chunk is stored in a Buffer before being processed or written to the destination.

## What is a Callback?

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

Output:
Hello test
Bye
```

## What is Callback Hell?

Callback Hell is a situation where multiple asynchronous operations depend on each other, causing callbacks to be nested inside other callbacks. This results in deeply indented code that is difficult to read, maintain, and debug.

## What is Error-First Callback Pattern?

An Error-First Callback is a Node.js convention where the first argument of a callback is reserved for an error. If the operation succeeds, the error argument is null; if it fails, it contains an Error object. This allows developers to handle errors before processing the result.

## What is uncaughtException?

uncaughtException is a Node.js process event that is emitted when a synchronous exception is not caught anywhere in the application.

## What is unhandledRejection?

unhandledRejection is a Node.js process event that is emitted when a Promise is rejected but no .catch() handler is attached to it.

### What do you do when an uncaught exception occurs in production?

First, I log the error using a logging framework or monitoring service. Then I gracefully close open resources such as database connections, Redis connections, message queues, and the HTTP server. After cleanup, I terminate the process using process.exit(1) because the application's state may be inconsistent. Finally, I rely on a process manager such as PM2, Docker, or Kubernetes to automatically restart the application.

```
Log → Clean up → Exit → Restart
```

## What is JWT(JSON Web Token)?

JWT (JSON Web Token) is a token generated by the server after a user successfully logs in. The client sends this token with every request so the server can identify and verify the user without storing session information.

## What is Stateless?

Stateless means the server does not remember the user. Instead, the client sends the JWT with every request, and the server verifies the token to identify the user.

### What is the difference between an Access Token and a Refresh Token?

## What is an Access Token?

An Access Token is a short-lived JWT used to authenticate and authorize a user for accessing protected APIs. It is sent with every request and expires quickly to reduce security risks.

## What is a Refresh Token?

A Refresh Token is a long-lived token used to obtain a new Access Token after it expires, allowing the user to stay logged in without entering credentials again.

## Authentication (Who are you?)

Authentication is the process of verifying a user's identity, usually by checking credentials such as an email and password or validating a JWT.

## Authorization (What can you do?)

Authorization is the process of determining what an authenticated user is allowed to access or perform based on roles or permissions.

## What is CORS(Cross-Origin Resource Sharing)?

CORS allows or blocks requests from different domains to protect users from unauthorized access.

## What is Rate Limiting?

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

  // FIX: Changed options.count to options.maxRequest
  if (clientData.count > options.maxRequest) {
    return {
      allowed: false,
      statusCode: 429,
      message: "Too Many Requests",
    };
  }

  return { allowed: true };
}

// Quick Terminal Test (Limit: 2 requests)
const testOptions = { windowMS: 5000, maxRequest: 2 };

console.log(rateLimiterFn("127.0.0.1", testOptions)); // { allowed: true }  (Hit 1)
console.log(rateLimiterFn("127.0.0.1", testOptions)); // { allowed: true }  (Hit 2)
console.log(rateLimiterFn("127.0.0.1", testOptions)); // { allowed: false, statusCode: 429, ... } (Hit 3 - Blocked!)

```

## What is Clustering in Node JS?

Cluster creates multiple Node.js processes so the application can use all CPU cores and improve performance and scalability.
