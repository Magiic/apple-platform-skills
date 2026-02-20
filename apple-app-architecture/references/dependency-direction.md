# Dependency Direction Rules

## Allowed
- Feature depends on Interface and shared utilities.
- Infrastructure depends on Interface contracts.
- App target depends on Features and Infrastructure for wiring.

## Forbidden
- Interface depending on Feature or Infrastructure.
- Feature depending on App target.
- Circular dependencies between packages.
- Views directly depending on provider-specific SDK APIs when abstraction exists.

## Practical check
For each dependency edge `A -> B`, ask:
- Does `A` need `B`'s abstraction or only its implementation details?
- Could `A` be tested with a fake of `B`?
- Would replacing `B` force UI refactor in `A`?
If the answer reveals tight coupling, dependency direction is likely wrong.
