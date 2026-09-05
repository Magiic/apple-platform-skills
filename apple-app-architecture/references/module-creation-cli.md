# Creating a Feature Pair with Swift CLI

## Before scaffolding

For an existing project, inspect a matching feature pair and the package root, supported destinations, toolchain, product naming, and integration mechanism. For a new project, establish those choices from the product requirements and create a thin app target that will assemble the packages. A new screen within an existing capability usually extends that feature instead of creating another pair.

For a new product feature, create both `FooInterface` and `FooFeature` as local library packages. Use `swift package init` from the terminal rather than manually creating an unconfigured folder in Finder/Xcode. Never reinitialize or overwrite an existing package.

## Example

Run from repository root. Replace `Foo` with the actual feature name; use `Sources` for a new project, or the established root of an existing repository. Check `swift package init --help` if the project's Swift version differs.

```sh
mkdir -p Sources/FooInterface Sources/FooFeature
(cd Sources/FooInterface && swift package init --type library --name FooInterface)
(cd Sources/FooFeature && swift package init --type library --name FooFeature --enable-swift-testing)
```

Then:

1. Replace generated sample code with meaningful contracts and the feature's minimal implementation. Remove generated interface sample tests/test target if the interface has no behavior.
2. Configure each manifest using [swift-package-templates.md](swift-package-templates.md). Keep package/product/primary target names aligned.
3. Add interface and utility dependencies only where consumed. Never add a sibling Feature implementation to a feature target.
4. Add feature-owned resources and a feature test target. Tests exercise real behavior when introduced; do not retain an empty sample assertion as validation.
5. Expose `FooModuleConfiguration` and `StartModuleView`. Add starter entries, inputs, outputs, and builders only as needed; no mandatory ViewModel or layer folders.
6. Register the local packages/products through the project's existing Xcode/workspace/project-generation mechanism. The app imports concrete features for assembly; callers import interfaces.
7. Add focused app factories and output/routing wiring. Avoid modifying generated project files manually if the project uses a generator.

## Validation

- Inspect each manifest using `swift package dump-package --package-path Sources/FooInterface` and the equivalent for `FooFeature`.
- Build the affected targets for a supported destination. Use `swift test` for packages that support the host; use the project's Xcode scheme and simulator destination for iOS-only targets/dependencies.
- Run relevant behavioral tests and verify that app assembly compiles. Manifest validation alone is not a successful build.
- Check the actual dependency graph, public API, resources, and previews. Report a platform/toolchain/dependency limitation precisely if it prevents execution.

Do not impose macOS support merely to make host-based `swift test` convenient, and do not change unrelated deployment targets while scaffolding.
