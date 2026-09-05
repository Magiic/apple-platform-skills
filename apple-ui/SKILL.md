---
name: apple-ui
description: >
  Apply the creation user interface rules for Apple platforms (iOS, iPadOS, macOS, visionOS, tvOS).
  Use this skill when creating and updating new user interface.
---

# Apple UI

## When to use
Use this skill when:
- Creating a new user interface (swiftUI views and/or UIKit)
- Updating a new user interface (swiftUI views and/or UIKit)
- Have to localizable with string catalog
- Formatting number, date or any formatter

## Goals (non-negotiable)
- Always choose first swiftUI when possible to create user interface unless user ask using UIKit/AppKit
- Use UIKit/AppKit for specifc use cas listed here
- Make all texts localizable
- Use always `#Preview` to display user interface
- Colors are listed in a file named `Colors.xcassets` located in `Resources` folder
- Always handle light and dark mode

## Localizable
- All texts in user interface should be localizable unless user ask to not localized.
- When using text in swift local package, provide bundle as shown below:

```swift
Text("Hello World!", bundle: #bundle)
```

- When create button taht display text, use the following constructor to make it localizable:

```swift
Button {
    // action
} label: {
    Text("Hello world!", bundle: #bundle)
}
```

- Other swiftUI components use `Text("mytext", bundle: #bundle)` when possible to make it localizable.
- Update string catalog to add translation for all languages supported. Ask user what languages is supported.
- Support plurals when necessary
- Vary strings by device
- See `references/localizable.md` for canonical examples, patterns and more details.

## Formatter
- Formatting (date, number, etc...) use latest Api format style. See `references/date-formatter.md`, `references/number-formatter.md`.

## Asset Catalog
Asset catalog rules (mandatory):

When adding files to an .xcassets catalog:
- Always set the correct File Type (UTType / UTI) according to the asset kind.
- Never leave the File Type empty or incorrect.

Use Apple UTType identifiers (Uniform Type Identifiers) when setting File Type:
- Video: UTType.mpeg4 (public.mpeg-4)
- Audio: UTType.audio (public.audio)
- Image: UTType.image (public.image)
- PDF: UTType.pdf (com.adobe.pdf)

Example:
- Adding a video file "intro.mp4" to a Swift Package asset catalog:
  - Place it in an .xcassets catalog
  - Set File Type to: public.mpeg-4

Before finishing:
- Verify that every asset added to an .xcassets catalog has a valid File Type.
- Especially verify video and audio assets.

## Colors
- When adding new color, add it to a file named `Colors.xcassets` located in folder `Resources`.
- Always handle light and dark mode
- Use built-in Color symbol to access the color from `xcassets`. `Color(.myColor)` when this is possible.

## Images and Videos
- Add images to file named `Images.xcassets` located in folder `Resources`.
- Add videos to file named `Videos.xcassets` located in folder `Resources`.

## Fonts
- Add fonts to file named `Fonts.xcassets` located in folder `Resources`.
- Check the project and dependencies if there is a way to use existing font registration otherwise create it.

## Views
- Use SwiftUI first if possible.
- By default, keep each SwiftUI View at a maximum of 300 lines of code, including helpers/extensions across files; exclude comments, blank lines, and standalone previews. Extract cohesive child views before the limit.
- Previews should be at the same file than the swiftUI view
- Multiple Previews for most of the state of the view if existing.
- Create child swiftUI views if needed
- If project contains module Design System, use it to build user interface using its components.
- Keep layout and simple local UI state in the View. Introduce a ViewModel when presentation logic needs reuse or independent tests; keep business workflows out of the View. Do not impose MVVM, VIP, or a fixed stack of layers.
- Use `@Environment(.horizontalSizeClass)` and `@Environment(.verticalSizeClass)` to handle variants layout.
- Make views accessible.
- Provide accessibility identifier for relevant views. Act like all the page need to be testable with UI Tests
- Provide great accessibility labels for children and combine components.
- Provide accessibility label and value for components like progress view, slider etc.
- Handle accessibility `DynamicTypeSize`
- Use `@Observable` if available otherwise `ObservableObject` conformance.
- Be sure to ignore properties that the swiftUI view does not need when using `@Observable`.
- Use UIKit/AppKit when views has infinite scroll especially for scroll containing images or videos like *Instagram*. Ask permission to the user before start using UIKit
- When using UIKit, use latest api UITableView and UICollectionView to build it.
- When dealing with advanced gesture, use UIKit. Ask permission to the user before start using UIKit

## Preview composition
- Keep previews deterministic and isolated from production services, storage, telemetry, and permission requests.
- Create fresh mutable preview dependencies and cover meaningful states with named scenarios.
- Keep preview setup inside the owning module; do not import the app target or another feature implementation to make a preview work.
- For substantial preview setup or troubleshooting, use `apple-swiftui-previews` when available. Basic UI work remains covered by this skill.

## View Models
- Create a ViewModel for reusable or independently testable presentation behavior, not automatically for every screen. Pure rules can use focused helpers instead.
- Test the meaningful behavior of these implementations through injected dependencies.
- Give visual child views narrow values, bindings, and action closures instead of an entire ViewModel or module configuration.

## Module boundaries
- Put UI implementation and its resources in the owning `*Feature` package. Keep routes, startup entries, and cross-module contracts in `*Interface`.
- Keep `StartModuleView` as the unique public screen entry and select alternate initial screens using the configured starter enum.
- Embed another feature through an injected input contract or view builder assembled by the app. Never import its implementation into a feature.
- Follow [swift-user-interface.md](references/swift-user-interface.md) for package ownership and configuration scope.

## References
- See `references/localizable.md`, `references/date-formatter.md`, `references/number-formatter.md` for canonical examples, patterns and more details.
