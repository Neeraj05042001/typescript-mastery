# Lesson 8 — Unions & Intersections

Think about a login form. A user can sign in using their **email** or their **phone number** — just one of the two. Depending on which one they gave, you validate it differently: an email needs an `@`, a phone number needs 10 digits. You must figure out _which one you actually got_ before you can safely check it.

Now think about a job application system. To be marked complete, it needs a resume **and** a filled contact form **and** a signed consent form — all three, together, at once. Missing any one means the whole application is invalid.

The first is a **union**. The second is an **intersection**. Today we formalize both.

---

## Connecting to what you already know

You've been using both of these informally since Lesson 4, without the formal name attached:

```typescript
(string | number)[]                              // Lesson 4 — a union, inside an array
type Status = "pending" | "shipped" | "delivered"; // Lesson 5 — a union of literals
type User = BaseEntity & { name: string };         // Lesson 7 — an intersection
```

Today we slow down and understand exactly what `|` and `&` mean, why JavaScript needed them, and — critically — the two gotchas that trip up almost every developer the first time: **narrowing** (for unions) and **the `never` trap** (for intersections).

---

## Part 1 — Union Types

### What is it?

A union type describes a value that can be **exactly one** of several specified types, using the `|` operator.

```typescript
let id: string | number;
id = "abc123"; // ✅
id = 42; // ✅
id = true; // ❌ boolean isn't part of the union
```

### Why did JavaScript need this?

You already write this pattern in plain JavaScript — you just do it with runtime `typeof` checks and no safety net:

```javascript
// Plain JavaScript — this is everywhere in real codebases
function formatId(id) {
  if (typeof id === "string") {
    return id.toUpperCase();
  }
  return id.toString();
}
```

This works, but nothing stops a teammate from calling `formatId(true)`, or from someone adding a new code path and forgetting to handle it. TypeScript's union type takes this exact pattern — a value that could legitimately be one of a few things — and makes it explicit and enforced. You declare up front "this can only be a string or a number," and the compiler makes sure every path that touches it accounts for both.

### What problem does it solve?

Without unions, you have two bad options: use `any` and lose all safety, or write near-duplicate functions for each possible type. Unions let you say precisely "this can be exactly these things, nothing else" — keeping safety while allowing legitimate flexibility.

### Real-world motivation

A product's ID might come from MongoDB as a string `ObjectId` in one part of your system, and from a legacy SQL table as an auto-incrementing `number` in another. A function that needs to handle both, safely, needs a union.

### Syntax

```typescript
let id: string | number;

// more than two members works the same way
type Status = "pending" | "shipped" | "delivered" | "cancelled";

// unions of object shapes
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; side: number };
type Shape = Circle | Square;
```

### The single most important rule about unions

You can only safely do what **every member** of the union supports:

```typescript
function printId(id: string | number): void {
  console.log(id.toUpperCase()); // ❌ Error — number has no toUpperCase
}
```

TypeScript blocks this because if `id` happens to be a number at runtime, this call would crash. It only allows operations valid for _every_ type in the union. This forces you to **narrow** first:

```typescript
function printId(id: string | number): void {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // ✅ TypeScript knows it's a string HERE
  } else {
    console.log(id.toFixed(2)); // ✅ TypeScript knows it's a number HERE
  }
}
```

This is the exact `typeof` narrowing pattern you already used with `unknown` back in Lesson 3 — a union just narrows the possibilities down to a small, specific set instead of "could be absolutely anything." We'll go much deeper on narrowing techniques — `in`, `instanceof`, custom type guard functions, exhaustiveness checks — in the dedicated Lesson 9. For now, lock in this one rule: **a union must be narrowed before you use anything beyond what's common to every member.**

### A preview worth knowing now — discriminated unions

When you union several object shapes that each share one common literal property — a "tag" — you get something genuinely powerful:

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
    console.log(response.message); // TypeScript knows this is ErrorResponse here
  }
}
```

The shared `status` property is the discriminant — TypeScript uses it to know exactly which shape you're holding inside each branch. This is one of the most-used patterns in real production TypeScript.

### Industry usage

Redux Toolkit types every dispatched action this way — a union of action shapes discriminated by a `type` field. Stripe's TypeScript SDK models API responses as a union of success/error shapes discriminated by a `status` field. You'll see this pattern constantly once you start looking for it.

---

## Part 2 — Intersection Types

### What is it?

An intersection type combines multiple types into one — a value must satisfy **all** of them simultaneously, using the `&` operator.

### Why did JavaScript need this?

You already merge objects constantly in plain JavaScript:

```javascript
const defaults = { timeout: 5000, retries: 3 };
const userOptions = { timeout: 8000 };
const config = { ...defaults, ...userOptions }; // merged object
```

TypeScript's intersection type gives you a precise way to describe the **type** of that merged result — "this has everything from A _and_ everything from B."

### What problem does it solve?

Without intersections, you'd either repeat shared fields across many types, or build one giant, rigid base type trying to cover every case. Intersections let you compose types from small, focused, reusable building blocks — exactly what you did with `BaseEntity` in Lesson 7, generalized.

### Syntax

```typescript
type Identifiable = { readonly id: number };
type Timestamped = { readonly createdAt: Date; readonly updatedAt: Date };
type Named = { name: string };

type Product = Identifiable &
  Timestamped &
  Named & {
    price: number;
  };
