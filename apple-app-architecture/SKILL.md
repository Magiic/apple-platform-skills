---
name: apple-app-architecture
description: >
  Structure Apple apps as feature-based Swift Packages with separate Interface and Feature
  implementation modules. Use when creating features, defining public contracts, wiring
  dependencies or cross-feature navigation, reviewing SOLID boundaries, or improving the
  build dependency graph. Preserves freedom of architecture inside each feature.
---

# Apple App Architecture

## Intended result

Create Apple applications from independent functional modules, explicit contracts, and app-level composition. Apply these conventions to both new projects and existing applications. Examples use fictional feature names and require no reference repository.

The mandatory choice is **feature modularization with separate interface and implementation modules**. Do not impose MVVM, VIP, VIPER, Clean Architecture, TCA, or a fixed sequence of internal layers. Follow an explicitly requested or established internal pattern; otherwise start with small SwiftUI views and add collaborators only for a concrete responsibility.

## Essential rules

1. Create a pair of local Swift Packages for each cohesive product feature: `FooInterface` and `FooFeature`. `Feature` is the default implementation suffix. A folder inside the app target is not a module boundary.
2. Put public contracts in `FooInterface`; put feature behavior, state, screens, and resources in `FooFeature`. Keep implementation details `internal` or `private`.
3. A feature may depend on its own interface, other interfaces, and focused utilities. **Never make one feature implementation depend on another feature implementation**, directly or transitively through a shared wrapper.
4. Keep the dependency graph acyclic. Interfaces and utilities must not import feature implementations or the app target.
5. Let the app composition root instantiate concrete dependencies, create module configurations, connect outputs and inputs, build feature entries, and own cross-feature navigation.
6. For a UI feature, `StartModuleView` is its unique public screen entry. Expose configuration and intentional contract adapters as needed; do not expose internal screens or ViewModels.
7. Inject dependencies explicitly through `FooModuleConfiguration`, then distribute configuration inside the feature through SwiftUI environment values. The environment is a delivery mechanism, not a global service locator.
8. By default, keep each SwiftUI view at **300 lines of code maximum**, including its helpers and extensions. Extract cohesive child views before the limit. Introduce a ViewModel when presentation logic needs reuse or independent tests; no ViewModel is required for a simple screen.
9. Apply SOLID to both package boundaries and internal responsibilities. A protocol per class or a layer per operation is not a goal.

## Module ownership

| Location | Owns | Must not own |
| --- | --- | --- |
| `FooInterface` | Public input/output protocols, events, starter entries, cross-module routes, shared domain values and provider-independent DTOs | Business workflows, concrete SDK clients, screens, ViewModels, UI resources |
| `FooFeature` | Public configuration and `StartModuleView`; internal screens, optional ViewModels, feature rules, display models, resources, contract adapters | Other features' implementations or app composition |
| `*Client` / `*Infrastructure` | Concrete network/storage/provider adapters, SDK mapping and technical error translation behind interface contracts | Feature UI or app navigation |
| Focused utility packages | One reusable technical concern such as design system, media compression, image loading | Product feature workflows disguised as shared helpers |
| App target | Lifecycle, assembly, shared dependency lifetimes, platform entry points, cross-feature routing and output wiring | Feature business rules |

Use `Sources/FooInterface` and `Sources/FooFeature` at repository root for a new project; retain an existing package root if working in another project. Each package has its own manifest, library product, and target. Prefer one primary production target per package. Shared technical packages do not need artificial Interface/Feature pairs unless a replaceable implementation boundary is useful.

Do not move all models into a common package. The module owning a contract owns its types. Keep presentation-only values internal. A shared backend interface package may already own repository contracts and provider-independent request/response DTOs; reuse those contracts instead of duplicating them in each feature.

## Contracts and composition

- Use an `Input` contract for capabilities another module calls, an `Output` contract with typed events for intent/results emitted to the host, and a typed view-builder closure for simple embedded feature UI.
- The caller knows only the contract. The app connects it to the concrete implementation. A runtime call into another feature does not require a compile-time dependency on that feature.
- For multiple initial screens, define `FooStarterEntry` in `FooInterface`, pass it in configuration, and switch inside `StartModuleView`. Do not add public `CreateFooEntryView` or `EditFooEntryView` alternatives.
- Put cross-module route values in interfaces. Let the app map events/deeplinks to routes and construct destination modules; keep purely internal navigation within the feature.
- Define contracts according to consumer needs. A UI-building interface may import SwiftUI; a repository/domain interface should remain independent of UI when possible.
- Prefer explicit constructor inputs. Keep app factories such as `AppDependency.Foo` in the app; feature code must never reach back into them.
- Public API includes explicit public initializers for values constructed by other modules. Do not make internal types public just for tests.

