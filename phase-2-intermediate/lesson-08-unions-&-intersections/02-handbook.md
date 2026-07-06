# Lesson 8 — Unions & Intersections in TypeScript

## What is it?

A **union type** describes a value that can be exactly **one** of several
specified types, using the `|` operator.

An **intersection type** combines multiple types into one — a value must
satisfy **all** of them simultaneously, using the `&` operator.

```typescript
let id: string | number;              // union — either type
type Product = Identifiable & Named;  // intersection — both combined
```

---

## Why do we need it?

### Why JavaScript needed unions

You already write this pattern in plain JavaScript — just with runtime
`typeof` checks and no safety net:

```javascript
// Plain JavaScript — this pattern is everywhere
function formatId(id) {
  if (typeof id === "string") {
    return id.toUpperCase();
  }
  return id.toString();
}
```

Nothing stops a teammate from calling `formatId(true)`, or from someone
adding a new code path and forgetting to handle it. A union type takes
this exact "could legitimately be one of a few things" pattern and makes
it explicit and enforced by the compiler.

### Why JavaScript needed intersections

You already merge objects constantly in plain JavaScript:

```javascript
const defaults = { timeout: 5000, retries: 3 };
const userOptions = { timeout: 8000 };
const config = { ...defaults, ...userOptions }; // merged object
```

An intersection type gives you a precise way to describe the **type** of
that merged result — "this has everything from A *and* everything from B."

---

## Problem it solves

**Unions** — without them you have two bad options: use `any` and lose all
safety, or write near-duplicate functions for each possible type. Unions
let you say precisely "this can be exactly these things, nothing else."

**Intersections** — without them you'd either repeat shared fields across
many types, or build one giant, rigid base type trying to cover every case.
Intersections let you compose types from small, focused, reusable pieces.

---

## Syntax

### Union
```typescript
let id: string | number;

// more than two members works the same way
type Status = "pending" | "shipped" | "delivered" | "cancelled";

// unions of object shapes
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; side: number };
type Shape = Circle | Square;
```

### Intersection
```typescript
type Identifiable = { readonly id: number };
type Timestamped = { readonly createdAt: Date; readonly updatedAt: Date };
type Named = { name: string };

type Product = Identifiable & Timestamped & Named & {
  price: number;
};
// A Product must have id, createdAt, updatedAt, name, AND price — everything combined
```

---

## Examples

### The single most important rule about unions

You can only safely do what **every member** of the union supports:

```typescript
function printId(id: string | number): void {
  console.log(id.toUpperCase()); // ❌ Error — number has no toUpperCase
}
```

TypeScript blocks this because if `id` is a number at runtime, the call
would crash. It only allows operations valid for *every* type in the
union. This forces you to **narrow** first:

```typescript
function printId(id: string | number): void {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // ✅ TypeScript knows it's a string HERE
  } else {
    console.log(id.toFixed(2));    // ✅ TypeScript knows it's a number HERE
  }
}
```

This is the same `typeof` narrowing pattern used with `unknown` in Lesson 3 —
a union just narrows the possibilities to a small, specific set instead of
"could be absolutely anything." Full narrowing techniques (`in`, `instanceof`,
custom type guards, exhaustiveness checks) are covered in Lesson 9.

### Discriminated unions

When you union several object shapes that each share one common literal
property — a "tag" — you get a very powerful pattern:

```typescript
type SuccessResponse = {
  status: "success";
  data: { id: number; name: string };
};

type ErrorResponse = {
  status: "error";
  message: string;
};

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse): void {
  if (response.status === "success") {
    console.log(response.data.name); // TypeScript knows this is SuccessResponse here
  } else {
    console.log(response.message);   // TypeScript knows this is ErrorResponse here
  }
}
```

The shared `status` property is the discriminant — TypeScript uses it to
know exactly which shape you're holding inside each branch.

### The mental model — union vs intersection is genuinely inverted

- **Union (`|`)** — "this OR that." *More* values qualify, but you can
  safely do *less* with any one of them (only what's common to every
  member, until you narrow).
- **Intersection (`&`)** — "this AND that." *Fewer* values qualify (must
  satisfy everything at once), but once a value qualifies, you can do
  *more* with it — everything from every combined piece is available
  immediately, no narrowing needed.

```
UNION ( | ) — pick exactly one          INTERSECTION ( & ) — must satisfy both
┌──────────┐  ┌──────────┐              ┌─────────────┬────┬─────────────┐
│  string  │  │  number  │              │ Identifiable│████│ Timestamped │
└──────────┘  └──────────┘              └─────────────┴────┴─────────────┘
A value lives in exactly one box         A valid value must satisfy both boxes
```

### The intersection gotcha — incompatible types collapse to `never`

```typescript
type Impossible = string & number;
// No value can be both a string and a number at once — this becomes 'never'
```

This connects straight back to Lesson 3 — `never` is a type with no
possible values. It also happens with object types, whenever a shared
property has conflicting types:

```typescript
type A = { value: string };
type B = { value: number };
type C = A & B;
// C.value must satisfy BOTH string AND number at once — impossible
// So C['value'] silently collapses to 'never'

const c: C = { value: "hello" }; // ❌ Error
```

---

## Real-world usage

### React — combining custom props with native HTML attributes

