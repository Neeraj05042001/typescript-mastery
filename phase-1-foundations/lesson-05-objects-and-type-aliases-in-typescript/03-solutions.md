# Lesson 5 — Q&A Reference
## Objects & Type Aliases in TypeScript

---

## Conceptual Questions

---

### Q1. What is a type alias and why do we use it?

A type alias gives a reusable name to any type definition.

Instead of writing the same shape in every function and variable:
```typescript
// ❌ Without type alias — repeated everywhere
function getUser(user: { name: string; age: number }): { name: string; age: number } {}
function updateUser(user: { name: string; age: number }): void {}
```

You define it once and reference it by name:
```typescript
// ✅ With type alias — defined once, used everywhere
type User = {
  name: string;
  age: number;
};

function getUser(user: User): User {}
function updateUser(user: User): void {}
```

If the shape changes, you update it in one place. Every function using that alias is automatically updated.

---

### Q2. What does `?` mean on a property? What must you do before using it?

`?` marks a property as optional — it may or may not exist on the object.

```typescript
type User = {
  name: string;
  phone?: string; // optional — string | undefined
};
```

`phone?: string` is exactly the same as `phone: string | undefined`.

Before using an optional property, you must check it exists:
```typescript
const user: User = { name: "Neeraj" }; // phone not provided

// ❌ Unsafe — phone might be undefined
user.phone.toUpperCase(); // Error: possibly undefined

// ✅ Safe — check first
if (user.phone !== undefined) {
  user.phone.toUpperCase(); // TypeScript now knows it's a string
}
```

---

### Q3. What does `readonly` do on a property?

`readonly` prevents a property from being reassigned after the object is created.

```typescript
type User = {
  readonly id: number;
  name: string;
};

const user: User = { id: 1, name: "Neeraj" };

user.name = "Neeraj Kumar"; // ✅ allowed
user.id = 2;                // ❌ Error — cannot assign to readonly property
```

Use `readonly` for IDs, timestamps, and anything that should be set once and never changed.

---

### Q4. What is the difference between a required and optional property?

```typescript
type User = {
  name: string;     // required — must always be provided
  phone?: string;   // optional — can be omitted
};

const user1: User = { name: "Neeraj" };               // ✅
const user2: User = { name: "Rahul", phone: "999" };  // ✅
const user3: User = {};                                // ❌ name is required
```

Required = the object is invalid without it.
Optional = the object is valid with or without it.

---

### Q5. Why should pincodes, phone numbers, and similar fields be typed as `string` not `number`?

Three reasons:

**1. Leading zeros are lost**
```typescript
// India pincode 011001 stored as number
const pincode: number = 011001; // becomes 11001 — wrong pincode
const pincode: string = "011001"; // ✅ preserved exactly
```

**2. No arithmetic is ever done on these values**
You never add two pincodes or multiply a phone number. If you don't compute with it, it shouldn't be a number.

**3. Fixed length and format**
Pincodes are always 6 digits. Phone numbers have a specific format. Strings preserve this. Numbers don't.

**Rule:** If it has leading zeros, has a fixed length, or is never computed — use `string`.
Applies to: pincodes, phone numbers, Aadhaar, PAN, bank account numbers, zip codes.

---

### Q6. What is a literal union type? Why is it better than plain `string`?

A literal union restricts a variable to only specific allowed string values:

```typescript
// Plain string — any text allowed
type Status = string;
let s: Status = "anything goes"; // ✅ — no protection

// Literal union — only specific values allowed
type Status = "active" | "inactive" | "banned";
let s: Status = "active";    // ✅
let s: Status = "deleted";   // ❌ Error — not an allowed value
```

Benefits:
- Documents the full set of valid values in the type itself
- TypeScript catches invalid values at compile time
- Autocomplete shows all allowed values in your IDE

---

### Q7. What is excess property checking? When does it apply?

When you assign an object literal directly to a typed variable, TypeScript rejects any extra properties not defined in the type:

```typescript
type User = { name: string; age: number };

// ❌ Direct assignment — excess property rejected
const user: User = {
  name: "Neeraj",
  age: 25,
  email: "neeraj@gmail.com", // Error — not in User
};

// ✅ Indirect assignment — excess property allowed
const obj = { name: "Neeraj", age: 25, email: "neeraj@gmail.com" };
const user: User = obj; // No error — User's shape requirement is satisfied
```

This is TypeScript's **structural typing** — it checks that the required shape is present. In indirect assignment, extra properties don't violate that.

---

### Q8. Why must React component names be PascalCase?

This is a functional requirement, not just a style convention.

React uses the casing of a component name to decide how to process it in JSX:

```typescript
// Lowercase — React treats this as a standard HTML element
function userCard(props: UserCardProps) {}
<userCard name="Neeraj" /> // React looks for an HTML tag 'usercard' — not found

// PascalCase — React treats this as a custom component
function UserCard(props: UserCardProps) {}
<UserCard name="Neeraj" /> // React correctly renders your component
```

A lowercase component name compiles without errors but breaks at runtime. Always use PascalCase for React components.

---

## Exercises

---

### Exercise 1. Define type aliases for Product, Address, and Customer

**Answer:**

