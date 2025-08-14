# Step 1 — What is TypeScript?

TypeScript (TS) is a typed superset of JavaScript that compiles (transpiles) to plain JavaScript.

Key ideas:

* Superset: every JS program is valid TS (though TS may flag type errors).
* Static typing: types are checked before the program runs.
* Erased types: the emitted .js has zero type information; TS affects development time, not runtime.
* Gradual typing: you can adopt types file‑by‑file and even line‑by‑line.

Why this matters:

* Catches bugs early (before runtime).
* Better IDE help (intellisense, refactors).
* Clearer contracts between modules and teams.

Mental model:

* Think of TS as a smart spell‑checker for your JS that knows shapes of objects and functions.

# Step 2 — How TypeScript works in your toolchain

1. You write .ts files.
2. The TS compiler (tsc) type‑checks them.
3. tsc emits .js (ES2017, ES2020, etc.) according to your tsconfig.json.
4. Node runs the emitted JavaScript.
   Optional: ts-node runs TS directly by compiling in memory (convenient in dev).

Important: Since types are erased, TS can’t enforce types at runtime. If you need runtime guarantees (e.g., external input), you add validation (e.g., Zod/Yup).

# Step 3 — Setup done right

Minimal project:

```bash
mkdir ts-basics && cd ts-basics
npm init -y
npm i -D typescript ts-node @types/node
npx tsc --init
```

Recommended tsconfig to start strict and modern:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

Why these flags:

* strict: enables the full suite of safety checks.
* esModuleInterop: smooth import compatibility.
* skipLibCheck: faster builds while you learn.

# Step 4 — First program, with concepts attached

hello.ts

```ts
let message: string = "Hello, TypeScript";
console.log(message);
```

Run:

```bash
npx ts-node hello.ts
```

Concepts demonstrated:

* Type annotation (: string).
* Emitted JS will just log the string; types don’t exist at runtime.

# Step 5 — Basic types and when to use them

Primitives:

```ts
let username: string = "Rahman";
let age: number = 25;
let isAdmin: boolean = true;
let nothingHere: null = null;
let notAssigned: undefined = undefined;
let big: bigint = 9007199254740991n;
let unique: symbol = Symbol("id");
```

Arrays and tuples:

```ts
let scores: number[] = [85, 92, 77];
let pair: [string, number] = ["points", 10]; // fixed length and positions
```

Special types you must master:

```ts
let anything: any = 42;           // disables type checking for this value (avoid)
let mystery: unknown = "maybe";   // must narrow before using (prefer over any)

// unknown requires checks
if (typeof mystery === "string") {
  console.log(mystery.toUpperCase());
}

function fail(msg: string): never {
  throw new Error(msg);           // never = function doesn’t return
}

function logNothing(): void {
  console.log("done");            // void = no useful return value
}
```

Memory tip (A‑U‑N):

* any = skip safety.
* unknown = safety with a gate (narrow first).
* never = impossible or no return path.


Alright, let’s dive into **Lesson 1: Basic Types & Inference**.

---

### 1. Core Types in TypeScript

These are the building blocks:

* `string`
* `number`
* `boolean`
* `null`
* `undefined`
* `symbol`
* `bigint`

Special types:

* `any` → Turns off type checking (avoid unless necessary).
* `unknown` → Similar to `any`, but forces you to check before using.
* `never` → A type that never happens (like a function that always throws).
* `void` → For functions that don’t return a value.

---

### 2. Type Inference

TypeScript can *guess* the type from the value:

```ts
let age = 30;      // inferred: number
const pi = 3.14;   // inferred: 3.14 (literal type)
```

If a variable starts with a value, you usually **don’t need to explicitly type it**.

---

### 3. Unions

Allow multiple possible types:

```ts
let id: string | number;
id = "abc";
id = 123;
```

---

### 4. `unknown` vs `any`

```ts
function parse(input: string): unknown {
  return JSON.parse(input);
}

const result = parse('{"x":1}');

// Must check before using:
if (result && typeof result === "object") {
  console.log("Safe to access properties now");
}
```

If you used `any` here, TypeScript wouldn’t force you to check, which is unsafe.

---

### 5. `never`

A function that never finishes:

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

---

#### Mini Exercises

1. Declare a variable that can be either a string ID or a number ID.
2. Write a function to square a number and let TS infer the return type.
3. Use `unknown` for data from `JSON.parse` and narrow it before using.

---

The key difference between `unknown` and `any` in TypeScript is **how much type safety they give you**.

---

### `any`

* Disables type checking for that value.
* You can do **anything** with it without errors — even if it doesn’t make sense.
* Useful as a last resort, but risky because TypeScript stops protecting you.

Example:

```ts
let value: any = "hello";
value.foo.bar();  // ✅ No error at compile time, but crashes at runtime!
```

---

### `unknown`

