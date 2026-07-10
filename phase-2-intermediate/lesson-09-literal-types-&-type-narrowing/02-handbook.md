# Lesson 9 — Literal Types & Type Narrowing in TypeScript

## What is it?

A **literal type** represents not a general category of values, but one
exact value — `"active"` as a type means "this can only ever be the exact
text active."

**Narrowing** is the process by which TypeScript reduces a broad or union
type down to something more specific, within a certain scope, based on a
check you've written in your code.

```typescript
let status: "active";                        // literal type
type Status = "active" | "inactive" | "banned"; // union of literal types

function process(value: string | number): void {
  if (typeof value === "string") { /* narrowed to string here */ }
}
```

---

## Why do we need it?

### Why JavaScript needed literal types

String comparisons for state and status are everywhere in plain JS, and
nothing validates them:

```javascript
function setStatus(status) {
  // "active", "Active", "ACTIVE", "actve" — JavaScript accepts all of them
}
```

Literal types lock down the exact set of valid values, catching a typo or
invalid state at compile time instead of as a mystery bug later.

### Why JavaScript needed narrowing

You already write informal type checks in plain JS (`typeof`, `instanceof`)
to safely branch on a value that could be more than one thing. TypeScript's
narrowing formalizes this same pattern — the compiler recognizes these
checks and uses them to refine what it knows about a value's type inside
each branch.

---

## Problem it solves

- Literal types prevent "stringly typed" bugs — a function accepting any
  `string` when only a handful of specific values are actually meaningful
- Narrowing lets you safely access union-specific members without risking
  a runtime crash
- Exhaustiveness checking (narrowing + `never`) catches an unhandled case
  in a union **at compile time**, at the exact point a case goes missing

---

## Syntax

### Literal types and widening

```typescript
let x = "hello";   // inferred: string (widened — let can be reassigned)
const y = "hello";  // inferred: "hello" (the literal type — const can't change)
```

```typescript
function move(direction: "up" | "down"): void { /* ... */ }

let dir = "up";
move(dir); // ❌ Error — dir widened to string

const dir2 = "up";
move(dir2); // ✅ const kept the literal type
```

### `as const`

Widening also happens to properties inside an object, even a `const` one:

```typescript
const config = { status: "active" };
// config.status inferred as: string (widened)

const config2 = { status: "active" } as const;
// config2.status inferred as: "active" — as const locks every
// property to its literal type and makes the object readonly
```

A very common real-world pattern combining `as const` with a type query:

```typescript
const ROLES = ["admin", "editor", "viewer"] as const;
type Role = typeof ROLES[number]; // "admin" | "editor" | "viewer"
```

**Two meanings of `typeof`:**
- Runtime operator: `typeof x === "string"` — checks a value's type when the code runs
- Compile-time type query (inside a `type` position): `typeof someVariable` — extracts a variable's *type* for reuse elsewhere; never appears in the compiled JS output

---

## Examples — The Narrowing Toolkit

### 1. `typeof` narrowing
Best for distinguishing JS primitives.
```typescript
function process(value: string | number): void {
  if (typeof value === "string") { /* string here */ }
  else { /* number here */ }
}
```

### 2. Truthiness narrowing
Best for `null`/`undefined`. Gotcha: excludes **every** falsy value, not
just null/undefined — `0`, `""`, `NaN` also get filtered out.
```typescript
function displayCount(count: number | null): void {
  if (count) {
    console.log(count); // ⚠️ BUG — skips a legitimate 0
  }
}
// ✅ Fix
function displayCount(count: number | null): void {
  if (count !== null) {
    console.log(count);
  }
}
```

### 3. Equality narrowing
Comparing against a specific literal — literal types put to direct use.
```typescript
function check(value: "success" | "error" | "pending"): void {
  if (value === "success") { /* ... */ }
  else if (value === "error") { /* ... */ }
  else { /* pending */ }
}
```

### 4. `instanceof` narrowing
Best for class instances. `typeof` can only tell you `"object"` — it
cannot distinguish a `Date` from a plain object or array.
```typescript
function handle(error: Error | string): void {
  if (error instanceof Error) {
    console.log(error.message); // narrowed to Error
  } else {
    console.log(error); // narrowed to string
  }
}
```

### 5. The `in` operator
Best for object shapes without a shared discriminant tag — checks whether
a specific property exists.
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

### 6. `Array.isArray()`
Distinguishes an array from a single value in a union.
```typescript
function processIds(ids: number | number[]): void {
  if (Array.isArray(ids)) {
    ids.forEach(id => console.log(id)); // narrowed to number[]
  } else {
    console.log(ids); // narrowed to number
  }
}
```

### 7. Custom type guard functions
A function TypeScript recognizes as a narrowing check, via the return
type `value is Type`:
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
Lets you package complex, reusable narrowing logic into a named function
while TypeScript still narrows correctly at every call site.

---

### The capstone — exhaustive `switch` + `never`

