---
title: Config Migrate Command
description: Migrate old PatchFlow config to the current schema with patchflow config migrate
---

# patchflow config migrate

Migrate an old `.patchflow/rules.yaml` to the current schema. This command does
NOT overwrite your original config — it writes a migrated copy for you to review.

## Syntax

```bash
patchflow config migrate [path] [flags]
```

## What It Does

1. Reads the existing config file
2. Adds `schema_version: "1.0"` if missing
3. Copies `framework_overrides` entries to `framework_extensions` (for CWE
   scoping and safe pattern support)
4. Writes the migrated config to `.patchflow/rules.migrated.yaml`

## Flags

| Flag | Default | Description |
| --- | --- | --- |
| `--output` | `.patchflow/rules.migrated.yaml` | Output path for migrated config |

## Examples

```bash
# Migrate default config
patchflow config migrate

# Migrate a specific file
patchflow config migrate .patchflow/old-rules.yaml

# Migrate to a custom output path
patchflow config migrate --output .patchflow/rules.v2.yaml
```

## Migration Output

```text
✓ Migrated config written to .patchflow/rules.migrated.yaml

Changes made:
  + added schema_version: "1.0"
  + migrated framework_overrides.spring → framework_extensions.spring

Next steps:
  1. Review .patchflow/rules.migrated.yaml
  2. Validate: patchflow rules validate .patchflow/rules.migrated.yaml
  3. If satisfied, replace your old config: mv .patchflow/rules.migrated.yaml .patchflow/rules.yaml
```

## patchflow config validate

Validate a config file without running a scan:

```bash
patchflow config validate
patchflow config validate .patchflow/rules.yaml
```

Checks:
- File exists and is valid YAML
- `schema_version` is present (warns if missing)
- `framework_overrides` detected (suggests migration to `framework_extensions`)

## See Also

- [Custom Framework Extensions](../user-guides/custom-framework-extensions)
- [Schema Versioning](./schema-versioning)
- [Configuration Reference](./configuration)