* Also allows **any value**, but **forces you to check** the type before using it.
* Safer because you can’t accidentally call methods or access properties without narrowing.

Example:

```ts
let value: unknown = "hello";
value.toUpperCase(); // ❌ Error: Object is of type 'unknown'

if (typeof value === "string") {
  value.toUpperCase(); // ✅ Works after narrowing
}
```

---

### Simple Rule to Remember

> **`unknown` = I don’t know yet, but I’ll check.**
> **`any` = I don’t care, do what you want.**

---

Alright, let’s move on to **Lesson 2: Functions and Typing Functions**.

---

### 1. Typing Function Parameters and Returns

You explicitly tell TS the types of parameters and what the function returns.

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

If the return type is obvious, TS can **infer** it:

```ts
function square(n: number) {
  return n * n; // inferred return type: number
}
```

---

### 2. Optional and Default Parameters

Optional parameters use `?`.
Default parameters don’t require `?` because they’re always defined.

```ts
function greet(name: string, title?: string) {
  return title ? `${title} ${name}` : name;
}

function log(msg: string, level: "info" | "warn" = "info") {
  console.log(`[${level}] ${msg}`);
}
```

---

### 3. Function Types (as Variables)

You can store functions in variables with explicit types.

```ts
type MathFn = (a: number, b: number) => number;
const multiply: MathFn = (x, y) => x * y;
```

---

### 4. Generic Functions (Intro)

Generics let your functions work with *any type*, while keeping type safety.

```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const num = first([1, 2, 3]); // T = number
const str = first(["a", "b", "c"]); // T = string
```

---

### 5. Function Overloads (Intro)

When you want multiple input/output type combinations.

```ts
function len(x: string): number;
function len(x: any[]): number;
function len(x: string | any[]) {
  return x.length;
}

len("hello"); // 5
len([1, 2, 3]); // 3
```

---

#### Mini Exercises

1. Write a `max` function that returns the bigger of two numbers.
2. Create a type alias `Fn2` for a function taking two numbers and returning a number, then use it to define an `add` function.
3. Write a generic `wrap` function that takes a value and returns `{ value }` with the right type.

---
When TypeScript says the return type is **obvious**, it means the compiler can **look at the return statement and figure out the type without you writing it**.

---

### How TypeScript Figures It Out

When you return a value, TypeScript checks:

1. What is the type of that value?
2. Does that same type (or a union of possible types) happen in every return path?

If yes, it automatically **infers** the return type.

---

### Examples

**Obvious return type:**

```ts
function square(n: number) {
  return n * n; 
  // `n` is a number, n*n is a number → inferred return type is number
}
```

**Union return type:**

```ts
function maybeNumber(flag: boolean) {
  if (flag) return 42; // number
  else return null;    // null
}
// inferred return type: number | null
```

**Void return type:**

```ts
function logMessage(msg: string) {
  console.log(msg);
  // no return value → inferred return type is void
}
```

---

### When You Still Want to Specify the Return Type

Even if TS can guess, you might **explicitly** set it:

* For clarity in an API or shared codebase.
* To catch mistakes if you accidentally return the wrong type later.
* To enforce a broader or narrower type than TS would infer.

Example:

```ts
function getId(): string {
  return "123"; // if you accidentally return a number later, TS will complain
}
```

---

Alright — let’s slow down and go deeper into **objects, interfaces, and type aliases** so you understand *why* and *when* to use each one.

---

## 1. Typing Objects — the Shape Concept

In TypeScript, an object’s **type** is basically its **shape**: the property names and the types of their values.
TypeScript uses **structural typing**, meaning it doesn’t care about names of types — if the shape matches, it’s compatible.

Example:

```ts
const user: { id: string; name: string } = {
  id: "u1",
  name: "Rahman",
};
```

Here:

* The type `{ id: string; name: string }` means:

  * Must have an `id` property that is a string
  * Must have a `name` property that is a string
* Any extra properties are allowed **only if** the compiler’s strict object checks are off (or if the type allows them via index signatures).

If you initialize the object right away, TypeScript can **infer** its type:

```ts
const product = { id: "p1", price: 100 };
// product is automatically typed as { id: string; price: number }
```

---

## 2. Interfaces — Named Object Shapes

An **interface** is a named way to describe the shape of an object.

Example:

```ts
interface User {
  id: string;
  name: string;
  email?: string; // optional property
}

const person: User = {
  id: "123",
  name: "Rahman",
};
```

Key points:

* `email?: string` → the `?` makes it optional.
* If you try to assign a number to `name`, TS will give an error.
* Interfaces can be **extended** (inherited):

```ts
interface Admin extends User {
  role: string;
}
```

Now `Admin` has all properties of `User` plus `role`.

---

## 3. Type Aliases — Flexible Naming

A **type** alias can name *any* type, not just object shapes:

```ts
type Product = {
  id: string;
  name: string;
  price: number;
};

type ID = string | number; // union type
```

