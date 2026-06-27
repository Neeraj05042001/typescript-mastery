# Lesson 4 — Arrays in TypeScript

## What is it?

A typed array is a collection of values where **every item must match a declared type**.  
TypeScript enforces the type on every item, every push, every method call — not just at declaration.

---

## Why do we need it?

JavaScript arrays accept anything:

```javascript
// JavaScript — no errors, total chaos
const users = ["Neeraj", 42, true, null, { name: "someone" }];
```

In production apps with hundreds of files, this causes silent bugs that only surface at runtime.  
TypeScript locks an array to a specific element type — every item is guaranteed to be what you expect.

---

## Problem it solves

- Prevents mixed-type arrays from causing runtime crashes
- Gives you accurate autocomplete on every array item
- Makes `.map()`, `.filter()`, `.find()` return correctly typed results
- Catches wrong method calls before code runs (e.g. calling `.toUpperCase()` on a number)

---

## Syntax — Two forms

```typescript
// Form 1 — Shorthand (preferred for simple types)
let names: string[] = ["Neeraj", "Rahul", "Priya"];
let scores: number[] = [95, 87, 100];
let flags: boolean[] = [true, false, true];

// Form 2 — Generic syntax (clearer for complex types)
let names: Array<string> = ["Neeraj", "Rahul", "Priya"];
let scores: Array<number> = [95, 87, 100];
let flags: Array<boolean> = [true, false, true];
```

### Which form to use?

| Situation | Preferred syntax |
|---|---|
| Simple types | `string[]`, `number[]` |
| Complex/nested generics | `Array<Promise<string>>` |

---

## Type Inference

TypeScript can infer the type from initial values:

```typescript
const names = ["Neeraj", "Rahul"]; // inferred: string[]
const scores = [95, 87, 100];       // inferred: number[]
```

### Critical trap — the empty array problem

```typescript
// ❌ TypeScript infers: never[]
// Nothing can ever be pushed into this
const items = [];
items.push("hello"); // Error — Argument of type 'string' not assignable to 'never'

// ✅ Always annotate empty arrays explicitly
const items: string[] = [];
items.push("hello"); // ✅ Fine
```

**Rule: Never declare an empty array without a type annotation.**

---

## TypeScript Enforces Types on Every Operation

```typescript
const scores: number[] = [95, 87, 100];

scores.push(88);        // ✅ number — allowed
scores.push("ninety");  // ❌ string — error
scores[0] = 99;         // ✅ number — allowed
scores[0] = "99";       // ❌ string — error
```

---

## Array Methods — Return Types TypeScript Infers

This is one of TypeScript's biggest wins in real projects.  
TypeScript knows exactly what type each method returns:

```typescript
const names: string[] = ["Neeraj", "Raj", "Priya", "Rahul", "Ali"];

// .map() → string[] — same length, transformed values
const uppercased = names.map((name) => name.toUpperCase());
// inferred: string[]

// .filter() → string[] — same type, fewer items
const longNames = names.filter((name) => name.length > 4);
// inferred: string[]

// .find() → string | undefined — one match OR nothing
const startsWithR = names.find((name) => name.startsWith("R"));
// inferred: string | undefined  ← CRITICAL

// .reduce() → inferred from accumulator type
const total = [1, 2, 3].reduce((sum, n) => sum + n, 0);
// inferred: number
```

### Why `.find()` returning `string | undefined` matters

`.find()` might find nothing. TypeScript reflects this in the type.  
If you ignore it, you get a runtime crash:

```typescript
const startsWithR = names.find((name) => name.startsWith("R"));
// Type: string | undefined

// ❌ Dangerous — might crash if nothing found
console.log(startsWithR.toUpperCase()); // Error: possibly undefined

// ✅ Always check first
if (startsWithR !== undefined) {
  console.log(startsWithR.toUpperCase()); // Safe
}
```

**Return type reference:**

| Method | Return type |
|---|---|
| `.map()` | `T[]` — same length, transformed |
| `.filter()` | `T[]` — same type, fewer items |
| `.find()` | `T \| undefined` — one item or nothing |
| `.findIndex()` | `number` — index or -1 |
| `.reduce()` | accumulator type |
| `.some()` / `.every()` | `boolean` |

---

## Union Type Arrays

When an array needs to hold more than one type:

```typescript
// Array of strings OR numbers
const mixed: (string | number)[] = ["Neeraj", 25, "Delhi", 100];

// Array of strings OR null (common in API responses)
const usernames: (string | null)[] = ["Neeraj", null, "Rahul", null];
```

### Critical parentheses rule

```typescript
// ❌ Wrong — means: string OR array-of-number
const data: string | number[];

// ✅ Right — means: array of (string or number)
const data: (string | number)[];
```

---

## `readonly` Arrays

Arrays that must never be mutated after creation.  
Used for configuration, constants, lookup tables, initial state.

```typescript
const ALLOWED_ROLES: readonly string[] = ["admin", "editor", "viewer"];

ALLOWED_ROLES.push("superuser"); // ❌ Error — cannot mutate readonly
ALLOWED_ROLES[0] = "superadmin"; // ❌ Error
ALLOWED_ROLES.pop();             // ❌ Error

console.log(ALLOWED_ROLES[0]);   // ✅ Reading is fine
```

**Alternative syntax:**
```typescript
const ALLOWED_ROLES: ReadonlyArray<string> = ["admin", "editor", "viewer"];
// Same thing — just the generic form
```

### Real-world usage

