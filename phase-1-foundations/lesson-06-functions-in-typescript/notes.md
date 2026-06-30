# Lesson 6 — Functions in TypeScript

## First, connect to what you already know

In Lesson 5 you typed the **shape of objects** — what properties they have and what types those properties hold.

Functions are the same idea applied to behaviour — TypeScript lets you type what goes **in** (parameters) and what comes **out** (return value). Once typed, the compiler enforces the contract at every call site.

```typescript
// Lesson 5 — typing the shape of data
type User = {
  name: string;
  age: number;
};

// Lesson 6 — typing the shape of behaviour
function greetUser(name: string, age: number): string {
  return `Hello ${name}, you are ${age} years old`;
}
```

The same principles apply — wrong types are caught at compile time, not runtime.

---

## Why does TypeScript need typed functions?

In JavaScript, functions have no contract:

```javascript
function add(a, b) {
  return a + b;
}

add(10, 20);       // 30 — correct
add("10", 20);     // "1020" — string concatenation, silent bug
add(10);           // NaN — missing argument, no error
add(10, 20, 30);   // 30 — extra argument silently ignored
```

Every one of these would cause a real bug in production. TypeScript fixes all of them by enforcing parameter types, return types, and argument counts at compile time.

---

## Part 1 — Parameter Types and Return Types

### Basic syntax

```typescript
function functionName(param1: type1, param2: type2): returnType {
  // body
}
```

```typescript
// ✅ Fully typed function
function add(a: number, b: number): number {
  return a + b;
}

add(10, 20);       // ✅ 30
add("10", 20);     // ❌ Error — string not assignable to number
add(10);           // ❌ Error — expected 2 arguments, got 1
add(10, 20, 30);   // ❌ Error — expected 2 arguments, got 3
```

TypeScript enforces the **exact number of arguments** unless you explicitly mark them optional. This alone catches a huge category of bugs.

---

### Return types

Always annotate the return type explicitly. TypeScript can infer it, but explicit return types are:
- Easier to read
- Self-documenting
- Catch mistakes where you accidentally return the wrong thing

```typescript
// TypeScript infers return type as number — works but not explicit
function multiply(a: number, b: number) {
  return a * b;
}

// ✅ Explicit return type — preferred in production
function multiply(a: number, b: number): number {
  return a * b;
}

// TypeScript catches wrong return types
function getUsername(id: number): string {
  return id; // ❌ Error — number is not assignable to string
}
```

---

### `void` return type — recap

Functions that perform actions but return nothing:

```typescript
function logMessage(message: string): void {
  console.log(message);
  // No return statement — or bare return;
}
```

---

### `never` return type — recap

Functions that never reach a return point:

```typescript
function throwError(message: string): never {
  throw new Error(message);
}
```

---

## Part 2 — Optional Parameters

Just like optional properties on objects, functions can have optional parameters using `?`:

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

**Critical rule — optional parameters must come after required ones:**

```typescript
// ❌ Error — optional before required
function greet(greeting?: string, name: string): string {}

// ✅ Correct — optional always comes last
function greet(name: string, greeting?: string): string {}
```

This makes logical sense — JavaScript fills arguments left to right. If the optional parameter came first, there'd be no way to skip it cleanly.

---

## Part 3 — Default Parameters

Default parameters give a fallback value when the argument isn't provided:

```typescript
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}`;
}

greet("Neeraj");            // ✅ "Hello, Neeraj" — default used
greet("Neeraj", "Namaste"); // ✅ "Namaste, Neeraj" — default overridden
```

**Default vs Optional — the key difference:**

```typescript
// Optional — parameter is string | undefined inside the function
function greet(name: string, greeting?: string): string {
  // greeting is string | undefined here — must check before using
  if (greeting !== undefined) { ... }
}

