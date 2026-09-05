# Swift Package Templates

## Layout

For a new project, use this layout relative to repository root. For an existing project, preserve its established package root:

```text
Sources/
  BookInterface/
    Package.swift
    Sources/BookInterface/
      BookStarterEntry.swift
      BookOutput.swift
      BookOutputEvent.swift
  BookFeature/
    Package.swift
    Sources/BookFeature/
      Configuration/
        BookModuleConfiguration.swift
      Views/
        StartModuleView.swift
        ModuleConfiguration+Environment.swift
        Create/
          BookCreateView.swift
      Resources/
        Localizable.xcstrings
    Tests/BookFeatureTests/
MyApp/
  MyApp.xcodeproj
  MyApp/
    Dependencies/Book/
      Book+AppDependency.swift
    Features/Book/
      BookOutputManager.swift
```

`BookCreateViewModel.swift`, validators, display models, services, and extra folders are added only when needed. The tree is not a prescribed micro architecture. Tests belong beside the owning implementation package; an interface without behavior need not have a test target.

## Interface manifest

This example uses Swift 6.2 and illustrative iOS/macOS minimums. Match the actual project's toolchain and supported platforms rather than upgrading them to copy this template. Other interface or utility dependencies should be added only when required by public contracts.

```swift
// swift-tools-version: 6.2
import PackageDescription

let package = Package(
    name: "BookInterface",
    platforms: [.iOS(.v18), .macOS(.v15)],
    products: [
        .library(name: "BookInterface", targets: ["BookInterface"])
    ],
    targets: [
        .target(name: "BookInterface")
    ]
)
```

Interfaces contain public protocols, events, routes, shared domain/DTO values, and explicit public initializers. A repository contract can remain in an existing backend interface package; do not duplicate it to fill this tree. Do not add SDK clients, business workflows, localization, or assets to an interface.

## Feature manifest

```swift
// swift-tools-version: 6.2
import PackageDescription

let package = Package(
    name: "BookFeature",
    defaultLocalization: "en",
    platforms: [.iOS(.v18), .macOS(.v15)],
    products: [
        .library(name: "BookFeature", targets: ["BookFeature"])
    ],
    dependencies: [
        .package(path: "../BookInterface")
    ],
    targets: [
        .target(
            name: "BookFeature",
            dependencies: [
                .product(name: "BookInterface", package: "BookInterface")
            ],
            resources: [.process("Resources")],
            swiftSettings: [.defaultIsolation(MainActor.self)]
        ),
        .testTarget(
            name: "BookFeatureTests",
            dependencies: ["BookFeature"]
        )
    ]
)
```

Create the processed resource directory/catalog when resources are present; omit that declaration otherwise. The UI-target default isolation shown here is an optional UI-target setting; do not apply it blindly to domain/technical packages or incompatible toolchains. Resource access must use the package's bundle (`Bundle.module`, or `#bundle` when the configured toolchain provides it), never assume `Bundle.main` owns package resources.

To embed QuestionCollection, add its **interface** to both the package dependencies and the target product dependencies. Do not add `QuestionCollectionFeature` to Book. The app target links both feature implementations and supplies the factory.

## Public entry and configuration

The following fragments assume a public `BookStarterEntry` and `BookOutput` in `BookInterface`, as in [module-communication.md](module-communication.md). Internal Book views are intentionally unspecified. Each top-level type gets its own file.

```swift
import BookInterface

public struct BookModuleConfiguration {
    public let starterEntry: BookStarterEntry
    public let output: any BookOutput

    public init(starterEntry: BookStarterEntry, output: any BookOutput) {
        self.starterEntry = starterEntry
        self.output = output
    }
}
```

Add required repository abstractions, common dependencies, and builders explicitly as the feature needs them. Do not add dummy dependencies for completeness.

```swift
import BookInterface
import SwiftUI

public struct StartModuleView: View {
    private let moduleConfiguration: BookModuleConfiguration

    public init(moduleConfiguration: BookModuleConfiguration) {
        self.moduleConfiguration = moduleConfiguration
    }

    public var body: some View {
        Group {
            switch moduleConfiguration.starterEntry {
            case .create:
                BookCreateView()
            case .detail(let bookID):
                BookDetailView(bookID: bookID)
            }
        }
        .moduleConfiguration(moduleConfiguration)
    }
}
```

The environment helper and safe preview configuration are described in [module-communication.md](module-communication.md). Add previews using deterministic dependencies. If a child needs constructor injection to create its state owner, pass those dependencies from the entry rather than reading environment state during initialization.

## Client and utility packages

A client implements interface contracts and owns provider SDK mapping. The app instantiates it; consuming features receive the protocol. Utilities such as DesignSystem or media compression expose focused reusable APIs and must not import feature implementations.

Do not generate a client, repository, use case, presenter, or ViewModel package per screen. Separate packages represent capability and dependency boundaries; internal types represent responsibilities within those boundaries.
