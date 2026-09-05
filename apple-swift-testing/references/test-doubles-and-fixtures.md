# Test Doubles and Fixtures

## Choose the role before writing the type

| Role | Purpose | Typical use |
| --- | --- | --- |
| Stub | Supply selected results/errors | Empty list, failure, restricted state |
| Spy | Record meaningful interactions | Output event or submitted payload |
| Fake | Small working implementation | In-memory store when stateful behavior matters |
| Mock | Configurable behavior and/or expected interactions | Repository with queued outcomes and call recording |
| Fixture | Deterministic input/output data | An article, session, or persisted record |

The repository naming convention uses the `Mock` suffix for test dependency doubles and places them in `Tests/<Module>Tests/Mock/`. Describe the role through the implementation and test, without renaming established types as unrelated cleanup. Preview-only dependencies may use `PreviewStub` in feature-local preview support; they are not assertion-bearing test mocks.

Create a double for a real dependency boundary. Pure functions, value types, and simple internal collaborators can be tested directly without adding a protocol for each type.

## Contract and configuration

- Conform to the production interface and preserve its isolation, cancellation, error, ordering, and ownership semantics that consumers rely on.
- Configure success and error outcomes explicitly. Distinguish successful empty content from an unconfigured dependency.
- Record only interactions needed by the test: request arguments, emitted events, call counts, or significant ordering. Do not spy on every private implementation detail.
- Prefer a sequence of outcomes for retry/pagination behavior. Unexpected extra calls should produce a clear test failure or a recognizable unconfigured error, not silent default success.
- Use a fake when behavior requires a coherent store rather than an ever-growing collection of mock closures. Keep it bounded to the contract under test.
- Use `@testable import` for feature internals. Do not make production implementation types public for test access.

## Isolation and deterministic execution

Each test owns new mutable doubles, stores, outputs, and fixture graphs. Do not use static mutable state, production `UserDefaults.standard`, a live keychain/App Group, or real backend credentials.

Match the production contract: an actor can protect an asynchronous `Sendable` repository double; a synchronous `@MainActor` output generally needs a main-actor implementation instead. Do not remove isolation or add `@unchecked Sendable` just to silence a test compiler error. Remember that actor isolation still permits reentrancy across suspension points.

Use injectable clocks/IDs and explicit async control where timing matters. Avoid arbitrary sleeps to wait for calls. A controllable continuation must be resumed at most once, handle cancellation, and be cleaned up so tests cannot hang. Do not build a custom scheduler if a simple synchronous result or injected clock is enough.

## Small repository example

These illustrative files form a minimal example. Real tests import the actual contract/model; do not duplicate them in the test target. Here the repository returns `ArticleSummary` values and accepts a search query. Each type belongs in its own file.

`ArticleSummary.swift` — illustrative production value:

```swift
struct ArticleSummary: Equatable, Sendable {
    let id: String
    let title: String
}
```

`ArticleRepository.swift` — illustrative production contract:

```swift
protocol ArticleRepository: Sendable {
    func fetch(query: String) async throws -> [ArticleSummary]
}
```

`ArticleFixtures.swift` — test data factory:

```swift
enum ArticleFixtures {
    static func make(
        id: String = "article-001",
        title: String = "A sample article"
    ) -> ArticleSummary {
        ArticleSummary(id: id, title: title)
    }
}
```

`Mock/ArticleRepositoryMockError.swift`:

```swift
enum ArticleRepositoryMockError: Error, Equatable, Sendable {
    case offline
    case unconfigured
}
```

`Mock/ArticleRepositoryMock.swift`:

```swift
actor ArticleRepositoryMock: ArticleRepository {
    private var outcomes: [Result<[ArticleSummary], ArticleRepositoryMockError>]
    private(set) var requestedQueries: [String] = []

    init(outcomes: [Result<[ArticleSummary], ArticleRepositoryMockError>]) {
        self.outcomes = outcomes
    }

    func fetch(query: String) async throws -> [ArticleSummary] {
        try Task.checkCancellation()
        requestedQueries.append(query)
        guard !outcomes.isEmpty else {
            throw ArticleRepositoryMockError.unconfigured
        }
        return try outcomes.removeFirst().get()
    }
}
```

Configure a first `.failure(.offline)` then `.success([ArticleFixtures.make()])` when testing a consumer's retry behavior. The real test should assert the consumer's recovered state and meaningful requested payload, not merely prove this mock consumes its own array. Match the real repository's error contract; use a dedicated unconfigured diagnostic only for unexpected test setup/calls.

## Fixtures and previews

Keep fixtures small, deterministic, and intention-revealing. Fix IDs/dates that affect results; allow named overrides for the property under test. Prefer fresh value graphs over a mutable shared singleton.

Default ownership is the consuming test target. If tests and previews truly share substantial data builders, extract a focused development-support target without making production interfaces depend on it. Never import a test target from production/preview code. Avoid introducing a support package for one trivial inline value.

A preview stub displays a chosen scenario; a test mock may record and assert behavior. They may share immutable fixture builders but should not share mutable instances. Keep large sample assets and assertion frameworks out of production dependency closures.

## Review checks

- Does the double implement the same contract the production consumer receives?
- Are relevant errors/cancellation preserved and unexpected calls visible?
- Can tests run concurrently without affecting each other or production storage?
- Are assertions about externally meaningful behavior rather than implementation wiring?
- Is a shared fixture/support target justified by actual reuse?
- Are preview and test dependency graphs independent of the app target and live services?
