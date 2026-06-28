# Lesson 6 — Q&A Reference
## Functions in TypeScript

---

## Conceptual Questions

---

### Q1. How do you type a function's parameters and return value?

```typescript
// Syntax
function name(param1: type1, param2: type2): returnType {
  return value;
}

// Example
function add(a: number, b: number): number {
  return a + b;
}
```

TypeScript enforces:
- Every argument must match the declared type
- The exact number of arguments must be provided
- The return value must match the declared return type

---

### Q2. Why should you always annotate return types explicitly?

TypeScript can infer return types — but explicit annotation is best practice because:

- Makes the function contract readable at a glance
- Catches bugs where you accidentally return the wrong type
- Serves as documentation for other developers
- Prevents silent return type widening as code changes

```typescript
// ❌ Inferred — TypeScript figures it out, but not explicit
function multiply(a: number, b: number) {
  return a * b;
}

// ✅ Explicit — preferred in production
function multiply(a: number, b: number): number {
  return a * b;
}

// TypeScript catches wrong return types when explicit
function getUsername(id: number): string {
  return id; // ❌ Error — number is not assignable to string
}
```

---

### Q3. What is the difference between `void` and `never`?

| | `void` | `never` |
|---|---|---|
| Meaning | Returns, but no useful value | Does not return at all |
| Example | `console.log()`, event handlers | throws error, infinite loop |

```typescript
// void — runs, finishes, returns undefined implicitly
function logMessage(message: string): void {
  console.log(message);
}

// never — throws before it can return
function throwError(message: string): never {
  throw new Error(message); // execution ends here
}
```

**How to throw an error in TypeScript:**
```typescript
throw new Error(message); // ✅ — keyword 'throw', then 'new Error()'
throwError(message);       // ❌ — throwError is not a built-in
```

---

### Q4. What does `?` do on a function parameter? What must you always do before using it?

`?` marks a parameter as optional — it may or may not be provided by the caller.

Optional parameters have type `T | undefined` inside the function.
You **must check** before using them:

```typescript
function greet(name: string, title?: string): string {
  // title is string | undefined here

  // ❌ Unsafe — title might be undefined
  return `${title.toUpperCase()} ${name}`;

  // ✅ Safe — check first
  if (title !== undefined) {
    return `${title.toUpperCase()} ${name}`;
  }
  return name;
}

greet("Neeraj");          // ✅ title not provided — that's fine
greet("Neeraj", "Dr");   // ✅ title provided
```

**Critical rule — optional parameters must come AFTER required ones:**
```typescript
// ❌ Error — optional before required
function greet(title?: string, name: string): string {}

// ✅ Correct — optional always last
function greet(name: string, title?: string): string {}
```

---

### Q5. What is the difference between an optional parameter and a default parameter?

| | Optional `?` | Default `= value` |
|---|---|---|
| Type inside function | `string \| undefined` | always `string` |
| Requires check before use | ✅ Yes | ❌ No |
| Has a fallback value | ❌ No | ✅ Yes |

```typescript
// Optional — must check inside function
function greet(name: string, greeting?: string): string {
  if (greeting !== undefined) {
    return `${greeting}, ${name}`;
  }
  return `Hello, ${name}`;
}

// Default — always has a value, no check needed
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}`; // safe — greeting is always a string
}
```

**Rule:** When you have a sensible fallback value, prefer default parameters over optional.

---

### Q6. What is a rest parameter and how is it typed?

A rest parameter collects any number of arguments into a typed array:

```typescript
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3);         // ✅ 6
sum(1, 2, 3, 4, 5);   // ✅ 15
sum(1, "2", 3);       // ❌ Error — string not assignable to number
```

**Rules:**
- Always typed as an array — `...args: string[]`
- Must always be the **last** parameter

```typescript
// ✅ Rest parameter last
function log(prefix: string, ...messages: string[]): void {
  messages.forEach(msg => console.log(`${prefix}: ${msg}`));
}

// ❌ Error — rest not last
function log(...messages: string[], prefix: string): void {}
```

---

### Q7. What is a function type alias? Why is it useful?

A function type alias gives a name to the **shape of a function** — what parameters it takes and what it returns.

```typescript
// Syntax
type AliasName = (param: type) => returnType;

