# What is Execution Context?

Execution Context is the environment in which JavaScript code is prepared and executed. Whenever a JavaScript program starts, the engine creates a Global Execution Context, and every time a function is invoked, it creates a new Function Execution Context.

Each execution context contains the information required to execute the code, including variable bindings, the lexical environment for scope resolution, and the this binding.

An execution context goes through two phases: the Creation Phase, where memory is allocated for variables and functions, and the Execution Phase, where the code executes line by line and variables receive their values.

Once a function finishes execution, its execution context is removed from the call stack, while the Global Execution Context remains until the program terminates.

## Execution Context Phases

Every Execution Context (Global or Function) goes through 2 phases:

Execution Context

    │
    ▼

1. Creation Phase (Memory Creation Phase)

   │
   ▼

2. Execution Phase (Code Execution Phase)

## Why are functions hoisted completely while var variables are initialized with undefined?

During the Creation Phase:

Function declarations are stored with their complete function object so they can be invoked before their declaration in the source code.
var declarations are allocated memory and initialized to undefined, but their assignments happen later during the Execution Phase.
let and const are also allocated memory during the Creation Phase but remain uninitialized until execution reaches their declarations, creating the Temporal Dead Zone (TDZ).

## What is the state of let and const during the Creation Phase?

let and const are not assigned any JavaScript value during the Creation Phase. They are allocated memory, but they remain _uninitialized_ until execution reaches their declaration.

This is why the Temporal Dead Zone (TDZ) exists.

## What does "uninitialized" mean?

It means:

The variable exists.
Memory has been allocated.
But JavaScript has not assigned any value yet.
Accessing it before initialization throws a ReferenceError.

## What is the Call Stack?

The Call Stack is a stack data structure used by the JavaScript engine to manage the execution of execution contexts.
OR
The Call Stack is a stack data structure that stores execution contexts and determines which function should execute next.

Whenever a JavaScript program starts, the Global Execution Context is pushed onto the stack. Every time a function is invoked, a new Function Execution Context is created and pushed onto the Call Stack. Once the function finishes execution, its execution context is popped from the stack. Since it follows the Last In, First Out (LIFO) principle, the most recently invoked function always executes first.

## If JavaScript is single-threaded, how can it execute asynchronous operations without blocking the Call Stack?

JavaScript is a single-threaded language, which means it can execute only one task at a time. However, it can still perform asynchronous operations without blocking the main thread because of the Event Loop and the runtime environment.

When JavaScript encounters an asynchronous task like setTimeout, reading a file, or making an API request, it doesn't execute that task itself. Instead, it hands it over to the browser's Web APIs or to libuv in Node.js. This allows JavaScript to continue executing the remaining synchronous code without waiting for the asynchronous task to finish.

Once the asynchronous task is completed, its callback is placed into the Callback Queue or the Microtask Queue. The Event Loop keeps checking whether the Call Stack is empty. When it becomes empty, the Event Loop moves the callback from the queue to the Call Stack, and then JavaScript executes it. This is how JavaScript can handle asynchronous operations while still being single-threaded.

## What is a runtime environment?

A runtime environment is the software that provides everything JavaScript needs to run outside of the language itself.
JavaScript by itself is just a language. It doesn't know how to:

Read files
Make network requests
Create timers
Access the DOM
Open sockets

Browser Runtime

When JavaScript runs in Chrome, the browser provides:

Web APIs (setTimeout, fetch, DOM, etc.)
Event Loop
Callback Queue
Rendering engine
Browser Runtime

├── JavaScript Engine (V8)
├── Web APIs
├── Event Loop
├── Callback Queue
└── DOM
Node.js Runtime

When JavaScript runs in Node.js, it gets different capabilities.

Node.js provides:

File System
HTTP Server
TCP Connections
Streams
libuv
Event Loop

Node.js Runtime

├── V8 Engine
├── libuv
├── File System APIs
├── HTTP Module
├── Event Loop
└── Process APIs

## What is libuv?

