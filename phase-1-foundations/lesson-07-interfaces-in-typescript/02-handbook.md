# Lesson 7 — Interfaces in TypeScript

## What is it?

An **interface** is a way to declare the shape of an object or class — what
properties and methods it must have. It serves the same core purpose as a
type alias for object shapes, but with its own syntax and a few unique
capabilities: `extends`-based inheritance and declaration merging.

---

## Why do we need it?

TypeScript already gives you type aliases for describing object shapes
(Lesson 5). Interfaces exist alongside them because they offer two things
type aliases structurally cannot:

- **`extends`** — clean, readable inheritance between object shapes
- **Declaration merging** — multiple declarations with the same name
  automatically combine into one, which is essential for safely extending
  types you don't own (like Express's `Request`)

---

## Problem it solves

- Lets you build shared base shapes (`BaseEntity`) and extend them across models
- Provides a clean way to add properties to external library types (e.g. `req.user`)
- Gives classes a contract to fulfil via `implements`
- Avoids repeating common fields (`id`, `createdAt`) across every entity

---

## Syntax

```typescript
// Type alias — needs '='
type Product = { id: number; name: string; price: number };

// Interface — no '=', body directly after the name
interface Product {
  id: number;
  name: string;
  price: number;
}
```

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

---

## Optional and Readonly Properties

Identical syntax and behaviour to type aliases (Lesson 5):

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

```typescript
interface User {
  name: string;
  greet(): string;                  // method shorthand syntax
  updateEmail(email: string): void; // method with a parameter
}

const user: User = {
  name: "Neeraj",
  greet() {
    return `Hello, I'm ${this.name}`;
  },
  updateEmail(email) {
    this.name = this.name;
  },
};
```

`greet(): string` (shorthand) and `greet: () => string` (arrow-function
property) describe the same kind of thing — shorthand is more common for
objects with class-like behaviour, the arrow style for plain callback props.

---

## The `extends` Keyword — Interface Inheritance

### Single inheritance

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

`User` now requires everything from `BaseEntity` plus its own fields.

### Extending multiple interfaces

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

`extends` reads as "this interface *is a* BaseEntity, plus more."
`&` reads as "this type *is the combination* of these shapes."
Outcomes are very similar; syntax and mental model differ.

---

## Declaration Merging — Unique to Interfaces

Two interfaces with the **same name** in the same scope automatically merge:

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
// ❌ Type aliases CANNOT do this — hard error
type User = { name: string };
type User = { age: number }; // Error: Duplicate identifier 'User'
```

### Real-world use — augmenting library types

```typescript
// Simplified preview — full module augmentation syntax ('declare global')
// is covered in the Declaration Files lesson later in the course

interface AuthenticatedRequest extends Request {
  user: { id: number; role: "admin" | "customer" };
}

function getDashboard(req: AuthenticatedRequest, res: Response): void {
  console.log(req.user.id); // ✅ TypeScript knows req.user exists
  res.json({ message: "Welcome" });
}
```

This mechanic — interfaces merging onto existing types — is how production
apps add `req.user` after auth middleware, or augment the global `Window`
object in the browser.

### The gotcha

Merging happens silently with no error — an accidental duplicate name can
quietly change what shape your code requires:

```typescript
interface Product {
  name: string;
  price: number;
}

// ... much later, a typo or copy-paste reuses the same name
interface Product {
  category: string;
}

// No error. Every Product now requires name, price, AND category —
// which may not have been intended.
```

**Rule:** use deliberate, specific interface names to avoid accidental merging.

---

## Interface vs Type — Full Comparison

| Feature | `interface` | `type` |
|---|---|---|
| Object / class shapes | ✅ | ✅ |
| Optional `?` / `readonly` | ✅ | ✅ |
| Unions (`"a" \| "b"`) | ❌ Cannot | ✅ |
| Primitives, tuples, arrays as aliases | ❌ Cannot | ✅ |
| Inheritance syntax | `extends` | `&` (intersection) |
| Declaration merging | ✅ Yes | ❌ No — error |
| Used with `implements` on a class | ✅ Idiomatic | ✅ Works, less common |

### Practical guidance

```typescript
// ✅ Use 'type' — interfaces literally cannot express these
type ID = string | number;
type Status = "active" | "inactive" | "banned";
type Coordinates = [number, number];
type Callback = (value: string) => void;

// ✅ Use 'interface' — shapes that may be extended or augmented later
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
}

interface User extends BaseEntity {
  name: string;
}
```

