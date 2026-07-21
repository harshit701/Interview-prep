# React Interview Prep

## What is React?

React is an open-source JavaScript library used to build reusable, component-based user interfaces. It follows a declarative programming model and efficiently updates the UI using a Virtual DOM.

OR

React is an open-source JavaScript library for building reusable, component-based user interfaces. It follows a declarative programming model, where the UI is derived from the application's state, and uses a Virtual DOM to efficiently update the real DOM.

### Is React a framework?

No. React is a library because it focuses only on building the user interface. Features like routing, global state management, and API handling are provided by separate libraries or frameworks.

### Who developed React?

React was developed by Facebook (now Meta) and was first released in 2013.

### Why is React popular?

Component-based architecture
Reusable UI components
Declarative programming
Efficient updates using the Virtual DOM
Strong ecosystem
Excellent developer experience
Backed by Meta and a large community

## What is the DOM?

DOM stands for Document Object Model

The DOM (Document Object Model) is a tree-like representation of an HTML document created by the browser. It allows JavaScript to access and modify the webpage dynamically.

### What is the Real DOM?

The Real DOM is the actual DOM maintained by the browser. Any changes made to it are reflected on the webpage.

### Why are frequent DOM updates considered expensive?

Updating the DOM can trigger additional browser work such as recalculating styles, recalculating layout, repainting, and compositing. Frequent or unnecessary updates can reduce performance, especially in large applications.

## What is the Virtual DOM?

The Virtual DOM is a lightweight JavaScript representation of the Real DOM. React uses it to compare UI changes and update only the necessary parts of the Real DOM.

### Why is the Virtual DOM used?

Updating the Real DOM can trigger expensive browser work. React first compares Virtual DOM trees in memory and then applies only the required changes to the Real DOM.

### Is the Virtual DOM the same as the Real DOM?

No.

Real DOM → Browser
Virtual DOM → JavaScript objects managed by React

### Does React recreate the entire page?

No.

When state changes, React creates a new Virtual DOM for the affected tree, compares it with the previous one, and applies only the necessary changes to the Real DOM.

### Does every re-render update the Real DOM?

No.

A component may re-render, but if the rendered output is unchanged, React may not make any changes to the Real DOM.

## What is Diffing Algorithm?

The Diffing Algorithm is the process React uses to compare the old Virtual DOM with the new Virtual DOM to determine the minimum number of changes required to update the Real DOM.
OR
Diffing is the algorithm React uses during reconciliation to compare the old and new Virtual DOM trees.

## What is Reconciliation?

Reconciliation is React's process of comparing the old and new Virtual DOM, determining what changed, and efficiently updating the Real DOM.

### Is Reconciliation the same as Diffing?

No.

Diffing is one step inside the reconciliation process.

Diffing identifies changes.

Reconciliation uses those changes to efficiently update the Real DOM.

### What triggers Reconciliation?

Usually:

State updates
Props changes
Context changes
Parent component re-renders

These cause React to render again and begin the reconciliation process.

### Diffing is a part of reconciliation, not the whole reconciliation process.

```text
Reconciliation
│
├── 1. Render components
├── 2. Create new Virtual DOM
├── 3. Diffing (compare old vs new Virtual DOM)
├── 4. Decide what to update
└── 5. Update the Real DOM (Commit)
```

## What is Rendering?

Rendering is the process where React executes components and creates a Virtual DOM representing what the UI should look like.

## What is Re-rendering?

Re-rendering is when React executes a component again because state, props, context, or a parent update requires React to produce a new UI description.

### Does Rendering mean DOM update?

No.

Rendering creates a new Virtual DOM.

The DOM is updated later during the Commit Phase if React finds changes.

### What is the Render Phase?

The Render Phase is where React executes components, creates the new Virtual DOM, and compares it with the previous one. No changes are made to the Real DOM during this phase.

### What is the Commit Phase?

The Commit Phase is where React applies the necessary changes to the Real DOM, updates refs, and then runs effects like useEffect.

### Rendering phases

