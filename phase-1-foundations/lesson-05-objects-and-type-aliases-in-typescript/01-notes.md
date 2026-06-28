# Lesson 5 — Objects & Type Aliases

## First, connect to what you already know

In Lesson 3 you learned to type individual values:
```typescript
let name: string = "Neeraj";
let age: number = 25;
```

In Lesson 4 you learned to type collections of values:
```typescript
let names: string[] = ["Neeraj", "Rahul"];
```

But in real applications, data doesn't come in isolated pieces. A user has a name, age, email, and role — all together, describing one thing. That's an **object**. And TypeScript lets you type the entire shape of that object.

---

## Part 1 — Typing Objects

### The problem without typed objects

```typescript
// JavaScript — no structure enforced
const user = {
  name: "Neeraj",
  age: 25,
};

console.log(user.emal);   // undefined — typo, no error
user.age = "twenty five"; // reassigned wrong type, no error
```

TypeScript fixes this by letting you define exactly what shape an object must have.

---

### Inline object typing

The most basic way — write the type directly where the variable is declared:

```typescript
const user: { name: string; age: number; email: string } = {
  name: "Neeraj",
  age: 25,
  email: "neeraj@gmail.com",
};
```

TypeScript now enforces three things:
- Every property in the type **must exist** in the object
- Every property must hold the **correct type**
- You **cannot access properties** that aren't in the type

```typescript
console.log(user.name);  // ✅ "Neeraj"
console.log(user.emal);  // ❌ Error — property 'emal' does not exist
user.age = "twenty five"; // ❌ Error — string not assignable to number
```

---

### The problem with inline typing

Inline typing works — but it doesn't scale:

```typescript
// You'd have to write this full shape every single time
function createUser(user: { name: string; age: number; email: string }) { ... }
function updateUser(user: { name: string; age: number; email: string }) { ... }
function deleteUser(user: { name: string; age: number; email: string }) { ... }
```

This is verbose, repetitive, and a maintenance nightmare. If the shape changes, you update it in ten places. This is exactly what **Type Aliases** solve.

---

## Part 2 — Type Aliases

### What is a type alias?

A type alias gives a **name** to a type. Instead of writing the shape everywhere, you define it once and reference it by name.

```typescript
// Define once
type User = {
  name: string;
  age: number;
  email: string;
};

// Use everywhere
const user: User = {
  name: "Neeraj",
  age: 25,
  email: "neeraj@gmail.com",
};

function createUser(user: User): void { ... }
function updateUser(user: User): void { ... }
function deleteUser(user: User): void { ... }
```

Now if the `User` shape changes, you update it in **one place** and every function that uses `User` is automatically updated.

---

### Syntax

```typescript
type TypeName = typeDefinition;
```

**Naming convention — always PascalCase:**
```typescript
type User = { ... }       // ✅
type ProductItem = { ... } // ✅
type user = { ... }        // ❌ lowercase — wrong convention
type product_item = { ... } // ❌ snake_case — wrong
```

PascalCase for types is a universal TypeScript convention. Every library, every codebase, every team follows it.

---

## Part 3 — Object Type Features

### Optional properties — `?`

Not every property is always required. A user might not have a middle name. A product might not have a discount. Use `?` to mark a property as optional:

```typescript
type User = {
  name: string;
  age: number;
  email: string;
  phone?: string;       // optional — string | undefined
  middleName?: string;  // optional — string | undefined
};

// ✅ Valid — phone and middleName not required
const user: User = {
  name: "Neeraj",
  age: 25,
  email: "neeraj@gmail.com",
};

// ✅ Also valid — providing optional fields
const user2: User = {
  name: "Rahul",
  age: 30,
  email: "rahul@gmail.com",
  phone: "9999999999",
};
```

**What `?` actually means under the hood:**

`phone?: string` is exactly the same as `phone: string | undefined`. The property can hold a string, or it might not exist at all.

