Lesson 2 starts now. We move from _understanding_ TypeScript conceptually to _running_ TypeScript on your machine and understanding how it works under the hood.

---

# 🎓 LESSON 2 — Installing TypeScript, the Compiler, and tsconfig.json

---

## Step 1 — TypeScript vs JavaScript: the technical differences

In Lesson 1 you understood _why_ TypeScript exists. Now you need to understand _what is actually different_ between the two — at the code level.

There are five concrete differences. Everything else builds on these.

**Difference 1 — Static vs dynamic typing**

JavaScript decides what type a variable is while the code is running. TypeScript decides at the moment you write it.

```javascript
// JavaScript — dynamically typed
// The type of 'value' changes at runtime and JS doesn't care
let value = "hello";
value = 42; // Fine — JS allows this completely
value = true; // Still fine
value = [1, 2, 3]; // Also fine — any type, any time
```

```typescript
// TypeScript — statically typed
// TypeScript locks the type from first assignment
let value = "hello"; // TypeScript infers: string
value = 42; // ❌ Error: Type 'number' is not assignable to type 'string'
// The type is locked. You cannot change it.
```

**Difference 2 — Compilation step**

JavaScript runs directly. You write it, the browser or Node.js executes it. TypeScript cannot run directly — it must first be compiled to JavaScript. You write `.ts`, the compiler produces `.js`, and then that `.js` runs.

This is why TypeScript catches errors _before_ runtime. It has an extra step where it inspects your code.

**Difference 3 — New syntax that does not exist in JavaScript**

TypeScript adds syntax to the language. None of this syntax exists in JavaScript — it is erased before the code runs.

```typescript
// All of this is TypeScript-only syntax
let name: string = "Alice";                        // type annotation
let id: number | string = 1;                       // union type
function greet(name: string): string { ... }       // typed function
interface User { name: string; age: number; }      // interface
type Status = 'active' | 'inactive' | 'pending';  // type alias
```

**Difference 4 — Null safety**

In JavaScript, accessing a property on `null` or `undefined` causes the dreaded `TypeError: Cannot read properties of null` crash. TypeScript with `strict: true` forces you to handle null and undefined before they ever reach that point.

```typescript
// TypeScript forces you to handle the case where something might be null
function getUserName(userId: string): string {
  const user = database.find(userId); // might return null

  return user.name;
  // ❌ Error: Object is possibly 'null'
  // TypeScript refuses to let you access .name unless you prove user exists

  if (!user) return "Guest";
  return user.name; // ✅ Now TypeScript knows user is definitely not null
}
```

**Difference 5 — IDE intelligence**

This one is invisible in code but transforms your daily workflow. Because TypeScript knows the type of every variable and every function, your editor gets complete information to provide autocomplete, inline documentation, immediate error highlighting, and safe rename-refactoring across every file in your project simultaneously. JavaScript offers none of this with the same reliability.

---

## Step 2 — Why the compilation step exists

This diagram shows exactly what happens between you writing TypeScript and your code actually running.

![ts workflow](/assets/ts-flow.png)

Browsers and Node.js were built to understand JavaScript. They do not understand TypeScript syntax — `: string`, `interface`, or `<T>` are not valid JavaScript and would throw a syntax error if a browser tried to parse them directly.

The TypeScript compiler (`tsc`) solves this by doing two things in one pass: it checks your types and reports any errors, then it transforms your `.ts` files into `.js` files by stripping all TypeScript-specific syntax away. The output is completely standard JavaScript that any browser or Node version can run.

This is also why TypeScript has zero runtime performance cost — by the time your code runs, TypeScript simply does not exist anymore.

---

## Step 3 — Installing TypeScript and running your first file

**Prerequisites check**

TypeScript requires Node.js. Check that you have it:

```bash
node --version   # Should show v16.0.0 or higher
npm --version    # Should show 7.0.0 or higher
```

If you don't have Node.js, install it from nodejs.org (LTS version).

**Two ways to install TypeScript**

```bash
# Option 1 — Global install (for experimenting and learning)
# Installs tsc as a command available anywhere on your machine
npm install -g typescript

# Verify the install
tsc --version    # Should show Version 5.x.x

# Option 2 — Local install (for actual projects — this is the professional approach)
# Install TypeScript only inside a specific project folder
npm install --save-dev typescript
# Run it with: npx tsc
```

In real projects, always use a local install. This ensures every developer on your team uses the same TypeScript version. Global installs cause "works on my machine" problems.

**Creating and compiling your first TypeScript file**

Create a file called `hello.ts`:

```typescript
// hello.ts

function greetDeveloper(name: string, language: string): string {
  return `Hello ${name}! You are now writing ${language}.`;
}

const message = greetDeveloper("Neeraj", "TypeScript");
console.log(message);
```

Compile it:

```bash
tsc hello.ts
```

This produces a new file `hello.js` in the same folder. Open it and look at what TypeScript generated:

```javascript
// hello.js — this is what TypeScript produced
// Notice: the types are completely gone

function greetDeveloper(name, language) {
  return `Hello ${name}! You are now writing ${language}.`;
}

const message = greetDeveloper("Neeraj", "TypeScript");
console.log(message);
```

