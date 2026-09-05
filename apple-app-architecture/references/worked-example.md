# Worked Example: A New Modular App

## Product requirements

Imagine an app that displays a feed of articles, opens article details, and saves articles into collections. The app also has a backend. No existing codebase is required to apply this example; feature names are illustrative.

## Identify boundaries

Create three cohesive UI feature pairs:

- `FeedInterface` / `FeedFeature`: feed presentation and paging behavior.
- `ArticleInterface` / `ArticleFeature`: article content and interactions.
- `CollectionInterface` / `CollectionFeature`: create, edit, list, and detail flows for collections.

Collection's multiple screens belong to the same capability, so they share one feature pair. Do not make one pair per screen. Create a focused `BackendClientInterface` / `BackendClient` boundary for provider-independent repository contracts and concrete backend integration when shared by these features. Add a `DesignSystem` utility if shared visual components are needed.

The repository's `Sources/` directory contains the package folders. Each has its own manifest, library product, and primary target. Create the app target separately and link the concrete feature/client products there.

## Design the graph first

```text
App -> FeedFeature -> FeedInterface
                  -> ArticleInterface
                  -> BackendClientInterface
App -> ArticleFeature -> ArticleInterface
                     -> BackendClientInterface
App -> CollectionFeature -> CollectionInterface
                        -> ArticleInterface
                        -> BackendClientInterface
App -> BackendClient -> BackendClientInterface
                     -> Provider SDK
```

Features can also depend on focused technical utilities. No feature imports another feature implementation, and neither a utility nor an interface hides such an import.

## Place each API with its owner

| Requirement | Placement and mechanism |
| --- | --- |
| Feed embeds article content | `ArticleInput` or a UI factory contract in `ArticleInterface`; app injects the concrete factory |
| Article asks to save into a collection | `ArticleOutputEvent.saveRequested(articleID:)` emitted through `ArticleOutput` |
| App opens the collection selector | App maps the event to `CollectionStarterEntry.selectForArticle(articleID:)` |
| Collections have several initial screens | `CollectionStarterEntry` cases; one `CollectionFeature.StartModuleView` switches internally |
| A feature fetches backend data | Provider-independent repository in its owning interface; app injects `BackendClient` implementation |
| A list row needs a subtitle and tap action | Internal child view with those explicit inputs; no new package or protocol |

Cross-module payloads use public values/IDs. Internal display models stay inside the implementation. A concrete provider response is mapped before reaching the feature, while an already suitable provider-independent DTO can be reused directly.

## Wire the app

Create focused factories such as `AppDependency.Feed` and `AppDependency.Collection` in the app target. They construct module configurations, supply repositories, connect outputs, and build `Feature.StartModuleView` entries. Instantiate shared services at the intended app/session lifetime and output channels at the intended flow lifetime.

The app can import both `FeedFeature` and `ArticleFeature` to build the injected article factory. Feed itself imports only `ArticleInterface`. A factory closure may erase its result to `AnyView` at this boundary; internal article screens remain concrete and internal.

The app owns external deeplink parsing, tab selection, and cross-feature navigation paths. Internal collection navigation stays within Collection unless its destination crosses a module boundary.

## Choose internal structure by need

- A simple collection confirmation view uses local state and an injected action; no ViewModel is required.
- A collection editor coordinating loading, saving, errors, and reusable presentation mapping gets a focused ViewModel with tests.
- A feed ordering/insertion rule becomes a pure helper with its own tests; it does not need a new module or a use-case layer.
- Every SwiftUI view defaults to at most 300 lines, with cohesive child views extracted early.

These choices apply SOLID without forcing a uniform internal architecture: independent reasons to change, focused contracts, substitutable services, and explicit dependency injection.

## Check the outcome

Build the app and affected packages for the chosen supported destinations. Test feature behavior using injected doubles, provider mapping in the client package, and output-to-route translation at the app boundary. A feature test must not need another feature implementation or a real backend.

Review whether a change to Article's internal layout requires any change to Feed's API dependency. Feed should depend only on the stable Article contract; app linking may still be necessary. Measure build effects before claiming faster compilation.

## Changes the architecture should handle

- **Add a collection entry mode:** extend `CollectionStarterEntry`, its internal entry switch, and the relevant host route; preserve the single public screen entry.
- **Replace the backend provider:** implement the same repository contracts and change composition; avoid edits to feature UI.
- **Add a separate product capability:** create a new Interface/Feature pair and wire it at the app root.
- **Reuse a presentation rule:** extract a focused collaborator where it belongs; do not export a feature ViewModel to another feature.
- **Find a circular interface dependency:** reduce exchanged payloads, move coordination to the app, or extract the genuinely shared contract into a lower-level interface.