You can use `type` for:

* Objects
* Unions
* Intersections
* Primitive aliases
* Complex mapped types

---

## 4. Readonly and Optional Properties

You can make properties **immutable** with `readonly`:

```ts
interface Config {
  readonly baseUrl: string;
  timeout?: number; // optional
}
```

This means:

* `baseUrl` cannot be reassigned after initialization.
* `timeout` is optional — if you leave it out, no error.

---

## 5. Index Signatures — Dynamic Keys

Use when you don’t know all property names ahead of time:

```ts
interface Dictionary<T> {
  [key: string]: T;
}

const colors: Dictionary<string> = {
  red: "#ff0000",
  blue: "#0000ff",
};
```

Here, any string key must map to a string value.

---

## 6. Interface vs Type — When to Use

**Interfaces:**

* Best for describing object shapes.
* Can be extended with `extends`.
* Can merge automatically if declared multiple times.

**Types:**

* Can describe **anything** (objects, unions, primitives, etc.).
* Can’t merge automatically.
* Better for union/intersection-heavy types.

---

### Mini Exercise — With Solutions

1. **Address Interface**

```ts
interface Address {
  street: string;
  city: string;
  zip?: string; // optional
  country: string;
}
```

2. **Config Interface**

```ts
interface Config {
  readonly baseUrl: string;
  timeout?: number;
}
```

3. **Result Type**

```ts
type Result =
  | { success: true; data: string }
  | { success: false; error: string };
```

---

Alright — let’s go into a **full, detailed comparison** of `type` vs `interface` in TypeScript, with examples, edge cases, and when to choose one over the other.

---

## 1. Core Difference

Both `type` and `interface` can describe the **shape of an object**, but:

| Feature                                      | `interface`                         | `type`                 |
| -------------------------------------------- | ----------------------------------- | ---------------------- |
| Can describe object shapes                   | ✅                                   | ✅                      |
| Can describe primitives/unions/intersections | ❌                                   | ✅                      |
| Can be **extended**                          | ✅ (`extends`)                       | ✅ (via intersections)  |
| Declaration merging                          | ✅ (interfaces with same name merge) | ❌ (types cannot merge) |
| Computed property types, mapped types        | ❌                                   | ✅                      |
| Aliasing any type (primitive, tuple, union)  | ❌                                   | ✅                      |

---

## 2. When They Look the Same

For a simple object:

```ts
interface User {
  id: string;
  name: string;
}

type UserType = {
  id: string;
  name: string;
};
```

Both are interchangeable here — both say: "An object with `id` string and `name` string."

---

## 3. Extension — How They Differ

### Interface extension:

```ts
interface User {
  id: string;
  name: string;
}

interface Admin extends User {
  role: string;
}

const admin: Admin = {
  id: "1",
  name: "Rahman",
  role: "superadmin",
};
```

### Type extension (intersection):

```ts
type User = {
  id: string;
  name: string;
};

type Admin = User & { role: string };

const admin: Admin = {
  id: "1",
  name: "Rahman",
  role: "superadmin",
};
```

Functionally the same, but different syntax.

---

## 4. Declaration Merging — Interface-Only Feature

You can declare the **same interface name twice**, and TS will merge them.

```ts
interface User {
  id: string;
}
interface User {
  name: string;
}

const u: User = { id: "1", name: "Rahman" }; // merged automatically
```

With `type`, this is an error:

```ts
type User = { id: string };
type User = { name: string }; // ❌ Cannot redeclare
```

---

## 5. Non-Object Types — Type-Only Feature

`type` can alias **anything**, not just objects:

```ts
type ID = string | number;
type Coordinates = [number, number]; // tuple
type Status = "pending" | "success" | "error";
```

`interface` cannot do this — it only works for objects/classes.

---

## 6. Computed and Mapped Types — Type-Only Feature

```ts
type Keys = "id" | "name";
type User = {
  [K in Keys]: string;
};
```

You can’t do this directly with an `interface`.

---

## 7. Which Should You Use?

### Use **interface** when:

* You’re modeling the shape of objects/classes.
* You want declaration merging.
* You expect the type to be extended in many places (like public API contracts).

### Use **type** when:

* You need unions, intersections, or primitives.
* You’re building complex mapped types.
* You want to represent things other than object shapes.

---

## 8. Real-World Example — Combining Them

```ts
type ID = string | number;

interface User {
  id: ID;          // using a type alias inside an interface
  name: string;
}

interface Admin extends User {
  permissions: string[];
}

type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: string };
```

Here:

* `ID` is a **type alias** because it’s a union of primitives.
* `User` and `Admin` are **interfaces** because they describe object shapes.
* `ApiResponse<T>` is a **generic type alias** because it’s a union type.

---

Got it — from now on, I’ll go into **full depth with step-by-step reasoning, examples, and use cases** for every lesson.
Let’s redo **Lesson 4: Unions, Intersections, and Literal Types** with maximum detail.

