# Type-Driven Design — Claude Code Skill

A [Claude Code](https://code.claude.com/) skill that teaches Claude to apply **Type-Driven Design** when writing and reviewing Swift code: using the type system to make illegal states unrepresentable, parsing rather than validating, and eliminating defensive runtime checks.

## What it does

Once installed, Claude will automatically reach for type-driven patterns when you:

- Design Swift domain models
- Write failable initializers or validation logic
- Review code with optionals, `Bool` flags, or `fatalError("unreachable")` defaults
- Mention "type-driven design", "parse don't validate", "smart constructors", "phantom types", "algebraic data types", or similar concepts

The skill covers seven concrete patterns — domain primitives, sum types for OR relations, product types for AND relations, `NonEmpty` collections, phantom-typed identities, `Never` for impossible failures, and parse-at-the-boundary decoders — plus a review checklist and a list of anti-patterns to watch for.

## Credit

The content of this skill is distilled from the excellent **[Type-Driven Design with Swift](https://swiftology.io/collections/type-driven-design/)** article series by **[Alexey Naumov](https://swiftology.io/)** at swiftology.io. All credit for the ideas, patterns, and examples belongs to the original author — this skill exists only to make those ideas available to Claude Code as loadable guidance.

If you find the skill useful, go read the original articles. They are the real thing:

1. [Part 1: Fundamentals of type-driven code](https://swiftology.io/articles/tydd-part-1-fundamentals)
2. [Part 2: Type-safe validation](https://swiftology.io/articles/tydd-part-2)
3. [Part 3: Witness pattern — type-safe access control](https://swiftology.io/articles/tydd-part-3)
4. [Part 4: Domain modeling with types](https://swiftology.io/articles/tydd-part-4)
5. [Part 5: Making illegal states unrepresentable](https://swiftology.io/articles/making-illegal-states-unrepresentable)

Further reading that inspired the series:

- Alexis King, [Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
- Yaron Minsky, [Effective ML: Make illegal states unrepresentable](https://blog.janestreet.com/effective-ml-revisited/)

## Installation

### Option A — Clone into your skills directory

```bash
git clone https://github.com/MathisDetourbet/swift-type-driven-design-skill.git \
  ~/.claude/skills/type-driven-design
```

### Option B — Copy just the SKILL.md

```bash
mkdir -p ~/.claude/skills/type-driven-design
curl -fsSL https://raw.githubusercontent.com/MathisDetourbet/swift-type-driven-design-skill/main/SKILL.md \
  -o ~/.claude/skills/type-driven-design/SKILL.md
```

Then restart Claude Code (or start a new session). Verify by running `/type-driven-design` or by asking Claude to review a Swift domain model — the skill should auto-load from its description.

## Usage

The skill is **auto-invocable**: Claude will pull it in whenever it sees a task that matches its description. You can also invoke it explicitly:

```
/type-driven-design
```

Or reference it in a prompt:

> Review this struct using the type-driven-design skill — I think it has some illegal states.

## Contributing

Issues and PRs welcome. If you spot a pattern from the original series that's missing, or have an improved Swift example, open a PR. Please keep the skill focused on **Swift** — the patterns generalize but the audience here is Swift developers.

## License

Content is provided as-is. The underlying ideas are the work of Alexey Naumov; see the [Credit](#credit) section for links to the source material.
