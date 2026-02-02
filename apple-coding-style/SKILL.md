---
name: apple-coding-style
description: >
  Apply Swift coding style and conventions (naming, file structure, error handling, concurrency patterns).
  Use this skill when writing or refactoring Swift code.
---

# Swift Coding Style (Apple platforms)

## When to use
Use this skill when:
- writing new Swift code
- refactoring existing code
- creating public APIs in packages
- adding async/await or actors
- creating models, services, view models, or tests

## General principles
- Prefer clarity over cleverness
- Small types with single responsibility
- Avoid hidden side effects

## Naming
- Types/Protocols/Enums: `UpperCamelCase`
- Functions/vars: `lowerCamelCase`
- See and follow `references/swift-guidelines.md` for canonical examples, patterns and more details.

## Files & structure
- Use folder not group when possible.
- Folder name use `UpperCamelCase`.
- File name use `UpperCamelCase` and follow general guidelines.
- One primary type per file (exceptions: small related types)
- File name matches main type
- Keep extensions grouped by purpose:
  - `// MARK: - Lifecycle`
  - `// MARK: - Public`
  - `// MARK: - Private`
  - `// MARK: - Helpers`

## API design
- Prefer value types unless identity/mutation is required
- Prefer dependency injection (initializer) over singletons
- Expose protocols, hide concrete implementations
- Avoid global state

## Error handling
- Use `throws` for recoverable failures
- Prefer typed errors (`enum FeatureError: Error`) with actionable cases. Should always suffixed with `Error` (`enum FeatureError: Error`).
- Never swallow errors silently; log with `os.log` or propagate intentionally.

## Concurrency (Swift 6 mindset)
- Use structured concurrency first (`async let`, task groups)
- Mark UI-facing types `@MainActor`
- Use actors for shared mutable state
- Avoid `Task.detached` unless you fully understand isolation implications
- Set first new `struct` and `enum` `Sendable`. Avoid `@unchecked Sendable` unless unavoidable and documented.

## Formatting rules (high signal)
- Early returns to reduce nesting
- Keep functions short (target < ~50 lines, pragmatic)
- Prefer guard for preconditions
- Avoid huge parameter lists except for DTO; group into value types when it clarifies intent.

## Testing conventions
- When created stubs or mocks, suffixed the name with `Mock` (`ArticleRepositoryMock`).
- When created mocks, always place them inside a folder named `Mock`.
- Tests mirror module structure
- AAA pattern (Arrange / Act / Assert) or Given/When/Then consistently
- Make async tests deterministic (no arbitrary sleeps)
- Use `os.log` framework to log. Never use `print`.

## References
- See `references/swift-guidelines.md` for canonical examples, patterns and more details.