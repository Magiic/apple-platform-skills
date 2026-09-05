# SwiftUI Structure

## Default design

Start with a small SwiftUI view. No mandatory MVVM, VIP, or fixed sequence of layers applies. Keep a suitable existing internal pattern or the user's explicit choice. The macro contract remains separate Interface/Feature packages regardless of the internal pattern.

## View responsibilities

- Render state, own simple local interaction state, emit user intent, and compose child views.
- Keep `body` free of side effects. Lifecycle tasks and button actions may invoke injected actions directly when there is no substantive orchestration to extract.
- Compute simple derived UI values instead of duplicating state.
- Extract cohesive rows, sections, toolbars, and loading/error states using explicit values, bindings, and closures.
- Keep feature workflows and reusable/testable presentation rules out of view code.

## When to add a ViewModel

Introduce a ViewModel when presentation behavior needs reuse or independent tests: coordinating async requests, mapping domain values into display state, or managing meaningful loading/error/retry transitions. Do not add one for a label, toggle, simple sheet, or action forwarding alone.

Give the model one presentation responsibility. Inject its dependencies rather than reading SwiftUI environment values or app factories from the model. Use the project's supported observation/lifetime APIs and appropriate UI actor isolation. Keep task lifetime and cancellation explicit and test relevant behavior.

Extract a pure validator, mapper, or policy helper when that is the actual responsibility. Do not force every helper through a ViewModel, protocol, repository, and use case. Apply SOLID by ownership and substitution needs, not by counting layers.

## Size limits

- By default, a SwiftUI `View` type has at most **300 lines of code**, counting its helpers/extensions even across files, excluding comments, blank lines, and standalone previews.
- The existing ViewModel/state-handler ceiling remains 500 lines; split earlier when responsibilities diverge. This ceiling does not require creating a model.
- Extract real components rather than moving the same oversized type between extensions or compressing code.

## Boundaries and testability

Presentation types are internal to their feature. Another feature must consume a public interface contract or injected view builder, never import the implementation to share a ViewModel. Tests may use `@testable import` and injected doubles.

A reusable visual child receives what it needs, not an entire dependency container. Screen analytics stay at the view lifecycle source, action analytics at the interaction source, and recoverable failure reporting uses the module's injected monitor. Logging alone is not a reason to introduce a ViewModel.
