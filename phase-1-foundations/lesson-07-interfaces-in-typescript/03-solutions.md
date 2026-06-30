# Lesson 7 — Q&A Reference
## Interfaces in TypeScript

---

## Conceptual Questions

---

### Q1. What is an interface in TypeScript? How is its syntax different from a type alias?

An interface declares the **shape of an object or class** — what properties and
methods it must have.

```typescript
// Type alias — needs '='
type Product = {
  id: number;
  name: string;
  price: number;
};

// Interface — no '=', body directly after the name
interface Product {
  id: number;
  name: string;
  price: number;
}
```

Both enforce the same shape at compile time. The key differences are in
what each can do beyond basic object typing.

---

### Q2. What does `extends` do for an interface?

`extends` makes an interface **inherit all properties** from another interface,
plus add its own:

```typescript
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
}

interface User extends BaseEntity {
  name: string;
  email: string;
}

// User now requires: id, createdAt, name, and email
const user: User = {
  id: 1,
  createdAt: new Date(),
  name: "Neeraj",
  email: "neeraj@gmail.com",
};
```

An interface can also extend **multiple** interfaces at once using commas:

```typescript
interface Document extends Identifiable, Auditable {
  title: string;
  content: string;
}
```

---

### Q3. What is declaration merging? Why is it useful and what is the risk?

When two interfaces share the same name in the same scope, TypeScript
**automatically merges** them into one — silently, with no error:

```typescript
interface User {
  name: string;
}

interface User {
  age: number;
}

// TypeScript treats these as one:
// interface User { name: string; age: number; }

const user: User = { name: "Neeraj", age: 25 }; // both required
```

Type aliases **cannot** do this:
```typescript
type User = { name: string };
type User = { age: number }; // ❌ Error: Duplicate identifier 'User'
```

**Why it is useful:**
Declaration merging is how you safely add properties to types you don't own —
most commonly, adding `req.user` to Express's `Request` type after
authentication middleware:

```typescript
interface AuthenticatedRequest extends Request {
  user: { id: number; role: "admin" | "customer" };
}

function getDashboard(req: AuthenticatedRequest, res: Response): void {
  console.log(req.user.id); // ✅ TypeScript knows this exists
  res.json({ message: "Welcome" });
}
```

**The risk:**
Merging is silent. An accidentally reused interface name quietly changes what
shape your code requires across the entire codebase — and TypeScript won't warn you:

```typescript
interface Product { name: string; price: number }
// ...later in a different file
interface Product { category: string }
// Silently merged — Product now requires ALL THREE fields
```

**Rule:** Use deliberate, specific interface names to avoid accidental merging.

---

### Q4. What is the difference between `extends` (interface) and `&` (type alias)?

Both combine shapes from multiple types — but the syntax and mental model differ:

```typescript
// Interface — extends reads as "is a BaseEntity, plus more"
interface User extends BaseEntity {
  name: string;
}

// Type alias — & reads as "the combination of these shapes"
type User = BaseEntity & {
  name: string;
};
```

Outcomes are functionally very similar. The difference matters when:
- You want declaration merging — use `interface`
- You want to combine with a union or primitive — use `type` with `&`

---

### Q5. What are the key differences between `interface` and `type`?

| Feature | `interface` | `type` |
|---|---|---|
| Object / class shapes | ✅ | ✅ |
| Optional `?` / `readonly` | ✅ | ✅ |
| Unions (`"a" \| "b"`) | ❌ Cannot | ✅ |
| Primitives, tuples, arrays | ❌ Cannot | ✅ |
| Inheritance syntax | `extends` | `&` (intersection) |
| Declaration merging | ✅ Yes | ❌ No — error |
| Used with `implements` on a class | ✅ Idiomatic | ✅ Works |

**Practical guidance:**
```typescript
// ✅ Use 'type' — interfaces cannot express these
type ID = string | number;
type Status = "active" | "inactive" | "banned";
type Coordinates = [number, number];
type Callback = (value: string) => void;

// ✅ Use 'interface' — shapes that may be extended or augmented
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
}

interface User extends BaseEntity {
  name: string;
}
```

---

### Q6. What is `readonly` and is it deep?

`readonly` prevents a property from being reassigned after the object is created.
**It is shallow** — it only blocks reassigning the property itself, not mutating
values inside it:

```typescript
interface User {
  readonly address: { city: string };
}

const user: User = { address: { city: "Delhi" } };

user.address = { city: "Pune" };  // ❌ blocked — address is readonly
user.address.city = "Mumbai";     // ✅ allowed — city itself is not readonly
```

If deep immutability is needed, mark the nested properties `readonly` too:

```typescript
interface User {
  readonly address: { readonly city: string };
}
```

---

## Exercises

