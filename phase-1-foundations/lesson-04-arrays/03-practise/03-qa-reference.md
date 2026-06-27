# Lesson 4 — Questions, Answers & Review
## Arrays in TypeScript

---

## Section 1 — Exercise 1: Declare Typed Arrays

---

### Task: Declare these arrays with correct types

---

**1. A list of product names**

```typescript
// ✅ Your answer — both correct
const products: string[] = ["a", "b", "c", "d"];
const products: Array<string> = ["a", "b", "c", "d"];
```

**Assessment: ✅ Correct. Good that you showed both syntaxes.**

---

**2. A list of user ages**

```typescript
// ✅ Your answer — both correct
const ages: number[] = [12, 15, 34, 50];
const ages: Array<number> = [12, 15, 34, 50];
```

**Assessment: ✅ Clean and correct.**

---

**3. A list of login statuses (true/false per user)**

```typescript
// ❌ Your attempt 1 — invalid syntax, won't compile
const userStatus: string[]boolean[] = ["Neeraj"[true, false]];

// ❌ Your attempt 2 — cannot nest generics this way
const status: Array<string<boolean>> = [{ Neeraj: true }, { priya: false }];
```

**Assessment: ❌ Both invalid — compiler rejects these immediately.**

The task said "a list of login statuses (true/false)" — that is simply a `boolean[]`.  
Your confusion came from wanting to pair names with statuses — that is an **array of objects**, covered in Lesson 5.

```typescript
// ✅ Correct — just a list of true/false values
const userStatus: boolean[] = [true, false, true, false];

// If you wanted name + status — that's Lesson 5 (object arrays)
const userStatus: { name: string; isLoggedIn: boolean }[] = [
  { name: "Neeraj", isLoggedIn: true },
  { name: "Priya", isLoggedIn: false },
];
// But this was NOT what this exercise asked for
```

---

**4. An empty array that will hold email addresses**

```typescript
// ✅ Your answer — both correct
const emailAddress: string[] = [];
const emailAddress: Array<string> = [];
```

**Assessment: ✅ Exactly right. Explicitly typed empty array — the correct pattern.**

---

**5. A list that can hold strings OR numbers**

```typescript
// ✅ Your answer
const itemList: (string | number)[] = ["as", 34, 45, "err", "wd"];
```

**Assessment: ✅ Correct parentheses around the union. You remembered the lesson.**

---

### Exercise 1 Score

| Item | Status | Note |
|---|---|---|
| Products | ✅ | Both syntaxes shown |
| Ages | ✅ | Clean |
| Login statuses | ❌ | Should be `boolean[]` — invalid syntax on both attempts |
| Empty email array | ✅ | Correctly annotated empty array |
| Mixed string/number | ✅ | Correct parentheses in union |

---

## Section 2 — Exercise 2: Find and Fix the Bugs

---

### Original broken code

```typescript
const userIds = [];
userIds.push(1);
userIds.push("two");
userIds.push(3);

const permissions: string | string[] = ["read", "write"];
permissions.push("delete");
```

---

### Your bug identification

> 1. Empty array without type declaration
> 2. Multiple types pushed into empty array — sometimes string, sometimes number
> 3. Permissions has wrong union declaration and is being mutated incorrectly

**Assessment: ✅ All three bugs correctly identified.**

---

### Your fixes

```typescript
// Fix 1 — userIds
const userIds: (string | number)[] = [];
userIds.push(1);
userIds.push("two");
userIds.push(3);

// Fix 2 — permissions
const permissions: readonly string[] = ["read", "write"];
// permissions.push("delete") → ❌ Error — readonly
```

---

### Review of your fixes

**`userIds` fix ✅ — Correct**

Since the original code pushed both strings and numbers, `(string | number)[]` is the right type.

**`permissions` fix ⚠️ — Right syntax, questionable reasoning**

Your `readonly` solution is valid TypeScript, but the original bug was purely the **wrong type annotation** — not a mutation problem.

