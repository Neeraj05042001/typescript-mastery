# Lesson 4 — Arrays in TypeScript

## First, connect to what you already know

In Lesson 3 you learned that every variable has a type. A `string` variable can only hold text. A `number` variable can only hold numbers.

Arrays work the same way — but instead of holding **one** value of a type, they hold **many**.

```typescript
// A single string
let name: string = "Neeraj";

// A collection of strings
let names: string[] = ["Neeraj", "Rahul", "Priya"];
```

The rule is the same: once you type an array, **every item inside it must match that type.** TypeScript enforces this across the entire array, not just at declaration.

---

## Why does TypeScript need typed arrays?

In JavaScript, arrays are completely open:

```javascript
// JavaScript — perfectly valid, perfectly dangerous
let users = ["Neeraj", 42, true, null, { name: "someone" }];
```

This creates a nightmare in real applications. You never know what you're iterating over. You can't trust `.map()`, `.filter()`, or any method that depends on the item type.

TypeScript fixes this by locking the array to a specific type of element — so every item is guaranteed to be what you expect.

---

## Syntax — Two ways to write typed arrays

TypeScript gives you two syntaxes. Both are completely valid. Both do the same thing.

```typescript
// Syntax 1 — shorthand (more common, preferred)
let names: string[] = ["Neeraj", "Rahul", "Priya"];
let scores: number[] = [95, 87, 100];
let flags: boolean[] = [true, false, true];

// Syntax 2 — generic syntax (Array<Type>)
let names: Array<string> = ["Neeraj", "Rahul", "Priya"];
let scores: Array<number> = [95, 87, 100];
let flags: Array<boolean> = [true, false, true];
```

### Which one to use?

```typescript
// ✅ Use string[] for simple arrays — cleaner, more readable
let emails: string[] = [];

// ✅ Use Array<Type> when working with complex generics — clearer
let responses: Array<Promise<string>> = [];
// Promise<string>[] is harder to read at a glance
```

In most real codebases you'll see `string[]` for simple cases and `Array<Type>` for complex types. We'll cover **Generics** (the `<>` syntax) in depth in the Intermediate phase.

---

## TypeScript enforces the type on every operation

This is critical. The type lock isn't just at declaration — it applies to every push, assignment, and access:

```typescript
let scores: number[] = [95, 87, 100];

scores.push(88);       // ✅ Fine — 88 is a number
scores.push("ninety"); // ❌ Error — string is not assignable to number
scores[0] = 99;        // ✅ Fine
scores[0] = "99";      // ❌ Error
```

TypeScript also knows the **return type of every array method**:

```typescript
let scores: number[] = [95, 87, 100];

const doubled = scores.map(score => score * 2);
// TypeScript infers: doubled is number[]
// It knows score is a number because the array is number[]

const highScores = scores.filter(score => score > 90);
// TypeScript infers: highScores is number[]

const total = scores.reduce((sum, score) => sum + score, 0);
// TypeScript infers: total is number
```

This is one of TypeScript's biggest wins in real projects — your IDE knows the type of every value at every step.

---

## Type Inference with arrays

Just like primitives, TypeScript can infer array types:

```typescript
// TypeScript infers: string[]
const names = ["Neeraj", "Rahul", "Priya"];

// TypeScript infers: number[]
const scores = [95, 87, 100];

// TypeScript infers: boolean[]
const flags = [true, false, true];
```

**But here is a critical trap — the empty array problem:**

```typescript
// ❌ TypeScript infers: never[]
// An empty array with no annotation = TypeScript doesn't know the type
// It uses 'never' — meaning nothing can ever be pushed into it
const items = [];

items.push("hello"); // ❌ Error — Argument of type 'string' not assignable to 'never'
```

This is one of the most common beginner mistakes. **Always annotate empty arrays explicitly:**

```typescript
// ✅ Correct — TypeScript now knows what this array will hold
const items: string[] = [];
items.push("hello"); // ✅ Fine
```

---

## Arrays with union types

Sometimes an array needs to hold more than one type. You combine what you learned about types with what you're learning now:

```typescript
// Array that holds strings OR numbers
let mixed: (string | number)[] = ["Neeraj", 25, "Delhi", 100];

// Array that holds strings OR null (common in API responses)
let usernames: (string | null)[] = ["Neeraj", null, "Rahul", null];
```

Note the parentheses — `(string | number)[]` means "array of string-or-number." Without them, `string | number[]` would mean "string OR array-of-number" which is very different.

```typescript
// ❌ Wrong — means: string OR number[]
let data: string | number[];

// ✅ Right — means: array of (string or number)
let data: (string | number)[];
```

---

## `readonly` arrays

In real applications, you often have data that should **never be mutated** after creation — configuration, constants, lookup tables, initial state. TypeScript gives you `readonly` arrays for this.