Type erasure in action. The `: string` annotations on both parameters and the `: string` return type annotation have all disappeared. The JavaScript output is identical to what you would have written by hand.

Now run the JavaScript:

```bash
node hello.js
# Output: Hello Neeraj! You are now writing TypeScript.
```

**Watching TypeScript catch an error**

Create a file called `error-demo.ts`:

```typescript
// error-demo.ts

function add(a: number, b: number): number {
  return a + b;
}

// Deliberately passing a string where a number is required
add(10, "20");
```

Compile it:

```bash
tsc error-demo.ts
```

TypeScript refuses to compile and shows:

```
error-demo.ts:7:9 - error TS2345:
Argument of type 'string' is not assignable to parameter of type 'number'.

7 add(10, "20");
          ~~~~
```

Learn to read this error format — you'll see it constantly:

- The file name and line number: `error-demo.ts:7:9`
- The error code: `TS2345` (every TypeScript error has a unique code — useful for Googling)
- The human description: `Argument of type 'string' is not assignable to parameter of type 'number'`
- The exact character that is wrong: the `~~~~` underlines `"20"`

Notice also that TypeScript did not produce a `error-demo.js` file. When there are type errors, compilation fails and no output is generated by default. This prevents broken code from reaching production.

---

## Step 4 — tsconfig.json: The command center

Running `tsc hello.ts` works for a single file, but real projects have dozens of files, specific output requirements, and strict settings to configure. That's what `tsconfig.json` is for — it controls how the TypeScript compiler behaves across your entire project.

**Setting up a proper project**

```bash
# Create a project folder
mkdir typescript-project
cd typescript-project

# Initialize npm (creates package.json)
npm init -y

# Install TypeScript locally
npm install --save-dev typescript

# Generate a tsconfig.json with good defaults
npx tsc --init
```

The generated `tsconfig.json` will have most options commented out. Here is a clean, production-ready version with the key options you need to understand:

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

Now let's understand each option from first principles.

---

**`target` — what version of JavaScript to output**

TypeScript can compile to very old JavaScript (ES5, which runs in IE11) or very modern JavaScript (ES2022). The target controls this.

```json
"target": "ES2020"
```

```typescript
// You write this TypeScript
const greet = (name: string) => `Hello ${name}`;

// target: "ES5" output — arrow functions converted to regular functions
var greet = function (name) {
  return "Hello " + name;
};

// target: "ES2020" output — arrow functions kept as-is
const greet = (name) => `Hello ${name}`;
```

For Node.js v16 and above, `ES2020` is a safe and modern choice. For older browser support, you might target `ES5` or `ES2015`.

---

**`module` — how imports and exports work**

```json
"module": "commonjs"
```

This controls the module system in the output. Two important values:

`"commonjs"` — for Node.js. Compiles your `import/export` statements to Node's native `require()` and `module.exports`.

```typescript
// You write (TypeScript / ES module syntax)
import express from "express";
export const router = express.Router();

// Compiled output with "module": "commonjs"
const express = require("express");
const router = express.Router();
exports.router = router;
```

`"ESNext"` or `"ES2020"` — for modern bundlers like Vite, webpack, or for frontend code. Keeps your `import/export` as-is.

---

**`strict` — the most important option. Always `true`. Never negotiate on this.**

```json
"strict": true
```

`strict` is a shorthand that enables a bundle of strict type checks simultaneously. Without it, TypeScript is permissive in ways that let real bugs through. With it, TypeScript is maximally protective.

`strict: true` enables these sub-options:

`strictNullChecks` — the most important one. Without it, `null` and `undefined` are assignable to any type, which defeats a huge part of TypeScript's safety. With it, you must explicitly handle the possibility of null.

```typescript
// WITHOUT strictNullChecks (dangerous — default without strict)
function getUser(id: number): User {
  return database.find(id); // database.find might return undefined
  // TypeScript doesn't care — it trusts you
}
getUser(1).name; // Might crash at runtime if user doesn't exist

// WITH strictNullChecks: true (safe)
function getUser(id: number): User | null {
  return database.find(id) ?? null;
}
const user = getUser(1);
user.name; // ❌ Error: Object is possibly 'null'
if (user) {
  user.name; // ✅ TypeScript knows user is not null inside this block
}
```

`noImplicitAny` — without this, untyped function parameters silently become `any`, which disables all type checking on them. With it, you must always annotate parameters.

```typescript
// WITHOUT noImplicitAny
function greet(name) {
  // 'name' silently becomes 'any' — no checking
  return name.toUpperCase(); // TypeScript trusts you — no protection
}

// WITH noImplicitAny: true
function greet(name) {
  // ❌ Error: Parameter 'name' implicitly has an 'any' type
  return name.toUpperCase();
}
function greet(name: string) {
  // ✅ Must annotate
  return name.toUpperCase();
}
```

---

**`outDir` and `rootDir` — organizing your files**

```json
"outDir": "./dist",
"rootDir": "./src"
```