```typescript
// ❌ Original bug — wrong type
const permissions: string | string[] = ["read", "write"];
// Means: "either a string OR an array of strings"
// TypeScript can't confirm .push() exists on a plain string
// So it blocks the push even though the value is clearly an array

// ✅ Simplest correct fix — just fix the type
const permissions: string[] = ["read", "write"];
permissions.push("delete"); // ✅ Now works fine
```

If `readonly` was genuinely your design intent, you'd also remove the push entirely:

```typescript
// ✅ If readonly is the intent — remove the push too
const permissions: readonly string[] = ["read", "write"];
// No push — the array is declared complete at creation
```

**Key lesson:** In a code review, fix what is actually broken. Adding `readonly` is a new constraint — explain your reasoning when you do.

---

### Ideal Full Solution

```typescript
// Fix 1 — typed, allows string and number
const userIds: (string | number)[] = [];
userIds.push(1);       // ✅
userIds.push("two");   // ✅
userIds.push(3);       // ✅

// Fix 2a — if the push is intentional, just fix the type
const permissions: string[] = ["read", "write"];
permissions.push("delete"); // ✅

// Fix 2b — if immutability is the intent, use readonly and remove push
const permissions: readonly string[] = ["read", "write"];
// no push needed — array is complete at declaration
```

---

## Section 3 — Exercise 3: Array Methods

---

### Task

```typescript
const names: string[] = ["Neeraj", "Raj", "Priya", "Rahul", "Ali"];
// 1. Map — every name in uppercase
// 2. Filter — only names longer than 4 characters
// 3. Find — first name starting with "R"
```

---

### Your answers

```typescript
cosnt uppercaseNames = names.map((name) => name.toUpperCase())
const namesGreaterThan4char = names.filter((name) => name.length > 4)
const startingwithR = names.find((name) => name[0] === "R")
```

---

### Review

**Map ✅ — Correct logic, one typo**

```typescript
// ❌ Your version — typo
cosnt uppercaseNames = names.map((name) => name.toUpperCase())
//  ^ 'cosnt' is not a keyword — compiler rejects this

// ✅ Fixed
const uppercaseNames = names.map((name) => name.toUpperCase());
// Result: ["NEERAJ", "RAJ", "PRIYA", "RAHUL", "ALI"]
// TypeScript infers: string[]
```

**Filter ✅ — Clean and correct**

```typescript
const namesGreaterThan4char = names.filter((name) => name.length > 4);
// Result: ["Neeraj", "Priya", "Rahul"]
// TypeScript infers: string[]

// Style note — shorter variable names are preferred in production
const longNames = names.filter((name) => name.length > 4); // ✅ cleaner
```

**Find ✅ — Correct, but a better approach exists**

```typescript
// ✅ Your version — works
const startingwithR = names.find((name) => name[0] === "R");

// ✅ Better — intention-revealing, reads like English
const startingWithR = names.find((name) => name.startsWith("R"));
// Result: "Raj" (first match)
```

`name[0]` reads a character by index — it works but it's a JS trick.  
`startsWith("R")` says exactly what you mean. Prefer it.

---

### Critical TypeScript concept: `.find()` returns `T | undefined`

This is the most important TypeScript lesson from this exercise:

```typescript
const startsWithR = names.find((name) => name.startsWith("R"));
// TypeScript infers: string | undefined
// NOT string — because .find() might find nothing
```

If no element matches, `.find()` returns `undefined`. TypeScript knows this automatically.

```typescript
// ❌ Dangerous — crashes if nothing found
console.log(startsWithR.toUpperCase()); // Error: possibly undefined

// ✅ Always check before using
if (startsWithR !== undefined) {
  console.log(startsWithR.toUpperCase()); // Safe
}
```

This is TypeScript protecting you from a real runtime bug.  
In JavaScript this crashes silently. TypeScript stops you at compile time.

---

### Array method return type reference

