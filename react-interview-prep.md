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

```
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

```
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

```
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

```
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

*Functional Components* are the standard approach.

### 1. Functional Components

Definition

A functional component is a JavaScript function that returns JSX.

Example:

```
function Welcome() {
  return <h1>Hello React</h1>;
}
```

This is a component.

### How Do We Use a Component?

We use components like HTML tags.

Example:

```
function App() {
  return <Welcome />;
}
```

Notice:

```
<Welcome />
```

This is not an HTML element.

It is our custom React component.

## Component Composition

Component composition means building complex UIs by combining smaller components together.

Example:

Small components:

```
function Avatar() {
  return <img />;
}

function UserName() {
  return <h2>John</h2>;
}
```

Combine them:

```
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

```
Profile

├── Avatar

└── UserName
```

This is composition.

## React Component Lifecycle

The React component lifecycle describes the different stages a component goes through from creation to removal from the DOM.

A component generally has three major phases:

```
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

```
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

```
function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

Initial:

```
count = 0
```

User clicks:

```
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

```
App

|

Modal
```

After closing:

```
App
```

Modal is unmounted.

## What is Props?

Props are read-only data passed from a parent component to a child component. They allow components to communicate and make components reusable.

### Using Props

Component:

```
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

```
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

```
props = {
  name: "Laptop",
  price: "1000",
};
```

Second component:

```
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

```
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

```
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

```
let age = 20;

age = 21;
```

React has no idea this happened.

### Enter useState

React provides a Hook called useState.

```
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

```
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

```
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

```
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

```
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

```
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

```
function App() {
  function handleClick() {
    console.log("Clicked");
  }

  return <button onClick={handleClick}>Click</button>;
}
```

Notice:

```
onClick;
```

This is a React event prop.

### Why do we use preventDefault()?

It prevents the browser's default behavior, such as reloading the page when a form is submitted.

### What is event bubbling?

Event bubbling is the process where an event starts at the target element and propagates upward through its parent elements.

### Difference between preventDefault() and stopPropagation()?

```
| `preventDefault()`                 | `stopPropagation()`                       |
| ----------------------------------- | ------------------------------------------- |
| Stops the browser's default action | Stops the event from bubbling             |
| Doesn't stop propagation           | Doesn't stop the browser's default action |
```

### What is a controlled component?

A controlled component is a form element whose value is managed by React state using useState

## What is a Key?

A key is a unique identifier used by React to identify elements in a list so it can efficiently update, add, or remove items during reconciliation.

```
products.map((product) => <ProductCard key={product.id} name={product.name} />);
```

## What is Controlled Components?

A controlled component is a form element whose value is controlled by React state.

```
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

```
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

```
const [count, setCount];
```

## Why does changing state re-render a component?

Calling the setter function tells React that the state has changed. React schedules a new render, executes the component again, and updates the UI if needed.

## Why must Hooks always be called in the same order?

React associates Hook state with the order in which Hooks are called. Changing the order would cause React to associate the wrong state with the wrong Hook.

## Why Do We Need useEffect?

Let's start with a simple question.

Suppose we want to fetch users from an API.

```
function Users() {
  fetch("/api/users");

  return <h1>Users</h1>;
}
```

Looks fine?

No.

Why?

Remember what happens when state changes.

```
State Changes

↓

Component Re-renders

↓

Component Function Runs Again
```

That means:

```
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

```
Remember this:

Render UI First

↓

Run useEffect Later
```

## Dependency Array

The dependency array controls when the effect runs.

There are three common patterns.

### 1. No Dependency Array

```
useEffect(() => {
  console.log("Runs");
});
```

Runs:

```
Initial Render

↓

Every Re-render
```

### 2. Empty Dependency Array

```
useEffect(() => {
  console.log("Runs Once");
}, []);
```

Runs:

```
Component Mounts

↓

Effect Runs

↓

Never Again
```

### 3. Dependencies

```
useEffect(() => {
  console.log("User Changed");
}, [userId]);
```

Runs:

```
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

## What is useRef?

useRef is a React Hook that returns a mutable object whose .current property persists across renders without causing re-renders when it changes.

### Why doesn't useRef trigger a re-render?

Because updating ref.current changes a property on an existing object. React isn't notified that the UI needs to update, unlike when state is updated with a setter function.

### When should you use useRef?

Accessing DOM elements
Focusing inputs
Storing timer IDs
Keeping previous values
Storing mutable values that shouldn't trigger re-renders

### What's the difference between useState and useRef?

| `useState`                     | `useRef`                                |
| -------------------------------- | ------------------------------------------ |
| Stores UI state                | Stores mutable values                   |
| Triggers re-render             | Does **not** trigger re-render          |
| Returns `[value, setter]`      | Returns `{ current }`                   |
| Used when the UI should update | Used when the UI doesn't need to update |

### Why Do We Need useMemo?

Imagine you have a page with two buttons.

Count: 0

[Increment]

[Change Theme]

Clicking Increment should increase the count.

Clicking Change Theme should only change the background color.

Now imagine your component contains an expensive calculation.

const total = expensiveCalculation(count);

