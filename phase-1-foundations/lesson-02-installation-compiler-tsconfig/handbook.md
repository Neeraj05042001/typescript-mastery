# TypeScript Handbook — Entry 2: Installation, Compiler & tsconfig.json

---

## What is it?

This entry covers the TypeScript toolchain: how to install TypeScript, how the
compiler works, what tsconfig.json is and why every production project needs it,
and how to set up a project correctly from day one.

---

## Why do we need it?

TypeScript cannot run directly in browsers or Node.js — they only understand
JavaScript. You need a build step that transforms TypeScript source into
JavaScript output. The tsconfig.json file controls every aspect of how that
transformation happens. Without understanding these tools, you cannot write
TypeScript in any real project.

---

## Problem it solves

- Browsers and Node.js cannot parse TypeScript syntax
- Single-file compilation does not scale to multi-file projects
- Without tsconfig, every developer on a team may compile with different settings
- Without strict mode, TypeScript misses entire categories of real bugs

---

## TypeScript vs JavaScript — key differences

| Dimension | JavaScript | TypeScript |
|-----------|-----------|------------|
| Type checking | Runtime (bugs reach users) | Compile time (caught before code runs) |
| Variable types | Can change freely | Locked from first assignment (inference) |
| Null handling | Silently crashes | Must handle null/undefined explicitly (strict) |
| New syntax | None added | Annotations, interfaces, generics, enums |
| Runs directly | Yes (browser/Node) | No — must compile to JS first |
| IDE support | Limited | Full autocomplete, refactoring, navigation |
| Performance overhead | None | None (types erased — zero runtime impact) |

### Static vs dynamic typing

```typescript
// JavaScript — dynamically typed
let value = "hello";
value = 42;     // Fine — type can change
value = true;   // Also fine

// TypeScript — statically typed
let value = "hello"; // TypeScript infers: string
value = 42;          // ❌ Error: Type 'number' not assignable to type 'string'
```

---

## The compilation pipeline

```
Your .ts files
      ↓
TypeScript Compiler (tsc)
  • reads tsconfig.json for settings
  • checks all type rules
  • reports errors (if any)
  • types are erased from output
      ↓
Plain .js files (zero TypeScript remains)
      ↓
Browser / Node.js executes JavaScript
```

---

## Syntax — installation

```bash
# Check prerequisites
node --version   # v16+ required
npm --version

# Global install (for learning)
npm install -g typescript
tsc --version    # Verify: should show 5.x.x

# Local install (for projects — professional approach)
npm install --save-dev typescript
npx tsc --version

# Generate tsconfig.json
npx tsc --init
```

---

## Syntax — first .ts file

```typescript
// src/hello.ts
function greetDeveloper(name: string, language: string): string {
  return `Hello ${name}! You are now writing ${language}.`;
}

const message = greetDeveloper("Alice", "TypeScript");
console.log(message);
```

```bash
# Compile a single file
tsc hello.ts        # Produces hello.js

# Compile whole project (when tsconfig.json exists)
npx tsc             # Compiles everything in rootDir to outDir

# Watch mode — recompile on every save
npx tsc --watch
```

### What type erasure looks like

```typescript
// Input: hello.ts
function greetDeveloper(name: string, language: string): string {
  return `Hello ${name}! You are now writing ${language}.`;
}
const message = greetDeveloper("Alice", "TypeScript");
```

```javascript
// Output: hello.js — all type annotations removed
function greetDeveloper(name, language) {
  return `Hello ${name}! You are now writing ${language}.`;
}
const message = greetDeveloper("Alice", "TypeScript");
```

---

## tsconfig.json — key options explained

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### `target` — output JavaScript version

Controls what JavaScript syntax the compiled output uses.

```json
"target": "ES2020"  // Modern Node.js and browsers — recommended
"target": "ES5"     // Maximum browser compatibility (converts arrow functions, etc.)
```