Without these, TypeScript puts compiled `.js` files next to your `.ts` files, which becomes messy. These options create a clean separation:

```
typescript-project/
├── src/              ← Your TypeScript source files (rootDir)
│   ├── index.ts
│   ├── utils.ts
│   └── types.ts
├── dist/             ← Compiled JavaScript (outDir) — never edit these manually
│   ├── index.js
│   ├── utils.js
│   └── types.js
├── tsconfig.json
└── package.json
```

The `dist` folder is what you deploy to production. The `src` folder is what you work in.

---

**`esModuleInterop` — better import compatibility**

```json
"esModuleInterop": true
```

Without this, importing CommonJS modules (which is most of npm) requires an ugly syntax:

```typescript
// Without esModuleInterop
import * as express from "express"; // awkward
import * as fs from "fs";

// With esModuleInterop: true
import express from "express"; // clean, natural
import fs from "fs";
```

Always enable this. There is no downside.

---

**`include` and `exclude` — which files to compile**

```json
"include": ["src/**/*"],
"exclude": ["node_modules", "dist"]
```

`include: ["src/**/*"]` tells TypeScript to compile every file inside the `src` folder, recursively. `**` means "any subfolder" and `*` means "any file."

`exclude: ["node_modules", "dist"]` prevents TypeScript from trying to compile the thousands of files in `node_modules` or your already-compiled output in `dist`.

**Running the compiler on your whole project**

Once `tsconfig.json` exists, you just run `tsc` without arguments:

```bash
npx tsc           # Compile once
npx tsc --watch   # Watch mode — recompile on every file save (use this during development)
```

**Adding build scripts to package.json**

```json
{
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "start": "node dist/index.js"
  }
}
```

```bash
npm run build   # Compile everything once (for production)
npm run dev     # Watch mode (for development)
npm run start   # Run the compiled output
```

---

## Step 5 — Real-world project setup

**For a React project (using Vite — the modern standard)**

```bash
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
npm install
npm run dev
```

Vite generates a `tsconfig.json` pre-configured for React. You get TypeScript working out of the box — no manual setup needed.

**For a Node/Express project**

```bash
mkdir my-node-app
cd my-node-app
npm init -y
npm install --save-dev typescript ts-node @types/node
npx tsc --init
mkdir src
```

Note: `ts-node` is a tool that runs TypeScript files directly in Node.js without a manual compile step — extremely useful during development.

```bash
# With ts-node installed, you can run:
npx ts-node src/index.ts   # Runs TypeScript directly — no compile step needed during dev
```

The real project workflow is:

- Development: `ts-node` or `tsc --watch` — instant feedback
- Production: `tsc` to compile to `dist/`, then `node dist/index.js` to run

---

## Step 6 — Exercises

**Conceptual questions**

1. What are the two things the TypeScript compiler `tsc` does when you run it? Name both — most beginners only name one.

2. What is the difference between `target` and `module` in tsconfig.json? Give a one-sentence explanation of each.

3. Why is `strict: true` described as "the most important setting"? What are the two most critical sub-checks it enables, and what does each one catch?

4. A developer says: "I'll set `strict: false` to avoid all the extra errors TypeScript generates. My code is simple so I don't need strict mode." What is wrong with this reasoning?

---

**Compiler output exercise — predict the output**

Without running anything, look at this TypeScript and predict exactly what the compiled JavaScript will look like:

```typescript
interface Config {
  host: string;
  port: number;
  debug: boolean;
}

function startServer(config: Config): void {
  console.log(`Starting on ${config.host}:${config.port}`);
  if (config.debug) {
    console.log("Debug mode enabled");
  }
}

const serverConfig: Config = {
  host: "localhost",
  port: 3000,
  debug: true,
};

startServer(serverConfig);
```

Questions: (a) What will the compiled JavaScript look like? Write it out. (b) Which lines change? Which lines stay exactly the same? (c) Does the `Config` interface appear anywhere in the output?

---

**Error reading exercise**

Read this TypeScript error message and answer the questions below it:

```
src/app.ts:14:25 - error TS2339:
Property 'username' does not exist on type 'User'.

14   console.log(currentUser.username);
                             ~~~~~~~~
```

Questions: (a) In which file and on which line did the error occur? (b) What is error code TS2339 telling you conceptually? (c) What is the most likely cause — what did the developer probably mean to write?

---

**tsconfig detective**

This tsconfig.json has three problems. Identify each one and explain why it is wrong:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": false,
    "outDir": "./src",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

---

**Hands-on task (do this now)**

This is not a written exercise — actually do it on your machine:

1. Install TypeScript globally with `npm install -g typescript`
2. Create a folder called `ts-practice`
3. Create a file `first.ts` with a function that takes a `name: string` and `age: number` and logs a greeting
4. Deliberately introduce a type error (pass a string where a number is expected)
5. Run `tsc first.ts` and read the error message
6. Fix the error and compile successfully
7. Open the compiled `first.js` and confirm the types are gone

Report back what the error message said, and paste both your `.ts` file and the compiled `.js` file.

---
