# Module Communication and App Composition

## Choose the smallest explicit contract

| Situation | Contract | Concrete wiring |
| --- | --- | --- |
| Feed requests a Question capability | `QuestionInput` in `QuestionInterface` | App injects a conforming adapter into Feed configuration |
| Feed asks to open a profile | `FeedOutput` + `FeedOutputEvent` in `FeedInterface` | App observes the output and updates navigation |
| Book embeds a question collection | Builder accepting `QuestionCollectionStarterEntry` | App closure creates `QuestionCollectionFeature.StartModuleView` |
| A deeplink opens Book in edit/detail mode | `BookStarterEntry` in `BookInterface` | App creates Book configuration for that entry |
| A feature fetches data | Repository protocol in its owning interface | App injects backend/local/test implementation |

Inputs describe capabilities supplied to a consumer. Outputs describe events emitted to the host. Neither should expose a concrete screen, ViewModel, SDK object, or app factory.

## Contract examples

These are illustrative contracts; adapt their cases to the product requirements. Each declared type belongs in its own file.

`BookInterface/BookStarterEntry.swift`:

```swift
import Foundation

public enum BookStarterEntry: Hashable, Sendable {
    case create
    case detail(bookID: UUID)
}
```

`BookInterface/BookOutputEvent.swift`:

```swift
import Foundation

public enum BookOutputEvent: Sendable {
    case created(bookID: UUID)
    case closeRequested
}
```

`BookInterface/BookOutput.swift`:

```swift
@MainActor
public protocol BookOutput {
    func send(_ event: BookOutputEvent)
}
```

A synchronous method or callback is sufficient for simple outputs. Use the project's existing stream/callback pattern when appropriate. With `AsyncStream`, the host owns listeners and cancellation; define whether the channel has one consumer or supports broadcasting, avoid duplicate listeners, and finish streams when their owner ends. Do not introduce a global event bus.

UI-facing protocols and builders should declare their actor requirements. Make cross-actor value contracts `Sendable` when needed; follow the target's default isolation and supported Swift version. Do not impose `MainActor` on every data contract just because a consumer is UI.

## Configuration is the feature's assembly boundary

`FooModuleConfiguration` belongs in `FooFeature` and has a public initializer. It holds the startup data, repository abstractions, output, required view builders, and shared dependencies that the feature actually consumes. Prefer immutable properties; mutable service state remains owned by the injected service.

The app creates production configuration. The public entry receives it and distributes it to internal views. Use `@Entry` where supported, with the setter beside it inside the feature:

```swift
// FooFeature/Views/ModuleConfiguration+Environment.swift
import SwiftUI

extension EnvironmentValues {
    @Entry var moduleConfiguration: FooModuleConfiguration = .preview
}

extension View {
    func moduleConfiguration(_ configuration: FooModuleConfiguration) -> some View {
        environment(\.moduleConfiguration, configuration)
    }
}
```

This fragment assumes a feature-local `.preview` configuration built from safe deterministic dependencies. Match an existing `.default` convention if appropriate. Never create a live authenticated backend client in an environment default. If a default must exist in all build configurations, keep it safe in all of them; use real app injection for production. Do not silently mask missing production wiring with success-returning mocks.

- Construct configurations and stateful services at a deliberate lifetime boundary, not repeatedly during `body` evaluation.
- Treat SwiftUI environment injection as feature-local scope. Keep the environment entry/setter internal to avoid collisions between features using the same name.
- Plain ViewModels and services receive dependencies via initializers; they do not read SwiftUI environment values.
- Pass narrow inputs to reusable children instead of forwarding the entire configuration automatically.
- An injected session snapshot can become stale. Where the app stores mutable preferences persistently, use its established refresh mechanism before reading them. For example, a session abstraction may expose a refresh/read operation for persisted language preferences. Do not assume an initializer snapshot stays current.

## Single public screen entry

`StartModuleView(moduleConfiguration:)` is the only public screen entry of a UI feature. It selects internal content from `moduleConfiguration.starterEntry` and installs configuration for every branch. Public feature adapters may build through this entry without exposing those internal screens.

Keep the starter enum in the interface: another feature or a deeplink can request a mode without importing the implementation. Reuse an existing route type when it already fully represents startup; do not create competing enums for the same purpose. Avoid a large group of optional startup fields allowing invalid combinations.

## Embedding UI without Feature-to-Feature imports

Book's configuration may declare:

```swift
public typealias QuestionCollectionViewBuilder =
    @MainActor (QuestionCollectionStarterEntry) -> AnyView
```

Its stored builder and initializer parameter use this type. Book imports `QuestionCollectionInterface`, not `QuestionCollectionFeature`. Only the app closure constructs the real collection entry with its configuration.

For a richer capability contract, an input protocol can return a view through an associated type:

```swift
import SwiftUI

@MainActor
public protocol QuestionInput {
    associatedtype Content: View
    func makeQuestionView(_ input: QuestionPageViewInput) -> Content
}
```

This fragment assumes `QuestionPageViewInput` is a public value in the same interface. Use generics or a deliberate erased adapter at the consuming boundary; do not assume an arbitrary `any QuestionInput` is itself a concrete SwiftUI `View`.

`AnyView` is acceptable as a deliberate cross-module factory boundary, when a concrete return type would otherwise cross the implementation boundary. Keep concrete `some View`/child-view types within the feature; do not spread erasure throughout every row and modifier or make every screen generic solely to avoid one boundary erasure.

## Composition root responsibilities

Use focused app factories such as `AppDependency.Book.moduleConfiguration(...)` and `makeStarterView(...)`. The app is allowed to import both Book and QuestionCollection implementations because it assembles them. An input adapter owned by a feature can be instantiated by the app; an app-specific bridge that combines several implementations belongs at the composition root.

The app also owns output listeners and cross-feature routing. Keep its factories and routing handlers split by feature/responsibility; a thin app target means limited responsibilities, not a single enormous file. App-specific lifecycle/platform adapters may live beside composition, but reusable provider implementations belong in focused client packages. Do not force a separate infrastructure package for every tiny assembly adapter.

Define service lifetimes explicitly: shared session/backend services, per-flow outputs, and per-screen state must not be recreated or accidentally shared by a factory. Avoid strong reference cycles among callbacks, outputs, routers, and builders.

## Navigation boundary

- Use value-based `NavigationStack(path:)` and interface-owned route/starter values for cross-module navigation.
- Map outputs and external entry points (push, universal links, system callbacks) at the app/root layer. Feature internals must not manipulate another tab's root path.
- Feature-local routes may stay internal. Attach destination handlers to a non-lazy parent, not directly to `List` or `LazyVStack`.
- In a tab host, prefer a local `@State NavigationPath` binding and coordinate router requests deliberately to avoid multiple updates in one frame.
- A tap requiring analytics/side effects should call the routing action explicitly. Reserve `NavigationLink` for simple passive navigation; do not add `.simultaneousGesture` to it.

URL paths and payload parsing remain app contracts. Reuse the current project's canonical paths; define and test product-specific paths in the app rather than hard-coding them into these architectural conventions.
