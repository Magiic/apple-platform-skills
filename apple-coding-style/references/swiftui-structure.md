# SwiftUI Structure

## View responsibilities
- Render state.
- Emit user intent.
- Compose reusable subviews.

## ViewModel or state-handler responsibilities
- Manage async operations.
- Transform domain models into display state.
- Handle loading/success/error transitions.
- Trigger navigation/output events where applicable.

## Practices
- Keep `body` readable; extract subviews early.
- Prefer computed properties for derived UI state.
- Keep side effects out of `body`.
- Avoid business rules in view files.

## Size limits
- A SwiftUI `View` type must not exceed 500 lines.
- A ViewModel or state-handler type must not exceed 500 lines.
- Split large implementations into focused components before crossing limits.
