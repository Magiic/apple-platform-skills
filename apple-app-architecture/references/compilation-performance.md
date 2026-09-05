# Compilation Performance Through Module Boundaries

## Objective

Reduce the amount of implementation knowledge carried by each target. Separate SPM targets are actual compilation boundaries; organizing folders in a single target does not provide this isolation.

`FeedFeature -> QuestionInterface` lets Feed consume Question contracts without compiling against Question's implementation surface. The app still builds and links the required implementations. This architecture can improve incremental builds and enable independent target work; it does not eliminate app linking or guarantee a fixed speedup.

## Design the graph

- Keep interface modules small and stable. Include public contracts and their required values, not all types used inside the implementation.
- Avoid aggregating unrelated feature contracts in one common module: a widely imported module increases the potential impact of changes.
- Reuse genuinely shared contracts, but review their transitive dependencies. A lightweight-looking interface that imports a large umbrella package may still pull in substantial work.
- Declare only consumed package products and target dependencies. Remove unused manifest edges as well as unused imports; avoid umbrella re-exports.
- Keep provider SDKs, generated provider schemas, and concrete storage/network clients out of feature interface closures. Feature implementations consume repository protocols.
- Put feature resources and localization in the owning feature target. Interfaces normally do not process UI assets.
- Use private/internal implementation types and expose stable value/protocol boundaries. Avoid public generic signatures that carry implementation details into consumers.
- Do not add `@inlinable` or `@_alwaysEmitIntoClient` to implementation code as a compilation optimization; these can expose implementation bodies to clients.
- Split by cohesive capability, not by every type or mandatory horizontal layer. Extra tiny packages add graph, configuration, and maintenance overhead.
- Keep supported platforms, language modes, and build settings consistent with actual project requirements. Do not copy example deployment versions or changing remote branches as universal defaults.

## Reduce local type-checking complexity

Keep SwiftUI views within the 300-line convention and extract concrete child views with clear inputs. Break deeply nested expressions and overloaded chains into meaningful typed values when the compiler struggles. A line-count reduction alone is not a measured performance result.

Keep view type erasure deliberate at cross-module builder boundaries. Neither adding `AnyView` everywhere nor expanding every screen into public generics is a general build optimization.

## Verify before claiming improvement

For a requested build-performance change:

1. Record toolchain, scheme, configuration, destination, dependency resolution, and cache state.
2. Establish comparable clean and incremental baselines using the same settings; keep package downloads separate from compiler timing.
3. Make an implementation-only edit in one feature and inspect which compilation tasks actually run. Then compare an interface edit to understand the consumer impact.
4. Use build logs and, where available, Xcode build timing summaries (for example `xcodebuild -showBuildTimingSummary` with the project's scheme and destination).
5. Compare repeated equivalent runs before stating a speedup. Keep cache conditions consistent and do not clean before an incremental measurement.

Actual invalidation depends on compiler/build-system behavior, signatures, resources, settings, and linking. Report reduced graph coupling separately from measured elapsed-time changes. Ordinary feature work needs an appropriate build check, not a new benchmarking project.
