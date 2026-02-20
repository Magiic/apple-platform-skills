# Module Creation via Swift CLI

## Goal
Create modules in a reproducible way and avoid manual folder scaffolding.

## Standard creation flow
```bash
mkdir -p Packages/<ModuleName>
cd Packages/<ModuleName>
swift package init --type library --name <ModuleName> --enable-swift-testing
```

Notes:
- Prefer library type for most app modules.
- Keep package name aligned with module folder name.
- Update Package.swift immediately after scaffold.
- Register new package in your app/workspace dependency graph.

Optional helper script:

```bash
#!/usr/bin/env bash
set -euo pipefail

module_name="$1"
module_type="${2:-library}"
root_dir="${3:-Packages}"
module_dir="${root_dir}/${module_name}"

mkdir -p "${module_dir}"
cd "${module_dir}"
swift package init --type "${module_type}" --name "${module_name}" --enable-swift-testing
echo "Created ${module_name} at ${module_dir}"
```
