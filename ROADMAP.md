# TypeScript Mastery Roadmap

<p align="center">
  <img src="./assets/roadmap.png" alt="TypeScript Mastery: a five-phase, thirty-three-lesson roadmap" width="100%" />
</p>

TypeScript Mastery is a 33-lesson curriculum designed to take a developer from first principles to production-ready and enterprise-scale TypeScript patterns.

> **Current build status:** Foundations is published and Intermediate is underway. The roadmap below is the target curriculum; availability is updated as lessons, exercises, solutions, and assessments are completed.

## How to read this roadmap

| Status | Meaning |
| --- | --- |
| **Available** | The lesson is ready to study. |
| **In progress** | Content is currently being prepared or standardized. |
| **Planned** | The lesson belongs to the published curriculum but has not been released yet. |

## Phase 1 — Foundations

**Goal:** Build a confident base in TypeScript syntax, the compiler, and everyday data modeling.

**Status:** 7 of 7 lessons available

| # | Lesson | Outcome |
| ---: | --- | --- |
| 01 | What is TypeScript? | Explain what TypeScript is, what it does, and when it helps. |
| 02 | Installation, Compiler, and `tsconfig` | Configure a TypeScript project and understand the compiler's role. |
| 03 | Primitive Types | Type common values safely and intentionally. |
| 04 | Arrays | Model and work with typed collections. |
| 05 | Objects and Type Aliases | Describe data shapes using object types and aliases. |
| 06 | Functions | Type inputs, outputs, optional values, and reusable behavior. |
| 07 | Interfaces | Create clear contracts for objects and application boundaries. |

**Phase milestone:** Complete the Foundations assessment and explain the difference between type inference, annotations, aliases, and interfaces in your own words.

## Phase 2 — Intermediate

**Goal:** Model more realistic application states and create types that can be reused safely.

**Status:** In progress

| # | Lesson | Outcome |
| ---: | --- | --- |
| 08 | Union and Intersection Types | Model values that can take multiple forms and combine contracts safely. |
| 09 | Generics | Write reusable functions, types, and components without losing type information. |
| 10 | Enums | Model named sets of values and understand when simpler alternatives are better. |
| 11 | Tuples | Represent fixed-length, position-aware data. |
| 12 | Utility Types | Transform existing types with TypeScript's built-in utilities. |
| 13 | Type Guards and Narrowing | Safely refine uncertain values at runtime. |
| 14 | Modules | Organize code and types across files without losing clarity. |

**Phase milestone:** Build a small typed data-processing feature using unions, generics, utility types, and narrowing.

## Phase 3 — Advanced

**Goal:** Understand the expressive parts of TypeScript's type system and use them deliberately.

**Status:** Planned

| # | Lesson | Outcome |
| ---: | --- | --- |
| 15 | Advanced Generics | Apply constraints, defaults, and reusable abstractions. |
| 16 | Conditional Types | Build types that adapt based on other types. |
| 17 | Mapped Types | Transform object types property by property. |
| 18 | Indexed Access Types | Extract and reuse types from existing structures. |
| 19 | Template Literal Types | Create safe string patterns and derived API names. |
| 20 | Declaration Files | Type JavaScript libraries and understand ambient declarations. |

**Phase milestone:** Design a reusable type utility layer for a small application domain.

## Phase 4 — Production TypeScript

**Goal:** Apply TypeScript where application boundaries, data validation, and maintainability matter.

**Status:** Planned

| # | Lesson | Outcome |
| ---: | --- | --- |
| 21 | React + TypeScript | Type components, props, state, events, and reusable UI patterns. |
| 22 | Next.js + TypeScript | Use TypeScript effectively in a modern full-stack React framework. |
| 23 | Node.js + TypeScript | Structure server-side TypeScript code and its runtime boundaries. |
| 24 | Express + TypeScript | Type routes, middleware, requests, responses, and errors. |
| 25 | API Typing | Design dependable request and response contracts. |
| 26 | Form Typing | Model form values, validation states, and field errors safely. |
| 27 | Authentication Typing | Represent identities, permissions, sessions, and authorization boundaries. |
| 28 | Database Typing | Keep persistence models and application types clear and reliable. |

**Phase milestone:** Build a typed service with validated request data, explicit API contracts, and predictable error handling.

## Phase 5 — Enterprise TypeScript

**Goal:** Design TypeScript systems that stay understandable as teams, features, and codebases grow.

**Status:** Planned

| # | Lesson | Outcome |
| ---: | --- | --- |
| 29 | Type Architecture | Organize types by ownership, domain, and application boundary. |
| 30 | Reusable Type Systems | Build small, discoverable type utilities that teams can trust. |
| 31 | Type-Safe APIs | Connect runtime validation and static contracts across services. |
| 32 | Scalable Patterns | Apply conventions that make large TypeScript codebases easier to change. |
| 33 | Enterprise Project Structures | Choose a project structure that supports teams, testing, and growth. |

**Phase milestone:** Design the type architecture for a multi-feature application with shared contracts and clear ownership.

## Milestone projects

The projects are intentionally cumulative: each one uses the practices introduced earlier in the course.

| Project | Focus | Recommended point in the journey |
| --- | --- | --- |
| **1. Typed Todo API** | CRUD operations, Express, request typing, response typing | After Phase 4 begins |
| **2. Typed Express Application** | Middleware, validation, authentication, error handling | After Phase 4 |
| **3. Full-Stack Typed Application** | React, Next.js, Node.js, shared types, production architecture | After Phase 5 |

## Completion standard

A phase is complete only when it includes more than lesson notes. Every completed phase should provide:

- Clear lesson navigation and learning outcomes
- Practical exercises and worked solutions
- A phase assessment
- A cumulative check that connects prior learning
- Compilable solution code where code is provided
- A practical example or project connection

This is how TypeScript Mastery stays a dependable learning resource rather than a collection of disconnected notes.

---

Choose your path in [START-HERE.md](./START-HERE.md) or return to the [repository home page](./README.md).
