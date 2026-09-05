# Widget Presentation and Validation

## Presentation and configuration

Start with the families/platforms the product actually supports. Design each layout for its available space and interaction model rather than shrinking a full app screen.

- Keep entry views small, declarative, and fed by display values.
- Provide meaningful placeholder, unavailable, signed-out, stale, and content states when relevant.
- Use the owning package's resources/localization and existing extension-safe design-system components.
- Respect supported widget background, margin, rendering-mode, accessibility, and privacy APIs for the target SDK. Do not assume ordinary app backgrounds and fixed sizes translate to every widget surface.
- Add user configuration only when users need a meaningful choice. Give selected entities stable IDs and a clear missing/deleted-entity fallback.
- Keep selection/configuration behavior separate from action execution and app navigation.

Check current WidgetKit guidance when adopting a new platform or family rather than copying an exhaustive compatibility table into the project. [Apple: creating a widget extension](https://developer.apple.com/documentation/widgetkit/creating-a-widget-extension)

## Previews

Use WidgetKit previews with deterministic entries for each relevant family and important state. Entry-view previews help inspect layout; widget/timeline previews exercise the WidgetKit presentation context. Neither proves installed-widget refresh scheduling. [Apple: widget preview examples](https://developer.apple.com/documentation/widgetkit/creating-a-widget-extension)

Use fixed times when displaying relative dates or stale-state indicators, and bundled assets when images matter. Avoid a real backend, app account, production App Group, or system-permission request in preview setup. Reuse the preview conventions for isolated dependencies without importing the app or test target.

Choose long localized content, appearance/rendering mode, and accessibility variants based on actual layout risk. Do not generate every possible family/state combination automatically.

## Links and interactions

Create URLs from the app's canonical contract and verify they resolve through app-level routing at cold and warm launch. If access requires login, preserve the requested destination according to that routing policy.

Verify which interaction mechanisms the chosen family, platform, and deployment target support. Do not attach multiple `widgetURL` modifiers to one hierarchy. Test app-opening links separately from in-place actions; the latter need an appropriate intent-backed service operation and fresh data/authorization checks. [Apple: links](https://developer.apple.com/documentation/widgetkit/linking-to-specific-app-scenes-from-your-widget-or-live-activity), [Apple: interactivity](https://developer.apple.com/documentation/widgetkit/adding-interactivity-to-widgets-and-live-activities)

## Validation matrix

| Check | Scope |
| --- | --- |
| Mapping/provider tests | Content, errors, stale data, configuration and reload policy |
| Action tests | Relevant mutation semantics, authorization, persisted result |
| App route tests | Correct destination, readiness and login handling |
| Family previews | Layout and entry states in selected contexts |
| App/extension build | Target dependencies, resources, platform API compatibility |
| Installed widget | Configuration, launch links, supported interaction, observed updates |
| Signed/shared-container check | Correct entitlements and actual shared-data access |

For manual checks, exercise changes made while the app is open and after it closes, missing cache, account changes, and multiple configured widget instances as relevant. Report OS-controlled timing as an observation, not a guaranteed refresh contract.

Do not expand an ordinary widget task into Live Activities, push infrastructure, or additional extension types unless requested.
