# Lesson 9 — Literal Types & Type Narrowing

## Connecting to what you already know

You've been using pieces of this since Lesson 3 — `typeof` narrowing on `unknown`, string literal unions like `"active" | "inactive" | "banned"`, and the `instanceof` check we needed to fix your `formatDate` function in Lesson 8. Today we formalize all of it: what a literal type actually is, the "widening" trap that catches almost everyone once, and the *complete* narrowing toolkit — ending in the single most valuable safety pattern in production TypeScript.

---

## Part 1 — Literal Types

### What is it?

A literal type represents not a general category of values, but one **exact** value.

```typescript
let status: "active"; // this can ONLY ever be the exact text "active"
status = "active";    // ✅
status = "inactive";  // ❌ Error — not assignable to type "active"
```

On its own this looks pointlessly restrictive. Its real power shows up when you combine several literals into a union — exactly the pattern you've used since Lesson 5:

```typescript
type Status = "active" | "inactive" | "banned"; // a union of three literal types
```

### Why did JavaScript need this?

String comparisons for state and status are everywhere in plain JS, and nothing validates them:
```javascript
function setStatus(status) {
  // "active", "Active", "ACTIVE", "actve" — JavaScript accepts all of them
}
```
Literal types let you lock down the *exact* set of valid values, so a typo or an invalid state is caught at compile time instead of surfacing as a mystery bug.

### The widening problem — a genuinely common gotcha

```typescript
let x = "hello";   // TypeScript infers: string (widened!)
const y = "hello";  // TypeScript infers: "hello" (the literal type)
```

Why the difference? `let` can be reassigned, so TypeScript widens the inferred type to the general `string`, since something else might get assigned later. `const` can never be reassigned, so TypeScript is confident it will always be exactly `"hello"` and keeps the narrow literal type.

This has a real, practical consequence:
```typescript
function move(direction: "up" | "down"): void { /* ... */ }

let dir = "up";
move(dir); // ❌ Error! 'dir' widened to string — string isn't "up" | "down"

const dir2 = "up";
move(dir2); // ✅ works — const kept the literal type "up"
```
If you've ever wondered why the exact same value behaves differently depending on `let` vs `const`, this is why.

### `as const` — locking down objects and arrays too

Widening also happens to properties *inside* an object, even when the object itself is `const`:

```typescript
const config = { status: "active" };
// config.status is inferred as: string (widened!) — TypeScript assumes
// this property could be reassigned later, even though config itself can't be

const config2 = { status: "active" } as const;
// config2.status is inferred as: "active" — as const locks every
// property to its literal type and makes the whole object readonly
```

### A pattern you'll see constantly in real code

```typescript
const ROLES = ["admin", "editor", "viewer"] as const;
type Role = typeof ROLES[number]; // "admin" | "editor" | "viewer"
```

This gives you a real runtime array to loop over *and* a derived type from that exact same source — no duplication. Worth pausing on `typeof` here, because it's doing something different from every other use you've seen: at runtime, `typeof` is an operator checking a value's type (`typeof x === "string"`). Inside a `type` position like this, `typeof` is a **compile-time type query** — it extracts the *type* of a variable so you can reuse it elsewhere. It never appears in the compiled JavaScript output; it exists purely for the type checker. Same keyword, two completely different jobs depending on where it's written.

---

## Part 2 — Type Narrowing: The Complete Toolkit

Narrowing is the process by which TypeScript reduces a broad or union type down to something more specific, within a certain scope, based on a check you've written. You've used one form of this since Lesson 3. Here's the full toolkit.

### 1. `typeof` narrowing (recap)
Best for distinguishing JS primitives.
```typescript
function process(value: string | number): void {
  if (typeof value === "string") { /* string here */ }
  else { /* number here */ }
}
```

### 2. Truthiness narrowing
Best for handling `null`/`undefined`. One real gotcha: it excludes **every** falsy value, not just null/undefined — `0`, `""`, and `NaN` get filtered out too.
```typescript
function displayCount(count: number | null): void {
  if (count) {
    console.log(count); // ⚠️ BUG: if count is legitimately 0, this branch is skipped — 0 is falsy!
  }
}
// ✅ Fix — check explicitly against null when 0 is a valid value
function displayCount(count: number | null): void {
  if (count !== null) {
    console.log(count);
  }
}
```

### 3. Equality narrowing
Comparing against a specific literal — this is literal types (Part 1) put to direct use.
```typescript
function check(value: "success" | "error" | "pending"): void {
  if (value === "success") { /* ... */ }
  else if (value === "error") { /* ... */ }
  else { /* pending */ }
}
```

### 4. `instanceof` narrowing (recap)
Best for checking class instances. `typeof` can only tell you `"object"` — it can't distinguish a `Date` from a plain object or an array. `instanceof` can.
```typescript
function handle(error: Error | string): void {
  if (error instanceof Error) {
    console.log(error.message); // narrowed to Error
  } else {
    console.log(error); // narrowed to string
  }
}
```

### 5. The `in` operator — new
Best for object shapes that don't share a clean discriminant tag — checking whether a specific *property* exists.
```typescript
type Dog = { bark: () => void };
type Cat = { meow: () => void };

function makeSound(animal: Dog | Cat): void {
  if ("bark" in animal) {
    animal.bark(); // narrowed to Dog
  } else {
    animal.meow(); // narrowed to Cat
  }
}
```
Genuinely useful for third-party or legacy types where you don't control the shape and there's no convenient tag.

