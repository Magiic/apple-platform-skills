# Architecture Apple project

All local swift packages should be located in folder `/sources`. The folder `/sources` is located at the root of xcode project

project/
├── project.xcodeproj
└── sources/

## Module Feature Implementation and Interface

When creating a new feature there is always 2 swift local package to create. One for interface and business model, and the other for implementation of the feature based on the module interface.

### Module Interface
Should named FeatureNameInterface. For example:
- OnboardingInterface
- ArticleInterface
- PaywallInterface
- SettingsInterface

This module can only contains protocols and business model. Should not contains business logical or implementation. It defines dto, domain models, types errors, protocols. This module has not tests, no images and string localizable. It can contains default implementation but only tests and previews purposes to use in other modules.
The module contains no user interface. All files and fonctions inside should be public.

Dependencies in `Package.swift` should never contains feature implementation. When need to access to other feature module, use only feature interface.

Here an example of package for feature interface.

```swift
// swift-tools-version: 6.2
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "OnboardingInterface",
    platforms: [
        .macOS(.v10_15),
        .iOS(.v18),
        .tvOS(.v13),
        .watchOS(.v6),
        .macCatalyst(.v13)
    ],
    products: [
        .library(
            name: "OnboardingInterface",
            targets: ["OnboardingInterface"]
        ),
    ],
    dependencies: [
        .package(path: "../AppCommonInterface"),
        .package(path: "../Networker")
    ],
    targets: [
        .target(
            name: "OnboardingInterface",
            dependencies: [
                .product(name: "AppCommonInterface", package: "AppCommonInterface"),
                .product(name: "Networker", package: "Networker")
            ]
        ),
    ]
)
```

Use alphabetical order to list dependencies.

#### Structure

Module Feature interface has a following structure:

ModuleInterface/
├── Package.swift
└── sources/
    └── ModuleInterface/
        └── DTO/
        └── Domain/
        └── Mapper/
        └── Repository/
        └── InputOutput/
        └── ModuleInterface.docc

This structure with these folders are not always present. If the module does not communicate with server, DTO and reporistory folders are useless and should not be present. If there is specifics feature protocols create folder at same level.

For example, a feature to calculate rectangle area should contains a protocol `RectangleCalculation` in folder `Calculation`.

#### DTO
This folder can be omitted if there is no dto for the feature.
Contains all dto files when communicate with backend server. Should conform to `Sendable`, `Decodable`. Use foundation types and custom model themself created with foundation types. Suffixed with `DTO` for example: `UserDTO`. Use `struct` or `enum` if that makes sense.

```swift
public struct UserDTO: Sendable, Decodable {
    public let address: String
    public let phoneNumber: String?
    public let socialNetwork: SocialNetworkDTO?
    
    public init(
        address: String, 
        phoneNumber: String?, 
        socialNetwork: SocialNetworkDTO?
    ) {
        self.address = address
        self.phoneNumber = phoneNumber
        self.socialNetwork = socialNetwork
    }
}
```

#### Domain
Contains feature domain. These models will be used accross other modules and the app. Conforms to `Sendable` if possible.

```swift
public struct User: Sendable {
    public let address: String
    public let phoneNumber: String?
    public let socialNetwork: SocialNetwork?
    
    public init(
        address: String, 
        phoneNumber: String?, 
        socialNetwork: SocialNetwork?
    ) {
        self.address = address
        self.phoneNumber = phoneNumber
        self.socialNetwork = socialNetwork
    }
}
```

#### Folder Mapper
Contains extension to domain model to create domain from a dto. Use init constructor.

```swift
extension User {
    
    init?(_ value: UserDTO?) {
        guard let value else { return nil }
        self.init(
            address: value.address,
            phoneNumber: value.phoneNumber,
            socialNetwork: SocialNetwork(value.socialNetwork)
        )
    }
}
```

#### Repository
Contains repository protocols. All protocols for repository should conform to `Sendable`. This folder can be omitted if there is no repository for the feature.

#### Docc
Each module contains a xocde documentation. It describes what the purpose of the module, types and protocols. Should be concise.

#### Input and Output
Because module feature implementation import only module interface as dependencies, it is necessary to expose protocols and function to communicate with other modules. `Output` defines event on the module that can be observe by other modules and app. `Input` exposes methods to inject values to the module.

Below an example of output protocol:

```swift
public protocol OnboardingOutput: Sendable {

    /// Indicates that user completes onboarding
    var didComplete: ( @Sendable (OnboardingInformation) -> Void)? { get set }
}
```

And an example for `Input`:

```swift
import Combine

public protocol OnboardingInput {
    
    var usernameSubject: PassthroughSubject<String, Never> { get }

    func didChangeUsername(_ username: String)
}
```

### Module Feature (Implementation)
Should named FeatureNameFeature. For example:
- OnboardingFeature
- ArticleFeature
- PaywallFeature
- SettingsFeature

