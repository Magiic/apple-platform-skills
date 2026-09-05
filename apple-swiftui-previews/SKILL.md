---
name: apple-swiftui-previews
description: >
  Build and repair deterministic, isolated SwiftUI previews in Apple app and SPM feature
  modules. Use for preview dependency factories, state scenarios, fixture data, environment
  injection, layout variants, and previews that fail because they depend on app startup.
---

# Apple SwiftUI Previews

## Intended result

Make a component or feature inspectable in Xcode with realistic, reproducible states and no need to start the app, sign in, or contact a live backend. Apply the same conventions to a new project and an existing modular app.

Previews are development composition. They must not determine production architecture, expose internal screens as public API, or force a ViewModel onto a simple view. Keep the default 300-line ceiling for each view's code, excluding standalone previews, comments, and blank lines.

## Ownership and boundaries

- Keep `#Preview` declarations near the view they demonstrate, normally in the same file. Use descriptive scenario names.
- Keep fixtures, preview dependency implementations, and scenario factories in the owning feature. A feature preview must compile without the app target or another Feature implementation.
- Preview internal views directly inside their module. Also exercise `StartModuleView` with realistic injected configuration where entry/environment wiring matters.
- For an embedded feature, provide an interface-conforming preview adapter or builder; do not import that feature implementation just for the preview.
- Share fixture support across packages only when actual reuse justifies a separate development-support target. Do not grow the production Interface module with example data or test tooling.

## Isolation by default

- Construct a fresh mutable dependency graph for each preview. Avoid static mutable repositories, shared navigation paths, or a single ViewModel reused across scenarios.
- Use fixed identifiers, dates, content, and outcomes when they affect the rendered result. Use bundled images or deliberate placeholders instead of remote URLs.
- Inject no-op monitoring and deterministic session/repository/storage dependencies. Previews must not write to production preferences, keychain, databases, App Groups, analytics, or permission services.
- Use in-memory or isolated temporary stores when exercising persistence. Creating a preview must not request system permissions or start purchases, background jobs, or authenticated SDK initialization.
- Do not rely on an environment flag alone to neutralize real dependencies. Construct safe dependencies explicitly.
- A `.preview` factory is separate from production wiring. A safe environment fallback must still compile in every configuration where the entry exists and must not conceal missing production injection with fake success.

Read [scenarios-and-fixtures.md](references/scenarios-and-fixtures.md) for scenario design, support placement, and test/preview separation.

## State and composition

Preview the state the user can actually encounter: content, loading, empty, failure/retry, or restricted access when supported. Prefer scenario-specific repository outcomes or an existing explicit presentation-state input. Do not add production-only mutation hooks or bypass invariants just to force a screenshot.

For a binding-driven component, supply preview-owned state using supported preview APIs or a small local wrapper. For a view that owns an observable model, ensure the model is created once at the proper lifetime boundary. A lifecycle task should not immediately overwrite the scenario with real data.

`#Preview`, `@Previewable`, and `PreviewModifier` have SDK/deployment considerations. Verify availability before introducing a newer API into an older target. `PreviewModifier` may cache and share its context, so only share a context when its mutability/lifetime is deliberately suitable. [Apple: preview setup](https://developer.apple.com/documentation/xcode/adding-previews-to-your-interface-files), [Apple: PreviewModifier](https://developer.apple.com/documentation/swiftui/previewmodifier)

## Visual coverage

Choose a compact set of variants that exposes the component's risks: long text, empty/large values, relevant locales and RTL, Dynamic Type, light/dark appearance, and supported layouts. Do not mechanically create every combination or invent platform support.

Read [validation-and-troubleshooting.md](references/validation-and-troubleshooting.md) when adding coverage or diagnosing a broken canvas.

## Workflow and completion

1. Identify the view's inputs, environment requirements, resources, and lifecycle effects.
2. Add or reuse minimal deterministic fixtures and safe dependency factories; create explicit scenarios.
3. Add nearby previews, including feature-entry wiring when relevant.
4. Build the affected target and inspect rendered previews where the environment allows it. Confirm local actions remain isolated.
5. Report separately whether compilation, rendering, and interactive states were verified. A preview declaration alone is not visual verification.

Previews support visual development; unit tests verify behavior. When writing configurable doubles or assertions, use the project's testing conventions without making previews import its test target.