---

### Exercise 1. Convert this type alias to an interface

**Type alias:**
```typescript
type Employee = {
  id: number;
  name: string;
  department: string;
  salary?: number;
};
```

**Answer:**
```typescript
interface Employee {
  id: number;
  name: string;
  department: string;
  salary?: number; // optional — same ? syntax as type alias
}
```

All properties identical. Syntax difference: no `=`, body directly follows the name.

---

### Exercise 2. BaseEntity + Order with `extends`

**Answer:**
```typescript
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
}

interface Order extends BaseEntity {
  customerName: string;
  totalAmount: number;
  status: "pending" | "shipped" | "delivered";
}

// Usage
const order: Order = {
  id: 1,
  createdAt: new Date(),
  customerName: "Neeraj",
  totalAmount: 5000,
  status: "pending",
};
```

**Key points:**
- `BaseEntity` fields are `readonly` — they must not be changed after creation
- `status` uses a literal union — only these three exact strings are valid
- `Order` inherits `id` and `createdAt` from `BaseEntity` — no need to repeat them

---

### Exercise 3. Extend multiple interfaces

**Answer:**
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

// Usage — Document requires all five fields
const doc: Document = {
  id: 1,
  createdBy: "neeraj",
  updatedBy: "neeraj",
  title: "TypeScript Notes",
  content: "Interfaces are powerful",
};
```

Multiple base interfaces are listed in `extends` separated by commas.

---

### Exercise 4. Find and fix all bugs

**Original broken code:**
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

**All bugs:**

| # | Bug | Fix |
|---|-----|-----|
| 1 | `interface Product =` — `=` is type alias syntax, not valid for interface | Remove `=` |
| 2 | `extends Product & { discount: number }` — `&` is invalid in extends clause | Move `discount` inside the interface body |
| 3 | `item` missing `discount` property — it is required in `DiscountedProduct` | Add `discount` to the object |
| 4 | Missing comma after `price: 50000` | Add comma between object properties |

**Fixed code:**
```typescript
interface Product {
  name: string;
  price: number;
}

interface DiscountedProduct extends Product {
  discount: number;  // defined here — not extended from anywhere
}

const item: DiscountedProduct = {
  name: "Laptop",
  price: 50000,      // comma required
  discount: 200,
};
```

---

### Exercise 5. AuthenticatedRequest and route handler

**Answer:**
```typescript
import { Request, Response } from "express";

// Interface extends Request — adds the 'user' property
// Two closing braces needed: one for the user object type, one for the interface body
interface AuthenticatedRequest extends Request {
  user: { id: number; role: "admin" | "customer" };
} // ← closes the interface body

function getDashboard(req: AuthenticatedRequest, res: Response): void {
  console.log(req.user.id);          // ✅ TypeScript knows req.user.id is number
  res.json({ message: "Welcome" });  // ✅ object wrapped in {}
}
```

**Key points:**
- `extends Request` — inherits all standard Express Request properties
- The `user` property is added on top — TypeScript now allows `req.user.id` safely
- Two closing braces: one ends `{ id: number; role: "..." }`, one ends the interface
- `res.json({ message: "..." })` — value must be wrapped in `{}` as an object literal

---

## Key Mistakes to Remember

| Mistake | Wrong | Right |
|---------|-------|-------|
| `=` with interface | `interface X = { }` | `interface X { }` |
| `&` in extends clause | `extends A & { b: string }` | `extends A { b: string }` |
| Missing interface closing brace | `interface X { prop: string` | `interface X { prop: string }` |
| Missing comma in object | `price: 50000\ndiscount: 200` | `price: 50000,\ndiscount: 200` |
| Union in interface | `interface Status = "a" \| "b"` | `type Status = "a" \| "b"` |
| Assuming readonly is deep | `readonly address: { city: string }` | also mark `readonly city` |
| Accidental interface name reuse | Two `interface Config` silently merge | Use specific, distinct names |

---

## Terms from This Lesson

| Term | Meaning |
|------|---------|
| **Interface** | A named shape declaration — what properties/methods an object must have |
| **`extends`** | Inherits all properties from one or more base interfaces |
| **Declaration merging** | Same interface name in scope = auto-combined into one |
| **`readonly` (shallow)** | Blocks reassigning the property — but not mutating values inside it |
| **`implements`** | Keyword used on a class to enforce an interface contract — preview only |
| **Module augmentation** | Using merging to add properties to external library types (e.g. `req.user`) |
| **BaseEntity** | A shared interface containing common fields (`id`, `createdAt`) — extended by all models |

---

*TypeScript Handbook — Lesson 7 Q&A Reference*
*Foundations Phase Complete*
*Next: Intermediate Phase — Lesson 8: Unions & Intersections*