---

## 1. Union Types ( `|` )

A **union type** says:

> “This value can be *either* one type or another.”

It’s like an **OR** in logic.

### Example

```ts
type Id = string | number;

let userId: Id;
userId = "abc"; // ✅ allowed (string is part of the union)
userId = 123;   // ✅ allowed (number is part of the union)
userId = true;  // ❌ not allowed (boolean not part of the union)
```

How TypeScript uses it:

* If you try to use `userId` as a string method (`userId.toUpperCase()`), TS will stop you unless you **narrow** it first.
* Narrowing is telling TS: “Right now, I know it’s a string.”

Example with narrowing:

```ts
if (typeof userId === "string") {
  console.log(userId.toUpperCase()); // ✅ safe
}
```

**Real-world use case:**
API IDs that can be either a number (from DB) or a string (from UUID).

---

## 2. Literal Types

A **literal type** is a type whose value is exactly one thing — not just any string or number, but that **specific** value.

### Example

```ts
let direction: "up" | "down" | "left" | "right";

direction = "up";    // ✅
direction = "down";  // ✅
direction = "north"; // ❌ Error: not in the literal union
```

Why this is powerful:

* Prevents typos (no `"lef"` by mistake).
* Makes functions safer by restricting accepted values.

Example with function:

```ts
function move(dir: "up" | "down" | "left" | "right") {
  console.log(`Moving ${dir}`);
}

move("up");    // ✅
move("north"); // ❌
```

---

## 3. Intersection Types ( `&` )

An **intersection type** says:

> “This value must have **all** properties from both types.”

It’s like an **AND** in logic.

### Example

```ts
interface Timestamps {
  createdAt: Date;
  updatedAt: Date;
}

interface Post {
  id: string;
  title: string;
}

type DatedPost = Post & Timestamps;

const blogPost: DatedPost = {
  id: "1",
  title: "Learning TypeScript",
  createdAt: new Date(),
  updatedAt: new Date(),
};
```

Here:

* `DatedPost` must have **everything** from `Post` **and** `Timestamps`.

**Real-world use case:**
Combining a `User` type with an `AuditInfo` type to ensure every user has audit fields.

---

## 4. Exhaustiveness Checking with `never`

When working with unions of **literal types**, you can make TS tell you if you’ve missed a case in a `switch`.

### Example

```ts
type Direction = "up" | "down" | "left" | "right";

function handleDirection(dir: Direction) {
  switch (dir) {
    case "up":
    case "down":
    case "left":
    case "right":
      break;
    default:
      const _exhaustive: never = dir; 
      // ❌ Error if a new direction is added but not handled
      return _exhaustive;
  }
}
```

If you later add `"forward"` to `Direction`, TS will immediately tell you to update the `switch`.

---

## 5. Combining Unions and Intersections

You can **mix** them for powerful type combinations.

Example:

```ts
type ErrorResponse = { status: "error"; message: string };
type SuccessResponse = { status: "success"; data: string };

type ApiResponse = (ErrorResponse | SuccessResponse) & { requestId: string };

const res1: ApiResponse = {
  status: "success",
  data: "User created",
  requestId: "abc123",
};

const res2: ApiResponse = {
  status: "error",
  message: "Invalid request",
  requestId: "abc124",
};
```

Here:

* Base type is `ErrorResponse | SuccessResponse` (**either** success or error).
* Every variant must also have a `requestId` (**and** condition from intersection).

---

## Mini Exercises — Step-by-Step

**1. Status Literal Type**

```ts
type Status = "pending" | "success" | "error";
```

* Only allows these three exact strings.

---

**2. UserWithAddress Intersection**

```ts
interface User {
  id: string;
  name: string;
}

interface Address {
  street: string;
  city: string;
  country: string;
}

type UserWithAddress = User & Address;

const person: UserWithAddress = {
  id: "u1",
  name: "Rahman",
  street: "Main St",
  city: "Melbourne",
  country: "Australia",
};
```

---

**3. Exhaustive processStatus**

```ts
function processStatus(status: Status) {
  switch (status) {
    case "pending":
      console.log("Loading...");
      break;
    case "success":
      console.log("Operation succeeded!");
      break;
    case "error":
      console.log("Something went wrong.");
      break;
    default:
      const neverValue: never = status; // compiler enforces all cases
      return neverValue;
  }
}
```

---
Alright, let’s go deep into **Lesson 5: Type Narrowing & Control Flow Analysis** — this is where TypeScript starts feeling *smart*, because it figures out your variable’s type based on logic in your code.

---

## 1. What is Narrowing?

**Narrowing** is when TypeScript takes a broad type and “narrows” it down to a more specific type based on checks you write.

Example without narrowing:

```ts
function printId(id: string | number) {
  console.log(id.toUpperCase()); // ❌ Error: id might be number
}
```