Read [dependency-direction.md](references/dependency-direction.md) before changing dependency edges, and [module-communication.md](references/module-communication.md) when designing inputs, outputs, configuration, or navigation.

## SOLID as concrete decisions

- **Single responsibility:** split packages by cohesive functionality and objects by reason to change. Layout, pagination policy, provider mapping, and app routing have different owners.
- **Open/closed:** introduce a new provider or presentation policy through the existing contract and composition root. Changing Book UI should not require editing Feed internals. Explicit route/event switches may still change when adding a new product capability.
- **Liskov substitution:** production implementations, previews, and test doubles honor the same results, errors, cancellation, and isolation contracts. Do not silently succeed or trap on a required supported operation.
- **Interface segregation:** give consumers the capabilities they use. Split oversized input/repository protocols when clients need distinct subsets; do not pass a universal app container to every object.
- **Dependency inversion:** feature rules depend on abstractions owned by interface modules. Concrete implementations are selected outside those rules and injected.

For internal design choices, read [feature-internals.md](references/feature-internals.md). This guidance deliberately does not prescribe a micro architecture.

## Compilation performance

Separate interface targets are compilation boundaries, not just naming. Keep contracts small and stable, avoid unused dependencies in both manifests and source imports, and keep SDKs out of interface dependency closures. A feature implementation edit should not introduce a source dependency requiring unrelated feature consumers to rebuild against that implementation.

This reduces coupling and can reduce incremental rebuild work; it does not guarantee that Xcode skips all downstream work. Relinking, resources, build settings, compiler behavior, and public API changes still matter. Read [compilation-performance.md](references/compilation-performance.md) for graph review and measurement.

## Workflow

1. For an existing app, inspect manifests, a matching feature pair, app assembly, and public contracts; distinguish conventions from accidental violations. For a new project, establish the app target, supported platforms/toolchain, and repository-root `Sources/` directory, then identify cohesive product features from the requirements.
2. Identify the feature's responsibility, contract owner, consumers, and allowed dependency edges before adding files. Extend an existing feature for another screen of the same capability.
3. For a new pair, follow [module-creation-cli.md](references/module-creation-cli.md) and [swift-package-templates.md](references/swift-package-templates.md). Keep supported platforms and toolchain settings aligned with the project.
4. Implement only the internal responsibilities needed, wire real dependencies at the app root, and supply safe test/preview dependencies.
5. Test behavior at its owning boundary using [testing-boundaries.md](references/testing-boundaries.md). Validate manifests and build affected targets with an appropriate supported destination.
6. Review the final dependency graph and public API. Report actual checks and any build limitation; do not claim an unmeasured compilation speedup.

## Review checklist

- Does each new product feature have separate Interface and Feature packages?
- Can consumers use the interface without importing the implementation or its SDKs?
- Are cross-feature calls, UI embedding, outputs, and routes wired by the app?
- Is `StartModuleView` the unique public screen entry, including alternate starter modes?
- Are dependencies explicit, interfaces focused, and shared modules free of feature coupling?
- Are SwiftUI views within 300 lines and additional layers justified by responsibility, reuse, or tests?
- Can feature behavior be tested with injected dependencies without starting the app or a backend?
- Were affected builds/tests checked without inventing platform requirements or performance results?

For a greenfield example and worked architecture decisions, read [worked-example.md](references/worked-example.md). No existing app or reference repository is required.

## Specialized integration work

When available, use `apple-navigation-deeplinks` for external navigation delivery and readiness, `apple-app-clips` for Clip targets and app continuity, `apple-widgets` for WidgetKit extensions, and `apple-swiftui-previews` for isolated preview composition. These skills refine their platform boundary; the Interface/Feature dependency rules remain in force. They are optional companions, not prerequisites for using this skill.

A system extension's framework entry point is distinct from a feature's public screen entry. Keep WidgetKit registration/provider wiring in the extension target; do not require a SwiftUI `StartModuleView` for a widget entry renderer. Reused app UI features still expose their normal single screen entry.