```typescript
// This means you must check before using optional properties
function printPhone(user: User): void {
  // ❌ Dangerous — phone might be undefined
  console.log(user.phone.toUpperCase());

  // ✅ Safe — check first
  if (user.phone !== undefined) {
    console.log(user.phone.toUpperCase());
  }
}
```

---

### `readonly` properties

Some properties should never change after an object is created. A user's ID, a database record's creation timestamp — these are set once and must not be overwritten:

```typescript
type User = {
  readonly id: number;      // cannot be changed after creation
  readonly createdAt: Date; // cannot be changed after creation
  name: string;             // can be updated
  email: string;            // can be updated
};

const user: User = {
  id: 1,
  createdAt: new Date(),
  name: "Neeraj",
  email: "neeraj@gmail.com",
};

user.name = "Neeraj Kumar";  // ✅ name can change
user.email = "new@email.com"; // ✅ email can change
user.id = 2;                  // ❌ Error — cannot assign to readonly property
user.createdAt = new Date();  // ❌ Error — cannot assign to readonly property
```

---

### Nested objects

Real-world objects contain other objects. An address inside a user. A category inside a product:

```typescript
// ✅ Approach 1 — define nested types separately (recommended)
type Address = {
  street: string;
  city: string;
  state: string;
  pincode: string;
};

type User = {
  name: string;
  email: string;
  address: Address; // reusable — any type can use Address
};

// ✅ Approach 2 — inline the nested type (fine for one-off shapes)
type User = {
  name: string;
  email: string;
  address: {
    street: string;
    city: string;
    state: string;
    pincode: string;
  };
};
```

**Prefer Approach 1 in real projects** — `Address` can now be reused across `User`, `Supplier`, `Warehouse`, and any other type that has an address.

```typescript
// Real usage
const user: User = {
  name: "Neeraj",
  email: "neeraj@gmail.com",
  address: {
    street: "123 MG Road",
    city: "Delhi",
    state: "Delhi",
    pincode: "110001",
  },
};

// Accessing nested properties — fully typed and autocompleted
console.log(user.address.city);    // ✅ "Delhi"
console.log(user.address.country); // ❌ Error — 'country' doesn't exist on Address
```

---

### Excess property checking

TypeScript checks that you haven't added properties that don't belong in the type:

```typescript
type User = {
  name: string;
  age: number;
};

// ❌ Error — 'email' is not in User
const user: User = {
  name: "Neeraj",
  age: 25,
  email: "neeraj@gmail.com", // TypeScript rejects this
};
```

**But there's a nuance — this only applies to direct object literals:**

```typescript
// ✅ This works — TypeScript is less strict with indirect assignment
const obj = {
  name: "Neeraj",
  age: 25,
  email: "neeraj@gmail.com",
};

const user: User = obj; // No error — obj satisfies the shape of User
```

This is TypeScript's **structural typing** from Lesson 3 in action. TypeScript checks that the required structure is present — extra properties in an indirect assignment are allowed because the shape requirement is still fully satisfied.

This distinction comes up in interviews — know it.

---

## Part 4 — Type Aliases for Everything

Type aliases aren't just for objects. You can name **any type**:

```typescript
// For union types — extremely common
type ID = string | number;
type Status = "active" | "inactive" | "banned"; // literal union — more on this soon

// For arrays
type Names = string[];
type Users = User[];

// Using your aliases
const userId: ID = "abc123";   // ✅ string
const userId2: ID = 101;       // ✅ number

const userList: Users = [
  { name: "Neeraj", age: 25, email: "n@g.com" },
  { name: "Rahul", age: 30, email: "r@g.com" },
];
```

**Literal types — a powerful pattern worth knowing now:**

```typescript
type Status = "active" | "inactive" | "banned";

let userStatus: Status = "active";   // ✅
userStatus = "inactive";             // ✅
userStatus = "deleted";              // ❌ Error — "deleted" not in Status
```

This locks the variable to only the specific string values you allow. This is far more powerful than just `string` — it documents and enforces exactly what values are valid. You'll use this pattern constantly.

---

## Part 5 — Real-World Usage

### React — typing component props

