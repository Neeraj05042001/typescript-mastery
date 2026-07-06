# Lesson 8 — Q&A Reference
## Unions & Intersections

---

## Interview Questions

---

### Q1. What is a union type? What is an intersection type? What do `|` and `&` mean?

A **union type** (`|`) describes a value that can be exactly one of several
listed types. A **intersection type** (`&`) combines multiple types into
one, requiring a value to satisfy all of them simultaneously.

```typescript
type Role = "admin" | "moderator" | "developer"; // union — exactly one of these
type Product = Identifiable & Timestamped & { name: string }; // intersection — all combined
```

`|` means "accept any one of these." `&` means "must satisfy every one of
these at once." Type names are always PascalCase (`Role`, `Product`), and
intersection is best used to combine **facets of the same entity**
(`Identifiable & Timestamped & ProductFields`) rather than two unrelated
entities — a `Book` doesn't intersect with an `Author`; it *has* one as a
nested property.

---

### Q2. Why can't you call `.toUpperCase()` on a `string | number` without a check first?

```typescript
function printId(id: string | number): void {
  console.log(id.toUpperCase()); // ❌ does not compile
}
```

This is a **compile-time** rejection, not a runtime one. Without narrowing,
TypeScript's compiler refuses to type-check this code at all — there's no
"running it and discovering the mismatch," because the compiler already
knows `toUpperCase` doesn't exist on the `number` branch of the union, and
blocks it before any JavaScript is even produced.

```typescript
function printId(id: string | number): void {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // ✅ compiler now knows id is a string HERE
  } else {
    console.log(id.toFixed(2));
  }
}
```

The `typeof` check does genuinely execute at runtime — but its purpose
isn't to catch a live crash. Its purpose is to prove to the **static type
checker**, ahead of time, that within that branch the value can only be a
string, which is what makes the compiler allow `.toUpperCase()` there.
This is TypeScript's core value proposition: catching this category of
mistake before deployment, not in production.

---

### Q3. What makes a union "discriminated"? Why does the tag property matter?

A discriminated union is a union of object shapes that all share one
common property whose **value** (not type) differs across each shape —
called the discriminant, or tag.

```typescript
type SuccessResponse = { status: "success"; data: { id: number } };
type ErrorResponse = { status: "error"; message: string };
type ApiResponse = SuccessResponse | ErrorResponse;

function handle(response: ApiResponse): void {
  if (response.status === "success") {
    console.log(response.data.id); // narrowed to SuccessResponse
  } else {
    console.log(response.message); // narrowed to ErrorResponse
  }
}
```

Checking the tag at runtime (`response.status === "success"`) lets
TypeScript's compiler narrow the static type inside each branch, giving
you safe access to properties that only exist on that specific shape.

---

### Q4. Why does `string & number` become `never`? What happens with a conflicting shared property?

No value can be a `string` and a `number` at the same time — intersecting
two incompatible primitives produces `never`, a type with no possible
values:

```typescript
type Impossible = string & number; // never
```

The same collapse happens with object types when a shared property has
conflicting types across the two intersected shapes:

```typescript
type A = { status: "active" };
type B = { status: "inactive" };
type C = A & B;
// C.status must be "active" AND "inactive" simultaneously — impossible
// so C['status'] collapses to never, and C itself becomes unsatisfiable
```

---

### Q5. How would you design a type-safe API response using a discriminated union?

```typescript
type ApiSuccess<T> = {
  status: "success";
  data: T;
};

type ApiFailure = {
  status: "error";
  message: string;
};

type ApiResponse<T> = ApiSuccess<T> | ApiFailure;

function handleUserResponse(response: ApiResponse<{ id: number; name: string }>): void {
  if (response.status === "success") {
    console.log(response.data.name); // narrowed — safe
  } else {
    console.log(response.message);   // narrowed — safe
  }
}
```

The shared `status` field acts as the discriminant. Every caller narrows
on it before accessing shape-specific fields, so there's no risk of
reading `data` off a failure response or `message` off a success one.
(`<T>` is a generic — formally covered in a later lesson, but usable here
to make the response reusable across different data shapes.)

---

## Exercises

---

### Exercise 1. Declare union types

```typescript
// A variable holding either a Date or a raw string timestamp
let timestamp: Date | string;

// A restricted set of payment methods
type PaymentMethod = "card" | "upi" | "cash";

// A function parameter accepting a single ID or an array of IDs
type ProductIdInput = number | number[];
function getProducts(ids: ProductIdInput): void {
  // implementation
}
```

---

### Exercise 2. `formatDate` with correct narrowing

```typescript
function formatDate(date: Date | string): string {
  if (date instanceof Date) {
    return date.toLocaleDateString();
  }
  return date;
}
```