libuv is a C library used by Node.js to handle asynchronous operations and implement the Event Loop.

### Why does Node.js need libuv?

Remember:

JavaScript is single-threaded.

If JavaScript itself tried to read a huge file:

fs.readFile(...)

It would freeze the application until the file finished reading.

That's bad.

Instead,

Node.js asks libuv to do the work.

### What does libuv do?

It handles:

File System Operations
DNS
Timers
Network I/O coordination
Event Loop
Thread Pool
Visual Flow

Suppose you write:

fs.readFile("data.txt", callback);

What happens?

JavaScript

      │

      ▼

Node.js

      │

      ▼

libuv

      │

      ▼

Operating System

      │

(Read File)

      │

      ▼

libuv

      │

      ▼

Callback Queue

      │

      ▼

Event Loop

      │

      ▼

Call Stack

      │

      ▼

callback()

Notice:

JavaScript never reads the file itself.

libuv does.

```
Another Example
setTimeout(() => {

    console.log("Hello");

}, 2000);
```

Who waits for 2 seconds?

Not JavaScript.

libuv manages the timer.

After 2 seconds,

libuv says:

"The timer is finished."

Then it places the callback into the appropriate queue.

Is libuv only for asynchronous operations?

Mostly yes.

It provides asynchronous capabilities for Node.js.

Without libuv,

Node.js would behave much more like a synchronous program.

Is libuv the Event Loop?

This is a common interview question.

Answer:

❌ No.

libuv implements the Event Loop for Node.js.

You can think of it this way:

Node.js

│

├── V8 Engine

└── libuv

      │

      ├── Event Loop

      ├── Thread Pool

      ├── Timers

      └── Async I/O

The Event Loop is one part of libuv.

Interview Answer

If asked:

What is libuv?

You can answer:

libuv is a C library used by Node.js to provide asynchronous I/O operations. It is responsible for implementing the Event Loop and managing features like timers, file system operations, networking, and a thread pool for tasks that cannot be handled asynchronously by the operating system. It allows Node.js to perform non-blocking operations while JavaScript continues executing other code.

## What is a Thread Pool?

A Thread Pool is a group of worker threads managed by libuv that perform certain time-consuming operations in the background, allowing the JavaScript thread to continue executing other code.

## If JavaScript is single-threaded, how does it perform asynchronous operations? OR How does Node.js handle asynchronous operations?

JavaScript executes only synchronous code on its main thread. Whenever it encounters an asynchronous operation like reading a file, making an API request, or a timer, it hands that work over to the Node.js runtime, which uses libuv to manage it. While the asynchronous operation is running in the background, JavaScript continues executing the remaining synchronous code. Once the operation is complete, its callback is placed into the appropriate queue. The Event Loop keeps checking whether the Call Stack is empty, and when it is, it moves the callback from the queue to the Call Stack for execution.

```
As we continue preparing, let's not just memorize answers. Instead, let's build a mental map:

Execution Context → "Where does code execute?"
Call Stack → "How does JavaScript keep track of execution?"
Runtime Environment → "Who provides APIs like timers and file system?"
libuv → "Who manages asynchronous operations in Node.js?"
Thread Pool → "Who performs certain background tasks?"
Event Loop → "How are completed asynchronous callbacks scheduled back onto the Call Stack?"
```

# What is Hoisting?

Hoisting is JavaScript's behavior of allocating memory for variable and function declarations during the Creation Phase of the Execution Context, before the code starts executing. During this phase, variables declared with var are initialized with undefined, while let and const are allocated memory but remain uninitialized, placing them in the Temporal Dead Zone (TDZ) until execution reaches their declaration. Accessing a let or const variable before initialization results in a ReferenceError. Function declarations are also hoisted, with their complete function definition available during the Creation Phase.

# Does JavaScript actually move variables to the top of the code?

No. JavaScript does not actually move variables or functions to the top of the code. Hoisting is just a behavior that happens during the Creation Phase of the Execution Context. Before the code starts executing, JavaScript scans the code and prepares memory for variable and function declarations. The source code stays exactly where it is.