This is where literal types, unions, narrowing, and `never` (Lesson 3)
combine into the single most valuable safety pattern in production
TypeScript:

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
```

If a teammate later adds a fourth shape — `{ kind: "hexagon"; ... }` — and
forgets to add a `case` for it, the `default` branch receives something
that isn't narrowed to nothing. Assigning it to `never` throws a compile
error: *"Type 'Hexagon' is not assignable to type 'never'."* The forgotten
case is caught the moment it's introduced — not months later as a runtime
bug. This turns "did I handle every case?" from a manual review question
into a guarantee the compiler enforces automatically.

---

## Real-world usage

### React — request state as a discriminated union
```typescript
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; message: string };
```

### Express — validating an untyped request body
```typescript
function handleWebhook(req: Request, res: Response): void {
  if ("email" in req.body && typeof req.body.email === "string") {
    // safe to use req.body.email here
  }
}
```

### Redux-style reducers
Every dispatched action is a discriminated union narrowed via `switch`,
almost always guarded by a `never` exhaustiveness check against unhandled
action types — the same pattern shown above, applied to state updates
instead of shape areas.

---

## Common mistakes

```typescript
// ❌ Truthiness narrowing filtering out valid falsy values
if (count) { /* skips a legitimate 0 */ }

// ❌ 'in' checking a property that doesn't actually differ between members
type A = { value: string; extra: string };
type B = { value: number; extra: string };
if ("extra" in x) { /* doesn't narrow anything — both have 'extra' */ }

// ❌ A custom type guard that lies about what it checks
function isCircle(shape: Circle | Square): shape is Circle {
  return true; // compiles fine — TypeScript trusts the claim, doesn't verify it
}

// ❌ Forgetting the never exhaustiveness check
switch (shape.kind) {
  case "circle": return ...;
  case "square": return ...;
  // no default, no never check — a forgotten "triangle" case silently falls through
}
```

---

## Best practices

1. Reach for discriminated unions + exhaustive `switch` + `never` for any
   union of more than two or three object shapes
2. Use `!== null` / `!== undefined` explicitly instead of truthiness
   checks when `0` or `""` are valid values
3. Extract complex narrowing logic into named, reusable custom type
   guard functions
4. Pick one discriminant property name (`kind`, `type`, `status`) and
   use it consistently across the whole codebase
5. A custom type guard's logic must genuinely correspond to its claim —
   TypeScript trusts `is Type` without verifying it

---

## Interview questions

**Beginner:**
- What is a literal type?
- Why does `let x = "a"` infer differently from `const x = "a"`?
- What does `as const` do?

**Intermediate:**
- What is the `in` operator used for in narrowing?
- What does `value is Type` mean in a function's return type?
- Why does truthiness narrowing sometimes cause bugs?

**Advanced:**
- How does exhaustiveness checking with `never` work, and why is it
  valuable in production code?
- What's the danger of an incorrectly implemented custom type guard?
- What is the difference between `typeof` as a runtime operator and
  `typeof` as a compile-time type query?

---

## Summary

- A **literal type** is a type representing one exact value — the
  building block behind every union you've written since Lesson 5
- `let` **widens** an inferred literal to its general type; `const`
  keeps the narrow literal type
- **`as const`** locks every property of an object or array to its
  literal type and makes it deeply readonly
- **Narrowing** is how TypeScript's compiler safely reduces a union
  down to a specific type inside a branch — the toolkit includes
  `typeof`, truthiness, equality, `instanceof`, `in`, `Array.isArray()`,
  and custom type guard functions (`value is Type`)
- **Exhaustiveness checking** — a `switch` over a discriminated union
  with a `default: const check: never = value` — is one of the most
  valuable patterns in real TypeScript, converting "did I handle every
  case" into a compiler-enforced guarantee

---

## 2-Minute Revision Notes

```
Literal type        → represents ONE exact value: "active", 42, true
let x = "a"          → widened to: string
const x = "a"         → stays: "a"
as const             → locks object/array properties to literal types

typeof (two meanings):
  runtime operator    → typeof x === "string"
  compile-time query  → type X = typeof someVariable

Narrowing toolkit:
  typeof        → primitives (string, number, boolean...)
  truthiness    → null/undefined — WATCH: also filters 0, "", NaN
  equality      → comparing to a specific literal (===)
  instanceof    → class instances (Date, Error, custom classes)
  in            → checking a property exists on the object
  Array.isArray → distinguishing an array from a single value
  custom guard  → function returning "value is Type" — reusable logic

Exhaustiveness check pattern:
  switch (x.kind) {
    case "a": ...
    case "b": ...
    default:
      const check: never = x;   // compile error if a case was missed
      return check;
  }

Rule of thumb: 3+ object shapes in a union → use discriminated union
  + switch + never, every time.
```

---

*TypeScript Handbook — Lesson 9 (Intermediate Phase)*
*Next: Lesson 10 — Generics*