Example with narrowing:

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // ✅ id is now string
  } else {
    console.log(id.toFixed(2)); // ✅ id is now number
  }
}
```

---

## 2. Type Guards

A **type guard** is any check that changes the type inside its block.

### Common Guards

1. **`typeof`**

```ts
if (typeof value === "string") { /* value is string */ }
```

2. **`instanceof`**

```ts
if (error instanceof Error) { /* error is Error */ }
```

3. **`in` operator**

```ts
if ("role" in user) { /* user has role property */ }
```

4. **Equality checks**

```ts
if (status === "success") { /* status is literal "success" */ }
```

---

## 3. User-Defined Type Predicates

You can teach TS your own type guard.

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function isFish(animal: Fish | Bird): animal is Fish {
  return (animal as Fish).swim !== undefined;
}

function move(animal: Fish | Bird) {
  if (isFish(animal)) {
    animal.swim(); // ✅ now narrowed to Fish
  } else {
    animal.fly();  // ✅ now narrowed to Bird
  }
}
```

`animal is Fish` tells TS: *if this function returns true, the variable is a Fish inside that block.*

---

## 4. Control Flow Analysis

TypeScript tracks your variable’s type **through your code’s logic flow**.

Example:

```ts
function example(input: string | undefined) {
  if (!input) return; 
  // after this return, input is guaranteed to be string

  console.log(input.toUpperCase()); // ✅ safe
}
```

TS understands that after the `return`, `input` can’t be `undefined` anymore.

---

## 5. Discriminated Unions

Pattern: give each union variant a **literal property** that identifies it.

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number };

function area(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    default:
      const neverValue: never = shape; // exhaustiveness check
      return neverValue;
  }
}
```

This makes narrowing super easy and safe.

---

## 6. Real-World Example: API Responses

```ts
type ApiResponse =
  | { status: "success"; data: string }
  | { status: "error"; error: string };

function handleResponse(res: ApiResponse) {
  if (res.status === "success") {
    console.log(res.data); // ✅ TS knows it's the success type
  } else {
    console.error(res.error); // ✅ TS knows it's the error type
  }
}
```

---
Alright, moving on to **Lesson 6: Arrays, Tuples, and Readonly** — we’ll go into full depth so you understand not just the syntax, but *why* and *when* to use each.

---

## 1. Arrays in TypeScript

An array type is written in two ways:

```ts
let numbers: number[] = [1, 2, 3];       // preferred
let strings: Array<string> = ["a", "b"]; // generic form
```

### Notes:

* `number[]` is shorthand for `Array<number>`.
* Arrays are **homogeneous** by default — every element must match the specified type.
* For mixed arrays, use a **union**:

```ts
let mixed: (string | number)[] = [1, "a", 2];
```

---

## 2. Readonly Arrays

A `readonly` array prevents mutation methods:

```ts
const nums: readonly number[] = [1, 2, 3];

nums.push(4); // ❌ Error
nums[0] = 99; // ❌ Error
```

You can also use `ReadonlyArray<T>`:

```ts
const names: ReadonlyArray<string> = ["Rahman", "Ali"];
```

**When to use:**

* When you want to guarantee an array is immutable (e.g., function arguments that shouldn’t change).

---

## 3. Tuples — Fixed Length, Ordered Types

A tuple is like an array but with:

* **Fixed length**
* **Types in a specific order**

Example:

```ts
let point: [number, number] = [10, 20];
point = [15, 25];     // ✅ correct
point = [15, "25"];   // ❌ wrong: second must be number
```

---

### Named Tuples (for clarity)

```ts
type Coordinate = [x: number, y: number];
const home: Coordinate = [5, 10];
```

---

## 4. Optional and Rest Elements in Tuples

```ts
type RGB = [number, number, number?];
let color: RGB = [255, 0];       // ✅ last is optional
let color2: RGB = [255, 0, 255]; // ✅ full length
```

Rest elements:

```ts
type StringNumberPair = [string, ...number[]];
const example: StringNumberPair = ["Rahman", 1, 2, 3];
```

---

## 5. Readonly Tuples

```ts
const pair: readonly [string, number] = ["age", 30];
pair[1] = 40; // ❌ Error: cannot modify
```

---

## 6. Real-World Uses

* **Arrays** → lists of data, API responses, DB rows.
* **Tuples** → coordinates, key-value pairs, color codes.
* **Readonly** → safe constants, immutable config values.

---
Alright, let’s go into **Lesson 7: Generics — The Heart of Reusability** in full detail.

---

## 1. What Are Generics?

Generics let you create **reusable** code that works with **any type**, but still keeps **type safety**.

Without generics:

```ts
function wrap(value: any) {
  return { value };
}

const result = wrap(42);
result.value.toUpperCase(); // ❌ runtime error — no type safety
```

With generics:

```ts
function wrap<T>(value: T) {
  return { value };
}

