# Lesson 7 — Interfaces in TypeScript

## First, connect to what you already know

In Lesson 5 you learned to type the shape of an object using a **type alias**:

```typescript
type User = {
  name: string;
  age: number;
};
```

TypeScript gives you a **second way** to do exactly this — the `interface` keyword:

```typescript
interface User {
  name: string;
  age: number;
}
```

Both define the same shape. Both work the same way at the basic level. So why does TypeScript have two tools for the same job? Because interfaces have a few capabilities type aliases don't — and understanding exactly where they diverge is one of the most commonly tested things in TypeScript interviews.

---

## What is an interface?

An interface is a way to declare the **shape of an object or a class** — what properties and methods it must have. It's TypeScript's original, more "OOP-flavoured" way of describing object structure, borrowed conceptually from languages like Java and C#.

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
}

const laptop: Product = {
  id: 1,
  name: "Laptop",
  price: 50000,
};
```

Notice the syntax difference from a type alias — **no `=` sign**, and the body uses `{}` directly after the name.

```typescript
// Type alias — needs '='
type Product = { id: number; name: string; price: number };

// Interface — no '=', body directly after the name
interface Product { id: number; name: string; price: number }
```

---

## Optional and Readonly Properties — Same Syntax

Everything you learned in Lesson 5 carries over identically:

```typescript
interface User {
  readonly id: number;     // cannot be reassigned after creation
  name: string;
  email: string;
  phone?: string;          // optional — string | undefined
}

const user: User = { id: 1, name: "Neeraj", email: "neeraj@gmail.com" };
user.name = "Neeraj Kumar"; // ✅ allowed
user.id = 2;                 // ❌ Error — readonly
```

---

## Method Signatures

Interfaces commonly describe methods using a shorthand syntax:

```typescript
interface User {
  name: string;
  greet(): string;                  // method shorthand
  updateEmail(email: string): void; // method with a parameter
}

const user: User = {
  name: "Neeraj",
  greet() {
    return `Hello, I'm ${this.name}`;
  },
  updateEmail(email) {
    this.name = this.name; // example body
  },
};
```

This shorthand `greet(): string` is equivalent in meaning to `greet: () => string`. You'll see both styles in real codebases — shorthand is more common when an interface describes an object with behaviour (almost class-like), and the arrow-function style is more common for plain callback properties.

---

## The `extends` Keyword — Interface Inheritance

This is where interfaces start to genuinely differ from type aliases in *how* they express relationships. An interface can **extend** another interface, inheriting all its properties:

```typescript
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
}

interface User extends BaseEntity {
  name: string;
  email: string;
}

const user: User = {
  id: 1,
  createdAt: new Date(),
  name: "Neeraj",
  email: "neeraj@gmail.com",
};
```

`User` now requires everything from `BaseEntity` **plus** its own properties. This is an extremely common real-world pattern — almost every database-backed entity in a MERN app shares `id` and `createdAt`, so you define it once and extend it everywhere.

### Extending multiple interfaces

An interface can extend more than one interface at the same time, separated by commas:

```typescript
interface Identifiable {
  readonly id: number;
}

interface Auditable {
  readonly createdBy: string;
  readonly updatedBy: string;
}

interface Document extends Identifiable, Auditable {
  title: string;
  content: string;
}

const doc: Document = {
  id: 1,
  createdBy: "neeraj",
  updatedBy: "neeraj",
  title: "TypeScript Notes",
  content: "Interfaces are powerful",
};
```

### The type alias equivalent — intersection `&`

Type aliases achieve the same result using `&` instead of `extends`:

```typescript
type BaseEntity = {
  readonly id: number;
  readonly createdAt: Date;
};

type User = BaseEntity & {
  name: string;
  email: string;
};
```

Functionally very similar outcomes — but the syntax and mental model differ. `extends` reads as "this interface *is a* BaseEntity, plus more." `&` reads as "this type *is the combination* of these two shapes."

---

## Declaration Merging — Interface's Unique Superpower

This is something a type alias **cannot do at all**. If you declare two interfaces with the **same name** in the same scope, TypeScript automatically merges them into one:

```typescript
interface User {
  name: string;
}