This module contains logical and implementations of the feature. It uses module interface to build its components and services.

Dependencies in `Package.swift` should never contains feature implementation. When need to access to other feature module, use only feature interface.

Here an example of package for feature interface.

```swift
// swift-tools-version: 6.2
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "OnboardingFeature",
    defaultLocalization: "en",
    platforms: [.macOS(.v10_15),
        .iOS(.v18),
        .tvOS(.v13),
        .watchOS(.v6),
        .macCatalyst(.v13)
    ],
    platforms: [
        .macOS(.v10_15),
        .iOS(.v18),
        .tvOS(.v13),
        .watchOS(.v6),
        .macCatalyst(.v13)
    ],
    products: [
        .library(
            name: "OnboardingFeature",
            targets: ["OnboardingFeature"]
        ),
    ],
    dependencies: [
        .package(path: "../AppCommonInterface"),
        .package(path: "../OnboardingInterface"),
        .package(path: "../Networker")
    ],
    targets: [
        .target(
            name: "OnboardingFeature",
            dependencies: [
                .product(name: "AppCommonInterface", package: "AppCommonInterface"),
                .product(name: "Networker", package: "Networker")
            ],
            resources: [.process("Resources")]
        ),
        .testTarget(
            name: "OnboardingFeatureTests",
            dependencies: ["OnboardingFeature"]
        ),
    ]
)
```

- Use alphabetical order to list dependencies.
- Use default localization "en".
- Always contains test target

#### Structure

Module Feature implementation has a following structure:

ModuleFeature/
├── Package.swift
└── sources/
    └── ModuleFeature/
        └── AppLogger.swift
        └── Configuration/
            └── ModuleFeatureModuleConfiguration.swift
        └── Dependencies/
            └── ModuleFeatureInput.swift
            └── ModuleFeatureOutput.swift
        └── Fixtures/
            └── FixturesMaker.swift
        └── Mocks/
        └── Model/
        └── ModuleFeature.docc
            └── ModuleFeature.md
            └── GettingStarted.docc
        └── Resources/
            └── Localizable.xcstrings
            └── Images.xcassets
        └── Views/
            └── StartModuleView.swift

#### AppLogger
Use AppLogger when needed to log information. Log always errors in interface layer model (ViewModel if using MVVM pattern for example). Never use `print` function.
For subsystem use the bundle identifier of the project and append feature name for example : `com.mycompany.myproject.onboardingfeature`.

Here an example of `AppLogger.swift` expected.

```swift
import os.log

struct AppLogger: Sendable {
    
    static let subsystem = "com.mycompany.myproject.onboardingfeature"
    
    static let configuration = Logger(subsystem: subsystem, category: "Configuration")
    static let repository = Logger(subsystem: subsystem, category: "Onboarding Repository")
}
```

#### Configuration
Each module feature implementation should contains a *configuration*. The configuration contains all dependencies needed for the module. It centralize place to find which dependencies is necessary.

Below an example of module feature configuration.

```swift
public struct OnboardingModuleConfiguration: ModuleConfiguration {
    
    public let defaultConfiguration: DefaultModConfiguration
    public var output: OnboardingOutput
    public let user: User
    public let input: OnboardingInput

    public init(
        defaultConfiguration: DefaultModConfiguration,
        user: User,
        output: OnboardingOutput,
    ) {
        self.defaultConfiguration = defaultConfiguration
        self.user = user
        self.output = output
        self.input = DefaultOnboardingInput()
    }
}
```

#### Dependencies
This folder contains all implementation denpendencies. It contains implementation of `Input` and `Output` interfaces if existing.

Example of implementation Input:

```swift
import Combine
import OnboardingInterface

public final class DefaultOnboardingInput: OnboardingInput {
    
    public var usernameSubject = PassthroughSubject<String, Never>()
    public var username: String?
    
    public init() {}
    
    public func didChangeUsername(_ username: String) {
        self.username = username
        usernameSubject.send(username)
    }
}
```

Below an example of implementation output:

```swift
import OnboardingInterface

public struct DefaultOnboardingOutput: OnboardingOutput {
        
    public var didComplete: (@Sendable (OnboardingInformation) -> Void)?
        
    public init(
        didComplete: (@Sendable (OnboardingInformation) -> Void)?
    ) {
        self.didComplete = didComplete        
    }
}
```

#### FixturesMaker

A simple struct that provide fixtures for swiftUI previews or for tests purposes. Below an example:

```swift
struct FixturesMaker {
    
    struct Article {
        
        static let articleSport = Article(id: "abc", category: "sport")
        static let articleHistory = Article(id: "abc", category: "history")
    }
    
    struct ViewModel {
        
        @MainActor
        static let articleViewModel = ArticleViewModel(
            article: ArticleInterface.Article, 
            repository: ArticleRepositoryMock()
        )
    }
}
```