For a one-off object shape that will never be extended, either works —
many teams default to one for consistency. What matters is recognising
*when* a situation calls for one specifically.

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

### Preview — `implements` with classes

```typescript
// Classes aren't formally covered yet — preview only
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

`implements` forces the class to fulfil everything the interface promises.

---

## Common Mistakes

```typescript
// ❌ Mistake 1 — using '=' with interface
interface Product = { name: string }; // Error
// ✅ Fix
interface Product { name: string }

// ❌ Mistake 2 — using interface for a union
interface Status = "active" | "inactive"; // Error — invalid
// ✅ Fix — unions require type alias
type Status = "active" | "inactive";

// ❌ Mistake 3 — mixing '&' into an extends clause
interface DiscountedProduct extends Product & { discount: number } {} // Error
// ✅ Fix
interface DiscountedProduct extends Product { discount: number }

// ❌ Mistake 4 — assuming readonly is deep
interface User { readonly address: { city: string } }
const user: User = { address: { city: "Delhi" } };
user.address.city = "Mumbai"; // ✅ Allowed — readonly only blocks reassigning 'address'
user.address = { city: "Pune" }; // ❌ Blocked
// ✅ Fix — mark nested properties readonly too if deep immutability matters

// ❌ Mistake 5 — accidental declaration merging from a reused name
interface Config { apiUrl: string }
interface Config { timeout: number } // silently merges
// ✅ Fix — use distinct, deliberate interface names
```

---

## Best Practices

1. Use `interface` for object/class shapes that may be extended or augmented
2. Use `type` for unions, tuples, primitives, and function types
3. Prefer `extends` over duplicating shared fields across related interfaces
4. Define a shared `BaseEntity` for common fields (`id`, `createdAt`, `updatedAt`)
5. Use deliberate, specific interface names — avoid generic names that risk merging
6. Remember `readonly` is shallow — wrap nested objects too if deep immutability matters
7. Stay consistent with your team's established convention

---

## Interview Questions

**Beginner:**
- What is the basic syntax difference between an interface and a type alias?
- What does `extends` do for an interface?
- Can an interface have optional or readonly properties?

**Intermediate:**
- What is declaration merging? Give an example.
- What is the difference between `extends` (interface) and `&` (type)?
- Why can't an interface represent a union type?

**Advanced:**
- How is declaration merging used to augment Express's `Request` type in production?
- What is the practical risk of declaration merging? How do you avoid it?
- When would you choose `interface` over `type` in a real codebase, and why?

---

## Summary

- Interfaces declare the shape of an object or class — similar role to type aliases
- Syntax: no `=`, body directly after the interface name
- `?` and `readonly` work identically to type aliases
- `extends` provides clean inheritance — single or multiple base interfaces
- **Declaration merging** is unique to interfaces — same name in scope auto-combines
- Type aliases can do unions, tuples, primitives — interfaces cannot
- Interfaces are idiomatic for shapes that might be extended or augmented later
- `readonly` is shallow — nested objects need their own `readonly` for deep immutability

---

## 2-Minute Revision Notes

```
interface Name { ... }      → no '=', body directly after name
type Name = { ... }         → needs '='

? and readonly              → same syntax/behaviour as type aliases

extends (interface)         → interface User extends BaseEntity { ... }
extends multiple            → interface Doc extends A, B { ... }
& (type alias equivalent)   → type User = BaseEntity & { ... }

Declaration merging         → SAME interface name in scope = auto-merged
                             → type aliases CANNOT do this (hard error)
                             → used to safely extend library types (req.user)
                             → gotcha: silent merge on accidental duplicate name

interface vs type:
  unions, tuples, primitives    → type ONLY
  extends / merging             → interface ONLY
  simple object shape           → either works

readonly                    → SHALLOW only — nested objects need own readonly

implements (preview)        → class Circle implements Shape { ... }
                             → classes not yet formally covered — coming soon

Real-world pattern:
  interface BaseEntity { readonly id: number; readonly createdAt: Date }
  interface User extends BaseEntity { name: string; email: string }
```

---

*TypeScript Handbook — Lesson 7 of 7 (Foundations Phase Complete)*
*Next: Intermediate Phase — Unions & Intersections*