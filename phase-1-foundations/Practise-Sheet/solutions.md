# TypeScript Foundations — Practice Sheet Answer Key
## Ideal Answers, Lessons 1–7 Consolidated

---

## Section 1 — Primitive Types

### Q1. Declare variables with correct types

```typescript
const productDescription: string = "A lightweight, durable travel backpack";
const accountBalance: number = 15000;
const isOrderConfirmed: boolean = true;

// Intentionally empty — pair null with the real type, never leave it bare
let selectedUser: string | null = null;

// Declared now, assigned later — annotate with the EVENTUAL type, not 'undefined'
let username: string;
username = "Neeraj"; // assigned later
```

**Key rule:** `null` should always be paired with the real type (`string | null`), and a variable declared before assignment should be annotated with what it will eventually hold — not with `undefined` itself.

---

### Q2. Fix the bugs

```typescript
// ❌ Original
let orderId: Number = 1001;
let isActive: boolean = "true";
let pincode: number = 110001;
let score: unknown = 95;
score.toFixed(2);
```

```typescript
// ✅ Fixed
let orderId: number = 1001;        // lowercase primitive, not the wrapper object
let isActive: boolean = true;      // actual boolean, not the string "true"
let pincode: string = "110001";    // string — preserves leading zeros, no arithmetic done on it
let score: number = 95;            // number directly, since the value is a number
score.toFixed(2);                  // ✅ now valid — toFixed exists on number
```

**Why `score` can't stay `unknown`:** `unknown` blocks every operation — including method calls — until the type is proven. If you must keep something as `unknown` (e.g. it came from an untrusted source), narrow it first:
```typescript
let value: unknown = 95;
if (typeof value === "number") {
  value.toFixed(2); // ✅ safe after narrowing
}
```

---

### Q3. Safely read `price` from unknown API data

```typescript
function readPriceFromApiData(data: unknown): void {
  if (
    typeof data === "object" &&
    data !== null &&
    "price" in data &&
    typeof data.price === "number"
  ) {
    console.log(data.price);
  } else {
    console.log("The price should be a number");
  }
}
```

**Why four checks are needed:** `data` is `unknown`, so TypeScript won't let you touch `.price` until it knows `data` is an object (`typeof data === "object"`), that it isn't `null` (a classic JS quirk — `typeof null` is `"object"`), that the property exists (`"price" in data`), and finally that the property itself is the expected type (`typeof data.price === "number"`).

---

## Section 2 — Arrays

### Q4. Typed array declarations

```typescript
const cities: string[] = ["Delhi", "Mumbai", "Pune", "Noida"];
const productPrices: number[] = [4300, 3400, 3400, 5600, 7600];
const orderIds: number[] = []; // always annotate empty arrays explicitly
const mixedList: (string | number)[] = ["Neeraj", 44, "John", 24];
const httpMethods: readonly string[] = ["GET", "POST", "PUT", "DELETE", "PATCH"];
```

**Note:** The generic form is `Array<string>`, not `Arrays<string>` — no trailing "s". Both `string[]` and `Array<string>` are equivalent; the shorthand is preferred for simple types.

---

### Q5. Array method return types

```typescript
const names = ["Delhi", "Mumbai", "Pune"];

const result1 = names.map((n) => n.toUpperCase());
// Value: ["DELHI", "MUMBAI", "PUNE"]
// Type: string[]

const result2 = names.filter((n) => n.length > 4);
// Value: ["Delhi", "Mumbai"]   (Pune has 4 letters — not > 4, so excluded)
// Type: string[]

const result3 = names.find((n) => n.startsWith("D"));
// Value: "Delhi"   (a single value, NOT an array)
// Type: string | undefined

const result4 = names.reduce((acc, n) => acc + n.length, 0);
// Value: 15   (5 + 6 + 4)
// Type: number
```

**Critical rule:** `.find()` always returns `T | undefined` — one matching value or nothing. It never returns an array, and the possibility of `undefined` must be handled before using the result.

---