**Why `instanceof`, not `typeof`:** `typeof` only distinguishes JavaScript's
primitive categories (`"string"`, `"number"`, `"object"`, etc.) — it cannot
tell a `Date` apart from a plain object or an array, since both are
`"object"`. To check whether a value is an instance of a specific class,
use `instanceof`. And `Date.now()` is a **static** method on the `Date`
class returning the current timestamp as a number — it does not exist on
a `Date` instance. To format an existing instance, use a method like
`.toLocaleDateString()` or `.toISOString()`.

---

### Exercise 3. Fix the bugs

```typescript
// ✅ Fixed
function getDiscountLabel(discount: number | string): string {
  if (typeof discount === "number") {
    return discount.toFixed(2) + "% off";
  }
  return discount + "% off";
}

type ApiResponse = (SuccessResponse | ErrorResponse) & { timestamp: Date };
```

**Two distinct bugs:** `typeof discount === number` needed quotes around
`"number"` — `typeof` always returns a string, so it's always compared
against a string literal. Separately, the original `ApiResponse` type had
an operator precedence bug: `&` binds tighter than `|` in TypeScript,
so `SuccessResponse | ErrorResponse & { timestamp: Date }` actually parses
as `SuccessResponse | (ErrorResponse & { timestamp: Date })` — meaning only
the error branch would require a `timestamp`. Wrapping the union in
parentheses forces the intersection to apply to the whole union instead.

---

### Exercise 4. `PaymentResult` discriminated union

```typescript
type PaymentSuccess = { status: "success"; transactionId: string };
type PaymentFailure = { status: "failed"; reason: string };
type PaymentResult = PaymentSuccess | PaymentFailure;

function handlePayment(result: PaymentResult): void {
  if (result.status === "success") {
    console.log(result.transactionId);
    return;
  }
  console.log(result.reason);
}
```

**Key point:** the function is typed `void`, so both branches must log
rather than return a value — returning `result.reason` (a string) from a
`void`-typed function is a type error. Narrowing on `status` correctly
gives safe access to `transactionId` in one branch and `reason` in the
other.

---

### Exercise 5. `BlogPost` composition

```typescript
type Identifiable = { readonly id: number };
type Timestamped = { readonly createdAt: Date };
type BlogPost = Identifiable & Timestamped & {
  title: string;
  content: string;
};
```

A `BlogPost` now requires `id`, `createdAt`, `title`, and `content` —
composed from small, reusable, single-purpose pieces rather than one
large monolithic type.

---

### Exercise 6. What does this evaluate to, and why?

```typescript
type A = { status: "active" };
type B = { status: "inactive" };
type C = A & B;
```

`C['status']` evaluates to `never`. Intersecting `A` and `B` requires a
value's `status` to be `"active"` **and** `"inactive"` at the same time —
no string can satisfy both literal types simultaneously, so TypeScript
collapses that property to `never`. Since no value can ever provide a
valid `status`, `C` itself becomes an effectively unsatisfiable type — you
can declare a variable of type `C`, but you can never construct a valid
value for it.

---

## Key Mistakes to Remember

| Mistake | Wrong | Right |
|---------|-------|-------|
| `typeof` comparison | `typeof x === number` | `typeof x === "number"` |
| Checking for a class instance | `typeof date === Date` | `date instanceof Date` |
| Formatting an existing Date | `date.now()` | `date.toLocaleDateString()` |
| Type/operator precedence | `A \| B & C` | `(A \| B) & C` when both should require C |
| Returning from a `void` function | `return someValue;` | `console.log(someValue); return;` |
| Intersecting unrelated entities | `Book & Author` | `{ title: string; author: Author }` (nesting) |
| Narrowing reasoning | "runtime catches the crash" | "compiler refuses to compile without narrowing" |

---

## Terms from This Lesson

| Term | Meaning |
|------|---------|
| **Union (`\|`)** | A value that can be exactly one of several listed types |
| **Intersection (`&`)** | A value that must satisfy every combined type at once |
| **Narrowing** | Proving to the compiler, via a runtime check, which specific member of a union you're holding |
| **Discriminated union** | A union of object shapes sharing one literal "tag" property used to narrow between them |
| **`instanceof`** | Checks whether a value is an instance of a specific class — succeeds where `typeof` cannot distinguish object types |
| **`never` (intersection collapse)** | The result of intersecting incompatible types — a type with no possible values |
| **Operator precedence (`&` vs `\|`)** | Intersection binds tighter than union; use parentheses to control grouping explicitly |

---

*TypeScript Handbook — Lesson 8 Q&A Reference*
*Next: Lesson 9 — Literal Types & Type Narrowing*