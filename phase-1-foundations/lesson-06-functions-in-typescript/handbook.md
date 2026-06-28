# Lesson 6 — Functions in TypeScript

## What is it?

TypeScript lets you define the **contract of a function** — what types go in as parameters
and what type comes out as a return value. Once defined, the compiler enforces this
contract at every single call site.

---

## Why do we need it?

JavaScript functions have no enforced contract:

```javascript
function add(a, b) { return a + b; }

add(10, 20);       // 30 — correct
add("10", 20);     // "1020" — silent string concatenation bug
add(10);           // NaN — missing argument, no error
add(10, 20, 30);   // 30 — extra argument silently ignored
```

TypeScript fixes every one of these by enforcing parameter types, return types,
and exact argument counts at compile time.

---

## Problem it solves

- Prevents wrong types being passed as arguments
- Enforces the correct number of arguments
- Guarantees the return value matches what callers expect
- Catches missing return statements
- Makes callbacks and higher-order functions safe

---

## Syntax

```typescript
// Regular function
function name(param1: type1, param2: type2): returnType {
  return value;
}

// Arrow function
const name = (param1: type1, param2: type2): returnType => {
  return value;
};

// Arrow function — concise form
const name = (param1: type1, param2: type2): returnType => value;
```

---

## Parameter Types and Return Types

```typescript
function add(a: number, b: number): number {
  return a + b;
}

add(10, 20);       // ✅ 30
add("10", 20);     // ❌ Error — string not assignable to number
add(10);           // ❌ Error — expected 2 arguments, got 1
add(10, 20, 30);   // ❌ Error — expected 2 arguments, got 3
```

### Why annotate return types explicitly?

TypeScript can infer return types — but explicitly annotating them is best practice because:
- Makes the contract readable at a glance
- Catches bugs where you accidentally return the wrong type
- Serves as documentation for other developers

```typescript
// ❌ Inferred — works but not explicit
function multiply(a: number, b: number) {
  return a * b;
}

// ✅ Explicit — preferred in production
function multiply(a: number, b: number): number {
  return a * b;
}

// TypeScript catches wrong return types
function getUsername(id: number): string {
  return id; // ❌ Error — number not assignable to string
}
```

---

## `void` and `never` Return Types

```typescript
// void — function performs an action, returns nothing
function logMessage(message: string): void {
  console.log(message);
}

// never — function never reaches a return point
function throwError(message: string): never {
  throw new Error(message);
}
```

| Return type | Meaning | Example |
|---|---|---|
| `void` | Returns, but no useful value | `console.log()`, event handlers |
| `never` | Does not return at all | throws error, infinite loop |

---

## Optional Parameters

Mark a parameter optional with `?` — it may or may not be provided:

```typescript
function greet(name: string, greeting?: string): string {
  if (greeting !== undefined) {
    return `${greeting}, ${name}`;
  }
  return `Hello, ${name}`;
}

greet("Neeraj");            // ✅ "Hello, Neeraj"
greet("Neeraj", "Namaste"); // ✅ "Namaste, Neeraj"
```

### Critical rule — optional parameters must come AFTER required ones

```typescript
// ❌ Error — optional before required
function greet(greeting?: string, name: string): string {}

// ✅ Correct — optional always last
function greet(name: string, greeting?: string): string {}
```

### Always check optional parameters before using them

```typescript
function greet(name: string, title?: string): string {
  // ❌ Unsafe — title might be undefined
  return `${title.toUpperCase()} ${name}`;

  // ✅ Safe — check first
  if (title !== undefined) {
    return `${title.toUpperCase()} ${name}`;
  }
  return name;
}
```

---

## Default Parameters

Give a parameter a fallback value when it is not provided:

```typescript
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}`;
}

