# JavaScript Interview Prep

## Execution Context & Call Stack

### What is Execution Context?

Execution Context is the environment in which JavaScript code is prepared and executed. Whenever a JavaScript program starts, the engine creates a Global Execution Context, and every time a function is invoked, it creates a new Function Execution Context.

Each execution context contains the information required to execute the code, including variable bindings, the lexical environment for scope resolution, and the `this` binding.

An execution context goes through two phases: the **Creation Phase**, where memory is allocated for variables and functions, and the **Execution Phase**, where the code executes line by line and variables receive their values.

Once a function finishes execution, its execution context is removed from the call stack, while the Global Execution Context remains until the program terminates.

#### Execution Context phases

Every Execution Context (Global or Function) goes through 2 phases:

1. **Creation Phase** (Memory Creation Phase)
2. **Execution Phase** (Code Execution Phase)

### What is the Call Stack?

The Call Stack is a stack data structure used by the JavaScript engine to manage the execution of execution contexts — it stores execution contexts and determines which function should execute next.

Whenever a JavaScript program starts, the Global Execution Context is pushed onto the stack. Every time a function is invoked, a new Function Execution Context is created and pushed onto the Call Stack. Once the function finishes execution, its execution context is popped from the stack. Since it follows the Last In, First Out (LIFO) principle, the most recently invoked function always executes first.

### What is a runtime environment?

A runtime environment is the software that provides everything JavaScript needs to run outside of the language itself. JavaScript by itself is just a language — it doesn't know how to read files, make network requests, create timers, access the DOM, or open sockets.

When JavaScript runs in a browser like Chrome, the browser provides the runtime:

```text
Browser Runtime
├── JavaScript Engine (V8)
├── Web APIs
├── Event Loop
├── Callback Queue
└── DOM
```

> Node.js provides its own runtime with different capabilities (libuv, the file system, HTTP, streams). That's covered in detail in the **Node.js Interview Prep** file.

## Scope & Closures

### What is a Lexical Environment in JavaScript?

A Lexical Environment is the internal structure that JavaScript creates for every Execution Context. It stores the variables and functions of the current scope, along with a reference to its outer (parent) environment.

**Think of it like a folder 📁** — imagine every function gets its own folder:

```text
outer() Folder
├── b = 20
├── function inner()
└── Link to Global Folder
```

Now `inner()` gets another folder:

```text
inner() Folder
├── c = 30
└── Link to outer() Folder
```

Every folder has its own variables and a link to the parent folder. That folder is the Lexical Environment.

### What is Lexical Scope?

A function can access variables from its own scope and from the outer scopes where it was defined.

**How does `inner()` find the variable `a`?**

