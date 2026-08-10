## What is TypeScript?

TypeScript is a statically typed superset of JavaScript developed by Microsoft that adds optional static typing and compiles into plain JavaScript.

Let's break that down.

```
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

```
TypeScript (.ts)

↓

TypeScript Compiler (tsc)

↓

JavaScript (.js)

↓

Node.js / Browser
```

### Does TypeScript provide any runtime type checking?

No. TypeScript types are completely **erased** during compilation to JavaScript — there is zero runtime overhead and zero runtime type safety from TypeScript alone.

```ts
interface User { name: string; age: number; }

function greet(user: User) {
  console.log(user.name);
}

const data: User = JSON.parse(someApiResponse); // TS trusts you completely here
greet(data); // if the actual JSON doesn't have `name`, this crashes at RUNTIME
```

TypeScript protects you from mistakes *you* make while writing code — it does not protect you from untrusted data crossing a trust boundary (API responses, `JSON.parse`, user input). For that, you need a runtime validator like **Zod** or **Joi** alongside the TS type, so the compile-time type and the actual runtime check stay in sync.

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

```
function calculateSalary(salary: number, bonus: number): number
```

Anyone reading the code immediately knows:

Parameters and Return type

4. Easier Maintenance

Large teams can understand each other's code more easily.

5. Better Refactoring

Renaming a property across thousands of files becomes much safer because the compiler can detect broken references.

**Disadvantages** Extra compilation step
Slight learning curve
More code to write (types)
Build process becomes more complex

## What is Type Inference?

Type Inference means TypeScript automatically determines the type of a variable without you explicitly specifying it.

```
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

```
let user: [number, string];

user = [1, "Harshit"];
```

Correct

```
user = ["Harshit", 1];
```

Wrong. Compiler error.

## What is Enum?

Enums allow us to create a set of named constants.

```
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

### Why do many teams avoid Enums in favor of union literal types?

```ts
enum Role { Admin, User, Manager }
```

vs.

```ts
type Role = "Admin" | "User" | "Manager";
```

Reasons teams prefer union literals:
- Enums generate actual runtime JS code (an object), while union literals are purely compile-time and erased — zero runtime cost.
- Numeric enums allow unsafe implicit number assignment (`Role.Admin` is just `0` under the hood, and any `number` can sometimes sneak through where an enum is expected).
- Union literals work more naturally with discriminated unions and are simpler to reason about.

Enums are still reasonable when you specifically need a runtime-accessible mapping/object, but for most "one of these fixed string values" cases, union literal types are the more common modern default.

## What is Literal Types?

```ts
let status: "Pending" | "Completed" | "Cancelled";
```

Now only these values are allowed.

### What is a Discriminated Union?

A discriminated union is a union of object types that share a common literal property (the "discriminant" or "tag"), which TypeScript uses to narrow the type automatically.

```ts
interface SuccessResponse {
  status: "success";
  data: string;
}

interface ErrorResponse {
  status: "error";
  message: string;
}

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse) {
  if (response.status === "success") {
    console.log(response.data);    // TS knows this is SuccessResponse here
  } else {
    console.log(response.message); // TS knows this is ErrorResponse here
  }
}
```

Checking `response.status` narrows the type inside each branch — TypeScript won't let you access `.data` in the `else` branch, because it knows that branch can only be `ErrorResponse`. This pattern is extremely common for API response types, Redux-style actions, and state machines, and it's a very likely follow-up after any `interface`/`type` question.

## What are the special types?

1. any
2. unknown
3. void
4. never

### What is any?

any disables TypeScript's type checking.

```
let value: any = 10;

value = "Harshit";

value = true;

value = {};

value = [];
```

### What is unknown?

unknown can store any value, but you cannot use it directly until you check its type.

```
let value: unknown = "Harshit";
console.log(value.length);

❌ Error

Because TypeScript doesn't know if it's actually a string.
```

```
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
| ------------------------- | --------------------------- |
| Disables type checking    | Keeps type safety          |
| Can perform any operation | Must narrow the type first |
| Unsafe                    | Safe                       |

### What is void?

This function doesn't return anything.