| Method | Return type | Note |
|---|---|---|
| `.map()` | `T[]` | Same length, transformed values |
| `.filter()` | `T[]` | Same type, fewer items |
| `.find()` | `T \| undefined` | One item OR nothing — always check |
| `.findIndex()` | `number` | Index or -1 |
| `.reduce()` | accumulator type | Inferred from initial value |
| `.some()` / `.every()` | `boolean` | |

---

### Ideal solution — Exercise 3

```typescript
const names: string[] = ["Neeraj", "Raj", "Priya", "Rahul", "Ali"];

// 1. Uppercase — returns string[]
const uppercaseNames = names.map((name) => name.toUpperCase());
// ["NEERAJ", "RAJ", "PRIYA", "RAHUL", "ALI"]

// 2. Longer than 4 characters — returns string[]
const longNames = names.filter((name) => name.length > 4);
// ["Neeraj", "Priya", "Rahul"]

// 3. First starting with R — returns string | undefined
const startsWithR = names.find((name) => name.startsWith("R"));
// "Raj"

// Always guard .find() result before using
if (startsWithR !== undefined) {
  console.log(startsWithR.toUpperCase()); // "RAJ"
}
```

---

## Section 4 — Exercise 4: Express Route with Readonly Array

---

### Task

Write an Express route that returns a list of usernames.  
The usernames array should be `readonly`.

**Your answer:** Did not attempt — unfamiliar with Express + TypeScript syntax.

---

### Express + TypeScript — The Pattern Explained

**Step 1 — Import types from Express**

```typescript
import express, { Request, Response } from "express";
// Request and Response are TypeScript types exported by Express
// They describe exactly what req and res are
```

**Step 2 — Route handlers are typed functions**

```typescript
router.get("/path", (req: Request, res: Response): void => {
  // req is typed — you get autocomplete on req.body, req.params etc.
  // res is typed — you know what methods exist: res.json(), res.send()
  // void — route handlers don't return values, they send responses
});
```

**Step 3 — Full solution**

```typescript
import express, { Request, Response } from "express";

const router = express.Router();

// readonly — no route handler can accidentally modify this
const USERNAMES: readonly string[] = ["Neeraj", "Rahul", "Priya", "Anjali"];

router.get("/users", (req: Request, res: Response): void => {
  res.json(USERNAMES);
  // TypeScript knows USERNAMES is readonly string[]
  // If any handler tried USERNAMES.push(...) → ❌ compile error
});

export default router;
```

**Why `readonly` specifically here:**

Route handlers run on every incoming request. Without `readonly`, any handler could accidentally mutate the shared array — corrupting data for all subsequent requests. `readonly` prevents this at the **type level**, before the code even runs.

---

## Key Mistakes to Remember

| Mistake | Wrong | Right |
|---------|-------|-------|
| Untyped empty array | `const x = []` | `const x: string[] = []` |
| Wrong union syntax | `string \| number[]` | `(string \| number)[]` |
| Ignoring `.find()` result | `result.toUpperCase()` | `if (result) { result.toUpperCase() }` |
| Character check by index | `name[0] === "R"` | `name.startsWith("R")` |
| Typo in keyword | `cosnt` | `const` |
| Missing type parameter | `Array` | `Array<string>` |
| Untyped route handlers | `(req, res) =>` | `(req: Request, res: Response): void =>` |

---

## Terms Introduced This Lesson

| Term | Meaning |
|------|---------|
| **`T[]`** | Shorthand array syntax — array of type T |
| **`Array<T>`** | Generic array syntax — same as `T[]` |
| **`readonly T[]`** | Array that cannot be mutated after creation |
| **`never[]`** | What TypeScript infers for untyped empty arrays — useless |
| **Type inference on methods** | TypeScript automatically knows what `.map()`, `.filter()`, `.find()` return |
| **`T \| undefined`** | What `.find()` always returns — must be checked before use |
| **`Request`, `Response`** | Types imported from express to type route handler parameters |

---

*TypeScript Handbook — Lesson 4 Q&A Review*  
*Next: Lesson 5 — Objects & Type Aliases*