# Why are function declarations fully hoisted while var variables are initialized with undefined?

During the Creation Phase, JavaScript only prepares declarations. For a var variable, it creates memory and initializes it with undefined. The actual value is assigned later during the Execution Phase. A function declaration is different because the complete function is part of the declaration itself, so JavaScript stores the entire function during the Creation Phase. That's why we can call a function before its declaration, but a var variable only has the value undefined until its assignment is executed.

# What is a Function Declaration?

A Function Declaration is a function that is declared using the function keyword with a name. It is fully available during the Creation Phase, so it can be called before its declaration because JavaScript hoists the entire function definition.

# What is Function Expression?

A Function Expression is a function assigned to a variable. Unlike a Function Declaration, only the variable is hoisted, not the function itself. So we cannot call it before its assignment.

# What is Arrow Function?

An Arrow Function is also a Function Expression because it is assigned to a variable. Therefore, its hoisting behavior is the same as a Function Expression. The main differences are that it has a lexical this, cannot be used as a constructor with new, and doesn't have its own arguments object.

# What is this?

this refers to the object that is calling the function.

const person = {
name: "HD",

greet() {
console.log(this.name);
},
};

person.greet();

Output: HD

Here,
this === person

because person is calling greet().

### Normal Function

const person = {
name: "HD",

greet: function () {
console.log(this.name);
},
};

person.greet();

Output:

HD

No problem.

### Arrow Function

const person = {
name: "HD",

greet: () => {
console.log(this.name);
},
};

person.greet();

Output:

undefined

Why?

Because Arrow Functions don't create their own this.

Instead,

they use the `this` value from where they were created.

This is called Lexical this.

A simpler way to say it:

Arrow Functions don't have their own this. They use the this from their surrounding scope.

That's all "lexical this" means.

## What is the Temporal Dead Zone (TDZ) in JavaScript?

The Temporal Dead Zone (TDZ) is the period between the Creation Phase of the Execution Context and the point where a let or const variable is initialized. During this time, memory is already allocated for the variable, but it is not initialized. If we try to access it before execution reaches its declaration, JavaScript throws a ReferenceError.

## What is a Lexical Environment in JavaScript?

A Lexical Environment is the internal structure that JavaScript creates for every Execution Context.
It stores the variables and functions of the current scope, along with a reference to its outer (parent) environment.

Think of It Like a Folder 📁

Imagine every function gets its own folder.

Inside the folder:

outer()

Folder

│

├── b = 20

├── function inner()

└── Link to Global Folder

Now inner() gets another folder.

inner()

Folder

│

├── c = 30

└── Link to outer() Folder

Every folder has:

Its own variables
A link to the parent folder

That folder is the Lexical Environment.

## What is Lexical Scope?

A function can access variables from its own scope and from the outer scopes where it was defined.

### How does inner() find the variable a?

```
let a = 10;

function outer() {
let b = 20;

function inner() {
console.log(a);
console.log(b);
}

inner();
}

outer();
```

When JavaScript creates the inner() function, it also creates its Lexical Environment. This Lexical Environment contains a reference to its outer environment. If JavaScript cannot find a inside inner(), it follows that reference to the outer environment. If it still doesn't find it there, it continues following the outer references until it reaches the global scope, where it finds a.

## What is Scope chain?

The Scope Chain is the process JavaScript uses to find a variable. It first looks in the current scope. If the variable is not found, it follows the reference to the outer scope. It continues searching through each parent scope until the variable is found or it reaches the global scope.

## What is a Closure in JavaScript?

A Closure is a function bundled toghether with its Lexical Environment. It allows an inner function to access the variables of its outer function even after the outer function has finished executing. This happens because the inner function keeps a reference to the outer function's Lexical Environment.

function outer() {
let count = 0;

function inner() {
count++;
console.log(count);
}

return inner;
}

const counter = outer();

counter();
counter();
counter();