const num = wrap(42);      // T = number
const str = wrap("hello"); // T = string
str.value.toUpperCase();   // ✅ safe
```

---

## 2. Syntax

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

* `<T>` → type parameter (name it anything, but `T` is common).
* `T` is inferred from the argument, so you don’t always have to specify it manually.

Explicit type parameter:

```ts
const result = identity<string>("Rahman");
```

---

## 3. Generic Functions in Action

Example: **Getting the first element**

```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

first([1, 2, 3]);     // T = number
first(["a", "b", "c"]); // T = string
```

---

## 4. Generic Constraints

Sometimes you want “any type, but it must have certain properties.”

Example:

```ts
function lengthOf<T extends { length: number }>(item: T): number {
  return item.length;
}

lengthOf("hello"); // ✅ string has length
lengthOf([1, 2, 3]); // ✅ array has length
lengthOf(123); // ❌ number has no length
```

---

## 5. `keyof` with Generics

`keyof` lets you get all property names of a type:

```ts
function pluck<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map(k => obj[k]);
}

const person = { name: "Rahman", age: 30 };
pluck(person, ["name"]); // returns string[]
```

---

## 6. Default Generic Types

You can set defaults:

```ts
interface ApiResponse<T = any> {
  data: T;
  status: number;
}

const res: ApiResponse<string> = { data: "ok", status: 200 };
const res2: ApiResponse = { data: { id: 1 }, status: 200 }; // T = any
```

---

## 7. Generics in Interfaces and Types

```ts
interface Box<T> {
  value: T;
}

const stringBox: Box<string> = { value: "hello" };
const numberBox: Box<number> = { value: 42 };
```

---

## 8. Generics in Classes

```ts
class Storage<T> {
  private items: T[] = [];
  add(item: T) {
    this.items.push(item);
  }
  getAll(): T[] {
    return this.items;
  }
}

const numberStore = new Storage<number>();
numberStore.add(1);
numberStore.add(2);

const stringStore = new Storage<string>();
stringStore.add("a");
```

---

## 9. Real-World Uses

* **Reusable functions** (like `map`, `filter`, `reduce`)
* **Reusable components** in React (e.g., `<Select<T>>`)
* **API response wrappers**
* **Data structures** (Stacks, Queues, Trees)

---
Alright, let’s go deep into **Lesson 8: Utility Types & Mapped Types** — this is where TypeScript really starts feeling powerful because you can **transform existing types** without rewriting them.

---

## 1. What Are Utility Types?

Utility types are **built-in TypeScript helpers** that take an existing type and create a modified version of it.
They save you from repeating type definitions and keep your code DRY (Don’t Repeat Yourself).

---

## 2. Common Built-in Utility Types

### `Partial<T>`

Makes all properties optional.

```ts
interface User {
  id: string;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// equivalent to:
// { id?: string; name?: string; email?: string }

const u1: PartialUser = { name: "Rahman" }; // ✅ only name
```

---

### `Required<T>`

Makes all properties required.

```ts
type RequiredUser = Required<PartialUser>;
// { id: string; name: string; email: string }
```

---

### `Readonly<T>`

Makes all properties readonly (immutable).

```ts
type ReadonlyUser = Readonly<User>;

const u2: ReadonlyUser = { id: "1", name: "Rahman", email: "r@example.com" };
u2.name = "Ali"; // ❌ Error
```

---

### `Pick<T, K>`

Selects specific properties from a type.

```ts
type UserPreview = Pick<User, "id" | "name">;
// { id: string; name: string }
```

---

### `Omit<T, K>`

Removes specific properties from a type.

```ts
type UserWithoutEmail = Omit<User, "email">;
// { id: string; name: string }
```

---

### `Record<K, T>`

Creates an object type with keys of `K` and values of `T`.

```ts
type Role = "admin" | "user" | "guest";
type RolePermissions = Record<Role, string[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"],
};
```

---

### `ReturnType<T>`

Gets the return type of a function.

```ts
function getUser() {
  return { id: "1", name: "Rahman" };
}

type UserReturn = ReturnType<typeof getUser>;
// { id: string; name: string }
```

---

### `Parameters<T>`

Gets the parameter types of a function as a tuple.

```ts
type UserParams = Parameters<typeof getUserName>;
```

---

## 3. Mapped Types

Mapped types allow you to create new types by **looping** over the keys of another type.

Example:

```ts
type Optional<T> = {
  [K in keyof T]?: T[K];
};

type OptionalUser = Optional<User>;
```

* `keyof T` → gets all property keys of `T`.
* `K in keyof T` → loops through those keys.
* `T[K]` → gets the property type for each key.

---

## 4. Modifiers in Mapped Types

* `readonly` → makes properties immutable.
* `?` → makes properties optional.
* `-readonly` → removes readonly.
* `-?` → removes optional.

Example:

```ts
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};
```

---

## 5. Real-World Use Case

Imagine you have:

```ts
interface Product {
  id: string;
  name: string;
  price: number;
}
```

You can quickly make:

* `Partial<Product>` → for update API
* `Pick<Product, "id" | "name">` → for dropdown lists
* `Readonly<Product>` → for immutable constants

---


Alright, let’s go into **Lesson 9: Modules & Project Structure** — this is where we make your TypeScript project organized, scalable, and easy to navigate.

---

## 1. Why Modules Matter in TypeScript

* A **module** is just a file that exports something.
* Modules help split large codebases into smaller, reusable pieces.
* TypeScript uses ES Modules (`import/export`) or CommonJS (`require/module.exports`) depending on your `tsconfig.json`.

---

## 2. Creating a Module

Example `math.ts`:

```ts
export function add(a: number, b: number): number {
  return a + b;
}