```typescript
const SUPPORTED_CURRENCIES: readonly string[] = ["INR", "USD", "EUR", "GBP"];
const HTTP_METHODS: readonly string[] = ["GET", "POST", "PUT", "DELETE", "PATCH"];
// No request handler can accidentally modify these at runtime
```

---

## Multi-Dimensional Arrays

```typescript
// 2D number array — a grid or matrix
const matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

console.log(matrix[0][1]); // 2

// Real-world — parsed CSV data
const csvData: string[][] = [
  ["Name", "Email", "Role"],
  ["Neeraj", "neeraj@gmail.com", "admin"],
  ["Rahul", "rahul@gmail.com", "editor"],
];
```

---

## Real-World Usage

### React — typed state

```typescript
import { useState } from "react";

const [tags, setTags] = useState<string[]>([]);

setTags(prev => [...prev, "typescript"]); // ✅
setTags(prev => [...prev, 42]);           // ❌ Error — caught immediately
```

### React — rendering a typed list

```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

const users: User[] = [
  { id: 1, name: "Neeraj", email: "neeraj@gmail.com" },
  { id: 2, name: "Rahul", email: "rahul@gmail.com" },
];

users.map((user) => (
  <div key={user.id}>{user.name}</div> // full autocomplete on user
));
```

### Node.js / Express — typed route with readonly data

```typescript
import express, { Request, Response } from "express";

const router = express.Router();

// readonly — route handlers cannot accidentally mutate this
const USERNAMES: readonly string[] = ["Neeraj", "Rahul", "Priya"];

router.get("/users", (req: Request, res: Response): void => {
  res.json(USERNAMES);
});

export default router;
```

**Key Express + TypeScript pattern:**
- `Request` and `Response` are types imported from express
- Route handlers return `void` — they send responses, not return values
- Constants shared across routes should be `readonly`

---

## Common Mistakes

```typescript
// ❌ Mistake 1 — Untyped empty array
const items = [];              // never[] — nothing can be pushed
// ✅ Fix
const items: string[] = [];

// ❌ Mistake 2 — Wrong union syntax
const data: string | number[]; // means: string OR number[]
// ✅ Fix
const data: (string | number)[];

// ❌ Mistake 3 — Using capital Array without type parameter
const names: Array = [];      // Error — needs a type parameter
// ✅ Fix
const names: Array<string> = [];

// ❌ Mistake 4 — Ignoring .find() returning undefined
const result = names.find((n) => n.startsWith("R"));
result.toUpperCase(); // Error — possibly undefined
// ✅ Fix
if (result !== undefined) {
  result.toUpperCase();
}

// ❌ Mistake 5 — Using index instead of startsWith
names.find((n) => n[0] === "R"); // works but fragile
// ✅ Fix — intention-revealing
names.find((n) => n.startsWith("R"));

// ❌ Mistake 6 — Typo in keyword
cosnt names: string[] = [];    // 'cosnt' not recognised — compiler error
// ✅ Fix
const names: string[] = [];
```

---

## Best Practices

1. **Always annotate empty arrays** — `const items: string[] = []`
2. **Use `readonly`** for constants, config, lookup tables
3. **Always handle `| undefined`** from `.find()` — check before using
4. **Use `startsWith()`** instead of `[0] ===` for character checks
5. **Use `string[]`** for simple types, `Array<T>` for complex generics
6. **Name arrays as plurals** — `users`, `names`, `scores`, not `user`, `name`, `score`
7. **In Express** — always import `Request` and `Response` types from express

---

## Interview Questions

**Beginner:**
- What is the difference between `string[]` and `Array<string>`?
- Why must you annotate an empty array explicitly?
- What does `readonly` do to an array?

**Intermediate:**
- What type does `.find()` return, and why?
- What is the difference between `string | number[]` and `(string | number)[]`?
- When would you use `Array<T>` over `T[]`?

**Advanced:**
- What is `ReadonlyArray<T>` and when would you use it over `readonly T[]`?
- How does TypeScript infer return types for `.map()` and `.filter()`?
- What happens when you declare `const items = []` without a type annotation?

---

## Summary

- Two syntaxes — `string[]` and `Array<string>` — both valid, shorthand preferred
- TypeScript enforces the type on **every operation** — push, index, method calls
- **Always annotate empty arrays** — untyped empty arrays become `never[]`
- `.find()` returns `T | undefined` — always check before using
- `readonly` locks an array after creation — use for constants and config
- Array methods return predictable, inferred types — TypeScript tracks them all
- In Express — import `Request`, `Response` from express; route handlers return `void`

---

## 2-Minute Revision Notes

```
string[]        → array of strings
number[]        → array of numbers
boolean[]       → array of booleans
string[][]      → 2D array (array of string arrays)

Array<string>   → same as string[] — use for complex generics

const x = []   → ❌ becomes never[] — always annotate empty arrays
const x: string[] = [] → ✅ correct

(string|number)[] → array of string OR number
string|number[]   → ❌ means string OR number-array — wrong

readonly string[] → cannot push, pop, or mutate — use for constants

.map()     → T[]           (same length, transformed)
.filter()  → T[]           (same type, fewer items)
.find()    → T | undefined (one item or nothing — always check!)
.reduce()  → accumulator type

Express pattern:
  import { Request, Response } from "express"
  (req: Request, res: Response): void => { ... }
  readonly for shared data across routes
```

---

*TypeScript Handbook — Lesson 4 of 5 (Foundations Phase)*  
*Next: Lesson 5 — Objects & Type Aliases*
