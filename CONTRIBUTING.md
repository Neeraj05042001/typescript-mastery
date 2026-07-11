# Contributing to TypeScript Mastery

Thank you for helping make TypeScript Mastery more useful for learners. The goal is not to publish the most material—it is to publish the clearest, most practical material.

Before contributing, read the [course roadmap](./ROADMAP.md), the [live progress](./PROGRESS.md), and the [community guidelines](./CODE_OF_CONDUCT.md).

## Good ways to contribute

- Report an unclear explanation, incorrect example, or broken link.
- Suggest a more realistic exercise or project scenario.
- Improve grammar, accessibility, or code readability.
- Add a missing edge case to a worked solution.
- Propose a diagram when it makes a difficult relationship easier to understand.

For larger curriculum changes, open an issue first. This keeps the learning path intentional and prevents duplicate work.

## Before opening an issue

Please search existing issues and include:

1. The lesson, file, or section affected.
2. What is confusing, incorrect, or missing.
3. A concrete improvement when you have one.
4. Code, screenshots, or error output when relevant.

## Content standards

Keep contributions aligned with the course principles:

- Prefer practical examples over abstract definitions.
- Teach one primary idea at a time.
- Explain *why* a type is useful, not only its syntax.
- Use realistic names and scenarios; avoid placeholder-only examples when a domain example would teach more.
- Keep beginner-facing language clear and free of unnecessary jargon.
- Do not add generated filler, copied tutorials, or unverified code.

## Code and lesson changes

- Preserve the phase-first, lesson-first repository structure.
- Do not rename or move lessons without discussing the curriculum impact first.
- Keep solution code compilable where applicable.
- Do not include secrets, personal data, or credentials in examples.
- Add or update navigation when creating a lesson, assessment, or project.

Run the repository check before opening a pull request:

```bash
npm ci
npm run check
```

## Pull request checklist

Before submitting a pull request, confirm that:

- [ ] The change has a clear learning benefit.
- [ ] Links and navigation are correct.
- [ ] Code examples are readable and accurate.
- [ ] Solutions compile when code is included.
- [ ] The change does not duplicate an existing lesson or exercise.
- [ ] The pull request description explains the learner-facing impact.

## Scope and review

Small, focused pull requests are easier to review and more likely to be merged. A pull request may be asked to change direction when it conflicts with the curriculum, makes a lesson harder to scan, or introduces complexity without improving learning outcomes.

By participating, you agree to follow the [Code of Conduct](./CODE_OF_CONDUCT.md).