```text
User Action
      │
      ▼
State/Props Change
      │
      ▼
Component Re-renders
      │
      ▼
Render Phase
      │
      ├── Execute Components
      ├── Create New Virtual DOM
      └── Diff Old vs New
      │
      ▼
Commit Phase
      │
      ├── Update Real DOM
      ├── Update Refs
      └── Run Effects
      │
      ▼
Browser Paint
```

## React Fiber Architecture

React Fiber is React's rendering engine introduced in React 16. It allows React to prioritize, pause, resume, and interrupt rendering work, making applications more responsive and enabling features like Concurrent Rendering.

**Important:** Concurrent rendering doesn't mean React renders multiple components at the exact same time on multiple CPU cores. It means React can make rendering interruptible and schedule work so higher-priority updates are handled first.

### Is Fiber Faster?

Many people answer

Fiber is faster.

Not exactly.

Fiber isn't primarily about raw speed.

It makes React more **responsive**.

### Why was Fiber introduced?

Before Fiber, React's rendering was synchronous and couldn't be interrupted. Fiber introduced scheduling and prioritization, allowing React to respond more quickly to high-priority user interactions.

## What is JSX?

JSX (JavaScript XML) is a syntax extension for JavaScript that allows developers to write HTML-like code inside JavaScript. React uses JSX to describe what the UI should look like.

### Is JSX HTML?

No.

It looks similar to HTML but is not HTML. JSX is JavaScript syntax that is compiled into JavaScript function calls.

### an React work without JSX?

Yes.

JSX is optional.

You can use

React.createElement(...)

or, in modern React, the underlying JSX runtime functions directly (though developers almost always write JSX).

### Why can't browsers understand JSX?

Because browsers understand JavaScript, HTML, and CSS—not JSX. JSX must be transformed into JavaScript during the build process.

## React Fragments

A React Fragment is a special component that allows you to group multiple elements together without adding an extra DOM element.

### When Should You Use the Full Syntax?

Usually, you'll use the shorthand.

However, if you need to assign a key to a fragment (for example, inside a list), you must use the full syntax.

Example:

```js
import { Fragment } from "react";

function App() {
  return (
    <>
      <h1>Hello</h1>
    </>
  );
}
```

or

```js
import { Fragment } from "react";

function List() {
  return (
    <Fragment key="1">
      <h1>Hello</h1>
      <p>World</p>
    </Fragment>
  );
}
```

The shorthand <>...</> cannot accept props or keys.

## What is a Component?

A React component is a reusable, independent piece of UI that returns JSX describing what should be rendered on the screen.

### Types of React Components

Historically React had two types:

1. Class Components
2. Functional Components

Today:

_Functional Components_ are the standard approach.

### 1. Functional Components

Definition

A functional component is a JavaScript function that returns JSX.

Example:

```js
function Welcome() {
  return <h1>Hello React</h1>;
}
```

This is a component.

### How Do We Use a Component?

We use components like HTML tags.

Example:

```js
function App() {
  return <Welcome />;
}
```

Notice:

```js
<Welcome />
```

This is not an HTML element.

It is our custom React component.

## Component Composition

Component composition means building complex UIs by combining smaller components together.

Example:

Small components:

```js
function Avatar() {
  return <img />;
}

function UserName() {
  return <h2>John</h2>;
}
```

Combine them:

```js
function Profile() {
  return (
    <div>
      <Avatar />
      <UserName />
    </div>
  );
}
```

Now:

```text
Profile

├── Avatar

└── UserName
```

This is composition.

## React Component Lifecycle

The React component lifecycle describes the different stages a component goes through from creation to removal from the DOM.

A component generally has three major phases:

```text
Mounting
   |
   |
Updating
   |
   |
Unmounting
```

#### 1. Mounting Phase

Mounting is the phase when a component is created and inserted into the DOM for the first time.

Example:

You open a website.

Initially:

```text
Browser
   |
   |
React loads App
   |
   |
Header component created
   |
   |
Inserted into DOM
```

The component is now visible.

#### 2. Updating Phase

