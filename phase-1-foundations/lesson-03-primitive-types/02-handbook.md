# Lesson 3 — Primitive Types in TypeScript

## What is it?

A **type** tells TypeScript what kind of value a variable can hold.  
TypeScript has **11 primitive types** — the building blocks of every program you will write.

---

## Why do we need it?

JavaScript lets you assign anything to anything:

```javascript
let age = 25;
age = "twenty five"; // No error in JavaScript
age = true;          // Still no error
```

This causes silent bugs that only appear at runtime — meaning your users find them.  
TypeScript attaches a type to every variable and catches mismatches **before the code runs**.

---

## Problem it solves

- Catches type mismatches at compile time, not runtime
- Makes code self-documenting — types tell you what a variable holds
- Enables better IDE autocomplete and refactoring
- Prevents entire categories of bugs common in large JavaScript codebases

---

## The 11 Primitive Types — Quick Reference

| # | Type | What it holds | Example |
|---|------|--------------|---------|
| 1 | `string` | Text | `"Neeraj"` |
| 2 | `number` | All numbers | `25`, `99.99`, `-5` |
| 3 | `boolean` | True or false | `true`, `false` |
| 4 | `null` | Intentional absence | `null` |
| 5 | `undefined` | Not yet assigned | `undefined` |
| 6 | `bigint` | Very large integers | `9007199254740993n` |
| 7 | `symbol` | Unique identifiers | `Symbol("id")` |
| 8 | `any` | Anything — no checks | ⚠️ Avoid |
| 9 | `unknown` | Anything — but safe | ✅ Prefer over `any` |
| 10 | `never` | Value that never occurs | throw / infinite loop |
| 11 | `void` | No return value | Side-effect functions |

---

## Syntax

```typescript
let variableName: type = value;

// Examples
let name: string = "Neeraj";
let age: number = 25;
let isLoggedIn: boolean = false;
```

---

## Each Type in Detail

---

### 1. `string`

Any text — names, emails, messages, URLs, IDs.

```typescript
let name: string = "Neeraj";
let email: string = "neeraj@gmail.com";
let greeting: string = `Hello, ${name}`; // template literals work fine
```

**Common mistake:**
```typescript
// ❌ Capital S — this is the wrapper object, not the primitive
let name: String = "Neeraj";

// ✅ Always lowercase
let name: string = "Neeraj";
```

**Real-world — Node.js/Express:**
```typescript
function sendWelcomeEmail(email: string, name: string): void {
  console.log(`Sending to ${email}, Hello ${name}`);
}

sendWelcomeEmail("neeraj@gmail.com", "Neeraj"); // ✅
sendWelcomeEmail("neeraj@gmail.com", 42);        // ❌ Compile-time error
```

---

### 2. `number`

All numbers — integers, floats, negatives, hex, binary.

```typescript
let age: number = 25;
let price: number = 99.99;
let temperature: number = -5;
let hex: number = 0xFF;       // still a number
let binary: number = 0b1010;  // still a number
```

**Common mistake:**
```typescript
// ❌ Wrong
let age: Number = 25;   // wrapper object

// ✅ Correct
let age: number = 25;   // primitive
```

**Real-world:**
```typescript
function calculateDiscount(price: number, discountPercent: number): number {
  return price - (price * discountPercent) / 100;
}

calculateDiscount(1000, 10);     // ✅ Returns 900
calculateDiscount("1000", 10);   // ❌ Caught at compile time
```

---

### 3. `boolean`

Only `true` or `false`. Used for flags, toggles, conditions.

```typescript
let isLoggedIn: boolean = false;
let hasVerifiedEmail: boolean = true;
let isPremiumUser: boolean = false;
```

**Common mistake:**
```typescript
// ❌ Truthy values are NOT booleans
let isActive: boolean = 1;     // Error
let isActive: boolean = "yes"; // Error

// ✅ Correct
let isActive: boolean = true;
```

**Real-world — React props:**
```typescript
type ButtonProps = {
  disabled: boolean;
  loading: boolean;
};

<Button disabled={true} />   // ✅
<Button disabled="true" />   // ❌ TypeScript catches this
```

---

### 4. `null`

**Intentional absence of value.** Set by the developer on purpose.

```typescript
let selectedUser: string | null = null; // No user selected — intentional
```

