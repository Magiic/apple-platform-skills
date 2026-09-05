---
name: apple-coding-style
description: >
  Apply clean, maintainable Swift coding conventions across Apple platforms.
  Use this skill when writing or refactoring Swift code in modules, features,
  view models, repositories, and tests.
---

# Apple Coding Style (Project-Agnostic)

## When to use this skill
Use this skill when:
- Writing new Swift files.
- Refactoring existing Swift code.
- Adding view models, repositories, mappers, or domain models.
- Defining error handling and async flows.
- Improving readability and consistency.

## Core goals
- Prefer clarity over cleverness.
- Keep code easy to test.
- Keep side effects explicit.
- Make error paths first-class.
- Preserve platform-safe behavior on Apple targets.

## General code conventions
- Prefer explicit names that describe intent.
- Keep functions focused and small.
- Prefer value types (`struct`, `enum`) unless reference semantics are required.
- Avoid global mutable state.
- Use `final` for classes unless inheritance is needed.
- Keep access control minimal and intentional (`private`/`internal` by default).
- Avoid force unwraps and force casts in production code.

## File organization
Use this order when applicable:
1. Imports
2. Main type declaration
3. Public API
4. Internal logic
5. Private helpers
6. Extensions (protocol conformances, formatting, mapping)

### Hard file limits
- Exactly one top-level type per file.
- Do not place multiple top-level types in the same file (for example: 2 `struct`, or `struct` + `class`).
- Nested types are allowed, but with strict limits:
- Maximum 3 nested types per parent type.
- Each nested type must not exceed 10 lines.

## Naming rules
- Types: `UpperCamelCase`
- Functions/properties/variables: `lowerCamelCase`
- Boolean names should read as predicates (`isLoading`, `hasAccess`, `canSubmit`)
- Acronyms should stay readable (`URL`, `ID`, `OTP`)
- Avoid ambiguous names like `data`, `item`, `manager` when a specific name exists.

## Error handling
- Use typed errors (`enum`) for domain and feature layers.
- Include actionable context in error cases.
- Map low-level errors to higher-level errors at boundaries.
- Do not leak provider-specific raw errors to UI.
- Avoid silent failure; log and propagate intentionally.

See: `references/error-handling.md`.

## Concurrency and async style
- Prefer `async/await` over callback pyramids.
- Mark UI-facing state handlers with `@MainActor` when required.
- Use actors or isolation boundaries for shared mutable state.
- Handle cancellation explicitly when needed.
- Avoid detached tasks unless there is a strong reason.

### Default isolation + `Sendable` rule
If the module or target uses default actor isolation (for example `.defaultIsolation(MainActor.self)`), and you define `Sendable` types meant to be cross-actor data, mark them `nonisolated`.

Example:

```swift
nonisolated enum User: Sendable {
    case authenticated
    case anonymous
}
```

Use this for immutable DTOs/domain types that should not inherit actor isolation by default.

See: `references/concurrency-guidelines.md`.

## SwiftUI style
- Keep Views declarative and lightweight.
- Do not impose MVVM, VIP, or another internal architecture. A simple screen may keep local UI state and forward actions directly.
- Introduce a ViewModel when presentation logic needs reuse or independent tests; extract focused helpers for pure rules. Keep business workflows out of Views.
- Keep derived view state computed, not duplicated.
- Minimize view-level business logic.
- Prefer small reusable subviews for complex layouts.
- Line-count limit:
- By default, a SwiftUI `View` type must not exceed 300 lines of code, including its helpers and extensions across files (excluding comments, blank lines, and standalone previews).
- A ViewModel (or equivalent state handler) must not exceed 500 lines.
- Split by responsibility before reaching a limit; moving one oversized type into extensions or compressing formatting is not a solution.

See: `references/swiftui-structure.md`.

## Dependencies and boundaries
- Depend on protocols/contracts in feature code.
- Inject dependencies explicitly through initializers or module configuration.
- Avoid hidden singleton dependencies in business logic.
- Keep provider/SDK details in infrastructure modules.

## SOLID in everyday code
- Give each view, ViewModel, validator, mapper, and service one reason to change.
- Extend behavior through focused injected contracts where substitution is useful; do not create a protocol or pass-through layer for every class.
- Keep production implementations and test doubles consistent with the same results, errors, cancellation, and isolation contracts.
- Pass only the data/actions a component needs. Avoid injecting an entire module configuration into a purely visual child.
- Define shared contracts in the owning `*Interface` module; keep implementation-only collaborators internal to `*Feature`.
- Never import another feature implementation to reuse its view or ViewModel. Use its interface and app-provided adapter or view builder.

## Logging and diagnostics
- Log meaningful events at boundaries (network, persistence, auth, critical actions).
- Avoid noisy logs in hot paths.
- Never log secrets or sensitive personal data.
- Include identifiers useful for debugging (non-sensitive IDs, operation names, status).

## Localization and resources
- Keep user-facing strings localizable.
- In Swift packages, use package-safe bundle access (`#bundle`) for localized strings/assets.
- Avoid hard-coded user-visible text in business logic layers.

## Code review checklist
- Naming is clear and intention-revealing.
- Functions are focused and readable.
- Error paths are explicit and typed.
- Async/concurrency boundaries are safe.
- Dependencies are injected and testable.
- UI layer is not carrying business workflows; views stay within 300 lines and extra layers have a concrete purpose.
- Localization and bundle access are correct.

## Definition of done
- Code is readable without extra explanation.
- Error handling is complete for expected failures.
- Concurrency choices are safe and intentional.
- Boundaries and dependency direction are respected.
- Tests are updated where behavior changed.

## References
- `references/swift-file-structure.md`
- `references/error-handling.md`
- `references/concurrency-guidelines.md`
- `references/swiftui-structure.md`
- `references/naming-and-api-design.md`