Updating occurs when a component's state or props change, causing React to render the component again.

Example:

Counter:

```js
function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

Initial:

```text
count = 0
```

User clicks:

```text
count = 1
```

Component updates.

##### What Causes Updates?

1. State Changes
2. Props Changes
3. Context Changes

#### 3. Unmounting Phase

Unmounting is the phase when a component is removed from the DOM.

Example:

You close a modal:

Before:

```text
App

|

Modal
```

After closing:

```text
App
```

Modal is unmounted.

## What is Props?

Props are read-only data passed from a parent component to a child component. They allow components to communicate and make components reusable.

### Using Props

Component:

```js
function ProductCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.price}</p>
    </div>
  );
}
```

Use it:

```js
<ProductCard
    name="Laptop"
    price="1000"
/>

<ProductCard
    name="Phone"
    price="500"
/>
```

React passes:

First component:

```js
props = {
  name: "Laptop",
  price: "1000",
};
```

Second component:

```js
props = {
  name: "Phone",
  price: "500",
};
```

Same component.

Different data.

### How Props Flow

React follows:

**One-way Data Flow**

Data always moves:

```text
Parent
|
|
▼
Child
```

### Can Child Send Data to Parent?

Directly?

❌ No.

React does not allow child components to modify parent data directly.

The child can communicate upward using a function passed as a prop.

## What is State?

State is data managed by a component that can change over time. When state changes, React re-renders the component to update the UI.

### Why Do We Need State?

Let's build a simple counter.

Without state:

```js
function Counter() {
  let count = 0;

  function increment() {
    count++;
    console.log(count);
  }

  return (
    <>
      <h1>{count}</h1>
      <button onClick={increment}>Increment</button>
    </>
  );
}
```

Looks correct?

Let's see.

User clicks:

count = 1

Console:

1

UI:

0

Why?

Because changing a normal JavaScript variable does not tell React to update the UI.

### React Doesn't Watch Normal Variables

React only watches:

State
Props
Context

It does not track ordinary variables.

Example:

```js
let age = 20;

age = 21;
```

React has no idea this happened.

### Enter useState

React provides a Hook called useState.

```js
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <>
      <h1>{count}</h1>

      <button onClick={() => setCount(count + 1)}>Increment</button>
    </>
  );
}
```

It allows React to:

Store data
Remember it between renders
Re-render when it changes

### What Happens Internally?

Let's connect this to everything we've already learned.

When the component first loads:

```text
Counter()

↓

useState(0)

↓

count = 0

↓

JSX Returned

↓

Virtual DOM

↓

Browser
```

**User clicks:**

setCount(1)

React does this:

```text
State Changed

↓

Component Re-renders

↓

Counter() runs again

↓

New JSX Returned

↓

New Virtual DOM

↓

Diffing

↓

Commit Phase

↓

Browser Updated
```

Notice something important:

**React does not update the existing JSX. It calls your component function again.**

### Functional Updates

Interview favorite.

Suppose:

```text
setCount(count + 1);

setCount(count + 1);

What do you expect?

Many say:

+2

Actually:

+1

Why?

Both updates read the same current value.

Example:

Current:

count = 0

Both calls become:

setCount(1)

setCount(1)

Final:

1
```

### Correct Way

Use a functional update.

```text
setCount(previous => previous + 1);

setCount(previous => previous + 1);

Flow:

0

↓

1

↓

2

Now React uses the latest pending state for each update.
```

### State Batching

React batches multiple state updates together to improve performance.

Example:

```text
setName("John");

setAge(30);

setCity("London");

Instead of rendering three times:

Render

Render

Render

React usually batches them into:

One Render

