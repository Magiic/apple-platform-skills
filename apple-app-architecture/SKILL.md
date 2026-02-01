---
name: apple-app-architecture
description: >
  Apply the project's macro architecture rules for Apple platforms (iOS, iPadOS, macOS).
  Use this skill when creating modules, choosing dependencies, structuring features, created files, or adding new architecture patterns.
---

# Apple App Architecture (Swift Packages, modular app)

## When to use
Use this skill when:
- Creating a new feature/module/utilities
- Changing folder/module boundaries
- Adding dependencies
- Defining app layers (UI / domain / data)
- Setting up the root app target or shared code across iOS/iPadOS/macOS
- Creating a new file

## Goals (non-negotiable)
- Keep the app target thin: composition + routing + app lifecycle only + create implementation and injecting dependencies into modules.
- Modularize by feature using local Swift Packages
- Enforce clear dependency direction (no circular deps)
- Make code testable by default (protocol boundaries, dependency injection)

## References
- See `references/swift-package-templates.md` for canonical examples, patterns and more details.

## Dependency direction (strict)
Allowed directions:
- UI -> Domain
- Data -> Domain
- AppComposition -> (UI, Domain, Data, Core*)
- Feature packages -> Core*

Forbidden:
- Domain importing UI or Data
- Feature importing another feature directly (use shared Domain/Core abstractions instead)
- Circular dependencies

## Public API rules
- Default to `internal`
- Expose only what other modules need
- Expose protocols from Domain; implementations live in Data/AppComposition

## State management (default guidance)
- SwiftUI: MVVM is default (View + ViewModel)
- UIKit: Coordinator/Router + MVVM (or VIP if already chosen), but keep the same module boundaries

## Concurrency (defaults)
- UI-bound APIs are `@MainActor`
- Background work is isolated (actors or structured concurrency)
- Avoid detached tasks unless justified

## Testing strategy (module-level)
- Domain: fast unit tests (pure business rules)
- Data: integration tests with mocked network/storage
- UI: minimal snapshot/UI tests if needed

## References
- See `references/package-templates.md` for Package.swift templates