**Real-world:**
```typescript
// No user logged in yet — we chose to represent this as null
let currentUser: string | null = null;

function login(username: string): void {
  currentUser = username;
}
```

---

### 5. `undefined`

**Declared but not yet assigned.** Set automatically by JavaScript.

```typescript
let middleName: string | undefined; // May not exist
let score: number | undefined;       // Not submitted yet
```

**null vs undefined — Key Distinction:**

| | `null` | `undefined` |
|---|---|---|
| Meaning | Intentionally empty | Not yet assigned |
| Set by | Developer | JavaScript (automatically) |
| Use case | User logged out | Value hasn't been given yet |

```typescript
let user: string | null = null;      // intentionally empty
let rating: number | undefined;       // hasn't been given yet
```

---

### 6. `bigint`

For integers larger than `Number.MAX_SAFE_INTEGER` (2^53 - 1).  
Used in financial systems, cryptography, large-scale IDs.

```typescript
let bigNum: bigint = 9007199254740993n;  // note the 'n' suffix
let another: bigint = BigInt(9007199254740993);
```

**Important rule:**
```typescript
// ❌ Cannot mix bigint and number
let result = 10n + 5;   // Error

// ✅ Keep them consistent
let result = 10n + 5n;  // Works
```

---

### 7. `symbol`

Creates a **guaranteed unique value** — even if two symbols have the same description.

```typescript
const id1 = Symbol("id");
const id2 = Symbol("id");

console.log(id1 === id2); // false — always unique
```

**Use case:** Unique object property keys — avoids name collisions in large codebases.

```typescript
const USER_ID = Symbol("userId");
const user = {
  [USER_ID]: 123,
  name: "Neeraj"
};
```

> Note: Beginners rarely write symbols directly. You'll encounter them in advanced library code and React internals.

---

### 8. `any`

TypeScript's escape hatch. Disables **all** type checking for that variable.

```typescript
let data: any = "hello";
data = 42;        // no error
data = true;      // no error
data.foo.bar.baz; // no error — even if this crashes at runtime
```

**Why it exists:**  
Migrating a large JavaScript codebase to TypeScript — you can't type everything at once.

**Why it's dangerous:**
```typescript
function processUser(user: any) {
  console.log(user.nmae); // Typo — but TypeScript won't catch it
}
```

**Rule:** Treat `any` as a **code smell**. Every `any` is a potential hidden bug.  
If you're tempted to use `any`, use `unknown` instead.

---

### 9. `unknown`

The **safe version of `any`**. Accepts any value — but TypeScript won't let you use it until you've checked its type.

```typescript
let input: unknown = "hello";

// ❌ Cannot use it directly
input.toUpperCase(); // Error — input is 'unknown'

// ✅ Must check type first (this is called a type guard)
if (typeof input === "string") {
  input.toUpperCase(); // ✅ Safe
}
```

**Real-world — API response:**
```typescript
async function fetchUser(url: string): Promise<unknown> {
  const response = await fetch(url);
  return response.json(); // Could be anything
}

const data = await fetchUser("/api/user");

// Must validate before using
if (typeof data === "object" && data !== null && "name" in data) {
  const user = data as { name: string };
  console.log(user.name); // ✅ Safe
}
```

**any vs unknown:**

| | `any` | `unknown` |
|---|---|---|
| Accepts any value | ✅ | ✅ |
| Can be used freely | ✅ | ❌ |
| Requires type check | ❌ | ✅ |
| Safe to use | ❌ | ✅ |

**Rule: Always prefer `unknown` over `any`.**

---

### 10. `never`

Represents a value that **literally never occurs**.  
Used for functions that never return — they either throw or loop forever.

```typescript
// Throws — never reaches a return point
function throwError(message: string): never {
  throw new Error(message);
}

// Loops forever — never returns
function infiniteLoop(): never {
  while (true) {}
}
```

**Real-world — Exhaustive type checking:**
```typescript
type Status = "active" | "inactive" | "banned";

function handleStatus(status: Status): string {
  if (status === "active") return "User is active";
  if (status === "inactive") return "User is inactive";
  if (status === "banned") return "User is banned";

  // If you add a new Status and forget to handle it,
  // TypeScript will error here — powerful safety net
  const exhaustiveCheck: never = status;
  return exhaustiveCheck;
}
```

