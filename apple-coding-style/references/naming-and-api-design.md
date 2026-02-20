# Naming and API Design

## Naming guidelines
- Prefer explicit intent:
- `fetchReviewQuestions()` over `loadData()`
- `submitDecision()` over `send()`
- Use domain vocabulary consistently across modules.
- Keep boolean names readable as questions (`is`, `has`, `can`).

## API shape
- Keep parameter labels meaningful.
- Avoid long parameter lists; use input structs where useful.
- Prefer immutable inputs and predictable outputs.
- Keep function side effects obvious in naming.

## Consistency
- Use the same term for the same concept across layers.
- Avoid aliases that create cognitive overhead.