// A Product must have id, createdAt, updatedAt, name, AND price — everything, combined.
```

### The mental model that trips almost everyone up the first time

- **Union (`|`)** — "this OR that." _More_ values qualify, but you can safely do _less_ with any one of them (only what's common to every member, until you narrow).
- **Intersection (`&`)** — "this AND that." _Fewer_ values qualify (must satisfy everything at once), but once a value qualifies, you can do _more_ with it — everything from every combined piece is available immediately, no narrowing needed.

It's genuinely inverted. Let me make it concrete.

![unions&intersections](/assets/unions-intersections.png)

### The intersection gotcha — incompatible types collapse to `never`

```typescript
type Impossible = string & number;
// No value can be both a string and a number at once — this becomes 'never'
```

This connects straight back to Lesson 3 — `never` is a type with no possible values. It's not only a primitive trap either. It happens with object types too, whenever a shared property has conflicting types:

```typescript
type A = { value: string };
type B = { value: number };
type C = A & B;
// C.value must satisfy BOTH string AND number at once — impossible for any real value
// So C['value'] silently collapses to 'never'

const c: C = { value: "hello" }; // ❌ Error
```

This is a genuine, common interview trap. Remember it precisely.

---

## Real-World Usage

**React** — combining custom props with native HTML attributes:

```typescript
type ButtonProps = {
  label: string;
  onClick: () => void;
} & React.ButtonHTMLAttributes<HTMLButtonElement>;

function Button({ label, onClick, ...rest }: ButtonProps) {
  return <button onClick={onClick} {...rest}>{label}</button>;
}
// Button now accepts your custom props AND every native button attribute
// (disabled, type, className) without you redeclaring any of them
```

**Express/Node** — a discriminated union for consistent API results:

```typescript
type ApiSuccess<T> = { success: true; data: T };
type ApiFailure = { success: false; error: string };
type ApiResult<T> = ApiSuccess<T> | ApiFailure;
// the <T> is a generic — a small preview of Lesson 11, don't worry about it yet

function sendUserResponse(
  res: Response,
  result: ApiResult<{ id: number; name: string }>,
): void {
  if (result.success) {
    res.status(200).json(result.data);
  } else {
    res.status(400).json({ message: result.error });
  }
}
```

**Entity composition** — the `BaseEntity` pattern from Lesson 7, generalized into small reusable pieces:

```typescript
type Identifiable = { readonly id: number };
type Timestamped = { readonly createdAt: Date; readonly updatedAt: Date };

type Order = Identifiable &
  Timestamped & {
    customerId: number;
    totalAmount: number;
  };
```

### Industry usage

This exact composition pattern — small, single-purpose types combined with `&` — is how most large TypeScript codebases build their entity layer (Prisma and TypeORM-adjacent code both lean on it). React's own type definitions use intersections constantly to let your components accept native DOM attributes alongside custom props, which is why `<Button disabled />` just works even though you never explicitly typed `disabled`.

---

## Common Mistakes

```typescript
// ❌ Using a union member's specific property without narrowing
function getLength(value: string | number) {
  return value.length; // Error — number has no .length
}

// ❌ Forgetting a discriminant when designing a union of object shapes
type Shape = { radius: number } | { side: number };
// no shared "kind" field — narrowing becomes clumsy; you'd have to check
// for the PRESENCE of a property instead of comparing one clean tag value

// ❌ Confusing which operator does what
type Wrong = User | { role: string }; // meant "combine", wrote "either/or"
type AlsoWrong = User & AdminUser; // meant "one of these", wrote "must be both"

// ❌ Accidentally producing 'never' via a conflicting intersected property
type A = { status: "active" };
type B = { status: "inactive" };
type C = A & B; // status becomes "active" & "inactive" → never
```

## Best Practices

1. Always give a union of object shapes a shared literal "tag" property — that's what makes it a _discriminated_ union and lets you narrow cleanly
2. Keep intersected building blocks small and single-purpose (`Identifiable`, `Timestamped`) rather than one large, rigid base type
3. Narrow a union before using anything beyond what's common to every member
4. Watch for accidental `never` when two intersected types share a property name with incompatible types
5. Reach for a union when a value is legitimately one of several distinct things; reach for an intersection when you're composing several required pieces into one whole

## Interview Questions

**Beginner:** What is a union type? What is an intersection type? What do `|` and `&` mean?

**Intermediate:** Why can't you call `.toUpperCase()` on a `string | number` without a check first? What makes a union "discriminated," and why does the tag property matter?

**Advanced:** Why does `string & number` become `never`? What happens when two intersected object types share a property with incompatible types? How would you design a type-safe API response using a discriminated union?

---

## Exercises

**Exercise 1.** Declare these with correct union types:

- A variable that can hold either a `Date` object or a raw string timestamp
- A type alias `PaymentMethod` restricted to `"card"`, `"upi"`, or `"cash"`
- A function parameter that accepts either a single product ID (`number`) or an array of product IDs (`number[]`)

**Exercise 2.** Write a function `formatDate(date: Date | string): string` that returns a readable date string regardless of which type was passed in — narrow correctly before using any type-specific method.

**Exercise 3.** Find and fix every bug:

```typescript
function getDiscountLabel(discount: number | string) {
  return discount.toFixed(2) + "% off";
}

type ApiResponse = SuccessResponse | (ErrorResponse & { timestamp: Date });
```

**Exercise 4.** Design a `PaymentResult` discriminated union with two shapes:

- `{ status: "success"; transactionId: string }`
- `{ status: "failed"; reason: string }`

Then write `handlePayment(result: PaymentResult): void` that logs the transaction ID on success or the reason on failure.

**Exercise 5.** Compose a `BlogPost` type from three small pieces: `Identifiable` (`readonly id: number`), `Timestamped` (`readonly createdAt: Date`), and a shape with `title: string` and `content: string`.

**Exercise 6.** No code — just explain: what does this evaluate to, and why?

```typescript
type A = { status: "active" };
type B = { status: "inactive" };
type C = A & B;
```

Take your time. Write complete code, and verify with the solutions in `solutions.md`
