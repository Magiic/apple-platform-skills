# Swift Package Templates (Architecture-Oriented)

## Interface module skeleton
- Public domain models.
- Public protocols.
- No concrete SDK imports.
- No app-specific composition logic.

## Feature module skeleton
- Module configuration object.
- ViewModel/state handler.
- SwiftUI entry view.
- Internal mappers for display models/errors.
- Unit test target required.

## Infrastructure module skeleton
- Concrete repository/client implementations.
- External SDK integration and request/response mapping.
- Error translation to domain/infrastructure errors.
- No UI concerns.

## Naming guidance
- Prefer explicit names:
- `UserProfileInterface`, `UserProfileFeature`, `UserProfileClient`
- Keep names consistent across package/product/target when possible.