This reduces unnecessary work.
```

## What is an Event?

An event is an action or occurrence that happens in the browser, such as a click, typing, scrolling, or submitting a form. React allows us to respond to these events by attaching event handlers to elements.

### Event Handling in React

React makes this much simpler.

```js
function App() {
  function handleClick() {
    console.log("Clicked");
  }

  return <button onClick={handleClick}>Click</button>;
}
```

Notice:

```js
onClick;
```

This is a React event prop.

### Why do we use preventDefault()?

It prevents the browser's default behavior, such as reloading the page when a form is submitted.

### What is event bubbling?

Event bubbling is the process where an event starts at the target element and propagates upward through its parent elements.

### Difference between preventDefault() and stopPropagation()?

```text
| `preventDefault()`                 | `stopPropagation()`                       |
| ---------------------------------- | ----------------------------------------- |
| Stops the browser's default action | Stops the event from bubbling             |
| Doesn't stop propagation           | Doesn't stop the browser's default action |
```

### What is a controlled component?

A controlled component is a form element whose value is managed by React state using useState

## What is a Key?

A key is a unique identifier used by React to identify elements in a list so it can efficiently update, add, or remove items during reconciliation.

```js
products.map((product) => <ProductCard key={product.id} name={product.name} />);
```

## What is Controlled Components?

A controlled component is a form element whose value is controlled by React state.

```js
import { useState } from "react";

function Login() {
  const [username, setUsername] = useState("");

  return (
    <input
      type="text"
      value={username}
      onChange={(event) => setUsername(event.target.value)}
    />
  );
}
```

## What is Uncontrolled Components?

An uncontrolled component stores its own state in the DOM instead of React state.

```js
import { useRef } from "react";

function Login() {
  const inputRef = useRef();

  function handleClick() {
    console.log(inputRef.current.value);
  }

  return (
    <>
      <input ref={inputRef} />

      <button onClick={handleClick}>Submit</button>
    </>
  );
}
```

### What is useRef used for?

useRef is used to access DOM elements directly or store mutable values that persist across renders without causing re-renders.

### Why do we use event.preventDefault()?

It prevents the browser's default form submission behavior, allowing React to handle the submission without reloading the page.

# React Hooks

## What is useState?

useState is a React Hook that allows functional components to store and update state. When the state changes, React schedules a re-render of the component.

## What does useState return?

useState returns an array containing the current state value and a state update function.

```js
const [count, setCount];
```

## Why does changing state re-render a component?

Calling the setter function tells React that the state has changed. React schedules a new render, executes the component again, and updates the UI if needed.

## Why must Hooks always be called in the same order?

React associates Hook state with the order in which Hooks are called. Changing the order would cause React to associate the wrong state with the wrong Hook.

## Why Do We Need useEffect?

Let's start with a simple question.

Suppose we want to fetch users from an API.

```js
function Users() {
  fetch("/api/users");

  return <h1>Users</h1>;
}
```

Looks fine?

No.

Why?

Remember what happens when state changes.

```text
State Changes

↓

Component Re-renders

↓

Component Function Runs Again
```

That means:

```text
Users()

↓

fetch()

↓

Users()

↓

fetch()

↓

Users()

↓

fetch()
```

Every render makes another API request.

Not good.

## What is Side Effects?

A side effect is any operation that interacts with something outside the component's rendering process.

Examples:

API requests
Timers (setTimeout, setInterval)
Event listeners
Updating the document title
Accessing Local Storage
Direct DOM manipulation

Returning JSX is not a side effect.

Fetching data is.

## What is useEffect?

useEffect is a React Hook used to perform side effects after a component has rendered.

```text
Remember this:

Render UI First

↓

Run useEffect Later
```

## Dependency Array

The dependency array controls when the effect runs.

There are three common patterns.

### 1. No Dependency Array

```js
useEffect(() => {
  console.log("Runs");
});
```

Runs:

```text
Initial Render

↓

Every Re-render
```

### 2. Empty Dependency Array

```js
useEffect(() => {
  console.log("Runs Once");
}, []);
```

Runs:

```text
Component Mounts

↓

Effect Runs

↓

Never Again
```

### 3. Dependencies

```js
useEffect(() => {
  console.log("User Changed");
}, [userId]);
```

Runs:

```text
Mount

↓

userId Changes

↓

Runs Again

↓

userId Changes Again

↓

Runs Again
```

If userId doesn't change, the effect doesn't run again.