```
function greet(): void {
  console.log("Hello");
}
```

### What is never?

This function never finishes normally.
It either:

throws an error
or runs forever

```
function throwError(): never {
  throw new Error("Something went wrong");
}
```

### When would a function's return type actually be `never`, in practice?

Two real cases: a function that always throws, and a function with an infinite loop that never returns control. It's also used by TypeScript itself for exhaustiveness checking:

```ts
type Status = "pending" | "completed" | "cancelled";

function handleStatus(status: Status) {
  switch (status) {
    case "pending": return "Waiting";
    case "completed": return "Done";
    case "cancelled": return "Cancelled";
    default:
      const exhaustiveCheck: never = status; // errors if a case is missed
      return exhaustiveCheck;
  }
}
```

If someone adds a new value to the `Status` union later and forgets to handle it in the switch, this pattern causes a compile error at the `default` branch — `never` is used here as a safety net for exhaustive matching.

### What is an Interface?

An interface defines the shape (structure) of an object.

Think of it as a contract.

If an object claims to follow an interface, it must contain all the required properties with the correct types.

```
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

## What is Type Aliases?

A type alias lets you create a new name (alias) for any type.

```
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

*Differences*

1. Interface can be extended

```
interface Person {
  name: string;
}

interface Employee extends Person {
  salary: number;
}
```

Type also supports inheritance using intersections:

```
type Person = {
  name: string;
};

type Employee = Person & {
  salary: number;
};
```

Both achieve similar results.

2. Type can represent more than objects

```
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

```
type Admin = User & Permissions;
```

### What is Structural Typing?

TypeScript uses **structural typing** — it checks compatibility based on the *shape* of a value, not its declared name. This is different from nominal typing (Java/C#), where a type is only compatible if it's explicitly declared as implementing/extending it.

```ts
interface Point { x: number; y: number; }

function printPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}

const obj = { x: 1, y: 2, z: 3 }; // never declared as a Point
printPoint(obj); // works — structurally compatible (has x and y as numbers)
```

If two types have the same shape, they're interchangeable — regardless of name, regardless of whether one was ever explicitly declared to satisfy the other. This is sometimes called "duck typing at compile time," and it's a common thing interviewers probe for specifically, since developers coming from Java/C# often assume TS requires an explicit `implements` relationship. It doesn't.

```ts
type Circle = { radius: number; area(): number; };
interface Shape { area(): number; }

const circle: Circle = { radius: 5, area() { return Math.PI * this.radius ** 2; } };

function printArea(shape: Shape) {
  console.log(shape.area());
}
printArea(circle); // compiles fine — Circle structurally satisfies Shape
```

### Which should you use?

*Use Interface when*:
Modeling objects
Creating contracts
Designing APIs
OOP-style applications

*Use Type when*:
Union types
Intersection types
Function types
Primitive aliases
Tuples
Utility types

## What are Generics?

Generics let you write a function, class, or interface once that works across multiple types while preserving type safety — instead of using `any` (which discards type checking entirely) or duplicating code per type.

```ts
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

const firstNum = getFirst([1, 2, 3]);       // T inferred as number
const firstStr = getFirst(['a', 'b', 'c']); // T inferred as string
```

### What is a Generic Constraint?

`extends` restricts what types `T` can be, when the function needs to rely on some property of `T`.

```ts
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}
getLength('hello');  // works — strings have .length
getLength([1, 2, 3]); // works — arrays have .length
getLength(42);        // ERROR — number has no .length, caught at compile time
```

### What is a Generic Interface, and where would you use one?

```ts
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  save(item: T): Promise<T>;
  delete(id: string): Promise<void>;
}

interface User { id: string; name: string; }