```js
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

When JavaScript creates the `inner()` function, it also creates its Lexical Environment, which contains a reference to its outer environment. If JavaScript cannot find `a` inside `inner()`, it follows that reference to the outer environment. If it still doesn't find it there, it continues following the outer references until it reaches the global scope, where it finds `a`.

### What is the Scope Chain?

The Scope Chain is the process JavaScript uses to find a variable. It first looks in the current scope. If the variable is not found, it follows the reference to the outer scope, continuing through each parent scope until the variable is found or it reaches the global scope.

### What is a Closure in JavaScript?

A Closure is a function bundled together with its Lexical Environment. It allows an inner function to access the variables of its outer function even after the outer function has finished executing. This happens because the inner function keeps a reference to the outer function's Lexical Environment.

```js
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
```

**Why isn't `count` removed from memory?**

The variable is not removed from memory because the inner function still has a reference to the outer function's Lexical Environment. Since the variable is still being referenced, JavaScript's Garbage Collector cannot remove it. It stays in memory until there are no more references to it.

**Real-world use cases of Closures:**

- Data encapsulation (private variables)
- Function factories
- Event handlers
- `setTimeout`
- Callbacks
- React Hooks
- Middleware in Node.js

## Hoisting & Temporal Dead Zone

### What is Hoisting?

Hoisting is JavaScript's behavior of allocating memory for variable and function declarations during the Creation Phase of the Execution Context, before the code starts executing. During this phase, variables declared with `var` are initialized with `undefined`, while `let` and `const` are allocated memory but remain uninitialized, placing them in the Temporal Dead Zone (TDZ) until execution reaches their declaration. Accessing a `let` or `const` variable before initialization results in a `ReferenceError`. Function declarations are also hoisted, with their complete function definition available during the Creation Phase.

### Does JavaScript actually move variables to the top of the code?

No. JavaScript does not actually move variables or functions to the top of the code. Hoisting is just a behavior that happens during the Creation Phase of the Execution Context. Before the code starts executing, JavaScript scans the code and prepares memory for variable and function declarations — the source code stays exactly where it is.

### Why are function declarations fully hoisted while `var` variables are initialized with `undefined`?

During the Creation Phase, JavaScript only prepares declarations:

- Function declarations are stored with their complete function object, so they can be invoked before their declaration in the source code.
- `var` declarations are allocated memory and initialized to `undefined`; the actual assignment happens later during the Execution Phase.
- `let` and `const` are also allocated memory during the Creation Phase but remain uninitialized until execution reaches their declarations, creating the Temporal Dead Zone (TDZ).

That's why we can call a function before its declaration, but a `var` variable only has the value `undefined` until its assignment is executed.

### What is the state of `let` and `const` during the Creation Phase?

`let` and `const` are not assigned any JavaScript value during the Creation Phase. They are allocated memory, but they remain *uninitialized* until execution reaches their declaration. This is why the Temporal Dead Zone (TDZ) exists.

### What does "uninitialized" mean?

It means:

- The variable exists
- Memory has been allocated
- But JavaScript has not assigned any value yet
- Accessing it before initialization throws a `ReferenceError`

### What is the Temporal Dead Zone (TDZ) in JavaScript?

The Temporal Dead Zone (TDZ) is the period between the Creation Phase of the Execution Context and the point where a `let` or `const` variable is initialized. During this time, memory is already allocated for the variable, but it is not initialized. If we try to access it before execution reaches its declaration, JavaScript throws a `ReferenceError`.

## Functions

### What is a Function Declaration?

A Function Declaration is a function that is declared using the `function` keyword with a name. It is fully available during the Creation Phase, so it can be called before its declaration because JavaScript hoists the entire function definition.

### What is a Function Expression?

A Function Expression is a function assigned to a variable. Unlike a Function Declaration, only the variable is hoisted, not the function itself — so we cannot call it before its assignment.

### What is an Arrow Function?

An Arrow Function is also a Function Expression because it is assigned to a variable. Therefore, its hoisting behavior is the same as a Function Expression. The main differences are that it has a lexical `this`, cannot be used as a constructor with `new`, and doesn't have its own `arguments` object.

### What is a First Class Function?

A First Class Function is a function that is treated like any other value in JavaScript. This means a function can be:

- Assigned to a variable
- Passed as an argument
- Returned from another function
- Stored in objects or arrays

Because JavaScript treats functions as first-class citizens, they have the same capabilities as other data types like strings, numbers, and objects.

**Example — assigned to a variable:**

```js
function greet(name) {
  return `Hello ${name}`;
}

const sayHello = greet;

console.log(sayHello("Harshit"));
// Output: Hello Harshit
```

**Example — returned from another function:**

```js
function multiply(multiplier) {
  return function (num) {
    return num * multiplier;
  };
}

const double = multiply(2);

console.log(double(5));
// Output: 10
```

**Example — stored inside an object:**

```js
const calculator = {
  add(a, b) {
    return a + b;
  },
};

console.log(calculator.add(5, 6));
```

**Real-world examples:**

```js
// Passed as a value to app.get()
app.get("/users", getUsers);

