# Lesson Authoring Guide

Use this guide when adding or improving a lesson. Consistency is part of the learner experience.

## Recommended lesson structure

```text
lesson-XX-topic/
├── README.md
├── 01-notes.md
├── 02-exercises/
│   ├── beginner.ts
│   ├── practical.ts
│   └── challenge.ts
├── 03-solutions/
│   └── solutions.ts
└── 04-interview/
    └── qa-reference.md
```

## Lesson README checklist

- Learning outcomes
- Why the concept matters in real work
- Prerequisites and estimated time
- Links to notes, exercises, solutions, and the next lesson
- A short completion checklist

## Writing rules

- Lead with the practical problem before introducing syntax.
- Keep examples small enough to understand, but realistic enough to matter.
- Explain limitations and common mistakes.
- Use descriptive names; avoid `foo`, `bar`, and vague placeholders.
- Make code blocks independently understandable.
- Prefer one excellent exercise over several repetitive ones.

## Exercise rules

- Provide foundation, practical, and challenge levels when the topic benefits from them.
- Separate questions from worked solutions.
- State acceptance criteria or expected behavior.
- Keep solutions compilable and include an explanation of the reasoning.

## Visual rule

Add a diagram only when it makes a relationship easier to understand than a short explanation. Diagrams should explain a compiler flow, type relationship, decision tree, or application boundary—not decorate the page.