// Default — parameter always has a value inside the function
function greet(name: string, greeting: string = "Hello"): string {
  // greeting is always string here — no check needed
  return `${greeting}, ${name}`;
}
```

Default parameters are almost always preferable over optional when you have a sensible fallback value.

---

## Part 4 — Rest Parameters

When a function can accept any number of arguments of the same type:

```typescript
// ...args collects all extra arguments into a typed array
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3);       // ✅ 6
sum(1, 2, 3, 4, 5); // ✅ 15
sum(1, "2", 3);     // ❌ Error — string not assignable to number
```

**Rest parameters are always typed as an array** and must be the last parameter:

```typescript
// ✅ Rest parameter — always last
function logMessages(prefix: string, ...messages: string[]): void {
  messages.forEach(msg => console.log(`${prefix}: ${msg}`));
}

logMessages("INFO", "Server started", "Listening on port 3000");
// INFO: Server started
// INFO: Listening on port 3000
```

---

## Part 5 — Function Type Aliases

In JavaScript, functions are first-class values — you can assign them to variables, pass them as arguments, and return them from other functions.

TypeScript lets you define the **type of a function itself** — what parameters it takes and what it returns:

```typescript
// Defining the shape of a function
type MathOperation = (a: number, b: number) => number;

// Any function matching this shape can be assigned
const add: MathOperation = (a, b) => a + b;        // ✅
const subtract: MathOperation = (a, b) => a - b;   // ✅
const multiply: MathOperation = (a, b) => a * b;   // ✅

// Wrong shape — caught immediately
const broken: MathOperation = (a: string) => a;    // ❌ Error
```

### Why this matters — callbacks and higher-order functions

This is where function types become essential in real projects:

```typescript
// A function that accepts another function as a parameter
type Transformer = (value: string) => string;

function processName(name: string, transform: Transformer): string {
  return transform(name);
}

// ✅ Passing functions that match the Transformer shape
processName("neeraj", (name) => name.toUpperCase()); // "NEERAJ"
processName("neeraj", (name) => name.trim());        // "neeraj"

// ❌ Wrong shape — caught at compile time
processName("neeraj", (name) => name.length);        // Error — returns number, not string
```

---

## Part 6 — Arrow Functions with Types

Arrow functions are typed the same way as regular functions:

```typescript
// Regular function
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function — same type, different syntax
const add = (a: number, b: number): number => {
  return a + b;
};

// Arrow function — concise form (return type inferred)
const add = (a: number, b: number): number => a + b;
```

**When to use which:**

```typescript
// Regular functions — named utilities, exported functions
function calculateTax(price: number, rate: number): number {
  return price * rate;
}

// Arrow functions — callbacks, component handlers, inline functions
const handleClick = (event: React.MouseEvent): void => {
  event.preventDefault();
};
```

---

## Part 7 — Typing Functions in Objects and Types

Functions appear as properties of objects and type aliases constantly:

```typescript
type User = {
  id: number;
  name: string;
  greet: () => string;             // method with no params
  updateName: (name: string) => void; // method with params
};

const user: User = {
  id: 1,
  name: "Neeraj",
  greet: () => `Hello, I am ${user.name}`,
  updateName: (name) => { user.name = name; },
};
```

### Real-world — React component props with function types

```typescript
type ButtonProps = {
  label: string;
  onClick: () => void;                          // no args, no return
  onChange?: (value: string) => void;           // optional, takes string
  onSubmit?: (data: FormData) => Promise<void>; // async function type
};
```

---

## Part 8 — Real-World Usage

### React — event handlers

```typescript
import { useState } from "react";