**void vs never:**

| | `void` | `never` |
|---|---|---|
| Does the function return? | Yes — returns `undefined` | No — never returns |
| Example | `console.log()` | throws error / infinite loop |

---

### 11. `void`

A function that **returns nothing meaningful** — it produces a side effect.

```typescript
function logMessage(message: string): void {
  console.log(message);
  // No return statement needed — implicitly returns undefined
}
```

**Real-world — React event handlers:**
```typescript
function handleClick(event: React.MouseEvent): void {
  event.preventDefault();
  console.log("Clicked");
  // No return value needed
}
```

---

## Type Inference

TypeScript can often **infer** the type from the assigned value.  
You don't always need to write the type explicitly.

```typescript
let name = "Neeraj";  // inferred: string
let age = 25;          // inferred: number
let active = true;     // inferred: boolean
```

**When you MUST annotate explicitly:**

```typescript
// 1. Variable declared without a value
let username: string;  // must annotate — nothing to infer from
username = "Neeraj";

// 2. Function parameters — TypeScript can't infer what will be passed
function greet(name: string): void {  // must annotate
  console.log(name);
}

// 3. When inference is too broad for your needs
let status = "active"; 
// TypeScript infers: string
// You might want: "active" | "inactive" | "banned"
// Annotate explicitly to control it (covered in Union Types lesson)
```

---

## Common Mistakes — Summary

| Mistake | Wrong | Right |
|---------|-------|-------|
| Wrapper objects | `String`, `Number`, `Boolean` | `string`, `number`, `boolean` |
| Using `any` | `let x: any` | `let x: unknown` |
| typeof comparison | `typeof x === string` | `typeof x === "string"` |
| Bare null type | `let id: null = null` | `let id: string \| null = null` |
| Mixing bigint + number | `10n + 5` | `10n + 5n` |
| Incomplete type checks | Using `unknown` without checking | Always guard with `typeof` / `in` |

---

## Best Practices

1. Always use **lowercase** primitive types — `string`, `number`, `boolean`
2. **Never use `any`** in production code — use `unknown` instead
3. When using `unknown`, always **type guard** before accessing properties
4. Use `null` for intentional absence, `undefined` for not-yet-assigned
5. Name functions as **verbs** — they do things (`logUser`, not `user`)
6. Always write **complete code** — open braces must have closing braces
7. Let TypeScript infer when obvious, annotate when it adds clarity

---

## Interview Questions

**Beginner:**
- What is the difference between `null` and `undefined`?
- What does TypeScript do that JavaScript doesn't?
- What is type inference?

**Intermediate:**
- What is the difference between `any` and `unknown`?
- When would you use `unknown` over `any`?
- What is a type guard? Give an example.

**Advanced:**
- What is the `never` type used for in practice?
- How does `never` help with exhaustive type checking?
- What happens when TypeScript infers a type that is too wide?

**Ideal answers are in the lesson notes above.**

---

## Summary

- TypeScript has **11 primitive types** — each with a specific purpose
- Types are attached to variables, parameters, and return values
- TypeScript **infers** types from assigned values — but you must annotate when no value exists yet
- **`any`** disables type checking — avoid it
- **`unknown`** is safe — requires a type check before use
- **`void`** = function returns nothing; **`never`** = function never returns
- **`null`** = intentionally empty (developer sets it); **`undefined`** = not yet assigned (JS sets it)
- Always use **lowercase** primitive type names

---

## 2-Minute Revision Notes

```
string   → text → always lowercase
number   → all numbers (int + float) → lowercase
boolean  → true/false only → not 1/0
null     → intentional empty → set by developer
undefined → not yet assigned → set by JS automatically
bigint   → large integers → use 'n' suffix: 10n
symbol   → always unique → Symbol("id")
any      → ❌ disables all checks → avoid
unknown  → ✅ safe any → must type-check before use
never    → function that never returns (throws/loops)
void     → function returns nothing meaningful

typeof comparison → always use quotes → typeof x === "string"
Type inference → TS reads your value and figures out the type
Must annotate → when no value assigned, or for parameters
```

---

*TypeScript Handbook — Lesson 3 of 5 (Foundations Phase)*  
*Next: Lesson 4 — Arrays*