// Examples
type StringToBool  = (value: string) => boolean;
type MathOperation = (a: number, b: number) => number;
type UserHandler   = (user: { name: string; email: string }) => void;
type AsyncFetcher  = (id: number) => Promise<string>;
```

Once defined, any function matching the shape can be assigned:

```typescript
type MathOperation = (a: number, b: number) => number;

const add: MathOperation      = (a, b) => a + b;       // ✅
const subtract: MathOperation = (a, b) => a - b;       // ✅
const broken: MathOperation   = (a: string) => a;      // ❌ wrong shape
```

**Why it matters — callbacks and higher-order functions:**

```typescript
type Transformer = (value: string) => string;

function processName(name: string, transform: Transformer): string {
  return transform(name);
}

processName("neeraj", (name) => name.toUpperCase()); // ✅ "NEERAJ"
processName("neeraj", (name) => name.length);        // ❌ returns number, not string
```

---

### Q8. What is the `NextFunction` type in Express middleware?

`NextFunction` is a type imported from express that represents the `next()` callback in middleware. Calling it passes control to the next middleware or route handler.

```typescript
import { Request, Response, NextFunction } from "express";

function authMiddleware(req: Request, res: Response, next: NextFunction): void {
  const token = req.headers.authorization;

  if (!token) {
    res.status(401).json({ message: "Unauthorised" });
    return; // IMPORTANT — stops execution. Without return, next() would also run.
  }

  next(); // passes control to the next middleware
}
```

**Critical pattern — always `return` after sending a response:**

Without `return`, code continues executing after `res.json()`. If `next()` then gets called, Express attempts to send a second response — causing a crash.

---

### Q9. How do you type an async function?

Async functions return a `Promise<T>` where `T` is the type of the resolved value:

```typescript
type UserResponse = {
  id: number;
  name: string;
  email: string;
};

async function fetchUser(id: number): Promise<UserResponse> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// TypeScript knows the resolved type
const user = await fetchUser(1);
console.log(user.name);  // ✅ TypeScript knows this is string
console.log(user.nmae);  // ❌ Error — typo caught immediately
```

---

## Exercises

---

### Exercise 1. Write four typed functions

**Answer:**

```typescript
// 1. Full name — takes two strings, returns a string
function fullName(firstName: string, lastName: string): string {
  return `${firstName} ${lastName}`;
}

// 2. Highest number — takes a number array, returns a number
function highestNumber(nums: number[]): number {
  return Math.max(...nums);
}

// 3. Log user ID — takes a number, returns nothing
// Note: function name should be a verb describing the action
function logUserId(userId: number): void {
  console.log(userId);
}

// 4. Always throws — return type is 'never', not 'void'
// 'throw new Error()' is the correct syntax — not a function call
function throwError(message: string): never {
  throw new Error(message);
}
```

**Key rules demonstrated:**
- `never` — when a function always throws, it never returns. Use `never`.
- `throw new Error(message)` — this is the correct throw syntax. `throwError()` is not built-in.
- Function names should be verbs — `logUserId` not `user`

---

### Exercise 2. Find and fix all bugs

**Original broken code:**
```typescript
function createProduct(name, price, category?: string, description: string) {
  return { name, price, category, description };
}

function getDiscount(price: number, percent: number): string {
  return price - (price * percent / 100);
}
```

**All bugs:**

| # | Bug | Fix |
|---|-----|-----|
| 1 | `name` and `price` have no types | Add `: string` and `: number` |
| 2 | `category?` (optional) comes before `description` (required) | Move optional to last position |
| 3 | `getDiscount` returns a number but is typed as `: string` | Change return type to `: number` |
| 4 | `createProduct` has no return type annotation | Add explicit return type |

**Fixed code:**

```typescript
type Product = {
  name: string;
  price: number;
  description: string;
  category?: string;
};

function createProduct(
  name: string,
  price: number,
  description: string,   // required — comes before optional
  category?: string      // optional — always last
): Product {
  return { name, price, description, category };
}

function getDiscount(price: number, percent: number): number {
  return price - (price * percent / 100);
}
```

---

### Exercise 3. Define function type aliases

**The correct syntax for a function type alias:**
```typescript
type AliasName = (param: type) => returnType;
// No curly braces. No object wrapper. Just the function signature.
```

**Answer:**

```typescript
// 1. Takes a string, returns a boolean
type StringToBool = (value: string) => boolean;

