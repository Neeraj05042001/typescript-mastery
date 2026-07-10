# Lesson 9 — Q&A Reference
## Literal Types & Type Narrowing

---

## Interview Questions

---

### Q1. What is a literal type? Why does `let x = "a"` infer differently from `const x = "a"`?

A literal type represents one exact value rather than a general category —
`"active"` as a type means the value can only ever be the exact text
`"active"`.

```typescript
let x = "a";   // inferred: string (widened)
const x = "a"; // inferred: "a" (the literal type)
```

This is purely about **inference defaults**, not a rule that literal types
require `const`. A `let` variable can absolutely hold an explicit literal
type — `let x: "active" = "active";` is completely valid. The difference
only shows up when no explicit type is given: since `let` can be
reassigned, TypeScript widens its inferred type to the general `string`
(anticipating it might hold other strings later). Since `const` can never
change, TypeScript keeps the most precise type it can — the literal itself.

---

### Q2. What does `as const` do? What is the `in` operator used for? What does `value is Type` mean?

**`as const`** does two things together: it locks every property of an
object (or every element of an array) to its literal type, *and* makes the
whole structure deeply readonly.

```typescript
const config = { status: "active" } as const;
// config.status: "active" (literal, not string)
// config itself and all its properties are now readonly
```

**The `in` operator** checks whether a specific named property exists on
an object — useful for narrowing a union of object shapes that don't
share a common discriminant tag:

```typescript
type Dog = { bark: () => void };
type Cat = { meow: () => void };

function makeSound(animal: Dog | Cat): void {
  if ("bark" in animal) {
    animal.bark(); // narrowed to Dog
  } else {
    animal.meow(); // narrowed to Cat
  }
}
```

**`value is Type`** does not change what the function itself returns — at
runtime, the function still returns a plain `boolean`, nothing else. What
it does is tell the *compiler*: "if this returns `true`, treat the
parameter as `Type` in the calling code from that point forward." The
narrowing happens to the argument at the **call site**, not to the
function's own return value.

```typescript
function isCircle(shape: Circle | Square): shape is Circle {
  return shape.kind === "circle"; // a real boolean check
}

if (isCircle(shape)) {
  // shape is narrowed to Circle HERE, in the caller — not inside isCircle
}
```

---

### Q3. How does exhaustiveness checking with `never` work, and why is it valuable? What's the danger of an incorrectly implemented type guard?

In a properly exhaustive `switch`, the `default` branch should **never
execute at runtime** — it exists purely as a compile-time safety net.

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      const exhaustiveCheck: never = shape;
      return exhaustiveCheck;
  }
}
```

If every case above `default` is handled, TypeScript knows nothing is
left for `shape` to be — its type inside `default` is `never`, and
assigning it to a `never`-typed variable compiles cleanly. If a new shape
variant is added later and its `case` is forgotten, something *is* left
over, and assigning it to `never` throws a compile error — at the exact
point the case was left unhandled, not months later as a runtime bug.
This converts "did I handle every case?" from a manual review question
into a guarantee the compiler enforces automatically.

**The danger of an incorrect type guard:** `value is Type` is a claim you
make yourself — TypeScript trusts it without verifying the logic actually
matches:

```typescript
function isCircle(shape: Circle | Square): shape is Circle {
  return true; // compiles fine — always claims true, regardless of the actual shape
}
```

Every caller of `isCircle` now silently believes any `Circle | Square`
value is a `Circle`, even when it's actually a `Square`. The bug hides
completely — no compile error, no obvious symptom, until code inside the
"narrowed" branch tries to access a `Circle`-only property that doesn't
exist on the actual runtime value.

---

## Exercises

---

### Exercise 1. Explain the widening, then fix the call

```typescript
function setDirection(dir: "north" | "south" | "east" | "west"): void { /* ... */ }
```

`let direction = "north";` infers `direction` as the wide `string`, since
`let` allows reassignment. Passing a `string` where `"north" | "south" |
"east" | "west"` is expected fails, because `string` includes far more
values than the four allowed.

```typescript
// ✅ Fixed — const keeps the literal type
const direction = "north";
setDirection(direction); // works — direction is inferred as "north"
```

---

### Exercise 2. Fix with `as const`, without changing `applyTheme`

```typescript
// ✅ applyTheme stays exactly as originally given
function applyTheme(theme: "dark" | "light"): void { /* ... */ }