class UserRepository implements Repository<User> {
  async findById(id: string): Promise<User | null> { /* ... */ return null; }
  async save(item: User): Promise<User> { /* ... */ return item; }
  async delete(id: string): Promise<void> { /* ... */ }
}
```

One generic `Repository<T>` interface instead of writing a near-identical interface per entity — a realistic pattern for NestJS/Prisma-style backend work.

## What are Utility Types?

Utility Types are built-in TypeScript types that help us create new types from existing types without rewriting them.

Instead of writing new interfaces again and again, we can transform existing ones.

---

Suppose we have:

```
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}
```

We'll use this interface for all examples.

---

# 1. Partial ⭐⭐⭐⭐⭐

Makes **all properties optional**.

```
type PartialUser = Partial<User>;
```

Now it becomes:

```
{
  id?: number;
  name?: string;
  email?: string;
  age?: number;
}
```

Example:

```
const updateUser: Partial<User> = {
  name: "Harshit",
};
```

Only updating the name.

---

## Real-world Use

**PATCH API**

```
PATCH /users/1
```

The client only sends the fields that need updating.

---

# 2. Required ⭐⭐⭐⭐☆

Makes **all properties required**.

Suppose

```
interface User {
  id: number;
  name?: string;
}
```

Now

```
type RequiredUser = Required<User>;
```

becomes

```
{
  id: number;
  name: string;
}
```

Everything is mandatory.

---

# 3. Readonly ⭐⭐⭐⭐☆

Makes every property read-only.

```
type ReadonlyUser = Readonly<User>;
```

Now

```
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

```
type UserSummary = Pick<User, "id" | "name">;
```

Result:

```
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

```
type UserWithoutAge = Omit<User, "age">;
```

Result:

```
{
  id: number;
  name: string;
  email: string;
}
```

## Real-world Example

Never send passwords.

```
interface User {
  id: number;
  name: string;
  password: string;
}
```

```
type SafeUser = Omit<User, "password">;
```

---

# 6. Record<K, T> ⭐⭐⭐⭐☆

Creates an object with predefined key and value types.

Example:

```
type Scores = Record<string, number>;
```

Now:

```
const scores: Scores = {
  Harshit: 95,
  Rahul: 88,
};
```

Keys → `string`

Values → `number`

## Real-world Example

Caching:

```
Record<string, User>;
```

---

# 7. Exclude<T, U> ⭐⭐⭐⭐☆

Removes specific types from a union.

```
type Status = "Pending" | "Completed" | "Cancelled";
```

Now:

```
type ActiveStatus = Exclude<Status, "Cancelled">;
```

Result:

```
"Pending" | "Completed";
```

---

# 8. Extract<T, U> ⭐⭐⭐⭐☆

Opposite of Exclude.

Keeps only matching types.

```
type Status = "Pending" | "Completed" | "Cancelled";
```

```
type Finished = Extract<Status, "Completed" | "Cancelled">;
```

Result:

```
"Completed" | "Cancelled";
```

---

# Summary

| Utility Type | Purpose                           |
| ------------ | ---------------------------------- |
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

```
let value: string | number;

if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

2. instanceof

Used for classes.

```
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

```
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

```
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

```
Hello Harshit
```

---

# Constructor

A constructor is a special method that runs automatically when an object is created.

```
constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
}
```

When you do:

```
new User(1, "Harshit");
```

the constructor executes automatically.

---

# Access Modifiers

## 1. public

Default access modifier.

Accessible from anywhere.

```
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

```
class User {
  private password: string;

  constructor(password: string) {
    this.password = password;
  }
}
```

Trying to access it outside:

```
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

```
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

```
dog.age;
```

Compile Error.

---

# readonly

A readonly property can only be assigned once.

```
class User {
  readonly id: number;

  constructor(id: number) {
    this.id = id;
  }
}
```

Later:

```
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

```
class MathUtil {
  static PI = 3.14;
}
```

Usage:

```
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

```
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

```
abstract class Animal {
  abstract sound(): void;
}
```

Wrong:

```
new Animal();
```

Compile Error.

Correct:

```
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
| ----------------------------- | -------------------------------------------------- |
| Only defines a contract      | Can define a contract and provide implementation |
| Cannot have constructor      | Can have constructor                             |
| Cannot maintain state        | Can have properties/state                        |
| Supports multiple interfaces | Supports single inheritance                      |

---

## What is an Abstract Class?

An abstract class is a class that cannot be instantiated directly. It serves as a base class and may contain abstract methods that must be implemented by child classes.
