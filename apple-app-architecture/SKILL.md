---
name: apple-app-architecture
description: >
  Apply the project's macro architecture rules for Apple platforms (iOS, iPadOS, macOS, visionOS, tvOS).
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

## General direction (strict)
Allowed directions:
- Inject dependency from module configuration
- Use swift environment values as described `references/swift-package-templates.md` to inject configuration accross all children swiftui views.
- DTO and Domains are located in module interface
- Protocols are located in module interface
- Module feature implementation contains implementation and logical of the feature
- Module feature implementation has tests to validate business logical and rules
- Module utilities can be imported in other modules
- Module utilities should focus on one topic. For example, a module for a client network, a module to contains Design System (atomic design)...

Forbidden:
- Implementation of logical business in module interface
- Add module feature implementation dependencies in `Package.Swift`. Only use module interface or module utilities as a dependency for modules interface and implementation.
- Circular dependencies

## Public API rules
- For all files in module interface, use `public`
- Default to `internal` for module feature implementation
- Expose only what other modules need

## State management (default guidance)
- SwiftUI: MVVM is default (View + ViewModel)
- Use `@Observable` if available otherwise use `ObservableObject` conformance for *ViewModel*.
- UIKit: Coordinator/Router + MVVM but keep the same module boundaries

## Concurrency (defaults)
- UI-bound APIs are `@MainActor`
- Background work is isolated (actors or structured concurrency)
- Avoid detached tasks unless justified

## Testing strategy (module-level)
- Domain: fast unit tests (pure business rules)
- Data: integration tests with mocked network/storage
- UI: minimal snapshot/UI tests if needed
- Use Swift Testing framework

## References
- See `references/package-templates.md` for Package.swift templates