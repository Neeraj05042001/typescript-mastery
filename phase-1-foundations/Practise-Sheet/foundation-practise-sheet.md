# TypeScript — Foundations Phase Practice Sheet

## Lessons 1–7 Consolidated Drill

This sheet covers every concept from the Foundations phase in one place.
Use it to test yourself before moving to the Intermediate phase.
No peeking at notes first. Write every answer as complete, working TypeScript.

---

## Section 1 — Primitive Types

**Q1.** Declare variables for each of these with the correct type:

- A product description
- A user's account balance
- Whether an order has been confirmed
- A variable that starts empty intentionally (no user logged in)
- A variable declared now but assigned later

**Q2.** What is wrong with each of these? Write the correct version:

```typescript
let orderId: Number = 1001;
let isActive: boolean = "true";
let pincode: number = 110001;
let score: unknown = 95;
score.toFixed(2);
```

**Q3.** A function receives data from an external API. The data could be
anything. You need to read a `price` property from it safely.
Write the function with the correct parameter type and a safe property access.

---

## Section 2 — Arrays

**Q4.** Declare these arrays with correct types:

- A list of city names
- A list of product prices
- An empty array that will hold order IDs (numbers)
- A list that can hold either strings or numbers
- A list of allowed HTTP methods that must never be changed at runtime

**Q5.** What does each of these return? Write the inferred type next to each:

```typescript
const names = ["Delhi", "Mumbai", "Pune"];

const result1 = names.map((n) => n.toUpperCase());
const result2 = names.filter((n) => n.length > 4);
const result3 = names.find((n) => n.startsWith("D"));
const result4 = names.reduce((acc, n) => acc + n.length, 0);
```

**Q6.** Fix everything wrong in this code:

```typescript
const tags = [];
tags.push("typescript");
tags.push(42);

const methods: string | string[] = ["GET", "POST"];
methods.push("DELETE");
```

---

## Section 3 — Objects & Type Aliases

**Q7.** Define type aliases for:

- A `BlogPost` with a readonly ID, title, content, published status,
  optional tags (array of strings), and optional views count
- An `Author` with a readonly ID, name, email, and optional bio
- A `BlogPostWithAuthor` that combines both using a nested `author`
  property typed as `Author`

**Q8.** What is wrong here? Fix everything:

```typescript
type orderItem = {
  ProductId: number;
  productName: string;
  quantity: Number;
  unitPrice: number;
  discount?: number;
};

const item: orderItem = {
  ProductId: 1,
  productName: "Laptop",
  quantity: 2,
  unitPrice: 50000,
  discount: "10%",
  notes: "Gift wrap please",
};
```

**Q9.** Define a `DeliveryAddress` type. Use it inside a `CustomerOrder` type.
The order should have: a readonly ID, customer name, delivery address,
total amount, status (only `"pending"`, `"processing"`, `"delivered"`),
and an optional notes field.

---

## Section 4 — Functions

**Q10.** Write fully typed versions of:

- A function that takes an array of strings and returns them sorted
  alphabetically
- A function that takes a price and an optional discount percentage,
  and returns the final price (default discount of 0)
- A function that takes a user ID and always throws
  `"User not found"` as an error
- An async function that takes an order ID (number) and returns
  a `Promise` that resolves to an `{ id: number; status: string }` object

**Q11.** Define function type aliases for:

- A function that takes a string and returns a string (a transformer)
- A function that takes two numbers and returns a boolean (a comparator)
- An async function that takes a number ID and returns `Promise<void>`

**Q12.** Fix every bug:

```typescript
function createOrder(
  customerId?: number,
  productId: number,
  quantity: number,
  notes: string,
): object {
  return { customerId, productId, quantity, notes };
}

function applyDiscount(price: number, discount: number): string {
  return price - (price * discount) / 100;
}

function logOrder(orderId: number): never {
  console.log(`Processing order ${orderId}`);
}
```

**Q13.** Write an Express middleware function called `requireAdmin` that:

- Takes `req`, `res`, `next` with proper types
- Checks if `req.body.role === "admin"`
- If not — sends `403` with `{ message: "Forbidden" }` and stops
- If yes — calls `next()`

---

## Section 5 — Interfaces

**Q14.** Create:

- A `BaseEntity` interface with `readonly id`, `readonly createdAt`,
  `readonly updatedAt`
- A `Product` interface that extends `BaseEntity` with `name`,
  `price`, `category`, and optional `description`
- A `DigitalProduct` interface that extends `Product` with
  `downloadUrl` and `fileSizeInMb`

**Q15.** What makes declaration merging different from type alias duplication?
Write an example showing why it matters for Express's `Request` type.

**Q16.** Fix everything wrong:

```typescript
interface = BlogComment {
  id: number;
  content: string;
  likes: number;
}

interface FeaturedComment extends BlogComment & { isFeatured: boolean } {}

const comment: FeaturedComment = {
  id: 1,
  content: "Great post"
  likes: 10,
  isFeatured: true
}
```

---

## Section 6 — Mixed Challenges

These combine concepts across multiple lessons — the kind of questions
that appear in real interviews.

**Q17.** You are building a product listing page in React.
Define all the types needed and write the component signature:

- A `Category` type — only `"electronics"`, `"clothing"`, `"books"`
- A `Product` interface with `id`, `name`, `price`, `category`
  (using your `Category` type), optional `rating`
- A `ProductListProps` type with a `products` array (using `Product`),
  an `onSelect` callback that receives a `Product` and returns nothing,
  and an optional `isLoading` boolean
- The `ProductList` component function signature (no JSX needed)

**Q18.** You are building an Express API for a bookstore.
Write everything needed for a `POST /books` route:

- A `CreateBookBody` type with `title`, `author`, `price`,
  `isbn` (string — not number), and optional `genre`
- A `BookResponse` type with all of the above plus a readonly `id`
  and readonly `createdAt`
- The route handler with proper types

**Q19.** Identify which tool (`type` or `interface`) you would use for each
of these, and explain why in one sentence each:

- A union of three possible user roles
- The shape of a database entity that other entities will inherit from
- A function callback type for an event handler
- A shape that might be augmented by a third-party library later
- A tuple representing latitude and longitude

**Q20.** You have this interface:

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  readonly version: string;
}
```

Write a `createConfig` function that takes all three values as parameters
with defaults of `"https://api.example.com"` for `apiUrl` and `5000`
for `timeout`, and returns a fully typed `Config` object.

---

## Answer Guidelines

When reviewing your own answers, check:

- Every variable has a type annotation where inference would be unclear
- Every function has typed parameters AND an explicit return type
- Optional parameters come AFTER required parameters
- `rest` parameters come LAST and are typed as an array
- Empty arrays are always explicitly typed
- `pincode`, `isbn`, phone numbers are `string` — not `number`
- `null` is always paired with its real type — `string | null`, never just `null`
- Interfaces use no `=` sign
- `extends` in interfaces does NOT use `&`
- Every `{` has a closing `}`
- Every `(` has a closing `)`
- Object properties are separated by commas
- React component names are PascalCase
- `throw new Error(message)` — not a function call
- `res.json({ ... })` — object wrapped in `{}`
- `return` after `res.json()` in Express middleware
- `next: NextFunction` — always typed, not left as `any`

---

_TypeScript Foundations Practice Sheet_
_Complete before starting: Intermediate Phase — Lesson 8: Unions & Intersections_
