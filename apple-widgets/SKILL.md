---
name: apple-widgets
description: >
  Create and integrate WidgetKit widgets with modular Apple apps. Use for widget extension
  boundaries, shared data, timeline providers and refresh policy, configuration and actions,
  deep links into the app, and family-specific previews and validation.
---

# Apple Widgets

## Intended result

Build a widget as a focused system-facing presentation of app data with explicit data access and update policy. Apply these conventions to a new app or an existing modular codebase. Example names and layouts are fictional.

This skill covers WidgetKit widgets. Live Activities use a distinct ActivityKit lifecycle and are not ordinary timeline widgets; do not scaffold them automatically. An App Clip also has a distinct lifecycle and must not be treated as a widget extension.

## Target and module boundaries

- Create the Widget Extension using the project's Xcode/project-generation mechanism. The extension target owns WidgetKit registration, provider/configuration adapters, and its composition root.
- Share domain values, repository contracts, and route values through focused Interface modules. Select extension-safe concrete data clients at the extension root.
- Do not import the app target, its dependency container, full-screen navigation hosts, or another feature's ViewModel into widget implementation code.
- Reuse small extension-safe UI/technical components when appropriate. Do not link an entire app feature merely to reuse a title formatter or row.
- If the widget capability warrants its own SPM implementation, preserve the Interface/Feature split for public product contracts. The WidgetKit extension remains the system entry; it does not need an artificial `StartModuleView` for a widget entry view.
- Audit transitive dependencies for extension compatibility, required capabilities, resource cost, and supported platforms. Keep app-only API access outside extension-linked code.
- Do not impose MVVM. A provider, small mapper, and entry view can be sufficient. Keep each SwiftUI view within the default 300-line limit.

## Data and updates

A widget is not a continuously running screen. WidgetKit schedules rendering and updates; refresh requests do not guarantee an exact delivery time. Design for cached data, missing data, and stale content rather than polling or a perpetual SwiftUI task. [Apple: widget updates](https://developer.apple.com/documentation/widgetkit/keeping-a-widget-up-to-date)

Read [data-and-timelines.md](references/data-and-timelines.md) for shared storage, snapshot ownership, timeline entries, refresh policies, and failure behavior.

## Navigation and actions

A content link should open the corresponding destination in the app through the app's validated route pipeline. Keep routes in interfaces and resolve screens in the app. Use one deliberate `widgetURL` in a hierarchy and additional `Link` controls only where supported for the selected family/platform. [Apple: linking into the app](https://developer.apple.com/documentation/widgetkit/linking-to-specific-app-scenes-from-your-widget-or-live-activity)

For a supported in-place action, use WidgetKit's App Intent interaction mechanisms. A button intended only to open the app should instead use the link mechanism. Keep mutations in injected services, re-check current authorization/data, and verify the relevant App Intents execution and availability rules before implementing. [Apple: widget interactivity](https://developer.apple.com/documentation/widgetkit/adding-interactivity-to-widgets-and-live-activities)

Do not invent a general App Intents implementation pattern in this skill. Read current Apple guidance or an available specialist reference when writing those intents; configuration intents and action intents have different purposes.

## Workflow

1. Define the useful glanceable information/action, supported platforms/families, configuration needs, and freshness expectations.
2. Establish the extension target and minimal shared contracts/data dependencies. For a new app, establish the data source and deep-link contract alongside the extension.
3. Implement provider/data mapping and entry rendering with explicit placeholder, preview snapshot, and timeline behavior.
4. Connect shared data and refresh requests where needed, then app links or supported actions.
5. Add representative family/state previews and test provider/mapping behavior with deterministic inputs.
6. Build the app and extension and perform relevant installed-widget checks. Report build, preview, interaction, and refresh observations separately.

Read [presentation-and-validation.md](references/presentation-and-validation.md) for UI, configuration, previews, and the test matrix. Use current official documentation for platform-specific budgets, families, and interaction support; never promise an exact periodic refresh or copy a universal memory limit into generated code.
