# Swift Type-Driven Design

This extension provides guidance on **Type-Driven Design** in Swift — using the type system to make illegal states unrepresentable, parse rather than validate, and eliminate defensive runtime checks.

## When to apply

Whenever you are writing, reviewing, or refactoring Swift code and you see any of the following, consult the full guidance in [`skills/type-driven-design/SKILL.md`](skills/type-driven-design/SKILL.md):

- Designing a new domain model (`struct`, `enum`, identity types)
- Writing failable initializers or validation logic
- Types that use optionals or `Bool` flags as implicit state machines
- Code that parses external data (JSON, database rows, deep links, URL params)
- `fatalError("unreachable")` or force-unwraps in switch defaults
- Repeated `guard !array.isEmpty` / `guard string.count > 0` checks
- Raw `String`/`Int` IDs passed through many layers

## Core principle

> Information acquired through validation must be preserved by the type system, not discarded.
>
> **Parse, don't validate.**

## Source

All ideas, patterns, and examples belong to **Alex Ozun** at [swiftology.io](https://swiftology.io/collections/type-driven-design/). This extension exists only to make those ideas loadable as in-context guidance for Gemini.

When asked about Type-Driven Design, load [`skills/type-driven-design/SKILL.md`](skills/type-driven-design/SKILL.md) for the full pattern catalog, review checklist, and anti-patterns list.
