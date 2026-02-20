---
name: apple-app-architecture
description: >
  Apply modular Apple app architecture using Swift Packages (iOS, macOS, and other Apple platforms).
  Use this skill when creating modules, defining dependencies, wiring composition root, and enforcing architecture/test boundaries.
---

# Apple App Architecture (Project-Agnostic)

## When to use this skill
Use this skill when:
- Creating a new module or package.
- Adding or changing module dependencies.
- Defining architecture boundaries.
- Integrating data sources (backend/local storage/system APIs).
- Refactoring feature structure.
- Creating new files that impact architecture direction.

## Core principles (non-negotiable)
- Keep the app target thin: app lifecycle, navigation, and dependency wiring only.
- Use local Swift Packages for modularity and scalability.
- Enforce strict dependency direction with no cycles.
- Keep features testable by default.
- Separate contracts from implementations.
- Preserve cross-platform compatibility (at least iOS + macOS when applicable).

## Module taxonomy
Use clear module types:
- `*Interface` modules:
- Purpose: public contracts, domain models, protocol abstractions.
- Implementation code: not allowed.
- `*Feature` modules:
- Purpose: feature implementation, orchestration, view models, SwiftUI views.
- Depends on interfaces and shared utilities.
- `*Client` / `*Infrastructure` modules:
- Purpose: concrete external integrations (network SDKs, storage SDKs, backend SDKs).
- Must implement interface contracts.
- `*DesignSystem` / `*Foundation` / `*Utility` modules:
- Purpose: reusable UI primitives or technical helpers.

## Dependency direction rules
Allowed:
- `Feature -> Interface`
- `Feature -> Utility/DesignSystem`
- `Infrastructure -> Interface`
- `App target -> Feature + Infrastructure` (composition root wiring)

Forbidden:
- `Interface -> Feature`
- `Feature -> concrete SDK client` when an interface exists
- `Feature -> App target`
- circular dependencies
- business logic in app target or pure view layer

## Composition root rules
- App target instantiates concrete implementations.
- App target injects dependencies into feature configurations.
- App target handles high-level routing.
- App target must not contain feature business logic.
- Environment selection/config (dev/staging/prod) belongs to composition/wiring, not feature internals.

## Backend integration boundary
- Define backend contracts in interface modules.
- Map external responses to domain models in infrastructure modules.
- Throw typed domain/infrastructure errors from repositories.
- Convert repository errors to displayable UI errors in feature layer.
- Never leak provider-specific naming/details into view layer.

## UI architecture rules
- SwiftUI Views render state and user intent only.
- ViewModels (or equivalent state handlers) own orchestration and async work.
- Keep side effects out of reusable view components.
- Localized strings and package resources should use package-safe bundle access (`#bundle`).

## Module creation workflow (mandatory)
Do not manually scaffold module folders in Finder/Xcode.

Required workflow:
1. Create package folder via CLI.
2. Initialize package with `swift package init`.
3. Configure `Package.swift` dependencies.
4. Integrate package into app/workspace.
5. Add tests based on module type rules.
See: `references/module-creation-cli.md`.

## Testing boundaries (mandatory)
- `*Interface` modules:
- Test targets are optional and generally not required.
- `*Feature` modules:
- Unit tests are required.
- `*Utility` modules:
- Add tests when they contain meaningful logic.
See: `references/testing-boundaries.md`.

## Definition of done (architecture)
- Module type and boundary are correct.
- Dependency direction is valid.
- No business logic in app target or pure views.
- No concrete provider leakage into feature UI contracts.
- Required tests exist and pass.
- Build succeeds for intended Apple platforms.

## References
- `references/swift-package-templates.md`
- `references/module-creation-cli.md`
- `references/testing-boundaries.md`
- `references/dependency-direction.md`
