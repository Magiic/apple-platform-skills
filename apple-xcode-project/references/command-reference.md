# Apple `xcrun xcodeproj` Command Map

Checked against `xcrun xcodeproj help` from Xcode 27.2 Beta 3. Use `xcrun xcodeproj help <branch> <command>` for the selected version's complete syntax. All commands except `init` accept `-P <name>.xcodeproj`; use it explicitly in multi-project directories. The CLI edits both `project.xcproj` and `project.pbxproj` but has no JSON output mode.

| Branch | Commands | Use |
| --- | --- | --- |
| `init` | `init <path>` | Create an empty JSON `.xcodeproj`. |
| `setting` | `get`, `list`, `set`, `set-multi`, `unset` | Read or change raw, unevaluated build settings at project or target level; `--config` narrows a configuration. |
| `target` | `list`, `info`, `add`, `add-aggregate`, `add-dependency`, `remove-dependency`, `remove` | Manage targets and target dependencies. `add` requires `--product-type`. |
| `target phase` | `info`, `add`, `add-script`, `add-copy`, `remove`, `move` | Manage numbered build phases. Re-read `target info` after every phase change. |
| `group` | `ls`, `add`, `add-file`, `remove-file`, `remove`, `include`, `exclude`, `set-header-visibility` | Manage navigator groups, file references, target membership, and header visibility. |
| `config` | `set`, `unset` | Attach or detach an existing `.xcconfig` as base configuration. Does not edit the file itself. |
| `pkg` | `list`, `add`, `add-local`, `remove`, `link`, `unlink`, `link-plugin`, `unlink-plugin` | Manage Swift package references and target product/plugin links. |

## Build settings and configurations

```sh
xcrun xcodeproj setting list -P MyApp.xcodeproj --target MyApp
xcrun xcodeproj setting get -P MyApp.xcodeproj SWIFT_VERSION --target MyApp --config Debug
xcrun xcodeproj setting set -P MyApp.xcodeproj SWIFT_VERSION 6.0 --target MyApp
xcrun xcodeproj setting set-multi -P MyApp.xcodeproj FRAMEWORK_SEARCH_PATHS '$(inherited)' '$(PROJECT_DIR)/Frameworks' --target MyApp
xcrun xcodeproj setting unset -P MyApp.xcodeproj SWIFT_VERSION --target MyApp --config Debug
xcrun xcodeproj config set -P MyApp.xcodeproj Debug.xcconfig --group /Config --config Debug
xcrun xcodeproj config unset -P MyApp.xcodeproj --config Debug
```

Omitting `--target` acts at project level; omitting `--config` applies to all configurations for mutations. `setting get/list` shows attached xcconfig names but not values defined inside those files. Use `xcodebuild -showBuildSettings -json` when evaluated build settings are needed.

## Targets and build phases

```sh
xcrun xcodeproj target list -P MyApp.xcodeproj
xcrun xcodeproj target info -P MyApp.xcodeproj --target MyApp
xcrun xcodeproj target add -P MyApp.xcodeproj MyExtension --product-type app-extension
xcrun xcodeproj target add-aggregate -P MyApp.xcodeproj AllTargets
xcrun xcodeproj target add-dependency -P MyApp.xcodeproj MyApp MyExtension
xcrun xcodeproj target remove-dependency -P MyApp.xcodeproj MyApp MyExtension
xcrun xcodeproj target remove -P MyApp.xcodeproj MyExtension
xcrun xcodeproj target phase add -P MyApp.xcodeproj --target MyApp sources
xcrun xcodeproj target phase add-script -P MyApp.xcodeproj --target MyApp --name Lint --script @Scripts/lint.sh
xcrun xcodeproj target phase add-copy -P MyApp.xcodeproj --target MyApp --destination frameworks --name 'Embed Frameworks'
xcrun xcodeproj target phase info -P MyApp.xcodeproj --target MyApp 1
xcrun xcodeproj target phase move -P MyApp.xcodeproj --target MyApp 2 --to 1
xcrun xcodeproj target phase remove -P MyApp.xcodeproj --target MyApp 1
```

These are individual command forms, not a sequence to run blindly. `target phase add` supports `sources`, `resources`, `frameworks`, and `headers`; use `add-script` or `add-copy` for configured phase types. Insert with `--position`, `--before`, or `--after`; all indexes are live and 1-based. `add-script` accepts repeatable `--input`, `--output`, `--input-file-list`, and `--output-file-list` plus shell, environment logging, and install-only options. `add-copy --destination` accepts an Xcode destination keyword and optional subpath; check its help for the selected beta. A new aggregate target supports scripts and copy phases, not compile phases.

## Groups and membership

```sh
xcrun xcodeproj group ls -P MyApp.xcodeproj -R
xcrun xcodeproj group add -P MyApp.xcodeproj MyApp --filesystem-path MyApp
xcrun xcodeproj group add -P MyApp.xcodeproj Config --without-folder
xcrun xcodeproj group add-file -P MyApp.xcodeproj Debug.xcconfig --group /Config
xcrun xcodeproj group include -P MyApp.xcodeproj --group /MyApp --target MyApp
xcrun xcodeproj group exclude -P MyApp.xcodeproj --group /MyApp --target MyApp
xcrun xcodeproj group include -P MyApp.xcodeproj Main.swift --group /Sources --target MyApp --phase 1
xcrun xcodeproj group set-header-visibility -P MyApp.xcodeproj Public.h --group /Headers --target MyFramework --visibility public
xcrun xcodeproj group remove-file -P MyApp.xcodeproj Debug.xcconfig --group /Config
xcrun xcodeproj group remove -P MyApp.xcodeproj /Config
```

Use a full navigator path or ID if a bare group name is ambiguous. `group add` can use `--parent`, `--path`, `--without-folder`, or `--filesystem-path`. For a regular group, `include/exclude` needs a file name and `--phase` index. For an entire synchronized or variant group, omit both. Inspect membership after adding it; a missing Sources phase may leave a synchronized source folder outside compilation. `remove-file` and `remove` can use `--force` to discard still-referenced entries; inspect references before using that option. Group commands affect the project model, not source files on disk.

## Swift packages

```sh
xcrun xcodeproj pkg list -P MyApp.xcodeproj
xcrun xcodeproj pkg add -P MyApp.xcodeproj https://github.com/apple/swift-argument-parser --from 1.5.0
xcrun xcodeproj pkg add-local -P MyApp.xcodeproj Sources/FooFeature --product FooFeature --target MyApp
xcrun xcodeproj pkg link -P MyApp.xcodeproj FooFeature --package FooFeature --target MyApp
xcrun xcodeproj pkg unlink -P MyApp.xcodeproj FooFeature --package FooFeature --target MyApp
xcrun xcodeproj pkg link-plugin -P MyApp.xcodeproj MyPlugin --package MyPackage --target MyApp
xcrun xcodeproj pkg unlink-plugin -P MyApp.xcodeproj MyPlugin --package MyPackage --target MyApp
xcrun xcodeproj pkg remove -P MyApp.xcodeproj FooFeature
```

Choose one version rule for a remote package: `--from`, `--up-to-next-minor`, `--exact`, `--range-min` with `--range-max`, `--branch`, or `--revision`. `pkg add` and `add-local` may link a product immediately with `--product` plus `--target`. `pkg link --embed` embeds a supported dynamic product; `--no-codesign` modifies that embedding. Ensure the target has the needed Frameworks phase. Use `pkg list` for exact package identities before unlinking or removing.