```typescript
// Define the shape of props as a type alias
type ButtonProps = {
  label: string;
  onClick: () => void;   // function that returns nothing
  disabled?: boolean;    // optional
  variant?: "primary" | "secondary" | "danger"; // literal union
};

// Use it in the component
function Button({ label, onClick, disabled, variant }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}

// TypeScript validates every prop at the call site
<Button label="Submit" onClick={handleSubmit} />                  // ✅
<Button label="Delete" onClick={handleDelete} variant="danger" /> // ✅
<Button onClick={handleSubmit} />                                  // ❌ label is required
<Button label={42} onClick={handleSubmit} />                       // ❌ label must be string
```

### Node.js / Express — typing request body

```typescript
type CreateUserBody = {
  name: string;
  email: string;
  password: string;
  role?: "admin" | "editor" | "viewer";
};

router.post("/users", (req: Request, res: Response): void => {
  const body = req.body as CreateUserBody; // we'll cover 'as' properly soon
  console.log(body.name);   // ✅ TypeScript knows this is string
  console.log(body.nmae);   // ❌ Error — typo caught immediately
  res.json({ message: "User created" });
});
```

### API response typing

```typescript
// What a user API response looks like
type ApiResponse<T> = {        // we'll cover generics properly later
  data: T;
  success: boolean;
  message: string;
  statusCode: number;
};

type User = {
  id: number;
  name: string;
  email: string;
};

// The full shape of the response
type UserResponse = ApiResponse<User>;

/*
{
  data: { id: 1, name: "Neeraj", email: "neeraj@gmail.com" },
  success: true,
  message: "User fetched successfully",
  statusCode: 200
}
*/
```

---

## Common Mistakes

```typescript
// ❌ Mistake 1 — using 'object' as a type (useless)
let user: object = { name: "Neeraj" };
console.log(user.name); // Error — 'object' type has no properties
// ✅ Fix — define the actual shape
type User = { name: string };
let user: User = { name: "Neeraj" };

// ❌ Mistake 2 — not using type aliases (inline types everywhere)
function getUser(user: { name: string; age: number }): { name: string; age: number } { }
// ✅ Fix — use a type alias
type User = { name: string; age: number };
function getUser(user: User): User { }

// ❌ Mistake 3 — accessing optional property without checking
type User = { name: string; phone?: string };
const user: User = { name: "Neeraj" };
user.phone.toUpperCase(); // Error — phone might be undefined
// ✅ Fix
if (user.phone !== undefined) { user.phone.toUpperCase(); }

// ❌ Mistake 4 — wrong naming convention
type user = { name: string };    // lowercase — wrong
type user_profile = { ... };     // snake_case — wrong
// ✅ Fix — PascalCase always
type User = { name: string };
type UserProfile = { ... };

// ❌ Mistake 5 — mutating readonly
type User = { readonly id: number; name: string };
const user: User = { id: 1, name: "Neeraj" };
user.id = 2; // Error
// ✅ readonly means set once — respect it
```

---

## Exercises — write all of these

**Exercise 1.** Create type aliases for the following. Think carefully about which properties should be optional and which should be readonly:
- A `Product` with an ID, name, price, optional description, and optional discount percentage
- An `Address` with street, city, state, and pincode
- A `Customer` with a readonly ID, name, email, optional phone, and an address (use your `Address` type)

**Exercise 2.** What is wrong with this code? Identify every issue and fix it:

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

**Exercise 3.** You're building a React component called `UserCard`. It should receive:
- A `name` (required string)
- An `email` (required string)
- A `role` — only one of `"admin"`, `"editor"`, or `"viewer"` (required)
- An `avatarUrl` (optional string)
- An `isActive` (required boolean)

Define the type alias for the props and write the component signature (just the function definition — no JSX needed).

**Exercise 4.** You're building an Express route to create a product. Define the type for the request body and write the route handler with proper types.

The request body should contain: name, price, category, and an optional description.

---

Take your time. Write complete code for all four. 
See `solutions.md` for solutions