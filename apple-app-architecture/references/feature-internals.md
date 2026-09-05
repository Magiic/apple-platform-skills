# Feature Internals: Small Views and SOLID

## No mandatory micro architecture

The Interface/Feature boundary is mandatory. The arrangement inside `FooFeature` is deliberately flexible. Do not impose MVVM, VIP, VIPER, a reducer, or `View -> ViewModel -> UseCase -> Repository` for every screen. Preserve an existing suitable pattern or an explicit user choice.

Start with a small SwiftUI view. Add the smallest collaborator that owns a real responsibility. Folder names such as `Views`, `Model`, `Validation`, or `Services` describe actual contents; they are not a required set of layers.

## Where behavior belongs

| Need | Default owner |
| --- | --- |
| Layout, focus, sheet visibility, selection, animation | View with local SwiftUI state |
| Simple derived display values and forwarding button intent | View or small child view |
| Reusable presentation mapping, testable loading/error transitions, coordinated async presentation | ViewModel, introduced for that need |
| Pure validation, ordering, pagination/insertion policy | Focused value/helper with independent tests when useful |
| Network/storage/provider mapping | Repository/client implementation behind an interface contract |
| Opening another feature or handling a universal link | Feature emits intent; app resolves and presents the destination |

A view may start a lifecycle task or call an injected action. Do not move every `.task`, `Button` closure, or state assignment into a ViewModel mechanically. Keep business workflows and substantive reusable/testable presentation logic out of the view. `body` must remain free of side effects.

## View size

By default, each SwiftUI `View` has at most **300 lines of code**. Count implementation helpers and extensions belonging to the same view, even across files; moving the same oversized type into extensions does not solve the problem. Blank lines, comments, and standalone `#Preview` blocks are not view code. Keep files readable too.

Extract by responsibility: a toolbar, a row, an empty/error state, or a reusable section. Use explicit values, bindings, and action closures for child views. Do not pass an entire module configuration or ViewModel to a purely visual child that only needs a title and an action.

The limit is a ceiling, not a target. Split earlier when a component has a clear independent purpose. Do not compress formatting to meet it. Existing oversized views are candidates for focused improvement when in scope, not templates to reproduce.

## When to introduce a ViewModel

Create a ViewModel when presentation behavior must be reused or tested independently: coordinating requests, mapping results to display state, managing meaningful transitions, handling retries or cancellation. Do not create one solely because a screen exists.

When introduced:

- Give it one screen/presentation responsibility. A ViewModel must not become a second app container or a backend client.
- Inject the contracts it consumes, directly or from module configuration. Keep SwiftUI environment lookup in views; ordinary objects receive dependencies through their initializer.
- Prefer `@Observable` where supported, otherwise `ObservableObject`. Isolate UI state on `@MainActor` as required and keep dependencies that do not drive rendering out of observation.
- Let the owning view retain the model using the appropriate SwiftUI lifetime mechanism; do not construct a new state owner during every `body` evaluation.
- Make task ownership and cancellation explicit. Test meaningful loading, success, failure, and cancellation behavior where applicable.
- Extract cohesive validation or policy objects when the model has multiple reasons to change. Do not add pass-through use cases or protocols for every internal helper.

A view-only screen and a screen with a ViewModel may coexist in the same feature. Existing VIP or other explicitly selected architecture is also compatible with these package boundaries.

## Apply SOLID without ceremony

- **SRP:** a feed page ordering helper can change without changing view rendering or SDK calls. A Book form can reuse a title validator without importing another feature.
- **OCP:** replace an injected repository or presentation strategy without rewriting its consumers. Do not build a plugin framework to avoid an ordinary local conditional.
- **LSP:** a test double preserves the contract's error/cancellation semantics. A no-op preview dependency is suitable only when the preview does not require real behavior.
- **ISP:** a row takes display data and actions; a pagination policy takes paging inputs; neither receives the entire app configuration. Split broad protocols where distinct consumers justify it.
- **DIP:** presentation code speaks to repository/output contracts. The app chooses concrete services. Pure internal helpers may remain concrete when substitution brings no value.

## Observability at the action source

Reuse the module's injected analytics and crash-reporting dependencies, such as an injected `commonConfiguration.monitor`. Screen analytics belong in the view's `onAppear`; interaction analytics belong at the action source. Report recoverable failures through the supplied crash reporter. Do not create a ViewModel only to log a screen event or introduce parallel telemetry services.