greet("Neeraj");            // ✅ "Hello, Neeraj" — default used
greet("Neeraj", "Namaste"); // ✅ "Namaste, Neeraj" — default overridden
```

### Default vs Optional — key difference

| | Optional `?` | Default `= value` |
|---|---|---|
| Type inside function | `string \| undefined` | always `string` |
| Must check before use | ✅ Yes | ❌ No |
| Has a fallback value | ❌ No | ✅ Yes |

```typescript
// Optional — must check inside
function greet(name: string, greeting?: string): string {
  if (greeting !== undefined) { // must check
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

## Rest Parameters

Accept any number of arguments of the same type:

```typescript
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3);        // ✅ 6
sum(1, 2, 3, 4, 5);  // ✅ 15
sum(1, "2", 3);      // ❌ Error — string not assignable to number
```

### Rules for rest parameters

- Always typed as an array — `...args: string[]`
- Must always be the **last** parameter

```typescript
// ✅ Rest parameter last
function log(prefix: string, ...messages: string[]): void {
  messages.forEach(msg => console.log(`${prefix}: ${msg}`));
}

// ❌ Rest parameter not last — Error
function log(...messages: string[], prefix: string): void {}
```

---

## Function Type Aliases

Define the **type of a function itself** — what it accepts and what it returns:

```typescript
// Defining a function type
type MathOperation = (a: number, b: number) => number;

// Any function matching this shape works
const add: MathOperation = (a, b) => a + b;       // ✅
const subtract: MathOperation = (a, b) => a - b;  // ✅
const broken: MathOperation = (a: string) => a;   // ❌ Error
```

### Why this matters — callbacks and higher-order functions

```typescript
type Transformer = (value: string) => string;

function processName(name: string, transform: Transformer): string {
  return transform(name);
}

processName("neeraj", (name) => name.toUpperCase()); // ✅ "NEERAJ"
processName("neeraj", (name) => name.trim());        // ✅ "neeraj"
processName("neeraj", (name) => name.length);        // ❌ Error — returns number, not string
```

---

## Arrow Functions

Typed the same way as regular functions — just different syntax:

```typescript
// Regular function
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const add = (a: number, b: number): number => {
  return a + b;
};

// Arrow function — concise form
const add = (a: number, b: number): number => a + b;
```

### When to use which

| Style | Use for |
|---|---|
| Regular function | Named utilities, exported functions, methods |
| Arrow function | Callbacks, event handlers, inline functions |

---

## Functions as Object Properties

Function types appear inside object type aliases constantly:

```typescript
type User = {
  id: number;
  name: string;
  greet: () => string;                 // no params
  updateName: (name: string) => void;  // one param
};

const user: User = {
  id: 1,
  name: "Neeraj",
  greet: () => `Hello, I am ${user.name}`,
  updateName: (name) => { user.name = name; },
};
```

### React component props with function types

```typescript
type ButtonProps = {
  label: string;
  onClick: () => void;                          // no args, no return
  onChange?: (value: string) => void;           // optional, takes string
  onSubmit?: (data: FormData) => Promise<void>; // async function type
};
```

---

## Async Functions

Async functions return a `Promise<T>` where `T` is the type of the resolved value:

```typescript
type UserResponse = {
  id: number;
  name: string;
  email: string;
};

// Async function — return type is Promise<UserResponse>
async function fetchUser(id: number): Promise<UserResponse> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// Calling it — TypeScript knows the resolved type
const user = await fetchUser(1);
console.log(user.name);  // ✅ TypeScript knows this is string
console.log(user.nmae);  // ❌ Error — typo caught immediately
```

---

## Real-World Usage

### React — typed event handlers

```typescript
function SearchBar(): JSX.Element {
  const [query, setQuery] = useState<string>("");

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>): void => {
    setQuery(event.target.value);
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>): void => {
    event.preventDefault();
    console.log("Searching:", query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={handleChange} />
      <button type="submit">Search</button>
    </form>
  );
}
```

### Express — middleware with NextFunction

```typescript
import { Request, Response, NextFunction } from "express";

function authMiddleware(req: Request, res: Response, next: NextFunction): void {
  const token = req.headers.authorization;

  if (!token) {
    res.status(401).json({ message: "Unauthorised" });
    return; // stops execution — does not call next()
  }

  next(); // passes control to next middleware or route handler
}
```

### Service layer — typed business logic

```typescript
type CreateUserInput = {
  name: string;
  email: string;
  password: string;
};

type UserResponse = {
  id: number;
  name: string;
  email: string;
};

async function createUser(input: CreateUserInput): Promise<UserResponse> {
  // database logic here
  return {
    id: 1,
    name: input.name,
    email: input.email,
  };
}
```

---

## Common Mistakes

```typescript
// ❌ Mistake 1 — no return type annotation
function getUser(id: number) {
  return { id, name: "Neeraj" };
}
// ✅ Fix — always annotate return types in production
function getUser(id: number): { id: number; name: string } {
  return { id, name: "Neeraj" };
}

// ❌ Mistake 2 — optional before required
function greet(greeting?: string, name: string): string {}
// ✅ Fix
function greet(name: string, greeting?: string): string {}

// ❌ Mistake 3 — using optional without checking
function greet(name: string, title?: string): string {
  return `${title.toUpperCase()} ${name}`; // Error — title might be undefined
}
// ✅ Fix
function greet(name: string, title?: string): string {
  if (title !== undefined) {
    return `${title.toUpperCase()} ${name}`;
  }
  return name;
}

// ❌ Mistake 4 — wrong return type
function isAdult(age: number): boolean {
  return age >= 18 ? "yes" : "no"; // Error — string not boolean
}
// ✅ Fix
function isAdult(age: number): boolean {
  return age >= 18;
}

// ❌ Mistake 5 — rest parameter not last
function log(...messages: string[], prefix: string): void {}
// ✅ Fix
function log(prefix: string, ...messages: string[]): void {}

// ❌ Mistake 6 — missing return
function add(a: number, b: number): number {
  const result = a + b; // forgot to return
}
// ✅ Fix
function add(a: number, b: number): number {
  return a + b;
}

// ❌ Mistake 7 — optional parameter with required after it
function createUser(name: string, role?: string, email: string): void {}
// ✅ Fix
function createUser(name: string, email: string, role?: string): void {}
```

---

## Best Practices

1. **Always annotate return types** — don't rely on inference in production code
2. **Optional parameters come last** — always, no exceptions
3. **Prefer default parameters** over optional when a fallback value makes sense
4. **Always check optional parameters** before using them
5. **Rest parameters are always last** and typed as an array
6. **Use function type aliases** for callbacks and higher-order function parameters
7. **Use `Promise<T>`** for async function return types — always specify T
8. **Use `NextFunction`** from express for middleware — not just `Function`
9. **Return early** in route handlers and middleware after sending a response
10. **Name functions as verbs** — `createUser`, `fetchProduct`, `validateBody`

---

## Interview Questions

**Beginner:**
- How do you type a function's parameters and return value in TypeScript?
- What is the difference between `void` and `never` as return types?
- What does `?` do on a function parameter?

**Intermediate:**
- What is the difference between an optional parameter and a default parameter?
- What is a function type alias? Give a real-world use case.
- What is a rest parameter and how is it typed?
- Why must optional parameters come after required ones?

**Advanced:**
- What does `Promise<T>` mean as a return type for async functions?
- What is the `NextFunction` type in Express middleware?
- How do you type a higher-order function that accepts a callback?
- What is the difference between these two:
  `type F = () => void` and `function f(): never`?

---

## Summary

- TypeScript enforces parameter types, return types, and argument count on every function call
- Always annotate return types explicitly — don't rely on inference
- Optional parameters use `?` and must come after required parameters — always check before use
- Default parameters always have a value inside the function — no checking needed
- Rest parameters collect multiple args into a typed array — must be last
- Function type aliases define the shape of a function — used for callbacks and props
- Async functions return `Promise<T>` — TypeScript knows the resolved type
- Express middleware takes `Request`, `Response`, `NextFunction` — all from express

---

## 2-Minute Revision Notes

```
Parameter types      → function add(a: number, b: number): number
Return type          → always annotate explicitly in production
void                 → returns nothing useful
never                → does not return at all (throws / infinite loop)

Optional param       → greeting?: string — must check before use
Default param        → greeting: string = "Hello" — always has value
Rest param           → ...args: number[] — must be last, typed as array

Optional rule        → optional params ALWAYS come after required ones
Rest rule            → rest params ALWAYS come last

Function type alias  → type MathOp = (a: number, b: number) => number
Callback typing      → processName(name: string, fn: Transformer): string

Async function       → async function fetch(id: number): Promise<UserResponse>

Express middleware   → (req: Request, res: Response, next: NextFunction): void
Return early         → after res.json() or res.send(), call return to stop execution

Arrow vs regular:
  Regular  → named utilities, exported functions
  Arrow    → callbacks, event handlers, inline
```

---

*TypeScript Handbook — Lesson 6 of 7 (Foundations Phase)*
*Next: Lesson 7 — Interfaces*