export const PI = 3.14;
```

Example `main.ts`:

```ts
import { add, PI } from "./math";

console.log(add(2, 3)); // 5
console.log(PI);        // 3.14
```

---

## 3. Default Exports

* A module can have **one** default export.
* Import default exports without curly braces.

`math.ts`:

```ts
export default function add(a: number, b: number): number {
  return a + b;
}
```

`main.ts`:

```ts
import add from "./math";
console.log(add(2, 3));
```

---

## 4. Project Structure Best Practices

Example:

```
src/
 ├── models/        // Types, interfaces, data models
 ├── services/      // Business logic, API calls
 ├── controllers/   // Request handling (for backend)
 ├── utils/         // Helpers, utility functions
 ├── index.ts       // Entry point
 └── tsconfig.json
```

---

## 5. Using `tsconfig` Paths for Clean Imports

Without paths:

```ts
import { User } from "../../models/user";
```

With paths in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@models/*": ["models/*"],
      "@utils/*": ["utils/*"]
    }
  }
}
```

Then:

```ts
import { User } from "@models/user";
```

---

## 6. Splitting Types Into Separate Files

`models/user.ts`:

```ts
export interface User {
  id: string;
  name: string;
}
```

`models/index.ts`:

```ts
export * from "./user"; // re-export everything
```

`main.ts`:

```ts
import { User } from "@models";
```

---

## 7. Barrel Files (Index Files)

A barrel file is an `index.ts` that re-exports from multiple files.

* Keeps imports clean.
* Reduces the number of paths to remember.

---

## 8. Real-World Example — Node.js + TypeScript

* `models/` for type definitions.
* `controllers/` for request handling.
* `services/` for business logic.
* `utils/` for shared helpers.
* Use `@aliases` to avoid long `../../` paths.

---

Alright — let’s deep-dive into **tsconfig.json** so you understand exactly how it works, what each important option does, and how to configure it for different projects.

---

## 1. What is `tsconfig.json`?

* A **configuration file** that tells the TypeScript compiler (`tsc`) how to compile your code.
* It controls:

  * Which files to include/exclude
  * Which JavaScript version to compile to
  * How strict the type checking should be
  * Module system, paths, and output locations

When you run:

```bash
npx tsc
```

TypeScript looks for a `tsconfig.json` in the current or parent directory and uses its settings.

---

## 2. Basic Structure

```json
{
  "compilerOptions": { ... },
  "include": ["src"],
  "exclude": ["node_modules", "dist"],
  "files": ["src/index.ts"]
}
```

* **`compilerOptions`** → main settings for how TS behaves.
* **`include`** → which files/directories to compile.
* **`exclude`** → which files/directories to skip.
* **`files`** → explicitly list files to compile (rarely used, usually use `include` instead).

---

## 3. Key `compilerOptions`

### A. Output Settings

| Option    | What it does                                              |
| --------- | --------------------------------------------------------- |
| `target`  | JS version output (e.g., `"ES5"`, `"ES2017"`, `"ES2022"`) |
| `module`  | Module system (e.g., `"CommonJS"`, `"ESNext"`)            |
| `outDir`  | Output folder for compiled JS                             |
| `rootDir` | Base folder for input TS files                            |

Example:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "outDir": "dist",
    "rootDir": "src"
  }
}
```

---

### B. Strictness Settings

Turning on `"strict": true` enables **all** strict checks.
You can also control them individually:

* `noImplicitAny` → Error if TS infers `any`.
* `strictNullChecks` → Disallows `null`/`undefined` unless explicitly allowed.
* `exactOptionalPropertyTypes` → Optional properties cannot be assigned `undefined` unless declared.
* `noUncheckedIndexedAccess` → Indexing into an array/object might return `undefined`.

Example:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

---

### C. Module Resolution & Path Aliases

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@models/*": ["models/*"],
      "@utils/*": ["utils/*"]
    }
  }
}
```

* `baseUrl` → base directory for non-relative imports.
* `paths` → create import shortcuts.

