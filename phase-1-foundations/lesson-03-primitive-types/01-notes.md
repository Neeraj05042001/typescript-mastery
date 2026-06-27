

# Lesson 3 — Primitive Types in TypeScript

## First, why do types even exist?

In JavaScript, you can do this:

```javascript
let age = 25;
age = "twenty five"; // JavaScript allows this. No error.
age = true;          // Still no error.
```

JavaScript doesn't care. It will let you assign anything to anything. This is fine for small scripts, but in a large production app with hundreds of files and dozens of developers, this becomes a **nightmare**. Bugs hide in plain sight, and you only discover them at runtime — which means your users discover them.

TypeScript solves this by **attaching a type to every value**. Once a variable is typed, it can only ever hold that kind of value. The compiler catches mistakes before your code ever runs.

---

## TypeScript has 11 primitive types

| # | Type | What it holds |
|---|------|--------------|
| 1 | `string` | Text |
| 2 | `number` | All numbers |
| 3 | `boolean` | true / false |
| 4 | `null` | Intentional absence of value |
| 5 | `undefined` | Declared but not yet assigned |
| 6 | `bigint` | Very large integers |
| 7 | `symbol` | Unique identifiers |
| 8 | `any` | Opt out of type checking |
| 9 | `unknown` | Safe version of `any` |
| 10 | `never` | A value that never occurs |
| 11 | `void` | Function that returns nothing |

Let's go through each one properly.

---

## 1. `string`

### What it is
Any text data — names, emails, messages, IDs, URLs.

### Syntax
```typescript
let name: string = "Neeraj";
let email: string = "neeraj@gmail.com";
let greeting: string = `Hello, ${name}`; // template literals work fine
```

### What TypeScript protects you from
```typescript
let username: string = "Neeraj";
username = 42; // ❌ Error: Type 'number' is not assignable to type 'string'
```

### Real-world usage in a Node/Express app
```typescript
function sendWelcomeEmail(email: string, name: string): void {
  console.log(`Sending email to ${email}, Hello ${name}`);
}

sendWelcomeEmail("neeraj@gmail.com", "Neeraj"); // ✅
sendWelcomeEmail("neeraj@gmail.com", 42);        // ❌ Caught at compile time
```

### Common mistake
```typescript
// ❌ Don't use String (capital S) — that's the wrapper object, not the primitive
let name: String = "Neeraj"; // works but is wrong practice

// ✅ Always use lowercase
let name: string = "Neeraj";
```

---

## 2. `number`

### What it is
TypeScript (like JavaScript) has only **one** number type. It covers integers, floats, negatives — everything.

```typescript
let age: number = 25;
let price: number = 99.99;
let temperature: number = -5;
let hex: number = 0xFF;     // hexadecimal — still a number
let binary: number = 0b1010; // binary — still a number
```

### Real-world usage
```typescript
function calculateDiscount(price: number, discountPercent: number): number {
  return price - (price * discountPercent) / 100;
}

const finalPrice = calculateDiscount(1000, 10); // ✅ Returns 900
calculateDiscount("1000", 10); // ❌ Error caught immediately
```

### Common mistake
```typescript
// ❌ Don't use Number (capital N)
let age: Number = 25;

// ✅ Use lowercase
let age: number = 25;
```

---

## 3. `boolean`

### What it is
Only two values: `true` or `false`. Used for flags, conditions, toggles.

```typescript
let isLoggedIn: boolean = false;
let hasVerifiedEmail: boolean = true;
let isPremiumUser: boolean = false;
```

### Real-world usage in React
```typescript
// A component prop
type ButtonProps = {
  disabled: boolean;
  loading: boolean;
};

// If someone passes a string "true", TypeScript catches it
<Button disabled="true" /> // ❌ Error — must be actual boolean
<Button disabled={true} />  // ✅
```

### Common mistake
```typescript
// ❌ Don't confuse truthy/falsy values with boolean type
let isActive: boolean = 1;     // ❌ Error — 1 is a number, not boolean
let isActive: boolean = "yes"; // ❌ Error

// ✅ Correct
let isActive: boolean = true;
```

---

## 4. `null`

### What it is
`null` means **intentional absence of a value**. You're explicitly saying "this has no value right now, and that is intentional."