### why isn't count removed from memory?

The variable is not removed from memory because the inner function still has a reference to the outer function's Lexical Environment. Since the variable is still being referenced, JavaScript's Garbage Collector cannot remove it. It stays in memory until there are no more references to it.

#### Can you give me a real-world use case of Closures?

Data encapsulation (private variables)
Function factories
Event handlers
setTimeout
Callbacks
React Hooks
Middleware in Node.js

## What is the this keyword in JavaScript?

this is a special keyword that refers to the object that is calling the function. Its value is decided at the time the function is called, not when it is created.

## What is call, apply and bind?

### Call

call() is a method that immediately invokes a function while explicitly setting the value of this. Any additional arguments are passed individually.

```
const person1 = {
  name: "HD",

  greet() {
    console.log(`Hello ${this.name}`);
  },
};

const person2 = {
  name: "John",
};

person1.greet.call(person2);


#### Example with Arguments
function greet(city, country) {
  console.log(
    `Hello I'm ${this.name} from ${city}, ${country}`
  );
}

const person = {
  name: "HD",
};

greet.call(person, "Ahmedabad", "India");
```

### Apply

apply() immediately invokes a function while explicitly setting the value of this. The only difference is that arguments are passed as an array.

```
greet.apply(person, ["Ahmedabad", "India"]);
```

### Bind

bind() does not invoke the function immediately. Instead, it returns a new function with the value of this permanently bound to the object provided.

## What is Synchronous Programming?

Synchronous programming is a way of executing code where one statement is executed at a time, in the order it appears. The next statement does not start until the current one has finished executing.

## What is event loop?

The Event Loop is a mechanism in JavaScript that continuously monitors the Call Stack. When the Call Stack becomes empty, it checks the Microtask Queue first and moves all pending callbacks to the Call Stack for execution. Once the Microtask Queue is empty, it processes callbacks from the Callback Queue. This allows JavaScript to handle asynchronous operations without blocking the main thread.

## What is starvation?

Starvation occurs when lower-priority tasks keep waiting because higher-priority tasks continue to execute. In JavaScript, this can happen when the Microtask Queue keeps receiving new tasks, preventing the Callback Queue from being processed.

## What is Promise?

Promise is an object which represent the future value of asynchronous operations. It has three states: Pending, Fulfilled and Rejected. It was introduced to simplify asynchronous programming and avoid callback hell.

## What is the difference between promise and async/await?

Promises and async/await are both used to handle asynchronous operations. A Promise uses .then(), .catch(), and .finally() for handling results, while async/await provides a cleaner syntax that makes asynchronous code look like synchronous code. Internally, async/await is built on top of Promises. An async function always returns a Promise, and await pauses the execution of the current async function until the Promise is fulfilled or rejected.

#### How does event loop executes below code?

console.log("A");

setTimeout(() => {
console.log("B");

Promise.resolve().then(() => {
console.log("C");
});
}, 0);

Promise.resolve().then(() => {
console.log("D");

setTimeout(() => {
console.log("E");
}, 0);
});

console.log("F");

###### Output

A
F
D
B
C
E

### what is Promise.all()?

Promise.all() is used to execute multiple independent asynchronous operations concurrently. It returns a single Promise that resolves only after all the Promises are fulfilled. If any one Promise fails, it immediately rejects with that error.

### what is Promise.allSettled()?

Promise.allSettled() accepts an iterable of Promises and waits for all of them to settle, whether they are fulfilled or rejected. It always resolves with an array of objects describing the status and result of each Promise. Unlike Promise.all(), it never rejects because of a single failed Promise.

### what is Promise.race()?

Promise.race() accepts multiple Promises and returns a Promise that settles as soon as the first Promise settles, whether it is fulfilled or rejected. The remaining Promises continue running, but their results are ignored.

### What is Promise.any()?

Promise.any() accepts multiple Promises and returns the first fulfilled Promise. It ignores rejected Promises unless all Promises reject. If every Promise rejects, it rejects with an AggregateError.
