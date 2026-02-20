# Concurrency Guidelines

## Preferred model
- Use `async/await` as default.
- Use structured concurrency (`Task`, task groups) intentionally.
- Use `@MainActor` for UI state ownership.
- Use actors for shared mutable state.

## Safety rules
- Avoid data races by design.
- Avoid long-running work on the main actor.
- Handle cancellation for network/persistence operations.
- Keep async functions single-purpose.

## Default isolation and `Sendable`
If a target uses default actor isolation (for example `.defaultIsolation(MainActor.self)`), data types intended to cross actor boundaries should be explicitly marked `nonisolated` when declared as `Sendable`.

Example:

```swift
nonisolated enum User: Sendable {
    case authenticated
    case anonymous
}
```

Use this for immutable data containers (DTOs, domain enums, simple value models) that should not be main-actor isolated by default.

## Avoid
- Unstructured detached tasks without lifecycle control.
- Mixing callbacks and `async/await` unless bridging legacy APIs.
