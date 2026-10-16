# Create or Convert an Xcode Project

## New project with the Apple CLI

`init` creates an empty `.xcodeproj` bundle with `project.xcproj` and Debug/Release configurations. It is suitable when you can explicitly assemble the project's target and build settings. It does not create source files, schemes, a workspace, or a fully configured app template. Discover valid product types and settings for the actual platform rather than copying a tool example into an app.

Example for a small macOS command-line tool (run in a scratch directory, with Xcode 27.2 selected):

```sh
mkdir -p Demo/Sources
printf 'print("Hello")\n' > Demo/Sources/main.swift
xcrun xcodeproj init Demo.xcodeproj
xcrun xcodeproj setting set -P Demo.xcodeproj SDKROOT macosx
xcrun xcodeproj setting set -P Demo.xcodeproj SWIFT_VERSION 6.0
xcrun xcodeproj target add -P Demo.xcodeproj Demo --product-type tool
xcrun xcodeproj target phase add -P Demo.xcodeproj --target Demo sources
xcrun xcodeproj group add -P Demo.xcodeproj Demo --filesystem-path Demo
xcrun xcodeproj group include -P Demo.xcodeproj --group /Demo --target Demo
xcrun xcodeproj target info -P Demo.xcodeproj --target Demo
xcodebuild -project Demo.xcodeproj -scheme Demo -derivedDataPath "$PWD/DerivedData" build
```

For an app or extension, determine its actual product type, bundle identifier, platform SDK, deployment target, Info.plist generation, signing, resources, capabilities, embedding, and scheme needs. Add only values supported by the selected Xcode and required by the product. `target add` supplies only minimal defaults. Use a matching Xcode template when the product needs substantial scaffold that the CLI does not create.

If a local package lives in `Sources/FooFeature`, link it after its `Package.swift` exists:

```sh
xcrun xcodeproj target phase add -P MyApp.xcodeproj --target MyApp frameworks
xcrun xcodeproj pkg add-local -P MyApp.xcodeproj Sources/FooFeature --product FooFeature --target MyApp
xcrun xcodeproj pkg list -P MyApp.xcodeproj
```

`pkg add-local` paths are relative to the project directory. Adapt the example if the `.xcodeproj` lives in a subfolder. A Swift package's own `Package.swift` remains the source of truth for that package's products and targets.

## Existing project and JSON conversion

Check the current file first:

```sh
ls MyApp.xcodeproj/project.xcproj MyApp.xcodeproj/project.pbxproj
xcodebuild -project MyApp.xcodeproj -list -json
```

When conversion is intended and Xcode 27.2 is selected:

```sh
xcodebuild -project MyApp.xcodeproj -convert-project xcproj
xcrun xcprojformatter --update MyApp.xcodeproj
xcodebuild -project MyApp.xcodeproj -list -json
```

Confirm that project consumers such as CI scripts and third-party project parsers handle `project.xcproj` before committing the conversion. Keep the project in its existing format when those integrations still require `project.pbxproj`; the Apple CLI can edit that format too. Review the conversion separately from functional changes when practical.
