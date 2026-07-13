## What is TypeScript?

TypeScript is a statically typed superset of JavaScript developed by Microsoft that adds optional static typing and compiles into plain JavaScript.

Let's break that down.

```text
1. Superset of JavaScript

A superset means:

Everything that is valid JavaScript is also valid TypeScript.

Example:

const name = "Harshit";

console.log(name);

This JavaScript code runs perfectly in TypeScript.

TypeScript extends JavaScript; it doesn't replace it.

2. Statically Typed

JavaScript is dynamically typed.

Example:

let age = 25;

age = "Twenty Five";

No error.

TypeScript

let age: number = 25;

age = "Twenty Five";

Error:

Type 'string' is not assignable to type 'number'

The compiler catches the mistake before the application runs.

3. Compiles to JavaScript

Browsers and Node.js cannot execute TypeScript directly.

They only understand JavaScript.

So TypeScript must be compiled.

Example

const name: string = "Harshit";

After compilation

const name = "Harshit";

Notice:

The type information disappears.
```

### Compilation Flow

```text
TypeScript (.ts)

↓

TypeScript Compiler (tsc)

↓

JavaScript (.js)

↓

Node.js / Browser
```

## Why was TypeScript created?

As applications became larger,

JavaScript became difficult to maintain.

Problems:

Runtime type errors
Difficult refactoring
Poor auto-completion
Hard to understand large codebases

TypeScript solves these by introducing a type system.

**Advantages**

1. Early Error Detection

Instead of finding bugs in production,

you find them while writing code.

2. Better IDE Support

Autocomplete

Go to Definition

Rename Symbol

Refactoring

3. Better Readability
   ```js
   function calculateSalary(salary: number, bonus: number): number
   ```

Anyone reading the code immediately knows:

Parameters and Return type

4. Easier Maintenance

Large teams can understand each other's code more easily.

5. Better Refactoring

Renaming a property across thousands of files becomes much safer because the compiler can detect broken references.

**Disadvantages**
Extra compilation step
Slight learning curve
More code to write (types)
Build process becomes more complex

## What is Type Inference?

Type Inference means TypeScript automatically determines the type of a variable without you explicitly specifying it.

```text
Example:

let name = "Harshit";

You didn't write:

let name: string

But TypeScript infers:

let name: string

Another example:

let age = 30;

TypeScript infers:

number
```

## What is Type?

A type tells TypeScript what kind of value a variable can store.

Example:
let age: number = 25;

### Primitive Types

Number
String
Boolean
Null
Undefined
BigInt
Arrays
Objects

## What is Tuple?

A tuple is a fixed-length array where each position has a predefined type.

```ts
let user: [number, string];

user = [1, "Harshit"];
```

Correct

```ts
user = ["Harshit", 1];
```

Wrong. Compiler error.

## What is Enum?

Enums allow us to create a set of named constants.

````ts
enum Role {
    Admin,
    User,
    Manager
}
```
Usage

```ts
const role = Role.Admin;
```

## What is Literal Types?

```ts
let status: "Pending" | "Completed" | "Cancelled";
````

Now only these values are allowed.

## What are the special types?

1. any
2. unknown
3. void
4. never

### What is any?

any disables TypeScript's type checking.

```ts
let value: any = 10;

value = "Harshit";

value = true;

value = {};

value = [];
```

### What is unknown?

unknown can store any value, but you cannot use it directly until you check its type.

```text
let value: unknown = "Harshit";
console.log(value.length);

❌ Error

Because TypeScript doesn't know if it's actually a string.
```

```text
Correct way:

if (typeof value === "string") {
    console.log(value.length);
}

Now it works.
```

#### Why is unknown better than any?

Because TypeScript forces you to perform a type check first.

It keeps type safety.

#### Difference between any and unknown

| any                       | unknown                    |
| ------------------------- | -------------------------- |
| Disables type checking    | Keeps type safety          |
| Can perform any operation | Must narrow the type first |
| Unsafe                    | Safe                       |

### What is void?

This function doesn't return anything.

```ts
function greet(): void {
  console.log("Hello");
}
```

### What is never?

This function never finishes normally.
It either:

throws an error
or runs forever

```ts
function throwError(): never {
  throw new Error("Something went wrong");
}
```

### What is an Interface?

An interface defines the shape (structure) of an object.

Think of it as a contract.

If an object claims to follow an interface, it must contain all the required properties with the correct types.

```ts
interface User {
  readonly id: number; // readonly property
  name: string;
  email: string;
  age?: number; //optional property

  login()?: void; //Interface can contain method
}

const user: User = {
  id: 1,
  name: "Harshit",
  email: "abc@gmail.com",
};
```

### What is Type Aliases?

A type alias lets you create a new name (alias) for any type.