#### Mocks
Mocks and stubs to work in module with swiftUI previews or for unit tests purposes. We have to test successfull, failures and edge case.

#### Model
All models and data structure needed to build feature implementation. Should be internal and never public.

#### ModuleFeature.docc
Xcode documentation with a getting started and other articles if needeed to specify how the module works. Provide screenshots of the pages for platforms.

#### Resources
Location of assets, images inside xcassets and string localization.

#### Views
All swiftUI, UIKit and viewmodel (MVVM pattern) or presenter for other patterns. Contains also extension to create environment values.:

To make previews working easily, each module configuration has an environment values similar to below:

```swift
struct OnboardingModuleConfigurationKey: @MainActor EnvironmentKey {
    typealias Value = OnboardingModuleConfiguration
    
    @MainActor
    static let defaultValue: OnboardingModuleConfiguration = OnboardingModuleConfiguration.default
}

extension EnvironmentValues {

    /*
     // Use @Entry if possible otherwise use old way
    @Entry
    var moduleConfiguration: OnboardingModuleConfiguration = OnboardingModuleConfiguration.default
     */
    
    // Old way
    @MainActor
    var moduleConfiguration: OnboardingModuleConfiguration {
        get { self[OnboardingModuleConfigurationKey.self] }
        set { self[OnboardingModuleConfigurationKey.self] = newValue }
    }
}

extension View {
    
    func moduleConfig(_ configuration: OnboardingModuleConfiguration) -> some View {
        environment(\.moduleConfiguration, configuration)
    }
}
```

Each module feature implementation has a file `StartModuleView`. It inject module configuration in child views using swiftui environments.

```swift
public struct StartModuleView: View {
    
    public typealias ModuleConfig = OnboardingModuleConfiguration
    
    public let moduleConfiguration: ModuleConfig
    
    private let viewModel: OnboardingViewModel
    
    public init(configuration: OnboardingModuleConfiguration) {
        self.viewModel = OnboardingViewModel(
            repository: RemoteOnboardingRepository(api: configuration.api)
        )
        self.moduleConfiguration = configuration
    }
    
    public var body: some View {
        OnboardingView(viewModel: viewModel)
            .moduleConfig(moduleConfiguration)
    }
}

#Preview {
    StartModuleView(
        configuration: OnboardingModuleConfiguration.default
    )
}
```

All swiftui view and uikit UIView/UIViewController have at least one `#Preview`. Multiple previews is preferable to check all main state of the view.

## Module Utilities
Module utilities are modules that are not features but help to solve specifics problems. For example a module with Foundation extension. A module containing Design System when working with Atomic Design, a module wrapper around network client to make HTTP call. These modules should have a name easy to understand what they do, short and upperCamelCase.
They can be imported as dependency in any other modules but especially in feature module.

## Integration in the app target

Once modules ready, import them in project target (or widget, extension) in the section *frameworks, libraries and Embeded content*.

The app target should have the following structure for modularization:

MyProject/
├── MyProject.docc
        ├── MyProject.md
        ├── GettingStarted.md
└── MyProject/
    └── Modularization/
        └── Implementation/
        └── Dependencies/
            └── AppDependency.swift
            └── FeatureA/
                └── AppDependency+FeatureA.swift
            └── FeatureB/
                └── AppDependency+FeatureB.swift
    └── Configuration/
    └── Resources/
    └── .../

`AppDependency` file contain empty struct to organize dependencies implementation, example below:

```swift
struct AppDependency {}

extension AppDependency {
    
    struct Onboarding {}
    
    struct Settings {}
}
```

And in `AppDependency+Feature`:

```swift
extension AppDependency.OnboardingFeature {
    
    static func makeConfiguration(
        windowSize: CGSize,
        output: ListenOutput
    ) -> OnboardingModuleConfiguration {
        OnboardingModuleConfiguration(
            configuration: DefaultModConfiguration(
                analytics: DefaultModConfiguration.Analytics(
                    crashLogger: AppDependency.CrashReporter.makeCrashReporting(),
                    analytics: AppDependency.Listen.makeAnalyticsReport(),
                    featureFlag: AppDependency.FeatureFlag.makeFeatureFlag()
                ),
                designSystem: DefaultModConfiguration.DS(),
                other: DefaultModConfiguration.Other(windowSize: windowSize)
            ),
            output: output
        )
    }
    
    @MainActor
    static func makeStarterView(
        windowSize: CGSize,
        output: ListenOutput
    ) -> some View {
        OnboardingFeature.StartModuleView(configuration: AppDependency.OnboardingFeature.makeConfiguration(windowSize: windowSize, output: output))
    }
    
    static func makeAnalyticsReport() -> AnalyticsReporting {
        OnboardingAnalyticsReporter()
    }
}
```