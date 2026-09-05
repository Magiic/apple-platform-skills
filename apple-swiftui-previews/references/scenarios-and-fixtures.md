# Preview Scenarios and Fixtures

## A focused support layout

```text
Sources/ArticleFeature/
  Views/
    ArticleListView.swift          # nearby #Preview declarations
    ArticleRowView.swift
    Previews/
      ArticlePreviewScenario.swift
      ArticlePreviewFactory.swift
      ArticleFixtures.swift
      ArticleRepositoryPreviewStub.swift
```

This is an illustrative layout, not a mandatory set of files. A simple row preview may only need an inline value. Extract factories when setup repeats or obscures the preview's intent. Keep one top-level type per file when adding support types.

## Choose meaningful scenarios

| Scenario | Setup | What to inspect |
| --- | --- | --- |
| Content | A few deterministic representative values | Hierarchy, alignment, supported actions |
| Empty | Successful response with zero values | Empty-state explanation/action |
| Loading | Explicit loading state or controllable pending request | Layout stability and disabled actions |
| Failure | Configured failure | Error copy and retry action |
| Recovery | Fail then succeed on retry | Transition back to usable content |
| Restricted | Explicit access/session state | Correct call to action without real authentication |
| Long content | Long title, larger count, multiline text | Truncation, wrapping, accessibility |

Only create states supported by the component. A row does not need a fake loading repository if it only renders a supplied value.

## Fixtures versus dependencies

A fixture builds data. A dependency implements a contract. Keep those responsibilities separate so a fixture can serve both a pure row and a repository-backed screen.

- Use deterministic fixture defaults and named variations; avoid opaque giant payloads.
- Factories return new values and new mutable services. Static immutable value fixtures are fine; a `static let` holding a mutable repository or ViewModel is still shared state.
- Give preview repository outcomes descriptive names. A test double's mutable call history is usually unnecessary for canvas rendering.
- Keep resource resolution inside the owning package. Local images can exercise rendering without a network dependency.
- Never make production interface APIs public merely to let an external preview target reach internals.

A `PreviewStub` may return a selected plausible result. Required capabilities must still have meaningful configured behavior; do not claim an unavailable payment/save operation succeeded. A no-op analytics implementation is appropriate because telemetry is not the previewed behavior.

## Loading and interaction

Prefer an existing presentation-state input for a purely visual loading preview. For a screen whose behavior owns loading, use a controllable async stub that supports cancellation. If it keeps a continuation, define completion/cancellation cleanup and resume it at most once. Avoid arbitrary sleeps, a busy loop, or a permanently hanging task used merely to hold the spinner.

Interactive preview state belongs to that preview instance. Re-running one preview should reset its effects without changing another. If using a cached `PreviewModifier` context, intentionally share only suitable immutable data or isolate mutable scenario state below it. [Apple: shared preview context](https://developer.apple.com/documentation/swiftui/previewmodifier)

## Debug and production availability

Guard preview-only declarations/support with `#if DEBUG` where appropriate for the project. Review every reference crossing that guard: an always-compiled `@Entry` default cannot reference a type or `.preview` factory that disappears in Release.

A safe production-compiled fallback is not the live configuration. Ensure app assembly explicitly supplies its real dependencies and that missing required wiring has a diagnosable failure, not a success-returning mock. Do not use the mere presence of a default value as proof that production injection is correct.

Tests may share data-building helpers through a deliberately separate support target if useful, but preview code must never import the test target. Avoid shipping large fixture assets or development tooling as a transitive production dependency.
