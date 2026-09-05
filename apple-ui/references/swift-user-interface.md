# SwiftUI Package Ownership

For a new project, use case-sensitive `Sources/` at repository root alongside the app project folder. For an existing app, retain its established package root. `MyApp` is an illustrative app name:

```text
repository/
  Sources/
    FooInterface/
      Package.swift
      Sources/FooInterface/
    FooFeature/
      Package.swift
      Sources/FooFeature/
        Configuration/
        Views/
          StartModuleView.swift
        Resources/
      Tests/FooFeatureTests/
  MyApp/
    MyApp.xcodeproj
    MyApp/
```

Each UI feature has separate interface and implementation packages. A new screen inside the same cohesive feature need not become a new module. Shared design-system components belong in the design-system package, not in a sibling feature's public UI surface.

## Entry and dependencies

- `StartModuleView(moduleConfiguration:)` is the unique public screen entry; internal screens remain internal.
- Represent alternate initial screens with a starter enum in `FooInterface`, store it in the feature configuration, and select content inside the entry.
- The app creates real dependencies and injects configuration. Distribute configuration internally with `@Entry` on `EnvironmentValues` where supported; keep the View setter beside that entry inside the feature.
- Use safe deterministic preview/default dependencies. Do not instantiate production clients in an environment default or a view body.
- Another feature's UI is supplied through an interface input or typed view builder wired by the app. Do not add a Feature-to-Feature package dependency.
- Keep routes shared across modules in interfaces; the app owns cross-feature destination assembly and external deeplinks.

## Internal UI composition

Default to SwiftUI views at most 300 lines of code including their helpers/extensions. Extract meaningful child views with narrow inputs. Local UI state and simple intent forwarding can stay in a view. Introduce a ViewModel only when presentation logic needs reuse or independent tests; preserve an explicitly selected internal architecture.

Keep package resources in the owning feature and resolve them with its package bundle. Consume existing design-system components. Do not duplicate UI or move a feature workflow into the design system merely to bypass dependency rules.
