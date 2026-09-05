# Dependency Direction

An arrow means a **compile-time dependency**, not a runtime call.

```text
App composition root
  |--> FeedFeature ------> FeedInterface
  |          |----------> QuestionInterface <------ QuestionFeature <--- App
  |          |----------> AppEvaluationInterface <- AppEvaluationFeature <- App
  |--> BookFeature ------> BookInterface
  |          |----------> QuestionCollectionInterface <- QuestionCollectionFeature <- App
  |--> BackendClient ---> BackendClientInterface <---- consuming features
  |--> focused utilities / DesignSystem
```

The app provides concrete objects and view factories. Feed can display Question UI while compiling against `QuestionInterface` alone.

## Allowed edges

| Consumer | Allowed dependency | Condition |
| --- | --- | --- |
| `FooFeature` | `FooInterface`, `BarInterface` | Consumes public contracts, never implementation symbols |
| `FooFeature` | Design system / focused technical utility | Utility does not hide feature dependencies |
| `FooInterface` | Another interface | Its public contract actually needs those types; graph stays acyclic |
| Interface | Focused utility / Apple framework | Required by its contract, with a reviewed transitive cost |
| Client / infrastructure | Interface + provider SDK | Implements that interface and contains SDK mapping |
| App | Interfaces + features + clients + utilities | Composition, lifecycle, routing, platform adapters |

## Forbidden edges

- `FooFeature -> BarFeature`, including imports, package products, re-exports, or a helper that transitively imports `BarFeature`.
- `FooInterface -> FooFeature`, any other feature implementation, or a concrete provider client.
- Any package depending on the app target.
- A shared/common package depending on product feature implementations.
- Interface cycles, even if neither side imports implementations.

A technical utility may contain implementation. A media compressor is not a product feature solely because it has concrete code. Conversely, renaming `QuestionFeature` to `QuestionUtility` does not make it a permitted dependency.

## Resolve a new dependency

1. **A feature needs another feature's action:** define/reuse the provider's input protocol; inject its implementation from the app.
2. **A feature embeds another feature's UI:** inject an interface factory or typed builder; keep construction at the app root.
3. **A feature opens another feature:** emit a typed output or route; let the host resolve the destination.
4. **Two features share a model:** place it in the interface of its actual owner. Extract a focused shared contract only if neither should own it.
5. **A feature needs a backend:** depend on the repository contract, not the SDK. Reuse an existing backend interface package if appropriate.
6. **Two interfaces would depend on each other:** reduce the exchanged payload to owned values/IDs, move coordination to the app, or extract the genuinely shared contract into a lower-level interface.

Do not create a generic protocol, mediator, service locator, or event bus solely to obscure a forbidden dependency.

## Public surface

Interface declarations intended for consumers are `public`, including required initializers. Keep only contract-level value construction/conformance in interfaces; business rules and provider work belong elsewhere. SwiftUI imports are legitimate for view-factory contracts, but not a reason to turn all domain interfaces into UI modules.

Feature declarations are `internal`/`private` by default. The normal public surface is configuration + `StartModuleView`, plus intentional implementations of public input/output contracts needed by the composition root. Such adapters may expose a factory capability; they must not expose internal screens or ViewModels. Use `@testable import` from tests instead of broadening production visibility.

## Review the real graph

Inspect `.package` declarations, target `.product` dependencies, and imports. An unused manifest dependency still adds graph complexity; removing an import alone does not remove that edge. Follow transitive dependencies through common/utility packages. Ask whether a consumer can compile and be tested using the interface without the provider feature, provider SDK, or app target.