---

### D. Interop & Compatibility

* `esModuleInterop`: Makes `import express from "express"` work with CommonJS modules.
* `allowSyntheticDefaultImports`: Allows default imports for modules without a default export.
* `resolveJsonModule`: Lets you `import` JSON files.

Example:

```json
{
  "compilerOptions": {
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true
  }
}
```

---

### E. Source Maps (For Debugging)

```json
{
  "compilerOptions": {
    "sourceMap": true
  }
}
```

Generates `.map` files so you can debug the original TypeScript code in browsers or Node.js debuggers.

---

### F. Declaration Files

```json
{
  "compilerOptions": {
    "declaration": true,
    "declarationDir": "types"
  }
}
```

* Creates `.d.ts` type definition files for libraries/packages.

---

## 4. Example: Strict Node.js Project Config

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": "./src",
    "paths": {
      "@models/*": ["models/*"],
      "@utils/*": ["utils/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 5. Tips for Working with `tsconfig.json`

* Start with `"strict": true` — catch problems early.
* Use `"skipLibCheck": true` to speed up builds if you don’t care about checking `node_modules`.
* Always set `outDir` and `rootDir` — keeps compiled JS separate.
* Use `paths` + `baseUrl` to avoid ugly `../../` imports.
* For libraries, enable `"declaration": true` so others get typings.

---

If you want, next I can make you a **cheat sheet** for all important `tsconfig` options grouped by category so you can quickly set up configs for Node, React, and library projects without guessing.

Do you want me to prepare that?


Alright — here’s your **TypeScript `tsconfig.json` Cheat Sheet** grouped by category so you can quickly set up for different kinds of projects.

---

## 1. Output & Compilation

| Option           | Description                          | Common Values                               |
| ---------------- | ------------------------------------ | ------------------------------------------- |
| `target`         | JavaScript version to compile to     | `"ES5"`, `"ES2017"`, `"ES2022"`, `"ESNext"` |
| `module`         | Module system for output JS          | `"CommonJS"` (Node), `"ESNext"` (ESM)       |
| `outDir`         | Folder for compiled JS               | `"dist"`                                    |
| `rootDir`        | Folder for source TS                 | `"src"`                                     |
| `sourceMap`      | Generates `.map` files for debugging | `true`                                      |
| `declaration`    | Emit `.d.ts` type files              | `true` (library projects)                   |
| `declarationDir` | Where to put `.d.ts` files           | `"types"`                                   |

---

## 2. Strictness & Type Safety

| Option                             | Description                                       | Default |
| ---------------------------------- | ------------------------------------------------- | ------- |
| `strict`                           | Enables **all** strict checks                     | `false` |
| `noImplicitAny`                    | Error if type is inferred as `any`                | `false` |
| `strictNullChecks`                 | `null`/`undefined` must be explicitly allowed     | `false` |
| `exactOptionalPropertyTypes`       | Optional props not auto-assignable to `undefined` | `false` |
| `noUncheckedIndexedAccess`         | Index access might return `undefined`             | `false` |
| `forceConsistentCasingInFileNames` | Prevent case mismatches in imports                | `false` |

---

## 3. Module Resolution & Imports

| Option                         | Description                          | Example                     |
| ------------------------------ | ------------------------------------ | --------------------------- |
| `baseUrl`                      | Base folder for non-relative imports | `"./src"`                   |
| `paths`                        | Create import aliases                | `"@models/*": ["models/*"]` |
| `moduleResolution`             | How TS finds modules                 | `"node"` (default)          |
| `esModuleInterop`              | Allows default import from CommonJS  | `true`                      |
| `allowSyntheticDefaultImports` | Same as above for type-only          | `true`                      |
| `resolveJsonModule`            | Import `.json` files                 | `true`                      |

---

## 4. Performance & Build Speed

| Option            | Description                             | Default            |
| ----------------- | --------------------------------------- | ------------------ |
| `skipLibCheck`    | Skip checking `node_modules` type files | `false`            |
| `incremental`     | Reuse build info for faster rebuilds    | `false`            |
| `tsBuildInfoFile` | Location for incremental build info     | `"./.tsbuildinfo"` |

---

## 5. File Inclusion & Exclusion

| Field     | Description              | Example                    |
| --------- | ------------------------ | -------------------------- |
| `include` | Files/folders to compile | `["src"]`                  |
| `exclude` | Skip these               | `["node_modules", "dist"]` |
| `files`   | Exact files to compile   | `["src/index.ts"]`         |

---

## 6. Recommended Templates

### A. **Node.js Project**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": "./src",
    "paths": {
      "@models/*": ["models/*"],
      "@utils/*": ["utils/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

---

### B. **React Project**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["DOM", "DOM.Iterable", "ES2020"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

---

### C. **Library / NPM Package**

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "ESNext",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "declaration": true,
    "declarationDir": "types",
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```