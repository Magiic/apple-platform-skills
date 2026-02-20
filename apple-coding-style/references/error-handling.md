# Error Handling

## Principles
- Use typed errors for expected failures.
- Keep error mapping at layer boundaries.
- Return user-safe messages in UI layer.
- Keep technical details for logs/diagnostics.

## Patterns
- Domain errors:
- `enum FeatureError: Error { ... }`
- Boundary mapping:
- Map provider/network/database errors into domain errors.
- UI mapping:
- Convert domain errors into displayable error data.

## Avoid
- Exposing raw SDK/network errors directly to UI.
- Swallowing errors silently.
- Using generic `Error` everywhere without structure.