```typescript
// Same TypeScript input
const greet = (name: string) => `Hello ${name}`;

// target: "ES5" output
var greet = function(name) { return "Hello " + name; };

// target: "ES2020" output
const greet = (name) => `Hello ${name}`;
```

### `module` — module system in output

Controls how `import`/`export` statements are compiled.

```json
"module": "commonjs"   // For Node.js — compiles to require() and module.exports
"module": "ESNext"     // For Vite/webpack — keeps import/export as-is
```

```typescript
// You write
import express from 'express';

// "module": "commonjs" output
const express = require('express');

// "module": "ESNext" output (kept as-is)
import express from 'express';
```

### `strict` — THE most important option. Always true.

Enables all strict type checks. Never set this to false.

```json
"strict": true
```

What `strict: true` actually enables:

**`strictNullChecks`** — must explicitly handle null/undefined:
```typescript
// Without: null can silently sneak into any type
function getName(): string { return null; } // No error — dangerous

// With strictNullChecks
function getName(): string { return null; } // ❌ Error: Type 'null' not assignable to 'string'
function getName(): string | null { return null; } // ✅ Explicit null is allowed
```

**`noImplicitAny`** — parameters cannot silently become `any`:
```typescript
// Without: untyped parameters become 'any' — type checking disabled
function greet(name) { return name.toUpperCase(); } // name is silently 'any'

// With noImplicitAny
function greet(name) { ... }         // ❌ Error: Parameter 'name' implicitly has 'any' type
function greet(name: string) { ... } // ✅ Must annotate
```

### `outDir` and `rootDir` — clean file organisation

```json
"outDir": "./dist",   // Compiled JS files go here — deploy this
"rootDir": "./src"    // TypeScript source files live here — edit this
```

```
project/
├── src/          ← rootDir — your TypeScript (edit these)
│   ├── index.ts
│   └── utils.ts
├── dist/         ← outDir — compiled JavaScript (never edit manually)
│   ├── index.js
│   └── utils.js
├── tsconfig.json
└── package.json
```

### `esModuleInterop` — cleaner imports

```json
"esModuleInterop": true
```

```typescript
// Without esModuleInterop
import * as express from 'express';  // awkward
import * as path from 'path';

// With esModuleInterop: true
import express from 'express';       // clean
import path from 'path';
```

### `include` and `exclude`

```json
"include": ["src/**/*"],                  // Compile all files in src/, recursively
"exclude": ["node_modules", "dist"]       // Never compile these folders
```

---

## Real-world usage

### React + TypeScript (Vite — modern standard)

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app && npm install && npm run dev
```

### Node.js + TypeScript

```bash
npm init -y
npm install --save-dev typescript ts-node @types/node
npx tsc --init
```

`ts-node` runs `.ts` files directly in Node.js during development — no manual
compile step needed:
```bash
npx ts-node src/index.ts   # Development: run TypeScript directly
npx tsc                    # Production: compile to dist/
node dist/index.js         # Production: run compiled output
```

### package.json scripts (standard pattern)

```json
{
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "start": "node dist/index.js"
  }
}
```

---

## Common mistakes

### Mistake 1 — Setting `strict: false`
```json
// ❌ Never do this — you lose null safety, implicit any protection, and more
"strict": false

// ✅ Always
"strict": true
```

### Mistake 2 — Setting `outDir` the same as `rootDir`
```json
// ❌ Compiled JS files will mix with your TypeScript source
"outDir": "./src",
"rootDir": "./src"

// ✅ Keep them separate
"outDir": "./dist",
"rootDir": "./src"
```

### Mistake 3 — Not including `node_modules` in exclude
```json
// ❌ TypeScript will try to compile every package in node_modules — takes forever
"exclude": []

// ✅ Always exclude node_modules and dist
"exclude": ["node_modules", "dist"]
```

### Mistake 4 — Ignoring TypeScript errors by force-compiling
```bash
# ❌ Bypasses all type checking — defeats the entire purpose
tsc --noEmitOnError false

