# Testing Boundaries by Module Type

## Interface modules
- Typical content: protocols, domain models, value types.
- Test target: optional.
- Rule: do not force interface-only modules to carry tests if they have no logic.

## Feature modules
- Typical content: view models, feature orchestration, state transitions, mapping.
- Test target: required.
- Minimum expectation:
- state transition coverage
- happy path and error path coverage
- repository interaction assertions (via mocks/spies)

## Utility/Infrastructure modules
- Test target: strongly recommended when logic is non-trivial.
- Cover parsing/mapping, retries, error mapping, and boundary conditions.

## UI tests
- Optional by default.
- Add when explicitly requested or when critical user journeys require regression protection.
