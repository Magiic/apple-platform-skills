# Preview Validation and Troubleshooting

## Choose coverage from layout risks

- Long translated text or RTL for directional layouts.
- Accessibility text sizes for dense controls, rows, and fixed-height regions.
- Light/dark appearance for custom colors, images, and contrast.
- Relevant compact/regular sizes or widget families, according to actual platform support.
- Empty, missing optional content, and large values for conditional layouts.

Keep variants easy to identify. Use the package's real localization and resources where possible; do not hard-code translated fixture text to simulate that localization wiring works.

## Diagnose in boundary order

| Symptom | Check first |
| --- | --- |
| Preview does not compile | Target scheme/destination, API availability, dependencies, conditional compilation |
| Missing environment crash | Entry-level configuration and all dependencies required by the selected internal view |
| Auth, network, or telemetry starts | Preview factory is accidentally constructing production services or a singleton |
| Loading preview instantly becomes content | `.task` or another lifecycle effect overrides the requested state |
| One preview changes another | Shared mutable singleton/static context or cached preview context |
| Resources or strings are missing | Owning package bundle, processed resources, localization configuration |
| Repeated state reset | Observable object recreated during `body` or unstable view identity |
| Debug works, Release fails | Unconditional references to `#if DEBUG` support types/factories |

Fix the cause at its boundary. Do not broaden production visibility, import the app target, or introduce a global preview mode service locator as a shortcut.

## Verify and report

1. Build the affected target using a compatible toolchain and destination.
2. Render the selected scenarios in the Xcode preview environment when available.
3. Inspect representative size, appearance, and accessibility variants.
4. Exercise local preview interactions when applicable and confirm they use isolated dependencies.
5. Check Release compilation when preview support or environment defaults changed conditional availability.

Do not call a SwiftUI preview visually verified when only a build passed. If the canvas is unavailable, report that limit and the specific compilation or alternative UI checks performed. Snapshot/UI tests are separate tools and should follow the user's task scope.

Use [Apple's preview documentation](https://developer.apple.com/documentation/xcode/adding-previews-to-your-interface-files) for supported declaration and state APIs. Keep this skill focused on deterministic composition and coverage rather than copying a version-specific catalog of preview modifiers.