```typescript
// readonly — cannot be modified after creation
const ALLOWED_ROLES: readonly string[] = ["admin", "editor", "viewer"];

ALLOWED_ROLES.push("superuser");  // ❌ Error — cannot mutate readonly array
ALLOWED_ROLES[0] = "superadmin";  // ❌ Error
ALLOWED_ROLES.pop();              // ❌ Error

// Reading is fine
console.log(ALLOWED_ROLES[0]);    // ✅ "admin"
```

**Alternative syntax using `ReadonlyArray<T>`:**

```typescript
const ALLOWED_ROLES: ReadonlyArray<string> = ["admin", "editor", "viewer"];
// Same thing — just the generic syntax form
```

### Real-world usage — why readonly matters

```typescript
// ✅ Real config in a Node.js/Express app
const SUPPORTED_CURRENCIES: readonly string[] = ["INR", "USD", "EUR", "GBP"];

const HTTP_METHODS: readonly string[] = ["GET", "POST", "PUT", "DELETE", "PATCH"];

// No one can accidentally modify these at runtime
// The array is locked at the type level
```

---

## Multi-dimensional arrays

Arrays can hold other arrays. The syntax just adds another `[]`:

```typescript
// 2D array — array of arrays of numbers (think: a grid or matrix)
const matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// Accessing values
console.log(matrix[0][1]); // 2

// 2D string array — think: a table of names
const teams: string[][] = [
  ["Neeraj", "Rahul"],
  ["Priya", "Anjali"],
];
```

### Real-world usage — CSV data

```typescript
// A CSV file parsed into rows and columns
const csvData: string[][] = [
  ["Name", "Email", "Role"],
  ["Neeraj", "neeraj@gmail.com", "admin"],
  ["Rahul", "rahul@gmail.com", "editor"],
];
```

---

## Real-world usage in your MERN stack

### React — typed state arrays

```typescript
import { useState } from "react";

// ✅ Typed state — TypeScript knows this is always string[]
const [tags, setTags] = useState<string[]>([]);

// Adding a tag
setTags(prev => [...prev, "typescript"]); // ✅

// This would be caught immediately
setTags(prev => [...prev, 42]); // ❌ Error — 42 is not a string
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

// TypeScript knows every 'user' in the map is a User object
users.map(user => (
  <div key={user.id}>{user.name}</div> // ✅ Full autocomplete on user
));
```

### Node.js/Express — typed API response

```typescript
import express, { Request, Response } from "express";
const router = express.Router();

const products: string[] = ["Laptop", "Phone", "Tablet"];

router.get("/products", (req: Request, res: Response) => {
  res.json(products); // TypeScript knows this is string[]
});
```

---

## Common mistakes — summarised

```typescript
// ❌ Mistake 1 — Untyped empty array
const items = [];              // inferred as never[] — useless
// ✅ Fix
const items: string[] = [];

// ❌ Mistake 2 — Wrong union syntax
const data: string | number[] = []; // means string OR number[]
// ✅ Fix
const data: (string | number)[] = [];

// ❌ Mistake 3 — Mutating a readonly array
const roles: readonly string[] = ["admin", "viewer"];
roles.push("superuser");       // Error
// ✅ Use a regular array if mutation is needed
const roles: string[] = ["admin", "viewer"];

// ❌ Mistake 4 — Ignoring inferred item types
const scores: number[] = [1, 2, 3];
scores.map(s => s.toUpperCase()); // Error — toUpperCase doesn't exist on number
// ✅ TypeScript told you the type — use the right methods
scores.map(s => s.toFixed(2));    // ✅

// ❌ Mistake 5 — Using capital Array without generics
const names: Array = [];      // Error — Array needs a type parameter
// ✅ Fix
const names: Array<string> = [];
```

---

## Exercises — write all of these

**Exercise 1.** Declare these arrays with correct types:
- A list of product names
- A list of user ages
- A list of login statuses (true/false per user)
- An empty array that will eventually hold email addresses
- A list of items that can be either a string or a number

**Exercise 2.** What is wrong with this code? Fix every issue:

```typescript
const userIds = [];
userIds.push(1);
userIds.push("two");
userIds.push(3);

const permissions: string | string[] = ["read", "write"];
permissions.push("delete");
```

**Exercise 3.** You have this array. Write three things:
- A `.map()` that returns each name in uppercase
- A `.filter()` that returns only names longer than 4 characters
- A `.find()` that returns the first name starting with "R"

```typescript
const names: string[] = ["Neeraj", "Raj", "Priya", "Rahul", "Ali"];
```

**Exercise 4.** You are building an Express route that returns a list of usernames. Write the route handler with proper types. The usernames array should be `readonly` — nobody should be able to accidentally modify it.

---

Take your time. Write complete, working code for all four. 