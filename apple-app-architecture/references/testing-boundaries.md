# Testing Boundaries

## Interfaces

Protocols, events, and plain data contracts generally need no test target. If a contract type has meaningful value behavior (for example encoding or normalization that legitimately belongs to the contract), test that behavior. Do not move business workflows into an interface to make them shared or easy to test.

## Feature implementations

Include a unit test target for a new Feature package. Test behavior when present: presentation transitions, validation, mapping, paging policies, output events, and error handling. A simple declarative view does not need an artificial ViewModel or a meaningless test just to satisfy a folder convention.

- Use Swift Testing for unit tests following the project's test conventions.
- Import implementation internals with `@testable import FooFeature`; do not make them public for tests.
- Inject repository/input/output test doubles through constructors or configuration. Tests must not require the app composition root, a sibling feature implementation, production credentials, or live network calls.
- Cover relevant success, empty, failure, cancellation, and retry behavior, according to the object's responsibility.
- Verify observable results and emitted intents. Assert dependency interactions only when ordering/count/payload is part of the behavior.
- Keep test doubles substitutable for the contract, including isolation and error semantics. Do not assert private implementation structure.

Introduce a ViewModel when independently testable or reusable presentation logic needs an owner. A pure validator or page-ordering builder can be tested directly without a ViewModel, UI harness, or repository layer.

## Infrastructure and utilities

Test meaningful provider mapping, parsing, persistence rules, and boundary failures in their owning packages using controlled external dependencies. Test a shared utility's behavior independently of consuming features. Keep live integration tests explicit and separate from fast unit tests.

## App composition

Keep tests for output-to-route mapping, deeplink parsing, and platform entry coordination at the app boundary. Feature tests prove the emitted intent; app tests prove how that intent becomes a destination. Do not require one large app test to validate every internal feature rule.

Previews use safe deterministic dependencies. UI tests follow the project's existing policy and the user's scope; this skill does not require a UI suite for every feature.

## Execution

Run the owning tests and build the affected assembly for an appropriate supported destination. An iOS-only package is not required to run `swift test` on macOS; use a configured Xcode scheme/simulator instead. Distinguish passing tests, compilation checks, and unavailable checks in the final report.