interface User {
  age: number;
}

// TypeScript silently merges these into:
// interface User { name: string; age: number; }

const user: User = { name: "Neeraj", age: 25 }; // ✅ requires both
```

```typescript
// ❌ This would NOT work with type aliases — it's a hard error
type User = { name: string };
type User = { age: number }; // Error: Duplicate identifier 'User'
```

### Why this matters in real projects

Declaration merging is how you safely extend types you don't own — most commonly, **augmenting Express's `Request` type** to add custom properties like `req.user` after an authentication middleware runs:

```typescript
// A simplified preview — the full module augmentation syntax
// (using 'declare global') is covered properly in the Declaration Files lesson later

interface AuthenticatedRequest extends Request {
  user: { id: number; role: "admin" | "customer" };
}

function getDashboard(req: AuthenticatedRequest, res: Response): void {
  console.log(req.user.id); // ✅ TypeScript knows req.user exists
  res.json({ message: "Welcome" });
}
```

In production Express + TypeScript apps, this exact mechanic — interfaces merging onto library types — is how teams type `req.user`, augment the global `Window` object in the browser, or extend third-party library types without modifying the library's source code.

### The gotcha you need to watch for

Because merging happens silently and without error, an accidental duplicate interface name can quietly change what shape your code requires — and TypeScript won't warn you:

```typescript
interface Product {
  name: string;
  price: number;
}

// ... 300 lines later, in a different file or further down, someone
// reuses the name "Product" thinking it's a new, separate type:
interface Product {
  category: string;
}

// TypeScript doesn't error. It just merges them.
// Now every Product in your codebase requires name, price, AND category —
// which might not have been the intention at all.
```

**Lesson:** name your interfaces deliberately and avoid generic, reusable names across unrelated files unless merging is exactly what you want.

---

## Interface vs Type — The Full Comparison

This is one of the most frequently asked TypeScript interview questions. Here's the precise, structural truth:

| Feature | `interface` | `type` |
|---|---|---|
| Object / class shapes | ✅ | ✅ |
| Optional `?` / `readonly` | ✅ | ✅ |
| Unions (`"a" \| "b"`) | ❌ Cannot | ✅ |
| Primitives, tuples, arrays as aliases | ❌ Cannot | ✅ |
| Inheritance syntax | `extends` | `&` (intersection) |
| Declaration merging | ✅ Yes | ❌ No — causes an error |
| Used with `implements` on a class | ✅ Idiomatic | ✅ Works, less common |

### Practical guidance

```typescript
// ✅ Use 'type' — interfaces literally cannot express these
type ID = string | number;                          // union
type Status = "active" | "inactive" | "banned";      // literal union
type Coordinates = [number, number];                  // tuple
type Callback = (value: string) => void;              // function type

// ✅ Use 'interface' — when the shape might be extended or augmented later
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
}

interface User extends BaseEntity {
  name: string;
}
```

For a simple, one-off object shape that will never be extended or merged, either works fine — many teams pick one as a default for consistency across their codebase, and that's a perfectly valid convention. What matters is knowing *why* you'd reach for one over the other when the situation calls for it, rather than treating them as interchangeable by accident.

---

## Real-World Usage

### React — extending shared prop shapes

```typescript
interface BaseInputProps {
  name: string;
  placeholder?: string;
}

interface TextInputProps extends BaseInputProps {
  type: "text" | "email" | "password";
}

interface NumberInputProps extends BaseInputProps {
  min?: number;
  max?: number;
}

function TextInput({ name, placeholder, type }: TextInputProps) {
  return <input name={name} placeholder={placeholder} type={type} />;
}
```

This pattern — a shared base interface extended by specific variants — is exactly how component libraries like MUI or shadcn structure their prop types.

### Express — base entity pattern across models

```typescript
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
  readonly updatedAt: Date;
}

interface User extends BaseEntity {
  name: string;
  email: string;
}

interface Product extends BaseEntity {
  name: string;
  price: number;
}