// Typing event handler functions
function SearchBar(): JSX.Element {
  const [query, setQuery] = useState<string>("");

  // ✅ TypeScript knows exactly what the event is
  const handleChange = (event: React.ChangeEvent<HTMLInputElement>): void => {
    setQuery(event.target.value);
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>): void => {
    event.preventDefault();
    console.log("Searching for:", query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={handleChange} />
      <button type="submit">Search</button>
    </form>
  );
}
```

### Node.js — utility functions

```typescript
// A reusable typed utility
function paginate<T>(items: T[], page: number, limit: number): T[] {
  const start = (page - 1) * limit;
  return items.slice(start, start + limit);
}

// TypeScript infers the return type from the input
const users = paginate(["Neeraj", "Rahul", "Priya"], 1, 2);
// returns string[]

// We'll cover generics (<T>) in depth in the Intermediate phase
```

### Express — route handlers and middleware

```typescript
import { Request, Response, NextFunction } from "express";

// Middleware function — note the NextFunction type
function authMiddleware(req: Request, res: Response, next: NextFunction): void {
  const token = req.headers.authorization;

  if (!token) {
    res.status(401).json({ message: "Unauthorised" });
    return; // important — stops execution without calling next()
  }

  next(); // passes to the next middleware or route handler
}

// Route handler
function getUsers(req: Request, res: Response): void {
  const users = [{ id: 1, name: "Neeraj" }];
  res.json({ data: users, success: true });
}
```

### Service layer pattern — common in production

```typescript
// Typed service functions — business logic separated from routes
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

// Async function — returns a Promise
async function createUser(input: CreateUserInput): Promise<UserResponse> {
  // database logic here
  return {
    id: 1,
    name: input.name,
    email: input.email,
  };
}

// The return type Promise<UserResponse> tells every caller
// exactly what they'll get when this resolves
```

---

## Common Mistakes

```typescript
// ❌ Mistake 1 — no return type annotation
function getUser(id: number) {
  return { id, name: "Neeraj" };
  // TypeScript infers it — but explicit is always better in production
}
// ✅ Fix
function getUser(id: number): { id: number; name: string } { ... }

// ❌ Mistake 2 — optional parameter before required
function greet(greeting?: string, name: string): string {}
// ✅ Fix — optional always comes last
function greet(name: string, greeting?: string): string {}

// ❌ Mistake 3 — not checking optional parameter before using it
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
  return age >= 18; // returns actual boolean
}

// ❌ Mistake 5 — rest parameter not last
function log(...messages: string[], prefix: string): void {} // Error
// ✅ Fix
function log(prefix: string, ...messages: string[]): void {}

// ❌ Mistake 6 — forgetting return in non-void function
function add(a: number, b: number): number {
  const result = a + b;
  // forgot to return result
}
// ✅ Fix
function add(a: number, b: number): number {
  return a + b;
}
```

---

## Exercises — write all of these

**Exercise 1.** Write fully typed versions of these functions:
- A function that takes a first name and last name (both strings) and returns a full name (string)
- A function that takes an array of numbers and returns the highest number
- A function that takes a user ID (number) and returns nothing — just logs it
- A function that takes a message (string) and always throws an error

**Exercise 2.** What is wrong with this code? Identify every bug and fix it:

```typescript
function createProduct(name, price, category?: string, description: string) {
  return {
    name,
    price,
    category,
    description
  };
}

function getDiscount(price: number, percent: number): string {
  return price - (price * percent / 100);
}
```

**Exercise 3.** Define function type aliases for these:
- A function that takes a string and returns a boolean
- A function that takes two numbers and returns a number
- A function that takes a user object `{ name: string; email: string }` and returns nothing
- An async function that takes an ID (number) and returns a Promise that resolves to a string

**Exercise 4.** Write an Express middleware function called `validateBody` that:
- Takes `req`, `res`, and `next` with proper types
- Checks if `req.body` exists
- If it doesn't — sends a `400` response with `{ message: "Body is required" }`
- If it does — calls `next()`

**Exercise 5.** You have this type:
```typescript
type Calculator = {
  add: (a: number, b: number) => number;
  subtract: (a: number, b: number) => number;
  multiply: (a: number, b: number) => number;
  reset: () => void;
};
```
Create a `calculator` object that satisfies this type with working implementations.

---

Take your time. Write complete, working code for all five. I'll review each one critically, then generate both markdown files.