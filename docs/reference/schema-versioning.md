---
title: Schema Versioning
description: Config schema versioning for .patchflow/rules.yaml
---

# Schema Versioning

PatchFlow config files (`.patchflow/rules.yaml`) use schema versioning to handle
config evolution across CLI versions.

## Current Schema

```yaml
schema_version: "1.0"

rules:
  PF-SPRING-SSRF-001: block
  G601: off

framework_extensions:
  spring:
    custom_sinks:
      - function: "LegacySql.run"
        cwe: "CWE-89"
        category: "sql_injection
```

## Schema Version Field

The `schema_version` field at the top of the config file identifies which schema
version the config uses:

```yaml
schema_version: "1.0"
```

If the field is missing, PatchFlow assumes `1.0` and emits a warning during
validation:

```text
⚠ schema_version missing, assuming 1.0 (add 'schema_version: "1.0"' to suppress this warning)
```

## Validating Schema Version

```bash
patchflow rules validate .patchflow/rules.yaml
```

Output with missing schema_version:

```text
✓ 0 rules, 1 framework overrides, and 1 framework extensions validated successfully
  ⚠ schema_version missing, assuming 1.0
```

## Migrating Old Configs

If you have a config without `schema_version`, use `patchflow config migrate`:

```bash
patchflow config migrate
```

This adds `schema_version: "1.0"` and suggests `framework_extensions`
equivalents for `framework_overrides`.

## Version History

| Version | Changes |
| --- | --- |
| `1.0` | Initial schema. Includes `rules`, `custom_rules`, `custom_taint_rules`, `frameworks`, `framework_overrides`, `framework_extensions`, `schema_version` |

## See Also

- [Config Migration](./config-migrate)
- [Custom Framework Extensions](../user-guides/custom-framework-extensions)
- [YAML Policy Reference](./yaml-policy)
