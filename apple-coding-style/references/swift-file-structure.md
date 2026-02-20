# Swift File Structure

## Recommended structure
1. Imports
2. Main type declaration
3. Initializer(s)
4. Public methods/properties
5. Internal methods/properties
6. Private methods/properties
7. Extensions for conformances

## Practices
- Exactly one top-level type per file.
- Do not declare multiple top-level types in the same file.
- Co-locate small helper extensions with the owning type.
- Split files when responsibility grows too broad.
- Prefer explicit `MARK:` sections for long files.

## Nested type limits
- Maximum 3 nested types per parent type.
- Each nested type must not exceed 10 lines.