```ts
type User = {
  id: number;
  name: string;
  email: string;
};

const user: User = {
  id: 1,
  name: "Harshit",
  email: "abc@gmail.com",
};
```

It looks very similar to an interface.

That's why interviewers almost always ask:

**What's the difference between interface and type?**

_Differences_

1. Interface can be extended

```ts
interface Person {
  name: string;
}

interface Employee extends Person {
  salary: number;
}
```

Type also supports inheritance using intersections:

```ts
type Person = {
  name: string;
};

type Employee = Person & {
  salary: number;
};
```

Both achieve similar results.

2. Type can represent more than objects

```ts
//Interface
interface User {
  id: number;
}

// Only describes object shapes.
```

```
Type
type ID = number;

or

type Status =
  | "Pending"
  | "Completed";

or

type Add = (
  a: number,
  b: number
) => number;

or

type Numbers = number[];

A type can represent almost anything.
```

3. Declaration Merging
   This is the biggest difference.

Interfaces support declaration merging.

```
interface User {
  name: string;
}

Later:

interface User {
  age: number;
}

TypeScript automatically merges them.

Equivalent to:

interface User {
  name: string;
  age: number;
}
```

```
Type aliases cannot do this.

type User = {
  name: string;
};

type User = {
  age: number;
};

Compile error.
```

4. Union Types

Only type aliases support unions.

type Status =
| "Pending"
| "Completed"
| "Cancelled";

Interfaces cannot.

5. Intersection Types

Type aliases naturally support intersections.

```ts
type Admin = User & Permissions;
```

### Which should you use?

_Use Interface when_:
Modeling objects
Creating contracts
Designing APIs
OOP-style applications

_Use Type when_:
Union types
Intersection types
Function types
Primitive aliases
Tuples
Utility types

## What are Generics?

Generics allow us to write reusable functions, classes, or interfaces that work with different data types while still maintaining type safety.

## What are Utility Types?

Utility Types are built-in TypeScript types that help us create new types from existing types without rewriting them.

Instead of writing new interfaces again and again, we can transform existing ones.

---

Suppose we have:

```ts
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}
```

We'll use this interface for all examples.

---

# 1. Partial<T> ⭐⭐⭐⭐⭐

Makes **all properties optional**.

```ts
type PartialUser = Partial<User>;
```

Now it becomes:

```ts
{
  id?: number;
  name?: string;
  email?: string;
  age?: number;
}
```

Example:

```ts
const updateUser: Partial<User> = {
  name: "Harshit",
};
```

Only updating the name.

---

## Real-world Use

**PATCH API**

```http
PATCH /users/1
```

The client only sends the fields that need updating.

---

# 2. Required<T> ⭐⭐⭐⭐☆

Makes **all properties required**.

Suppose

```ts
interface User {
  id: number;
  name?: string;
}
```

Now

```ts
type RequiredUser = Required<User>;
```

becomes

```ts
{
  id: number;
  name: string;
}
```

Everything is mandatory.

---

# 3. Readonly<T> ⭐⭐⭐⭐☆

Makes every property read-only.

```ts
type ReadonlyUser = Readonly<User>;
```

Now

```ts
user.name = "Rahul";
```

Compile Error.

Useful for:

- Configuration objects
- Constants
- API responses

---

# 4. Pick<T, Keys> ⭐⭐⭐⭐⭐

Pick only the properties you need.

Example:

```ts
type UserSummary = Pick<User, "id" | "name">;
```

Result:

```ts
{
  id: number;
  name: string;
}
```

## Real-world Example

Listing users.

You don't need:

- email
- age

Only:

- id
- name

---

# 5. Omit<T, Keys> ⭐⭐⭐⭐⭐

Opposite of Pick.

Remove unwanted properties.

```ts
type UserWithoutAge = Omit<User, "age">;
```

Result:

```ts
{
  id: number;
  name: string;
  email: string;
}
```

## Real-world Example

Never send passwords.

```ts
interface User {
  id: number;
  name: string;
  password: string;
}
```

```ts
type SafeUser = Omit<User, "password">;
```

---

# 6. Record<K, T> ⭐⭐⭐⭐☆

Creates an object with predefined key and value types.

Example:

```ts
type Scores = Record<string, number>;
```

Now:

```ts
const scores: Scores = {
  Harshit: 95,
  Rahul: 88,
};
```

Keys → `string`

Values → `number`

## Real-world Example

Caching:

```ts
Record<string, User>;
```

---

# 7. Exclude<T, U> ⭐⭐⭐⭐☆

Removes specific types from a union.

```ts
type Status = "Pending" | "Completed" | "Cancelled";
```

Now:

```ts
type ActiveStatus = Exclude<Status, "Cancelled">;
```

Result:

```ts
"Pending" | "Completed";
```

---

# 8. Extract<T, U> ⭐⭐⭐⭐☆

