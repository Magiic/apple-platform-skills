---
name: apple-xcode-project
description: >
  Create or modify Xcode project configuration with Apple's xcodeproj CLI.
  Use when creating a project, adding newly created local Swift packages to an
  app target, changing project or target build settings, or editing targets,
  phases, groups, xcconfig links, and package dependencies in a .xcodeproj.
---

# Apple Xcode Project

Use Apple's `xcrun xcodeproj` for supported `.xcodeproj` edits when the selected Xcode provides it (verified with Xcode 27.2 Beta 3). Prefer the JSON project configuration, `MyApp.xcodeproj/project.xcproj`, for a new project and when the existing project's toolchain and integrations support it. A `.xcodeproj` is still the outer bundle; `.xcproj` is the configuration file inside it. Xcode 27 supports both JSON and legacy `project.pbxproj`. Check the team's minimum Xcode and tools that parse the project before converting an established project.

## Choose the project toolchain

1. Inspect the repository's project generator or source of truth, current project format, minimum Xcode, and selected developer directory. If a generator owns `.xcodeproj`, edit its source and regenerate instead of editing generated output.
2. Run `xcodebuild -version` and `xcrun --find xcodeproj`. With multiple Xcodes, set `DEVELOPER_DIR=/Applications/<Xcode>.app/Contents/Developer` for each command rather than changing the machine-wide selection. Invoke Apple's binary as `xcrun xcodeproj`; the bare `xcodeproj` name may resolve to the Ruby gem or another tool.
3. Read `xcrun xcodeproj help <subcommand>` on the selected beta before a mutation. If Apple's CLI is unavailable, use the project's established Xcode UI/MCP or generator workflow. Do not install an unrelated CLI as a substitute.
4. Supply `-P MyApp.xcodeproj` when project discovery is ambiguous. Otherwise the CLI uses `XCODEPROJ_PROJECT`, then a single `.xcodeproj` in the current directory. `init` instead takes the new project path directly.

`xcodeproj` reads and writes the project configuration, including legacy projects. It does not create or edit workspaces, schemes, source files, or the contents of `.xcconfig` files. Its output is human-readable; do not assume a `--json` output switch. For machine-readable inspection use `xcodebuild -list -json` or `xcodebuild -showBuildSettings -json` where appropriate. Use `xcprojformatter` for canonical formatting of `project.xcproj`; do not rewrite it through a generic JSON serializer because Xcode's format can contain trailing commas.

## Workflows

- **Create from zero:** Read [new-project.md](references/new-project.md). `xcodeproj init` makes an empty JSON project; add the target, build phases, file membership, settings, package links, source files, and any scheme or workspace needed by the product. A template-capable Xcode UI/MCP flow can supply app-specific scaffolding, followed by `xcodeproj` for project edits.
- **Add a local Swift package to the app:** Create and configure its `Package.swift` with Swift Package Manager, then use `xcrun xcodeproj pkg add-local` to register its path and `--product`/`--target` (or `pkg link`) to link the required product into the app target. Do this when a newly created package must be consumed by an `.xcodeproj`; `xcodeproj` does not create the package's files or manifest. Follow the example in [new-project.md](references/new-project.md).
- **Change build settings:** Use `xcrun xcodeproj setting get/list` to inspect raw values and `setting set`, `set-multi`, or `unset` to change them at project or `--target` level, optionally for one `--config`. Use `config set/unset` only for the base `.xcconfig` association, and edit the `.xcconfig` contents separately. See [command-reference.md](references/command-reference.md).
- **Modify a project:** Read [command-reference.md](references/command-reference.md) for the complete command tree and flags that affect each operation. Discover actual target names, group paths, and live phase indexes before editing. Use `setting` for build settings, `config` to attach an existing `.xcconfig`, `group` for file membership, `target` for targets/phases/dependencies, and `pkg` for Swift packages.
- **Move to JSON:** Only when the project and its integrations are ready, convert with `xcodebuild -project MyApp.xcodeproj -convert-project xcproj` or use Xcode's Project Format control. Verify that `project.xcproj` replaces `project.pbxproj` and review the diff. The JSON format opens in Xcode 27+, not earlier major versions.

After a structural edit, inspect `target info` again: phase indexes are 1-based live positions. In a new target, add a Sources phase before including a synchronized source folder; `target add` alone does not create build phases or all required build settings. Check actual build settings, target membership, package links, and product behavior with `xcodebuild -list -json`, a relevant build/test, and a focused diff. A successful build can miss source files that were never included.

These commands were checked against locally installed Xcode 27.2 Beta 3. Recheck `--help` as beta syntax changes. See [Apple's JSON format guide](https://developer.apple.com/documentation/xcode/updating-your-xcode-project-configuration-file-format), [Xcode 27.2 release notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27_2-release-notes), and [Apple's format library](https://github.com/apple/xcode-project-format).