```typescript
type Product = {
  readonly id: number;    // IDs should not change — readonly
  name: string;
  price: number;
  description?: string;  // optional
  discount?: number;      // optional, percentage — number not string
};

type Address = {
  street: string;
  city: string;
  state: string;
  pincode: string;        // string — not number, preserves leading zeros
};

type Customer = {
  readonly id: number;    // camelCase — not 'ID'. PascalCase is for TYPE names only
  name: string;
  email: string;
  phone?: string;         // optional
  address: Address;       // reuse the Address type
};
```

**Key decisions explained:**
- `id` is `readonly` — identifiers should never be reassigned
- `pincode` is `string` — Indian pincodes have leading zeros, no arithmetic needed
- `discount` is `number` — a percentage is numeric; `"string"` in quotes is a literal type, not the string type
- Property names use `camelCase` — `id` not `ID`, `firstName` not `FirstName`
- `Address` is defined separately — reusable across `Customer`, `Supplier`, `Warehouse`, etc.

---

### Exercise 2. Find and fix all bugs

**Original broken code:**
```typescript
type blogPost = {
  id: number;
  title: string;
  content: string;
  published: boolean;
}

const post: blogPost = {
  id: 1,
  title: "Learning TypeScript",
  content: "TypeScript is great",
  published: false,
  author: "Neeraj",
}

post.id = 2;
console.log(post.ttle);
```

**All four bugs:**

| # | Bug | Fix |
|---|-----|-----|
| 1 | `blogPost` — wrong naming convention | Rename to `BlogPost` (PascalCase) |
| 2 | `author` exists in object but not in type | Add `author: string` to the type |
| 3 | `post.id = 2` — id should not be reassignable | Add `readonly` to `id` in the type |
| 4 | `post.ttle` — typo, property doesn't exist | Fix to `post.title` |

**Fixed code:**
```typescript
type BlogPost = {          // fix 1: PascalCase
  readonly id: number;     // fix 3: readonly — id must not change
  title: string;
  content: string;
  published: boolean;
  author: string;          // fix 2: added to match the object
};

const post: BlogPost = {
  id: 1,
  title: "Learning TypeScript",
  content: "TypeScript is great",
  published: false,
  author: "Neeraj",        // now valid
};

// post.id = 2;            // removed — readonly prevents reassignment

console.log(post.title);   // fix 4: ttle → title
```

---

### Exercise 3. Define React component props and function signature

**Task:**
- `name` — required string
- `email` — required string
- `role` — one of `"admin"`, `"editor"`, `"viewer"` (required)
- `avatarUrl` — optional string
- `isActive` — required boolean

**Answer:**

```typescript
type UserCardProps = {
  name: string;
  email: string;
  role: "admin" | "editor" | "viewer";  // literal union — only these values
  avatarUrl?: string;                    // optional
  isActive: boolean;                     // required — no ? here
};

// PascalCase — functionally required for React components
function UserCard({ name, email, role, avatarUrl, isActive }: UserCardProps): void {
  // component body
}
```

**Key points:**
- `isActive: boolean` — no `?`. The task said required. Optional and required are opposites.
- `UserCard` not `userCard` — React uses casing to distinguish components from HTML elements
- Literal union for `role` — only three specific values are allowed, not any string

---

### Exercise 4. Express route to create a product

**Task:** Type the request body with name, price, category, and optional description.

**Answer:**

```typescript
import express, { Request, Response } from "express";

const router = express.Router();

// Specific name — avoids confusion with other Product-related types
type CreateProductBody = {
  name: string;
  price: number;
  category: string;
  description?: string;  // optional
};

router.post("/product", (req: Request, res: Response): void => {
  const body = req.body as CreateProductBody;
  console.log(body.name);                    // ✅ TypeScript knows this is string
  res.json({ message: "Product created" });  // ✅ object with {}
});

export default router;
```

**Key points:**
- Name the type `CreateProductBody` not just `Product` — more specific, avoids conflicts
- `res.json()` takes an object literal — always wrap in `{ }`
- Route handler returns `void` — it sends a response, it doesn't return a value
- `Request` and `Response` imported from express

---

## Key Mistakes to Remember

| Mistake | Wrong | Right |
|---------|-------|-------|
| Type naming | `type user = {}` | `type User = {}` |
| Property naming | `ID: number` | `id: number` |
| Discount type | `discount?: "string"` | `discount?: number` |
| Pincode type | `pincode: number` | `pincode: string` |
| Optional when required | `isActive?: boolean` | `isActive: boolean` |
| React component casing | `function userCard()` | `function UserCard()` |
| res.json syntax | `res.json(message: "done")` | `res.json({ message: "done" })` |
| Useless type | `let x: object` | `type X = { name: string }` |

---

## Terms from This Lesson

| Term | Meaning |
|------|---------|
| **Type alias** | A named, reusable type definition — `type User = { ... }` |
| **Optional property** | A property that may not exist — marked with `?` |
| **Readonly property** | A property that cannot be reassigned after creation |
| **Literal union type** | A type restricted to specific string values — `"active" \| "inactive"` |
| **Nested type** | An object type that contains another object type as a property |
| **Excess property checking** | TypeScript rejecting extra properties on direct object literals |
| **Structural typing** | TypeScript checks shape, not name — if the shape fits, it's valid |
| **PascalCase** | Capitalised first letter of each word — used for type names |
| **camelCase** | Lowercase first word, capitalised subsequent words — used for property names |

---

*TypeScript Handbook — Lesson 5 Q&A Reference*  
*Next: Lesson 6 — Functions*