// Example implementation
const checkLength: StringToBool = (value) => value.length > 4;

// 2. Takes two numbers, returns a number
type MathOperation = (a: number, b: number) => number;

// Example implementations
const add: MathOperation = (a, b) => a + b;
const subtract: MathOperation = (a, b) => a - b;

// 3. Takes a user object, returns nothing
// Note: the task said "user object" — one parameter that is an object
type UserHandler = (user: { name: string; email: string }) => void;

// Example implementation
const printUser: UserHandler = (user) => {
  console.log(`${user.name} — ${user.email}`);
};

// 4. Async function — takes a number ID, resolves to a string
type AsyncFetcher = (id: number) => Promise<string>;

// Example implementation
const fetchById: AsyncFetcher = async (id) => {
  return `Result for ID: ${id}`;
};
```

**Common mistake — function type alias is NOT an object type:**
```typescript
// ❌ This is an object type with a method — not a function type alias
type MathOperation = {
  pqr: (a: number, b: number) => number;
};

// ✅ This is a function type alias
type MathOperation = (a: number, b: number) => number;
```

---

### Exercise 4. Express middleware — validateBody

**Answer:**

```typescript
import { Request, Response, NextFunction } from "express";

function validateBody(req: Request, res: Response, next: NextFunction): void {
  if (!req.body) {
    res.status(400).json({ message: "Body is required" });
    return; // critical — stops execution, prevents next() from being called
  }
  next();
}
```

**Four things that must be right:**
1. `next: NextFunction` — imported from express, not untyped
2. `res.json({ ... })` — object must be wrapped in `{ }`
3. `return` after sending response — without it, `next()` runs too, causing a crash
4. Return type is `void` — middleware sends a response or calls next, it doesn't return a value

---

### Exercise 5. Implement the Calculator type

**The given type:**
```typescript
type Calculator = {
  add: (a: number, b: number) => number;
  subtract: (a: number, b: number) => number;
  multiply: (a: number, b: number) => number;
  reset: () => void;
};
```

**Answer:**

```typescript
const calculator: Calculator = {
  add: (a: number, b: number): number => a + b,
  subtract: (a: number, b: number): number => a - b,
  multiply: (a: number, b: number): number => a * b,
  reset: (): void => {
    console.log("Calculator reset");
  },
};

// Usage
calculator.add(4, 5);       // 9
calculator.subtract(5, 4);  // 1
calculator.multiply(3, 5);  // 15
calculator.reset();          // "Calculator reset"
```

**Three things that must be right:**
1. `const calculator: Calculator` — typed as Calculator so TypeScript verifies the shape
2. Parameters `a: number, b: number` — all parameters must be typed
3. Return types annotated on each method — `: number`, `: void`

---

## Key Mistakes to Remember

| Mistake | Wrong | Right |
|---------|-------|-------|
| Throwing an error | `throwError(message)` | `throw new Error(message)` |
| Function always throws | return type `void` | return type `never` |
| Optional before required | `greet(title?: string, name: string)` | `greet(name: string, title?: string)` |
| Function type alias | `type F = { fn: () => void }` | `type F = () => void` |
| res.json without braces | `res.json(message: "x")` | `res.json({ message: "x" })` |
| No return after response | sends response then calls next() | `return` immediately after response |
| Untyped next param | `next` | `next: NextFunction` |
| Object not typed | `const calc = { ... }` | `const calc: Calculator = { ... }` |
| Function name is noun | `function user()` | `function logUser()` |

---

## Terms from This Lesson

| Term | Meaning |
|------|---------|
| **Parameter type** | The type declared on a function's input — `(name: string)` |
| **Return type** | The type declared on a function's output — `: string` |
| **Optional parameter** | Parameter that may not be provided — marked with `?` |
| **Default parameter** | Parameter with a fallback value — `greeting = "Hello"` |
| **Rest parameter** | Collects multiple args into a typed array — `...args: string[]` |
| **Function type alias** | A named type describing a function's shape — `type F = (x: string) => boolean` |
| **`void`** | Function returns nothing useful |
| **`never`** | Function does not return at all |
| **`Promise<T>`** | Return type of async functions — T is the resolved value type |
| **`NextFunction`** | Express type for the `next()` callback in middleware |

---

*TypeScript Handbook — Lesson 6 Q&A Reference*
*Next: Lesson 7 — Interfaces*