// Passed as a value to setTimeout()
setTimeout(sendEmail, 5000);
```

### What is a Higher Order Function?

A Higher Order Function (HOF) is a function that either accepts one or more functions as arguments, or returns another function. If either condition is true, it's a Higher Order Function.

Common built-in examples: `map()`, `filter()`, `reduce()`, `setTimeout()`, `forEach()`, `sort()` — all of these accept a function as an argument.

```js
const numbers = [1, 2, 3];

const doubled = numbers.map((num) => num * 2);
// map() → Higher Order Function
// (num => num * 2) → Callback Function

const even = numbers.filter((num) => num % 2 === 0);
// filter() is also a Higher Order Function
```

Express middleware is another example — `app.get()` accepts functions as arguments (`authMiddleware`, `getUsers`), so it's a Higher Order Function too:

```js
app.get("/users", authMiddleware, getUsers);
```

### What is Currying?

Currying is a technique where a function that takes multiple arguments is transformed into a sequence of functions, each taking one argument at a time.

```js
function multiply(a) {
  return function (b) {
    return a * b;
  };
}

console.log(multiply(2)(5));
// Output: 10
```

### What is Function Composition?

Function composition is a technique where the output of one function becomes the input of the next function. Multiple small functions are chained together to produce a final result.

```js
function double(num) {
  return num * 2;
}

function addFive(num) {
  return num + 5;
}

function toString(num) {
  return String(num);
}

const result = toString(addFive(double(10)));

console.log(result);
// Output: "25"
```

```text
10 → double() → 20 → addFive() → 25 → toString() → "25"
```

## `this`, call, apply, and bind

### What is `this`?

`this` is a special keyword that refers to the object that is calling the function. Its value is decided at the time the function is called, not when it is created.

```js
const person = {
  name: "HD",
  greet() {
    console.log(this.name);
  },
};

person.greet();
// Output: HD
```

Here, `this === person` because `person` is calling `greet()`.

**Normal function:**

```js
const person = {
  name: "HD",
  greet: function () {
    console.log(this.name);
  },
};

person.greet();
// Output: HD — this refers to person, no problem.
```

**Arrow function:**

```js
const person = {
  name: "HD",
  greet: () => {
    console.log(this.name);
  },
};

person.greet();
// Output: undefined
```

Why? Because Arrow Functions don't create their own `this`. Instead, they use the `this` value from where they were created — this is called **lexical `this`**. Put simply: Arrow Functions don't have their own `this`; they use the `this` from their surrounding scope.

### What is `call`, `apply`, and `bind`?

**`call`** immediately invokes a function while explicitly setting the value of `this`. Any additional arguments are passed individually.

```js
const person1 = {
  name: "HD",
  greet() {
    console.log(`Hello ${this.name}`);
  },
};

const person2 = { name: "John" };

person1.greet.call(person2);
```

With arguments:

```js
function greet(city, country) {
  console.log(`Hello I'm ${this.name} from ${city}, ${country}`);
}

const person = { name: "HD" };