Question:

When you click Change Theme, should React execute expensiveCalculation() again?

The answer is No, because count hasn't changed.

Unfortunately, by default, React executes the entire component again on every render.

## What is Memoization?

Memoization is the process of storing the result of an expensive computation and reusing the cached result when the inputs have not changed.

**useMemo**

```
React provides this optimization through:

const value = useMemo(callback, dependencies);
```

Example:

```
const total = useMemo(() => {
  return expensiveCalculation(count);
}, [count]);
```

```
Now React says:

"Only execute this calculation if count changes."
```

## useMemo

useMemo is a React Hook that memoizes the result of an expensive calculation and recomputes it only when its dependencies change.

```
const memoizedValue = useMemo(() => {
  return expensiveCalculation();
}, [dependencies]);
```

### Does useMemo prevent re-renders?

No.

It only avoids recalculating a value.

The component still re-renders whenever its state or props change.

### When should you avoid useMemo?

Avoid it for cheap calculations because memoization itself has a cost. Use it only when the computation is expensive or when it helps prevent unnecessary work.

## What is useCallback?

useCallback memoizes a function so React returns the same function reference until its dependencies change.

```
const memoizedFunction = useCallback(() => {}, [dependencies]);
```

### Why do we use useCallback?

To avoid creating a new function on every render when a stable function reference is important, especially for memoized child components.

### Difference between useMemo and useCallback?

useMemo

Memoizes a **value**

useCallback

Memoizes a **function**

### Does useCallback stop re-renders?

No.

It only returns the same function reference.

A component still re-renders when its state or props change.

### What is `React.memo`, and how is it different from `useMemo`/`useCallback`?

`React.memo` is a higher-order component that wraps a component and skips re-rendering it if its **props** haven't changed (shallow comparison).

```
const ProductCard = React.memo(function ProductCard({ name, price }) {
  console.log('Rendering ProductCard');
  return <div>{name} - {price}</div>;
});
```

If the parent re-renders but passes the same `name` and `price` values, `ProductCard` skips re-rendering.

**Why `React.memo` alone often doesn't work as expected:** if the parent passes a function or object as a prop, a new reference is created on every parent render — even if the function/object is logically "the same." Shallow comparison sees a new reference and re-renders anyway.

```
function Parent() {
  const [count, setCount] = useState(0);
  const handleClick = () => console.log('clicked'); // NEW function every render

  return <ProductCard onClick={handleClick} />; // React.memo doesn't help here
}
```

Fix: wrap the function passed down with `useCallback` so it keeps the same reference across renders, allowing `React.memo`'s shallow comparison to actually skip the re-render.

```
const handleClick = useCallback(() => console.log('clicked'), []);
```

**Summary of the three:**

| | Memoizes | Prevents |
|---|---|---|
| `useMemo` | A computed **value** | Recalculating that value |
| `useCallback` | A **function reference** | Creating a new function each render |
| `React.memo` | A **component's render output** | Re-rendering when props are unchanged |

`useCallback` and `React.memo` are usually used together — `useCallback` keeps the function reference stable, `React.memo` uses that stability to actually skip re-rendering the child.

## Before Learning useContext

Let's understand the problem.

Suppose we have this component tree:

```
App
│
├── Navbar
│
├── Dashboard
│    │
│    ├── Sidebar
│    │
│    └── Profile
│          │
│          └── UserCard
```

The logged-in user's name is stored in App.

```
function App() {
  const [user] = useState({
    name: "HD",
  });
}
```

Now imagine UserCard needs the user's name.

*Without Context*

```
How do we get it there?

App
 │
 │ user
 ▼
Dashboard
 │
 │ user
 ▼
Profile
 │
 │ user
 ▼
UserCard
```

Every intermediate component has to receive the user prop and pass it down.

Example:

```
<App user={user} />

↓

<Dashboard user={user} />

↓

<Profile user={user} />

↓

<UserCard user={user} />
```

Even though:

Dashboard doesn't need it.
Profile doesn't need it.

They only pass it down.

### This is called Prop Drilling

Prop drilling is the process of passing props through multiple intermediate components that don't actually use them, just so deeper components can access the data.

```
Example:

App

↓

Dashboard

↓

Profile

↓

UserCard

Only:

UserCard

needs the data.

The other components are just "middlemen."
```

### React's Solution: Context

React says:

"Instead of passing data through every component, let's make the data available directly to any component that needs it."

### How Context Works

Three steps:

```
Create Context

↓

Provide Data

↓

Consume Data
```

#### Step 1: Create Context

```
import { createContext } from "react";

export const UserContext = createContext(null);
```

This creates a Context object.

Think of it as an empty container.

#### Step 2: Provide Data

```
<UserContext.Provider value={user}>
  <Dashboard />
</UserContext.Provider>
```

Now every component inside Dashboard can access user.

#### Step 3: Consume Data

```
import { useContext } from "react";

const user = useContext(UserContext);
```

Now you have direct access to the value.

No prop drilling.

### When Should You Use Context?

Good for:

Logged-in user
Theme (Light/Dark)
Language
Authentication state
Shopping cart
Application settings

