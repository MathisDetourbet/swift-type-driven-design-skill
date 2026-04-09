# Swift Type-Driven Design

This repository provides a single AI coding-agent skill that teaches Type-Driven Design in Swift.

**Full skill content:** [`skills/type-driven-design/SKILL.md`](skills/type-driven-design/SKILL.md)

## When the skill applies

Any time you are writing, reviewing, or refactoring Swift code and you encounter:

- Domain model design (`struct`, `enum`, identity types)
- Failable initializers or input validation
- Optionals / `Bool` flags acting as implicit state machines
- Decoding external data (JSON, database rows, deep links)
- `fatalError("unreachable")` or force-unwraps in switch defaults
- Repeated `guard !array.isEmpty` / `guard string.count > 0` checks
- Stringly-typed APIs carrying invariants the compiler cannot see

...load `skills/type-driven-design/SKILL.md` for the full pattern catalog: domain primitives, sum/product types, `NonEmpty` collections, phantom-typed identities, `Never` for impossible failures, and parse-at-the-boundary decoders.

## Core principle

> Parse, don't validate. Information acquired through validation must be preserved by the type system, not discarded.

## Credit

All ideas and examples are distilled from the [Type-Driven Design with Swift](https://swiftology.io/collections/type-driven-design/) series by **Alex Ozun** at swiftology.io.