### Q6. Fix the bugs

```typescript
// ✅ Fixed
const tags: (string | number)[] = [];
tags.push("typescript");
tags.push(42);

const methods: string[] = ["GET", "POST"];
methods.push("DELETE");
```

**Two fixes:** the empty array needed an explicit union type since both strings and numbers get pushed into it; `methods` had the wrong union (`string | string[]`, meaning "a string OR an array" — not what was intended) and needed to simply be `string[]`.

---

## Section 3 — Objects & Type Aliases

### Q7. BlogPost, Author, BlogPostWithAuthor

```typescript
type BlogPost = {
  readonly id: number;
  title: string;
  content: string;
  published: boolean;
  tags?: string[];
  views?: number;
};

type Author = {
  readonly id: number;
  name: string;
  email: string;
  bio?: string;
};

type BlogPostWithAuthor = BlogPost & {
  author: Author;
};
```

**Note:** Only include exactly what's specified — resist adding extra fields beyond the stated requirements unless there's a clear reason to.

---

### Q8. Fix the bugs

```typescript
// ✅ Fixed
type OrderItem = {           // PascalCase — type names are always PascalCase
  productId: number;         // camelCase — 'p' not 'P'
  productName: string;
  quantity: number;          // lowercase primitive
  unitPrice: number;
  discount?: number;         // number, not a string like "10%"
  notes?: string;            // added to match the object, marked optional
};

const item: OrderItem = {
  productId: 1,
  productName: "Laptop",
  quantity: 2,
  unitPrice: 50000,
  discount: 10,
  notes: "Gift wrap please",
};
```

---

### Q9. DeliveryAddress and CustomerOrder

```typescript
type DeliveryAddress = {
  houseNo: string;
  street: string;
  city: string;
  state: string;
  pincode: string;   // string — not number. Leading zeros, no arithmetic.
  phone: string;      // string — same reasoning as pincode
};

type CustomerOrder = {
  readonly id: number;
  customerName: string;
  address: DeliveryAddress;
  totalAmount: number;
  status: "pending" | "processing" | "delivered";
  notes?: string;
};
```

**Rule to lock in:** any field that is an identifier or code — pincode, phone number, ISBN, account number — is `string`, never `number`. This applies even when the value looks purely numeric, because these fields are never used in arithmetic and can carry leading zeros.

---

## Section 4 — Functions

### Q10. Four typed functions

```typescript
// 1. Sort alphabetically
function sortAlphabetically(items: string[]): string[] {
  return [...items].sort(); // spread first — avoids mutating the original array
}

// 2. Price with default discount of 0
function finalPrice(price: number, discount: number = 0): number {
  return price - (discount / 100) * price;
}

// 3. Always throws
function throwUserNotFound(userId: number): never {
  throw new Error("User not found");
}

// 4. Async — resolves to { id, status }
type OrderResult = {
  id: number;
  status: string;
};

async function getOrderStatus(orderId: number): Promise<OrderResult> {
  return {
    id: orderId,
    status: "processing",
  };
}
```

**Two rules reinforced:** "default discount of 0" calls for an actual default parameter (`= 0`), not optional + manual check. "Always throws" means unconditional — no `if` — and the return type is `never`, since the function never reaches a return point.

---

### Q11. Function type aliases

```typescript
type Transformer = (value: string) => string;
type Comparator = (a: number, b: number) => boolean;
type AsyncIdHandler = (id: number) => Promise<void>;
```

**Reminder:** function type aliases use no curly braces — just `(params) => returnType`. And like all type names, they're PascalCase.

---

### Q12. Fix the bugs

```typescript
// ✅ Fixed
type OrderInput = {
  customerId: number;
  productId: number;
  quantity: number;
  notes?: string;
};

function createOrder(
  customerId: number,
  productId: number,
  quantity: number,
  notes?: string
): OrderInput {
  return { customerId, productId, quantity, notes };
}

function applyDiscount(price: number, discount: number): number {
  return price - (price * discount) / 100;
}

function logOrder(orderId: number): void {
  console.log(`Processing order ${orderId}`);
}
```