greet.call(person, "Ahmedabad", "India");
```

**`apply`** immediately invokes a function while explicitly setting the value of `this`. The only difference from `call()` is that arguments are passed as an array.

```js
greet.apply(person, ["Ahmedabad", "India"]);
```

**`bind`** does not invoke the function immediately. Instead, it returns a new function with the value of `this` permanently bound to the object provided.

## Asynchronous JavaScript

### What is Synchronous Programming?

Synchronous programming is a way of executing code where one statement is executed at a time, in the order it appears. The next statement does not start until the current one has finished executing.

### If JavaScript is single-threaded, how can it execute asynchronous operations without blocking the Call Stack?

JavaScript is a single-threaded language, which means it can execute only one task at a time. However, it can still perform asynchronous operations without blocking the main thread because of the Event Loop and the runtime environment.

When JavaScript encounters an asynchronous task like `setTimeout`, reading a file, or making an API request, it doesn't execute that task itself. Instead, it hands it over to the browser's Web APIs (or to libuv in Node.js). This allows JavaScript to continue executing the remaining synchronous code without waiting for the asynchronous task to finish.

Once the asynchronous task completes, its callback is placed into the Callback Queue or the Microtask Queue. The Event Loop keeps checking whether the Call Stack is empty. When it becomes empty, the Event Loop moves the callback from the queue to the Call Stack, and JavaScript executes it. This is how JavaScript handles asynchronous operations while still being single-threaded.

### What is the Event Loop?

The Event Loop is a mechanism in JavaScript that continuously monitors the Call Stack. When the Call Stack becomes empty, it checks the Microtask Queue first and moves all pending callbacks to the Call Stack for execution. Once the Microtask Queue is empty, it processes callbacks from the Callback Queue. This allows JavaScript to handle asynchronous operations without blocking the main thread.

> Node.js implements a more detailed, multi-phase Event Loop via libuv — see the **Node.js Interview Prep** file for the phase-by-phase breakdown.

### What is Starvation?

Starvation occurs when lower-priority tasks keep waiting because higher-priority tasks continue to execute. In JavaScript, this can happen when the Microtask Queue keeps receiving new tasks, preventing the Callback Queue from being processed.

### What is a Promise?

A Promise is an object that represents the future value of an asynchronous operation. It has three states: **Pending**, **Fulfilled**, and **Rejected**. It was introduced to simplify asynchronous programming and avoid callback hell.

### What is the difference between a Promise and `async`/`await`?

Promises and `async`/`await` are both used to handle asynchronous operations. A Promise uses `.then()`, `.catch()`, and `.finally()` for handling results, while `async`/`await` provides a cleaner syntax that makes asynchronous code look like synchronous code. Internally, `async`/`await` is built on top of Promises — an `async` function always returns a Promise, and `await` pauses the execution of the current async function until the Promise is fulfilled or rejected.

### How does the event loop execute the following code?

```js
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
```

**Output:**

```text
A
F
D
B
C
E
```

### What is `Promise.all()`?

`Promise.all()` is used to execute multiple independent asynchronous operations concurrently. It returns a single Promise that resolves only after all the Promises are fulfilled. If any one Promise fails, it immediately rejects with that error.

### What is `Promise.allSettled()`?

`Promise.allSettled()` accepts an iterable of Promises and waits for all of them to settle, whether they are fulfilled or rejected. It always resolves with an array of objects describing the status and result of each Promise. Unlike `Promise.all()`, it never rejects because of a single failed Promise.

### What is `Promise.race()`?

`Promise.race()` accepts multiple Promises and returns a Promise that settles as soon as the first Promise settles, whether it is fulfilled or rejected. The remaining Promises continue running, but their results are ignored.

### What is `Promise.any()`?

`Promise.any()` accepts multiple Promises and returns the first fulfilled Promise. It ignores rejected Promises unless all Promises reject. If every Promise rejects, it rejects with an `AggregateError`.

## Performance Optimization Techniques

### What is Debouncing?

Debouncing is a technique that delays the execution of a function until a specified amount of time has passed since the last event occurred.

```js
function debounce(fn, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      fn(...args);
    }, delay);
  };
}

function search(query) {
  console.log("Searching:", query);
}

const debouncedSearch = debounce(search, 500);

debouncedSearch("t");
debouncedSearch("te");
debouncedSearch("tes");
debouncedSearch("test");
// Output after 500ms: "Searching: test" — only once
```

### What is Throttling?

Throttling is a technique that limits how often a function can execute within a specified time interval.

```js
function throttle(fn, delay) {
  let shouldWait = false;

  return function (...args) {
    if (shouldWait) {
      return;
    }

    fn(...args);

    shouldWait = true;

    setTimeout(() => {
      shouldWait = false;
    }, delay);
  };
}

function log() {
  console.log("Scrolling...");
}

const throttledScroll = throttle(log, 1000);

window.addEventListener("scroll", throttledScroll);
```

## DOM Events

### What is Event Bubbling?

Event Bubbling is the default event propagation mechanism in JavaScript. When an event occurs on a child element, it first executes on the target element and then propagates upward through its parent elements until it reaches the document.