### When Should You NOT Use Context?

Don't use Context for every piece of state.

Local component state should usually stay in:

useState()

Context is for data that many components need.

## What is useContext?

*useContext* is a React Hook that allows a component to read the current value of a Context without passing props through intermediate components.

## What problem does it solve?

It solves prop drilling.

## What is Prop Drilling?

Passing props through components that don't use them, just so deeper components can receive the data.

## Can Context replace Redux?

Not completely.

Context is great for sharing data.

Redux (or Zustand, etc.) also provides advanced state management features like middleware, devtools, predictable updates, and more.

## Does Context cause re-renders?

Yes.

If a Provider's value changes, every consuming component that reads that Context is re-rendered.

### Why is a single large Context often a performance problem?

When a Provider's value changes, **every** component consuming that Context re-renders — even if it only reads a small part of the value that didn't actually change.

```
const AppContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'HD' });
  const [theme, setTheme] = useState('light');

  return (
    <AppContext.Provider value={{ user, theme, setUser, setTheme }}>
      <Dashboard />
    </AppContext.Provider>
  );
}
```

If `theme` changes, every component consuming `AppContext` re-renders — including ones that only care about `user`, because the entire context object is a new reference on every Provider render.

**Fixes:**
- Split into multiple, narrower Contexts (`UserContext`, `ThemeContext`) so components only re-render when the specific value they consume changes.
- Memoize the context value with `useMemo` so it doesn't create a new object reference on every render unless its actual contents changed.

```
const value = useMemo(() => ({ user, theme, setUser, setTheme }), [user, theme]);
```

This is a common follow-up after any Context question — interviewers often ask "does Context cause performance issues" specifically to see if you know about this re-render-everything behavior, not just how to set up a Provider.

## What is a Reducer?

A reducer is a pure function that takes the current state and an action, then returns a new state without mutating the existing one.

```
function reducer(state, action) {
  if (action.type === "increment") {
    return state + 1;
  }

  return state;
}
```

Notice something.

The reducer doesn't know who clicked the button.

It only knows:

I received an "increment" action.

## What is an Action?

An action is just a plain JavaScript object describing what happened.

Example:

```
{
  type: "increment";
}

or;

{
  type: "decrement";
}

or;

{
  type: "reset";
}
```

The key idea:

Actions describe what should happen.

The reducer decides how the state changes.

## What is dispatch?

Instead of calling:

```
setCount(...)
```

you call:

```
dispatch({
  type: "increment",
});
```

Think of dispatch as:

"Send this action to the reducer."

### The Complete Flow

This is one of the most important diagrams in React.

```
Button Click
│
▼
dispatch({
type: "increment"
})
│
▼
Reducer
(state, action)
│
▼
Returns New State
│
▼
React Updates UI
```

Notice:

The button never changes the state directly.

### useState vs useReducer

| `useState`                     | `useReducer`                 |
| --------------------------------- | ------------------------------- |
| Simple state                   | Complex state                |
| Few updates                    | Many update types            |
| Direct setter (`setState`)     | `dispatch(action)`           |
| Logic spread across components | Logic centralized in reducer |
| Easier to start                | Better as complexity grows   |

## Custom Hooks

### Suppose you're building two pages.

Page A

```
function Users() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch("/users")
      .then((res) => res.json())
      .then((data) => setUsers(data));
  }, []);
}
```

Page B

```
function Products() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch("/products")
      .then((res) => res.json())
      .then((data) => setProducts(data));
  }, []);
}
```

Notice something?

The API URL changes, but the logic is almost identical.

We're repeating:

useState
useEffect
Loading state
Error state
Fetch logic

This is called duplicate logic.

### What is a Custom Hook?

A custom hook is a JavaScript (or TypeScript) function that starts with use and allows you to reuse stateful logic between multiple components.

### It starts with use

```
useFetch();

useDebounce();

useAuth();

useTheme();
```

### It can use other hooks

Inside a custom hook:

```
function useFetch() {
  useState();

  useEffect();

  useRef();

  useMemo();
}
```

A custom hook is simply a function that calls other hooks.

### Does a custom hook share state between components?

No. A custom hook does not share state. Each component that calls a custom hook gets its own independent instance of the hook's state. Custom hooks are used to reuse stateful logic, not to create shared state. If shared state is required, I would use Context, Redux, Zustand, or another state management solution.

### What is Abort Controller?

AbortController is a browser API that allows you to cancel one or more asynchronous operations, such as a fetch() request.

### Why use AbortController in React?

When a component unmounts, a pending request may no longer be needed. Using AbortController in the useEffect cleanup function cancels the request, avoiding unnecessary work and ensuring only active requests continue.

### Can It Cancel Other Things?

Yes.

It's not limited to fetch.

Many modern browser APIs support AbortSignal.

For example:

fetch()
Streams
Some file operations
Some third-party libraries

It's becoming a standard way to support cancellation in JavaScript.

### Why use generics in a custom hook?

Generics make the hook reusable for different data types while preserving type safety. Instead of creating separate hooks for users, products, or orders, a single generic hook can work with all of them.