**Three original bugs:** `customerId` was wrongly optional and placed before required params (also a syntax error — optional can't precede required); `applyDiscount`'s return type didn't match its numeric return value; `logOrder`'s return type was `never`, but the function simply logs and finishes — that's `void`. `never` is reserved for functions that throw or loop forever; `void` is for functions that complete but return nothing useful. Logging a number does not make the *function's return type* a number — `console.log()` itself returns `undefined`.

---

### Q13. Express middleware — requireAdmin

```typescript
import { Request, Response, NextFunction } from "express";

function requireAdmin(req: Request, res: Response, next: NextFunction): void {
  if (req.body.role !== "admin") {
    res.status(403).json({ message: "Forbidden" });
    return; // critical — without this, next() would still run
  }
  next();
}
```

**Three things that must be right:** `next` typed as `NextFunction`; the comparison string matches exactly (case-sensitive); `return` immediately after sending the response, so a second call to `next()` never happens after a response has already been sent.

---

## Section 5 — Interfaces

### Q14. BaseEntity → Product → DigitalProduct

```typescript
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
  readonly updatedAt: Date;
}

interface Product extends BaseEntity {
  name: string;
  price: number;
  category: string;
  description?: string;
}

interface DigitalProduct extends Product {
  downloadUrl: string;
  fileSizeInMb: number; // a measurable quantity — not an identifier — so it's a number
}
```

**Distinction to hold onto:** fields like pincode or ISBN are identifiers with no arithmetic meaning — those are `string`. A file size is a genuine numeric quantity you'd compare or calculate with — that's `number`. The "make ID-like things strings" rule is about identifiers specifically, not about anything with a unit in its name.

---

### Q15. Declaration merging vs type alias duplication

Two interfaces sharing a name in the same scope **merge together** — every property from every declaration becomes part of one combined shape. It is additive, not a "last one wins" overwrite:

```typescript
interface User {
  name: string;
}
interface User {
  age: number;
}
// Merged result: interface User { name: string; age: number; }
// BOTH are now required — nothing from the first declaration is lost
```

Type aliases cannot do this at all — a duplicate name is a hard compile error:
```typescript
type User = { name: string };
type User = { age: number }; // ❌ Error: Duplicate identifier 'User'
```

**Why this matters for Express's `Request`:** merging is how you safely add properties to a type you don't own, like adding `req.user` after authentication middleware runs:

```typescript
interface AuthenticatedRequest extends Request {
  user: { id: number; role: "admin" | "customer" };
}

function getDashboard(req: AuthenticatedRequest, res: Response): void {
  console.log(req.user.id); // TypeScript knows this property exists
  res.json({ message: "Welcome" });
}
```

---

### Q16. Fix the bugs

```typescript
// ✅ Fixed
interface BlogComment {
  id: number;
  content: string;
  likes: number;
}

interface FeaturedComment extends BlogComment {
  isFeatured: boolean;
}

const comment: FeaturedComment = {
  id: 1,
  content: "Great post",  // comma required between every object property
  likes: 10,
  isFeatured: true,
};
```

**Three original bugs:** `=` used with `interface` (invalid — that's type alias syntax); `&` used inside an `extends` clause (invalid — `extends` takes interface names directly, separated by commas if there are multiple); and a missing comma between object properties, which is a plain syntax error regardless of types.

---

## Section 6 — Mixed Challenges

### Q17. React product listing

```typescript
type Category = "electronics" | "clothing" | "books";

interface Product {
  id: number;
  name: string;
  price: number;
  category: Category;   // uses the union type as a property — not as something to extend
  rating?: number;
}

type ProductListProps = {
  products: Product[];
  onSelect: (product: Product) => void;
  isLoading?: boolean;
};

function ProductList({ products, onSelect, isLoading }: ProductListProps): JSX.Element {
  // JSX body would go here
}
```

**Two rules to hold onto:** an interface can only `extends` another object shape — never a union, since a union has no fixed set of members to inherit. And a React component's signature always takes its **props type as the parameter** and returns JSX — never the reverse.

---

### Q18. Express bookstore — create route

```typescript
type CreateBookBody = {
  title: string;
  author: string;
  price: number;
  isbn: string;       // string — preserves format, no arithmetic
  genre?: string;
};

type BookResponse = CreateBookBody & {
  readonly id: number;
  readonly createdAt: Date;
};

app.post("/books", (req: Request, res: Response<BookResponse>): void => {
  const body = req.body as CreateBookBody;

  const book: BookResponse = {
    ...body,
    id: 5,                  // in a real app, generated by the database
    createdAt: new Date(),  // new Date() returns a Date object — Date.now() returns a number
  };

  res.status(201).json(book); // 201 Created — the conventional status for a new resource
});
```

**Key point:** a create route must build its response from the incoming `req.body`, not from hardcoded values — otherwise every request produces the same result regardless of what the client sent.

---

### Q19. type vs interface — five scenarios

| Scenario | Choice | Why |
|---|---|---|
| Union of three user roles | `type` | Interfaces cannot represent unions at all — this is a structural limitation, not a style preference |
| Database entity other entities inherit from | `interface` (preferred) | Both technically work (`extends` vs `&`), but `interface` is the idiomatic choice for extendable base shapes, and stays open to future augmentation via declaration merging |
| Function callback type for an event handler | `type` | Type aliases are the natural fit for function signatures; interfaces are for object/class shapes |
| A shape that might be augmented by a third-party library later | `interface` | Specifically because of **declaration merging** — only interfaces allow a shape to be reopened and extended after the fact |
| Tuple representing latitude and longitude | `type` | Same underlying reason as unions — interfaces cannot express tuples; only type aliases can (`type Coordinates = [number, number]`) |

---

### Q20. Config with default parameters

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  readonly version: string;
}

function createConfig(
  apiUrl: string = "https://api.example.com",
  timeout: number = 5000,
  version: string
): Config {
  return { apiUrl, timeout, version };
}

createConfig(undefined, undefined, "1.0.0");
// or, overriding a default:
createConfig("https://staging.example.com", 5000, "1.0.0");
```

**A cleaner real-world pattern** for functions with several defaultable parameters is an options object, which avoids the awkwardness of passing `undefined` to skip a positional default:
```typescript
type CreateConfigOptions = {
  apiUrl?: string;
  timeout?: number;
  version: string;
};

function createConfig({
  apiUrl = "https://api.example.com",
  timeout = 5000,
  version,
}: CreateConfigOptions): Config {
  return { apiUrl, timeout, version };
}

createConfig({ version: "1.0.0" });                    // both defaults used
createConfig({ version: "1.0.0", timeout: 3000 });     // one default overridden
```

---

## Master Checklist — Carry Into the Intermediate Phase

```
✓ null always paired with its real type — never bare
✓ A variable declared before assignment is annotated with its EVENTUAL type
✓ void = function completes, returns nothing useful
✓ never = function never completes (throws / infinite loop) — NOT "logs a number"
✓ .find() returns T | undefined — a single value, never an array
✓ Array<T>, not Arrays<T>
✓ pincode, isbn, phone → string, never number
✓ Type names → PascalCase. Properties → camelCase. Always.
✓ Function type aliases: (params) => returnType — no curly braces
✓ Optional parameters come after required ones; default params use " = value"
✓ Always return immediately after res.json()/res.send() in middleware
✓ next: NextFunction — always typed, never left bare
✓ interface can only extend an object shape — never a union or tuple
✓ React components take props IN, return JSX OUT — never the reverse
✓ Declaration merging COMBINES interfaces — it does not overwrite
✓ Date.now() → number. new Date() → Date object. Not interchangeable.
✓ Read the task's exact wording before designing a solution
```

---

*TypeScript Foundations — Practice Sheet Answer Key*
*Next: Intermediate Phase — Lesson 8: Unions & Intersections*