interface Order extends BaseEntity {
  userId: number;
  productId: number;
  quantity: number;
}
```

Every model shares the same audit fields without repeating them. This is the standard structure you'll see in real Node.js/Express + TypeScript backends.

### A quick preview — `implements` with classes

Interfaces are also used to enforce a contract on **classes**, using the `implements` keyword:

```typescript
// A preview only — Classes aren't formally covered in our curriculum yet
interface Shape {
  calculateArea(): number;
}

class Circle implements Shape {
  constructor(private radius: number) {}
  calculateArea(): number {
    return Math.PI * this.radius ** 2;
  }
}
```

`implements` forces the class to provide everything the interface promises — if `Circle` forgot `calculateArea()`, TypeScript would error immediately.

---

## Common Mistakes

```typescript
// ❌ Mistake 1 — using '=' with interface (that's type alias syntax)
interface Product = { name: string }; // Error
// ✅ Fix
interface Product { name: string }

// ❌ Mistake 2 — trying to alias a union with interface
interface Status = "active" | "inactive"; // Error — invalid
// ✅ Fix — unions require type alias
type Status = "active" | "inactive";

// ❌ Mistake 3 — mixing '&' into an extends clause
interface DiscountedProduct extends Product & { discount: number } {} // Error
// ✅ Fix — use comma-separated extends, or switch to type + intersection
interface DiscountedProduct extends Product { discount: number }

// ❌ Mistake 4 — assuming readonly is deep
interface User { readonly address: { city: string } }
const user: User = { address: { city: "Delhi" } };
user.address.city = "Mumbai"; // ✅ Allowed! readonly only blocks reassigning 'address' itself
user.address = { city: "Pune" }; // ❌ Blocked
// ✅ Fix — mark nested properties readonly too if deep immutability matters

// ❌ Mistake 5 — accidental declaration merging from a reused name
interface Config { apiUrl: string }
// ...later, unintentionally reusing the same name
interface Config { timeout: number }
// Silently merges — Config now requires BOTH apiUrl and timeout
// ✅ Fix — use distinct, deliberate interface names
```

---

## Best Practices

1. **Use `interface`** for object and class shapes that may be extended or augmented later
2. **Use `type`** for unions, tuples, primitives, and function types — interfaces cannot express these
3. **Prefer `extends`** over duplicating shared fields across related interfaces
4. **Define a shared `BaseEntity`** for common fields (`id`, `createdAt`, `updatedAt`) across your data models
5. **Use deliberate, specific interface names** — avoid generic names that risk accidental merging
6. **Remember `readonly` is shallow** — wrap nested objects in `readonly` too if deep immutability is required
7. **Stay consistent** with whatever convention your team has already established

---

## Exercises — write all of these

**Exercise 1.** Convert this type alias into an interface with identical behaviour:
```typescript
type Employee = {
  id: number;
  name: string;
  department: string;
  salary?: number;
};
```

**Exercise 2.** Using `extends`, create:
- A `BaseEntity` interface with `readonly id: number` and `readonly createdAt: Date`
- An `Order` interface that extends `BaseEntity`, adding `customerName: string`, `totalAmount: number`, and `status: "pending" | "shipped" | "delivered"`

**Exercise 3.** Create two interfaces — `Identifiable` (`readonly id: number`) and `Auditable` (`readonly createdBy: string`, `readonly updatedBy: string`) — then a `Document` interface that extends **both**, adding `title: string` and `content: string`.

**Exercise 4.** Find and fix every bug in this code:
```typescript
interface Product = {
  name: string;
  price: number;
}

interface DiscountedProduct extends Product & { discount: number } {
}

const item: DiscountedProduct = {
  name: "Laptop",
  price: 50000
}
```

**Exercise 5.** Define an `AuthenticatedRequest` interface that extends Express's `Request` and adds a `user` property typed as `{ id: number; role: "admin" | "customer" }`. Then write a route handler function `getDashboard(req: AuthenticatedRequest, res: Response): void` that logs `req.user.id` and responds with `res.json({ message: "Welcome" })`.

---