```typescript
let selectedUser: null = null; // explicitly empty
```

### When you actually use it
In real apps, `null` is combined with other types (called a **union**, which we'll cover later):

```typescript
let currentUser: string | null = null; // No one logged in yet

function login(username: string) {
  currentUser = username; // Now someone is logged in
}
```

### Real-world: API responses
```typescript
// A user from a database might not exist
let user: { name: string } | null = null;

// After fetching:
user = { name: "Neeraj" }; // or it remains null if not found
```

---

## 5. `undefined`

### What it is
`undefined` means **a variable was declared but no value was assigned yet**. JavaScript sets this automatically.

```typescript
let score: undefined = undefined;
```

### null vs undefined — the key distinction

| | `null` | `undefined` |
|---|---|---|
| Meaning | Intentionally empty | Not yet assigned |
| Set by | You (the developer) | JavaScript (automatically) |
| Example | User logged out → set to null | Variable declared, not filled yet |

```typescript
let user: string | null = null;       // intentionally empty — no user
let middleName: string | undefined;    // might not exist at all
```

### Real-world usage in Express
```typescript
// Query params may or may not exist
function getUser(id: string | undefined) {
  if (id === undefined) {
    return "No ID provided";
  }
  return `Fetching user ${id}`;
}
```

---

## 6. `bigint`

### What it is
JavaScript's `number` type can only safely handle integers up to `2^53 - 1`. For anything bigger — financial systems, cryptography, large IDs — you need `bigint`.

```typescript
let bigNumber: bigint = 9007199254740993n; // note the 'n' at the end
let anotherBig: bigint = BigInt(9007199254740993);
```

### When you'd actually use this
```typescript
// Payment systems — handling large amounts in paise/cents
let transactionAmount: bigint = 100000000000n; // ₹1,000 crore in paise
```

### Important rule
```typescript
// ❌ You cannot mix bigint and number
let result = 10n + 5; // Error — cannot mix bigint and number

// ✅ Keep them consistent
let result = 10n + 5n; // Works
```

---

## 7. `symbol`

### What it is
A `symbol` creates a **completely unique value**, even if two symbols have the same description.

```typescript
const id1 = Symbol("id");
const id2 = Symbol("id");

console.log(id1 === id2); // false — they are always unique
```

### When you'd use it
Symbols are used as unique property keys — especially useful in large codebases where you don't want property name collisions.

```typescript
const USER_ID = Symbol("userId");

const user = {
  [USER_ID]: 123,
  name: "Neeraj"
};
```

In practice, beginners rarely write symbols directly. You'll encounter them when working with advanced library code, iterators, and React internals. But you need to recognise them.

---

## 8. `any`

### What it is
`any` is TypeScript's escape hatch. It turns **off** all type checking for that variable.

```typescript
let data: any = "hello";
data = 42;        // ✅ no error
data = true;      // ✅ no error
data = { x: 1 }; // ✅ no error
data.foo.bar.baz; // ✅ no error — even if this crashes at runtime
```

### Why it exists
When migrating a large JavaScript codebase to TypeScript, you can't type everything overnight. `any` lets you incrementally add types.

### The problem
`any` completely defeats the purpose of TypeScript. It's like wearing a seatbelt and then unbuckling it at the most dangerous turn.

```typescript
function processUser(user: any) {
  console.log(user.nmae); // ❌ Typo — but TypeScript won't catch it
}
```

### Rule of thumb
**Treat `any` as a code smell.** Every `any` in production code is a potential bug waiting to happen. Use it only when absolutely necessary, and replace it as soon as possible.

---

## 9. `unknown`

### What it is
`unknown` is the **safe version of `any`**. It also accepts any value, but TypeScript won't let you *use* it until you've checked what type it actually is.

```typescript
let input: unknown = "hello";

// ❌ Cannot use it directly
console.log(input.toUpperCase()); // Error — 'input' is of type 'unknown'

// ✅ Must check the type first
if (typeof input === "string") {
  console.log(input.toUpperCase()); // Now TypeScript is happy
}
```

### Why this matters — real-world example
```typescript
// API responses are unknown — you don't control what the server sends
async function fetchData(url: string): Promise<unknown> {
  const response = await fetch(url);
  return response.json(); // Could be anything
}

const data = await fetchData("/api/user");

// ❌ Can't just use it
console.log(data.name); // Error

// ✅ Must validate first
if (typeof data === "object" && data !== null && "name" in data) {
  console.log(data.name);
}
```

### `any` vs `unknown` — the critical difference

| | `any` | `unknown` |
|---|---|---|
| Accepts any value | ✅ | ✅ |
| Can be used freely | ✅ | ❌ |
| Requires type check before use | ❌ | ✅ |
| Safe to use | ❌ | ✅ |

**Rule: If you're tempted to use `any`, use `unknown` instead.**

---

## 10. `never`

### What it is
`never` represents a value that **literally never occurs**. It's the type of a function that never returns — either it throws an error or runs forever.

```typescript
// A function that always throws — it never returns normally
function throwError(message: string): never {
  throw new Error(message);
}

// A function that loops forever
function infiniteLoop(): never {
  while (true) {}
}
```

### Real-world usage — exhaustive checks
This is where `never` becomes genuinely powerful. It ensures you've handled every possible case:

```typescript
type Status = "active" | "inactive" | "banned";

function handleStatus(status: Status): string {
  if (status === "active") return "User is active";
  if (status === "inactive") return "User is inactive";
  if (status === "banned") return "User is banned";

  // If you add a new Status and forget to handle it,
  // TypeScript will catch it here
  const exhaustiveCheck: never = status; // ❌ Error if any case is missed
  return exhaustiveCheck;
}
```

`never` is mostly used in advanced patterns. For now, just understand what it means — a value that can never exist.

---

## 11. `void`

### What it is
`void` means a function **does not return a meaningful value**. It's the return type for functions that produce side effects (logging, writing to database, sending emails).

```typescript
function logMessage(message: string): void {
  console.log(message);
  // No return statement — or return; with nothing
}
```

### `void` vs `never` — the critical difference

| | `void` | `never` |
|---|---|---|
| Function returns? | Yes, but `undefined` | No — never returns |
| Example | `console.log()` | throws error, infinite loop |

```typescript
function log(msg: string): void {
  console.log(msg);
  // returns undefined implicitly — that's fine for void
}

function crash(msg: string): never {
  throw new Error(msg);
  // literally never returns
}
```

### Real-world in React
```typescript
// Event handlers return void — they DO things, they don't produce values
function handleClick(event: React.MouseEvent): void {
  event.preventDefault();
  console.log("Button clicked");
}
```

---

## Type Inference — TypeScript is smart

You don't have to always write the type explicitly. TypeScript can often **infer** it:

```typescript
let name = "Neeraj";   // TypeScript infers: string
let age = 25;           // TypeScript infers: number
let isActive = true;    // TypeScript infers: boolean
```

TypeScript reads the value you assign and figures out the type. The **explicit annotation is required** when:
- You declare a variable without assigning a value
- TypeScript can't figure it out from context
- You want to be explicit for clarity

```typescript
// Must annotate — no value to infer from
let username: string;

username = "Neeraj"; // ✅
username = 42;       // ❌ Error
```

---

## Now let's test your understanding

### Conceptual Questions — answer these in your own words

**Q1.** What is the difference between `null` and `undefined`? Give a real-world scenario for each.

**Q2.** Why is `unknown` safer than `any`? What does TypeScript force you to do before using an `unknown` value?

**Q3.** A function logs a message to the console and returns nothing. What return type should it have — `void` or `never`? Why?

**Q4.** TypeScript infers types. So when would you still explicitly annotate a type?

---

### Exercises — write these out

**Exercise 1.** Declare these variables with correct types:
- A user's full name
- A product price
- Whether a user is logged in
- A value that starts as null (representing "no user selected")
- A function that takes a username (string) and logs it, returning nothing

**Exercise 2.** What's wrong with this code? Fix it.
```typescript
let score: number = "100";
let isVerified: boolean = 1;
let email: String = "user@example.com";
```

**Exercise 3.** You're building an Express API. A request handler receives a body that could be anything (you don't trust the client). Should you type it as `any` or `unknown`? Write a small function that safely reads a `name` property from it.

---