Opposite of Exclude.

Keeps only matching types.

```ts
type Status = "Pending" | "Completed" | "Cancelled";
```

```ts
type Finished = Extract<Status, "Completed" | "Cancelled">;
```

Result:

```ts
"Completed" | "Cancelled";
```

---

# Summary

| Utility Type | Purpose                           |
| ------------ | --------------------------------- |
| Partial      | Makes all properties optional     |
| Required     | Makes all properties required     |
| Readonly     | Makes properties read-only        |
| Pick         | Select specific properties        |
| Omit         | Remove specific properties        |
| Record       | Creates key-value object types    |
| Exclude      | Removes types from a union        |
| Extract      | Keeps matching types from a union |

---

## What is a Type Guard?

A Type Guard is a way to check the type of a variable at runtime so that TypeScript knows its exact type and allows us to safely use it.

**Common Type Guards**

1. typeof

Used for primitive types.

```ts
let value: string | number;

if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

2. instanceof

Used for classes.

```ts
class User {}

const user = new User();

if (user instanceof User) {
  console.log("Valid User");
}
```

Useful when working with objects created using classes.

3. in Operator

Checks whether a property exists.

Example

```ts
interface User {
  name: string;
}

interface Admin {
  permissions: string[];
}

function print(person: User | Admin) {
  if ("permissions" in person) {
    console.log(person.permissions);
  }
}
```

## What is Classes?

A class is a blueprint for creating objects. It defines the properties (data) and methods (behavior) that objects will have.

Example:

```ts
class User {
  id: number;
  name: string;

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }

  greet() {
    console.log(`Hello ${this.name}`);
  }
}

const user = new User(1, "Harshit");

user.greet();
```

Output:

```text
Hello Harshit
```

---

# Constructor

A constructor is a special method that runs automatically when an object is created.

```ts
constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
}
```

When you do:

```ts
new User(1, "Harshit");
```

the constructor executes automatically.

---

# Access Modifiers

## 1. public

Default access modifier.

Accessible from anywhere.

```ts
class User {
  public name: string;

  constructor(name: string) {
    this.name = name;
  }
}

const user = new User("Harshit");

console.log(user.name);
```

Accessible:

- Inside the class ✅
- Outside the class ✅
- Child classes ✅

---

## 2. private

Accessible **only inside the class**.

```ts
class User {
  private password: string;

  constructor(password: string) {
    this.password = password;
  }
}
```

Trying to access it outside:

```ts
console.log(user.password);
```

Compile Error.

### Real-world Examples

Use `private` for:

- Password
- OTP
- Refresh Token
- Secret Key

---

## 3. protected

Accessible:

- Inside the class ✅
- Child classes ✅
- Outside the class ❌

Example:

```ts
class Animal {
  protected age: number = 5;
}

class Dog extends Animal {
  printAge() {
    console.log(this.age);
  }
}
```

Outside:

```ts
dog.age;
```

Compile Error.

---

# readonly

A readonly property can only be assigned once.

```ts
class User {
  readonly id: number;

  constructor(id: number) {
    this.id = id;
  }
}
```

Later:

```ts
user.id = 10;
```

Compile Error.

### Real-world Examples

Use `readonly` for:

- User ID
- Order ID
- Employee ID

---

# static

A static member belongs to the class itself, not to its objects.

Example:

```ts
class MathUtil {
  static PI = 3.14;
}
```

Usage:

```ts
console.log(MathUtil.PI);
```

No object creation required.

### Why use static?

- Shared values
- Utility methods
- Constants

---

# Inheritance

A child class inherits properties and methods from a parent class.

```ts
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

class Employee extends Person {
  salary: number;

  constructor(name: string, salary: number) {
    super(name);

    this.salary = salary;
  }
}
```

---

# super()

`super()` calls the parent class constructor.

Without it, the child class cannot initialize the parent.

---

# Abstract Class ⭐⭐⭐⭐☆

An abstract class cannot be instantiated directly.

Example:

```ts
abstract class Animal {
  abstract sound(): void;
}
```

Wrong:

```ts
new Animal();
```

Compile Error.

Correct:

```ts
class Dog extends Animal {
  sound() {
    console.log("Bark");
  }
}
```

The child class **must implement** all abstract methods.

---

# Interface vs Abstract Class

| Interface                    | Abstract Class                                   |
| ---------------------------- | ------------------------------------------------ |
| Only defines a contract      | Can define a contract and provide implementation |
| Cannot have constructor      | Can have constructor                             |
| Cannot maintain state        | Can have properties/state                        |
| Supports multiple interfaces | Supports single inheritance                      |

---

## What is an Abstract Class?

An abstract class is a class that cannot be instantiated directly. It serves as a base class and may contain abstract methods that must be implemented by child classes.