```typescript
type ButtonProps = {
  label: string;
  onClick: () => void;
} & React.ButtonHTMLAttributes<HTMLButtonElement>;

function Button({ label, onClick, ...rest }: ButtonProps) {
  return <button onClick={onClick} {...rest}>{label}</button>;
}
// Button now accepts your custom props AND every native button attribute
// (disabled, type, className) without redeclaring any of them
```

### Express/Node — a discriminated union for consistent API results

```typescript
type ApiSuccess<T> = { success: true; data: T };
type ApiFailure = { success: false; error: string };
type ApiResult<T> = ApiSuccess<T> | ApiFailure;
// <T> is a generic — a small preview of Lesson 11

function sendUserResponse(
  res: Response,
  result: ApiResult<{ id: number; name: string }>
): void {
  if (result.success) {
    res.status(200).json(result.data);
  } else {
    res.status(400).json({ message: result.error });
  }
}
```

### Entity composition — the BaseEntity pattern from Lesson 7, generalized

```typescript
type Identifiable = { readonly id: number };
type Timestamped = { readonly createdAt: Date; readonly updatedAt: Date };

type Order = Identifiable & Timestamped & {
  customerId: number;
  totalAmount: number;
};
```

### Industry usage

Redux Toolkit types every dispatched action as a discriminated union tagged
by a `type` field. Stripe's TypeScript SDK models API responses the same
way, tagged by `status`. React's own type definitions rely on intersections
constantly — it's exactly why `<Button disabled />` works even though you
never explicitly typed `disabled` yourself.

---

## Common mistakes

```typescript
// ❌ Using a union member's specific property without narrowing
function getLength(value: string | number) {
  return value.length; // Error — number has no .length
}

// ❌ Forgetting a discriminant when designing a union of object shapes
type Shape = { radius: number } | { side: number };
// no shared "kind" field — narrowing becomes clumsy without one

// ❌ Confusing which operator does what
type Wrong = User | { role: string }; // meant "combine", wrote "either/or"
type AlsoWrong = User & AdminUser;    // meant "one of these", wrote "must be both"

// ❌ Accidentally producing 'never' via a conflicting intersected property
type A = { status: "active" };
type B = { status: "inactive" };
type C = A & B; // status becomes "active" & "inactive" → never
```

---

## Best practices

1. Always give a union of object shapes a shared literal "tag" property —
   that's what makes it a *discriminated* union and lets you narrow cleanly
2. Keep intersected building blocks small and single-purpose
   (`Identifiable`, `Timestamped`) rather than one large, rigid base type
3. Narrow a union before using anything beyond what's common to every member
4. Watch for accidental `never` when two intersected types share a
   property name with incompatible types
5. Reach for a union when a value is legitimately one of several distinct
   things; reach for an intersection when composing several required
   pieces into one whole

---

## Interview questions

**Beginner:**
- What is a union type? What is an intersection type?
- What do `|` and `&` mean in a type declaration?
- Can you access any property directly on a union-typed value?

**Intermediate:**
- Why can't you call `.toUpperCase()` on a `string | number` without a
  check first?
- What makes a union "discriminated"? Why does the tag property matter?
- What is the difference in behaviour between narrowing a union and using
  an intersection?

**Advanced:**
- Why does `string & number` become `never`?
- What happens when two intersected object types share a property with
  incompatible types?
- How would you design a type-safe API response shape using a
  discriminated union?

---

## Summary

- A **union (`|`)** means a value can be exactly one of several types —
  more values qualify, but only shared operations are safe without narrowing
- An **intersection (`&`)** means a value must satisfy every combined type
  at once — fewer values qualify, but everything combined is safely
  accessible once a value qualifies
- Unions must be **narrowed** (commonly with `typeof`) before you use
  anything beyond what's common to every member
- A **discriminated union** — object shapes sharing one literal "tag"
  property — is one of the most-used patterns in real TypeScript, powering
  API responses, Redux actions, and UI states
- Intersecting **incompatible types collapses to `never`** — a classic
  interview trap, worth remembering precisely
- Unions and intersections are the foundation for generics, utility types,
  and discriminated-union narrowing covered later in the curriculum

---

## 2-Minute Revision Notes

```
Union ( | )        → value is EXACTLY ONE of the listed types
Intersection ( & ) → value must satisfy ALL of the combined types

Union safety rule  → can only use what's common to EVERY member
                    → must narrow (typeof, in, instanceof) before more

Discriminated union → object shapes + one shared literal "tag" property
                    → e.g. { status: "success" | "error" }
                    → tag lets TypeScript narrow inside if/else branches

Mental model (inverted!):
  Union         → MORE values qualify, LESS safe to do without narrowing
  Intersection  → FEWER values qualify, MORE safe to do once qualified

never trap        → string & number = never (no value can be both)
                   → conflicting property types in intersected objects
                     also collapse that property to never

React             → ButtonProps = CustomProps & NativeHTMLAttributes
Express           → ApiResult<T> = ApiSuccess<T> | ApiFailure (discriminated)
Entities          → Order = Identifiable & Timestamped & { ...specific fields }

Rule of thumb:
  "one of several distinct things"  → union
  "composing required pieces"       → intersection
```

---

*TypeScript Handbook — Lesson 8 (Intermediate Phase)*
*Next: Lesson 9 — Literal Types & Type Narrowing*