### 6. `Array.isArray()` — new
Distinguishes an array from a single value in a union — this is exactly the tool you needed (but didn't have yet) for the `number | number[]` type you declared back in Lesson 8, Exercise 1.
```typescript
function processIds(ids: number | number[]): void {
  if (Array.isArray(ids)) {
    ids.forEach(id => console.log(id)); // narrowed to number[]
  } else {
    console.log(ids); // narrowed to number
  }
}
```

### 7. Custom type guard functions — new, and powerful
A function TypeScript recognizes as performing a narrowing check, using the special return type `value is Type`:
```typescript
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; side: number };

function isCircle(shape: Circle | Square): shape is Circle {
  return shape.kind === "circle";
}

function getArea(shape: Circle | Square): number {
  if (isCircle(shape)) {
    return Math.PI * shape.radius ** 2; // narrowed to Circle
  }
  return shape.side ** 2; // narrowed to Square
}
```
The value: you can package *complex* narrowing logic — several checks, helper calls — into one reusable, named function, and TypeScript still narrows correctly at every call site.

---

### The capstone — discriminated unions with exhaustive `switch` and `never`

This is where literal types, unions, narrowing, and `never` (all the way back from Lesson 3) come together into the single most valuable safety pattern you'll use in real TypeScript:

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "triangle"; base: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      const exhaustiveCheck: never = shape;
      return exhaustiveCheck;
  }
}
```Here's why this matters so much in real codebases: if a teammate later adds a fourth shape — say `{ kind: "hexagon"; ... }` — and forgets to add a `case` for it, the `default` branch now receives something that *isn't* narrowed to nothing. Assigning it to `never` throws a compile error: *"Type 'Hexagon' is not assignable to type 'never'."* The forgotten case is caught at the exact point it was introduced, not three months later when a user hits a blank area of your UI. This converts "did I handle every case?" from a manual code-review question into a guarantee the compiler enforces for you.

---

## Real-World Usage

**React** — loading/success/error states as a discriminated union, exhaustively handled in render logic:
```typescript
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; message: string };
```

**Express** — validating an untyped request body before trusting its shape:
```typescript
function handleWebhook(req: Request, res: Response): void {
  if ("email" in req.body && typeof req.body.email === "string") {
    // safe to use req.body.email here
  }
}
```

**Redux-style state management** — every reducer action is a discriminated union narrowed via `switch`, almost always with a `never` exhaustiveness check guarding against unhandled action types.

---

## Common Mistakes

```typescript
// ❌ Truthiness narrowing filtering out valid falsy values
if (count) { /* skips a legitimate 0 */ }

// ❌ 'in' checking a property that doesn't actually differ between members
type A = { value: string; extra: string };
type B = { value: number; extra: string };
if ("extra" in x) { /* doesn't narrow anything — both have 'extra' */ }

// ❌ A custom type guard that lies about what it checks
function isCircle(shape: Circle | Square): shape is Circle {
  return true; // compiles fine — TypeScript trusts your claim, doesn't verify it
}

// ❌ Forgetting the never exhaustiveness check — new cases silently fall through
switch (shape.kind) {
  case "circle": return ...;
  case "square": return ...;
  // no default, no never check — a forgotten "triangle" case is now a silent bug
}
```

## Best Practices

1. Reach for discriminated unions + exhaustive `switch` + `never` for any union of more than two or three object shapes
2. Use `!== null` / `!== undefined` explicitly instead of truthiness checks when `0` or `""` are valid values
3. Extract complex narrowing logic into named, reusable custom type guard functions
4. Pick one discriminant property name (`kind`, `type`, `status`) and use it consistently across your whole codebase
5. A custom type guard's logic must actually correspond to its claim — TypeScript trusts `is Type` without verifying it

## Interview Questions

**Beginner:** What is a literal type? Why does `let x = "a"` infer differently from `const x = "a"`?

**Intermediate:** What does `as const` do? What is the `in` operator used for in narrowing? What does `value is Type` mean in a function's return type?

**Advanced:** How does exhaustiveness checking with `never` work, and why is it valuable in production code? What's the danger of an incorrectly implemented custom type guard?

---

## Exercises

**Exercise 1.** Explain what TypeScript infers for each, then fix the broken call:
```typescript
function setDirection(dir: "north" | "south" | "east" | "west"): void { /* ... */ }

let direction = "north";
setDirection(direction); // this fails — why, and what's the smallest fix?
```

**Exercise 2.** Use `as const` to fix this:
```typescript
const settings = { theme: "dark", version: 2 };
function applyTheme(theme: "dark" | "light"): void { /* ... */ }
applyTheme(settings.theme); // fails — fix it without changing applyTheme
```

**Exercise 3.** Write `getUserLabel(count: number | null): string` that returns `"No users"` if `count` is `null`, and `` `${count} users` `` otherwise — correctly, without the truthiness trap (count could legitimately be `0`).

**Exercise 4.** Write a custom type guard `isAdmin` for this union, then a function that uses it:
```typescript
type AdminUser = { role: "admin"; permissions: string[] };
type RegularUser = { role: "user"; email: string };
type AppUser = AdminUser | RegularUser;
```
`isAdmin` should narrow to `AdminUser`. `describeUser(user: AppUser): string` should return the permissions joined by a comma for admins, or the email otherwise.

**Exercise 5.** Given this union, write an exhaustive `getShippingCost` function using `switch` and a `never` exhaustiveness check:
```typescript
type ShippingMethod =
  | { type: "standard"; days: number }
  | { type: "express"; days: number }
  | { type: "overnight" };
```
(standard: ₹50, express: ₹150, overnight: ₹300 — flat rates, ignore `days`)

**Exercise 6.** Find and fix every bug:
```typescript
type Bird = { fly: () => void };
type Fish = { swim: () => void };

function move(animal: Bird | Fish) {
  if ("fly" in animal) {
    animal.swim();
  }
}
```

Take your time — write complete code for all six. 