// ✅ the fix belongs on settings
const settings = { theme: "dark", version: 2 } as const;

applyTheme(settings.theme); // works — settings.theme is now the literal "dark"
```

`as const` locks `settings.theme` to the literal `"dark"` instead of the
widened `string`, satisfying `applyTheme`'s narrow parameter type.

---

### Exercise 3. `getUserLabel` without the truthiness trap

```typescript
function getUserLabel(count: number | null): string {
  if (count !== null) {
    return `${count} users`;
  }
  return "No users";
}
```

Using `count !== null` explicitly (rather than `if (count)`) correctly
handles `count === 0` — a legitimate value that truthiness would have
incorrectly treated as "no count."

---

### Exercise 4. Custom type guard `isAdmin` + `describeUser`

```typescript
type AdminUser = { role: "admin"; permissions: string[] };
type RegularUser = { role: "user"; email: string };
type AppUser = AdminUser | RegularUser;

function isAdmin(user: AppUser): user is AdminUser {
  return user.role === "admin";
}

function describeUser(user: AppUser): string {
  if (isAdmin(user)) {
    return user.permissions.join(", ");
  }
  return user.email;
}
```

`isAdmin` performs a genuine boolean check (`user.role === "admin"`).
`describeUser` calls it directly — the narrowing from `isAdmin` flows
straight into the `if` branch, giving safe access to `permissions`
without needing a separate switch statement.

---

### Exercise 5. Exhaustive `getShippingCost`

```typescript
type ShippingMethod =
  | { type: "standard"; days: number }
  | { type: "express"; days: number }
  | { type: "overnight" };

function getShippingCost(shippingType: ShippingMethod): number {
  switch (shippingType.type) {
    case "standard":
      return 50;
    case "express":
      return 150;
    case "overnight":
      return 300;
    default:
      const exhaustiveCheck: never = shippingType;
      return exhaustiveCheck;
  }
}
```

Two details that matter here: the switch runs on `shippingType.type` —
the discriminant *property* — not on `shippingType` itself (switching on
the whole object would compare an object to a string, which never
matches). And the return type is `number`, matching what every branch
actually returns.

---

### Exercise 6. Fix the `Bird | Fish` bug

```typescript
type Bird = { fly: () => void };
type Fish = { swim: () => void };

function move(animal: Bird | Fish): void {
  if ("fly" in animal) {
    animal.fly();
  } else {
    animal.swim();
  }
}
```

The `in` operator's left-hand side must be a string — `"fly"`, not the
bare identifier `fly` (which would be read as a variable reference and
fail to resolve). Once narrowed, the branch should call the method that
actually exists on the narrowed type: `fly()` when `"fly" in animal` is
true, `swim()` otherwise.

---

## Key Mistakes to Remember

| Mistake | Wrong | Right |
|---------|-------|-------|
| Type vs value position | `{ theme: "dark" \| "light" }` | `{ theme: "dark" } as const` |
| `case` syntax | `case: "value"` | `case "value":` |
| Switching on the wrong thing | `switch (shippingType)` | `switch (shippingType.type)` |
| `void` but returns a value | `function f(): void { return x; }` | `function f(): number { return x; }` |
| `in` operator without quotes | `if (fly in animal)` | `if ("fly" in animal)` |
| Type guard returning a non-boolean | `return "admin";` | `return user.role === "admin";` |
| "scope" vs "type" | "let widens the scope" | "let widens the inferred type" |
| Unused type guard | writing `isAdmin` but not calling it | `if (isAdmin(user)) { ... }` |

---

## Terms from This Lesson

| Term | Meaning |
|------|---------|
| **Literal type** | A type representing one exact value, not a general category |
| **Widening** | TypeScript inferring a broader type (e.g. `string`) instead of the exact literal, for reassignable (`let`) variables |
| **`as const`** | Locks every property/element to its literal type and makes the structure deeply readonly |
| **Narrowing** | Reducing a broad or union type to something specific within a scope, based on a check |
| **Type guard** | A function using `value is Type` to tell the compiler how to narrow its parameter at the call site |
| **Discriminant** | The shared property (e.g. `kind`, `type`, `status`) used to distinguish between members of a union |
| **Exhaustiveness check** | A `default: const check: never = value` pattern that turns an unhandled union case into a compile error |

---

*TypeScript Handbook — Lesson 9 Q&A Reference*
*Next: Lesson 10 — Generics*