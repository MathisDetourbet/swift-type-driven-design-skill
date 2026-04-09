---
name: type-driven-design
description: Expert guidance on Type-Driven Design in Swift — using the type system to make illegal states unrepresentable, parse rather than validate, and eliminate defensive runtime checks. Use when designing domain models, writing failable initializers, reviewing Swift types that use optionals/booleans as flags, adding validation logic, or whenever the user mentions "type-driven design", "parse don't validate", "make illegal states unrepresentable", "smart constructors", "phantom types", "domain modeling", or "algebraic data types" in Swift.
---

# Type-Driven Design in Swift

> **Credit:** This skill is distilled from the [Type-Driven Design with Swift](https://swiftology.io/collections/type-driven-design/) article series by **Alexey Naumov** at swiftology.io. All ideas, patterns, and examples belong to the original author; this skill exists only to make those ideas loadable as Claude Code guidance. Read the originals — they are the real thing.

Type-Driven Design (TyDD) is a discipline of using Swift's type system to **retain proofs of validation and domain invariants**, so that illegal states become compile-time errors instead of runtime bugs. It replaces scattered defensive checks with structural guarantees.

## Core Principle

> Information acquired through validation must be preserved by the type system, not discarded.

When you call `isValid(email)` and the call site gets a `Bool`, the proof that validation happened is **lost** the moment you pass the raw `String` to the next function. Every downstream function is forced either to re-validate or to trust an invariant the compiler cannot see. Type-Driven Design fixes this by encoding the invariant in a dedicated type.

**The mantra:** *Parse, don't validate.* A parser returns a more structured type on success; a validator returns a boolean and throws the structure away.

## When to Apply

Reach for TyDD when you see:

- Optionals used as pseudo-enums (`struct { let a: A?; let b: B? }` where exactly one should be set)
- `Bool` flags that gate other fields (`isPremium` + `premiumExpiry: Date?`)
- Repeated `guard !array.isEmpty` / `guard string.count > 0` checks
- Raw `String`/`Int` passed through many layers with re-validation at each
- `fatalError("should never happen")` in switch defaults
- Functions that take 5 parameters where only 3 combinations are legal
- `try!` or force-unwraps "because we already checked"

## The Core Patterns

### 1. Domain Primitives (Smart Constructors)

Wrap stringly/numerically-typed values in a struct with a **failable initializer**. Once you hold the type, the invariant is proven.

```swift
struct Email {
    let rawValue: String
    init?(_ rawValue: String) {
        guard rawValue.isValidEmail else { return nil }
        self.rawValue = rawValue
    }
}

struct Password {
    let rawValue: String
    init?(_ rawValue: String) {
        guard rawValue.count >= 8 else { return nil }
        self.rawValue = rawValue
    }
}

// Downstream APIs accept only validated types:
func logIn(email: Email, password: Password) async throws { ... }
```

The call site parses once at the boundary; every layer below is relieved of re-validation.

### 2. Sum Types for "Exactly One Of" (OR)

Whenever a comment says "either X or Y, but not both", that is an `enum`, not two optionals.

```swift
// ❌ Permits (nil, nil) and (some, some) — both illegal
struct ContactMethod {
    let email: Email?
    let phone: PhoneNumber?
}

// ✅ Only the legal shapes exist
enum ContactMethod {
    case email(Email)
    case phone(PhoneNumber)
    case both(Email, PhoneNumber)
}
```

Exhaustive `switch` then forces every consumer to handle every legal case — and no illegal ones.

### 3. Product Types for "All Of" (AND)

Use `struct` (prefer value semantics, reserve `final class` for genuine reference needs) for combinations where every field is required *together*.

```swift
struct Credentials {
    let email: Email
    let password: Password
}
```

### 4. Eliminate "Zero" From Types That Should Never Be Empty

Replace nullable/empty-permissive types with non-empty variants:

| Permissive | Proven |
|---|---|
| `[T]` | `NonEmpty<[T]>` |
| `String` | `NonEmpty<String>` |
| `Int` | `PositiveInt` |
| `Date?` (with companion flag) | `Date` directly on the correct sum-type case |

```swift
struct NonEmptyArray<Element> {
    let first: Element
    let rest: [Element]

    var all: [Element] { [first] + rest }
}

func send(batch: NonEmptyArray<Event>) { /* no isEmpty check needed */ }
```

### 5. Phantom Types for Type-Safe Identities

Prevent mixing unrelated IDs at compile time without runtime cost.

```swift
struct ID<Owner>: Hashable, RawRepresentable {
    let rawValue: String
}

enum User {}
enum Song {}

func play(song: ID<Song>) { ... }

let userID: ID<User> = .init(rawValue: "u_123")
play(song: userID) // ❌ compile error — exactly what we want
```

### 6. `Never` for Impossible Failures

When an API of a given protocol cannot fail in a specific implementation, encode that in the type.

```swift
protocol Cache {
    associatedtype Failure: Error
    func data(for key: String) -> Result<Data?, Failure>
}

final class InMemoryCache: Cache {
    func data(for key: String) -> Result<Data?, Never> { ... }
}

// At the call site, the error case is statically unreachable:
switch inMemoryCache.data(for: key) {
case .success(let data): ...
// no .failure — compiler knows Never has no values
}
```

### 7. Parse at the Boundary

Untrusted data (JSON, database rows, URL parameters, deep links, cross-module calls) enters through a parsing layer that refuses malformed combinations **once**. Everything downstream operates on the parsed domain model.

```swift
enum PaymentMethod: Decodable {
    case creditCard(CreditCard)
    case giftCard(GiftCard)

    init(from decoder: Decoder) throws {
        let c = try decoder.container(keyedBy: CodingKeys.self)
        let card = try c.decodeIfPresent(CreditCard.self, forKey: .creditCard)
        let gift = try c.decodeIfPresent(GiftCard.self, forKey: .giftCard)
        switch (card, gift) {
        case let (c?, nil): self = .creditCard(c)
        case let (nil, g?): self = .giftCard(g)
        case (.some, .some), (nil, nil):
            throw DecodingError.dataCorruptedError(
                forKey: .creditCard, in: c,
                debugDescription: "Exactly one payment method required"
            )
        }
    }
}
```

## Algebraic Distribution: A Diagnostic Tool

Think of `struct` as multiplication and `enum` as addition. The type `struct { a: (X|Y); b: (P|Q) }` has `(X+Y) * (P+Q) = XP + XQ + YP + YQ` possible shapes. If fewer than all four combinations are legal, the type is **lying**. Distribute it into an enum that lists only the legal shapes.

This is the mechanical way to find hidden illegal states in an existing model.

## Review Checklist

When reviewing or writing Swift domain code, ask:

- [ ] Does every `String`/`Int` field that carries an invariant have a dedicated wrapper type?
- [ ] Are any optionals co-dependent? (If yes → enum.)
- [ ] Are any `Bool` flags gating other fields? (If yes → enum.)
- [ ] Does any array/string need to be non-empty? (If yes → `NonEmpty`.)
- [ ] Are IDs of different entities the same raw type? (If yes → phantom-typed `ID<T>`.)
- [ ] Is parsing of external data concentrated at one boundary?
- [ ] Does any `switch` have a defaulted `fatalError`? (If yes → the type admits states the logic rejects.)
- [ ] Do any functions downstream of validation re-validate the same thing?

## Anti-Patterns to Call Out

- **"Stringly-typed" APIs** — `func fetch(id: String)` where `id` is really a `UserID`.
- **Boolean blindness** — `init(isAdmin: Bool, isActive: Bool, isDeleted: Bool)` should almost always be a sum type.
- **Optional soup** — a struct whose fields are mostly optional is usually a badly-expanded enum.
- **Defensive guards in "impossible" branches** — either the branch is possible (and the type is wrong) or it isn't (and the guard is noise).
- **`try!` to paper over an invariant the type doesn't express.**
- **`fatalError("unreachable")` in enum switches** — add the missing case or narrow the enum.

## Trade-offs and Limits

- **Cost of wrapping:** every domain primitive is extra code. Reserve it for values that carry invariants or cross API boundaries; don't wrap `String` for a label that is only displayed.
- **Decoding friction:** parsing at the boundary means writing `Decodable` conformances by hand when the JSON shape doesn't match the domain shape. This is intentional — the friction is where the illegal states get rejected.
- **API ergonomics:** failable initializers return `Optional`; for richer error reporting, prefer a throwing initializer or a `Result`-returning static factory.
- **Third-party APIs:** you can't force SDKs to accept your wrappers. Convert at the edge and go.
- **Over-modeling:** the goal is to capture *essential* domain invariants, not every conceivable property. If an "invariant" isn't actually enforced anywhere in the business logic, don't encode it.

## How to Use This Skill

When writing or reviewing Swift code:

1. Identify the invariants the code is relying on (read `guard` statements, comments, force-unwraps).
2. Ask: can this invariant be lifted into a type?
3. Apply the smallest pattern that removes the defensive check — usually a wrapper struct or a narrower enum.
4. Work outward from the boundary: parse once, then propagate the parsed type.
5. Delete the now-dead runtime checks.

## References

- [Part 1: Fundamentals of type-driven code](https://swiftology.io/articles/tydd-part-1-fundamentals)
- [Part 2: Type-safe validation](https://swiftology.io/articles/tydd-part-2)
- [Part 3: Witness pattern — type-safe access control](https://swiftology.io/articles/tydd-part-3)
- [Part 4: Domain modeling with types](https://swiftology.io/articles/tydd-part-4)
- [Part 5: Making illegal states unrepresentable](https://swiftology.io/articles/making-illegal-states-unrepresentable)
- Original inspiration: Alexis King, ["Parse, don't validate"](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
- Yaron Minsky, ["Effective ML: Make illegal states unrepresentable"](https://blog.janestreet.com/effective-ml-revisited/)