# ✅ Fix the errors — that's the whole point
```

### Mistake 5 — Confusing `target` and `module`
- `target`: what JavaScript features the *output* uses (ES5, ES2020, etc.)
- `module`: how *import/export* statements are compiled (commonjs, ESNext, etc.)
These are independent. You can target ES2020 syntax but use CommonJS modules.

---

## Best practices

1. Always start a project with `npx tsc --init` — never write tsconfig from scratch
2. Always set `strict: true` — no exceptions
3. Keep `outDir` and `rootDir` separate — clean project structure from day one
4. Add `dist/` to `.gitignore` — never commit compiled output
5. Use `"dev": "tsc --watch"` in package.json for fast development feedback
6. For Node projects, install `ts-node` for development (`npx ts-node src/index.ts`)
7. Read TypeScript error messages carefully — the line number, error code, and description all tell you exactly what to fix

---

## Interview questions

### Beginner
**Q: What does the TypeScript compiler `tsc` do?**

Ideal answer: `tsc` does two things: it checks the type correctness of your
code against the rules you have defined, and it transforms TypeScript syntax
(which browsers cannot read) into plain JavaScript (which they can). If type
errors exist, it reports them. The compiled output contains zero TypeScript —
all type annotations are erased.

### Intermediate
**Q: What is `strict: true` in tsconfig.json and why is it important?**

Ideal answer: `strict: true` is a shorthand that enables a bundle of strict
type-checking rules. The two most critical components are `strictNullChecks`
(which forces you to explicitly handle null and undefined rather than letting
them silently crash your app) and `noImplicitAny` (which prevents function
parameters from silently becoming the `any` type, which disables type
checking). Without strict mode, TypeScript misses entire categories of real
bugs that will reach production. It should always be enabled from day one.

### Advanced
**Q: What is the difference between `target` and `module` in tsconfig.json?**

Ideal answer: They control different things. `target` determines what JavaScript
syntax is used in the *compiled output* — for example, `"ES5"` causes TypeScript
to convert arrow functions to regular functions and `const` to `var`, while
`"ES2020"` keeps modern syntax as-is. `module` determines how import and export
statements are compiled — `"commonjs"` converts them to `require()` and
`module.exports` (for Node.js), while `"ESNext"` keeps them as native ES module
syntax (for bundlers like Vite or webpack). These two settings are independent
and you often need to set them differently depending on your deployment target.

---

## Summary

TypeScript cannot run directly — it compiles to JavaScript first. The TypeScript
compiler (`tsc`) both checks types and transforms syntax. `tsconfig.json`
controls the compilation. The most important setting is `strict: true`, which
enables null safety and implicit-any protection. Keep source files in `src/`
and compiled output in `dist/`. Use `ts-node` for local development in Node
projects, and `tsc --watch` for continuous compilation feedback.

---

## 2-Minute Revision Notes

| Concept | Key point |
|---------|-----------|
| Why compile? | Browsers and Node.js can't read TypeScript syntax |
| What `tsc` does | Type-checks your code AND transforms .ts → .js |
| Type erasure | All types removed from output — zero runtime presence |
| `tsconfig.json` | Controls every aspect of how TypeScript compiles |
| `strict: true` | The most important option — enables null safety + no implicit any |
| `strictNullChecks` | Must handle null/undefined explicitly — no silent crashes |
| `noImplicitAny` | Function parameters cannot silently become `any` |
| `target` | Controls JavaScript syntax in the output (ES5, ES2020, etc.) |
| `module` | Controls how imports compile (commonjs for Node, ESNext for bundlers) |
| `outDir` / `rootDir` | Keep compiled JS and TypeScript source in separate folders |
| `ts-node` | Runs TypeScript directly in Node.js during development |
| Production flow | `tsc` → `dist/` → `node dist/index.js` |

---

*Next entry: Primitive types — string, number, boolean, null, undefined, any, unknown, never, void*
