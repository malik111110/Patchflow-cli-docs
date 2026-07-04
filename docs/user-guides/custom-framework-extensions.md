---
title: Custom Framework Extensions
description: Extend official framework packs with organization-specific sources, sinks, sanitizers, and safe patterns
---

# Custom Framework Extensions

Custom framework extensions let you add organization-specific sources, sinks,
sanitizers, and safe patterns to official framework packs. This is how PatchFlow
understands your internal libraries — not just public framework APIs.

## Why Extensions?

Every engineering team has internal libraries that generic scanners don't know
about:

- A `LegacySql.run()` wrapper that all legacy code uses for database queries
- An `@TenantInput` annotation that marks multi-tenant request data
- A `TenantAuth.requireOwner()` call that validates ownership before sensitive
  operations
- A `CompanySql.safe()` sanitizer that wraps queries with parameterization

Without extensions, PatchFlow can't track taint through these internal APIs.
With extensions, PatchFlow understands them exactly like official framework
APIs — and scopes them by CWE to prevent cross-rule noise.

## Available Framework Packs

Extensions can be added to any of the 18 official framework packs. The framework
name in your YAML must match the pack name exactly:

| Framework | Pack name | Language |
| --- | --- | --- |
| Spring | `spring` | Java |
| Spring Security | `spring-security` | Java |
| Express | `express` | JavaScript |
| React | `react` | JavaScript |
| Next.js | `nextjs` | JavaScript |
| Angular | `angular` | TypeScript |
| NestJS | `nestjs` | TypeScript |
| Django | `django` | Python |
| Flask | `flask` | Python |
| FastAPI | `fastapi` | Python |
| GraphQL | `graphql` | Python |
| Rails | `rails` | Ruby |
| Laravel | `laravel` | PHP |
| Symfony | `symfony` | PHP |
| ASP.NET | `aspnet` | C# |
| Razor | `razor` | C# |
| Gin | `gin` | Go |
| Echo | `echo` | Go |

See the [Framework Packs](../frameworks/index) page for the full rule tables,
sources, sinks, and sanitizers of each pack.

## Configuration File

Extensions are defined in `.patchflow/rules.yaml`:

```yaml
schema_version: "1.0"

framework_extensions:
  spring:
    custom_sources:
      - annotation: "@TenantInput"
        categories: [sql_injection, path_traversal]
    custom_sinks:
      - function: "LegacySql.run"
        cwe: "CWE-89"
        category: "sql_injection"
      - function: "InternalHttp.fetch"
        cwe: "CWE-918"
        category: "ssrf"
    custom_sanitizers:
      - function: "CompanySql.safe"
    safe_patterns:
      - pattern: "TenantAuth.requireOwner"
        reason: "Ownership validation by internal auth helper"
```

Use the `--config` flag to specify the config file:

```bash
patchflow scan run --config .patchflow/rules.yaml
```

## Schema Reference

### `framework_extensions`

A map of framework name to extension entries. The framework name must match an
official pack name (e.g., `spring`, `express`, `rails`).

### `custom_sources`

Sources are entry points where untrusted data enters the application.

| Field | Required | Description |
| --- | --- | --- |
| `function` | One of function/annotation | Function name that returns untrusted data |
| `annotation` | One of function/annotation | Annotation that marks untrusted parameters |
| `is_subscript` | No | If true, source is a subscript access (e.g., `req["query"]`) |
| `categories` | No | Limit this source to specific vulnerability categories |

**Example: Function source**

```yaml
custom_sources:
  - function: "getTenantContext"
```

**Example: Annotation source**

```yaml
custom_sources:
  - annotation: "@TenantInput"
    categories: [sql_injection, path_traversal]
```

**Example: Subscript source**

```yaml
custom_sources:
  - function: "req"
    is_subscript: true
```

### `custom_sinks`

Sinks are dangerous functions where untrusted data should not reach.

| Field | Required | Description |
| --- | --- | --- |
| `function` | Yes | Function name (accepts `function` or `func` as YAML key) |
| `arg_index` | No | Which argument to track (default: 0) |
| `cwe` | No | CWE identifier (e.g., `CWE-89`) — scopes sink to matching rules |
| `category` | No | Vulnerability category (e.g., `sql_injection`) — scopes sink to matching rules |
| `severity` | No | Override severity for findings from this sink |

**Example: Scoped sink (recommended)**

```yaml
custom_sinks:
  - function: "LegacySql.run"
    cwe: "CWE-89"
    category: "sql_injection"
```

This sink will only attach to SQLi rules, not SSRF or redirect rules.

**Example: Unscoped sink (backward compatible, noisy)**

```yaml
custom_sinks:
  - function: "GenericDanger.run"
```

This sink attaches to ALL taint rules. `patchflow rules validate` will warn
about unscoped sinks.

### `custom_sanitizers`

Sanitizers are functions that neutralize untrusted data.

| Field | Required | Description |
| --- | --- | --- |
| `function` | One of function/regex | Function name that sanitizes input |
| `regex` | One of function/regex | Regex pattern that identifies sanitization |

**Example: Function sanitizer**

```yaml
custom_sanitizers:
  - function: "CompanySql.safe"
```

**Example: Regex sanitizer**

```yaml
custom_sanitizers:
  - regex: "ParameterizedQuery\\("
```

### `safe_patterns`

Safe patterns suppress findings when the pattern is present in the same
function as the finding. This works for both pattern/template rules and taint
rules.

| Field | Required | Description |
| --- | --- | --- |
| `pattern` | Yes | Regex pattern to match |
| `reason` | No | Human-readable explanation of why this is safe |

**Example:**

```yaml
safe_patterns:
  - pattern: "TenantAuth.requireOwner"
    reason: "Ownership validation by internal auth helper"
```

If `TenantAuth.requireOwner()` appears in the same function as a taint finding,
the finding is suppressed.

## Sink Scoping by CWE/Category (B11.5.1)

Custom sinks can be scoped by CWE or category to prevent cross-rule noise:

```yaml
custom_sinks:
  - function: "LegacySql.run"
    cwe: "CWE-89"           # only attaches to SQLi rules
  - function: "InternalHttp.fetch"
    category: "ssrf"        # only attaches to SSRF rules
```

A sink with no CWE/category attaches to ALL taint rules (backward compatible,
but noisy). Always scope your sinks.

### How scoping works

1. Each framework rule has a CWE and category (e.g., `CWE-89` / `sql_injection`)
2. Each custom sink can optionally carry a CWE and/or category
3. During `ApplyPackOverride`, a sink is only attached to rules whose CWE or
   category matches
4. If the sink has no CWE/category, it attaches to all rules (legacy behavior)

### Supported categories

| Category | CWE |
| --- | --- |
| `sql_injection` | CWE-89 |
| `xss` | CWE-79 |
| `open_redirect` | CWE-601 |
| `ssrf` | CWE-918 |
| `command_injection` | CWE-78 |
| `deserialization` | CWE-502 |
| `path_traversal` | CWE-22 |
| `csrf` | CWE-352 |
| `xxe` | CWE-611 |
| `idor` | CWE-639 |
| `mass_assignment` | CWE-915 |

## Source Category Scoping (B11.5.2)

Custom sources can be scoped by category:

```yaml
custom_sources:
  - annotation: "@TenantInput"
    categories: [sql_injection, path_traversal]
```

This source will only attach to SQLi and path traversal rules, not SSRF or XSS
rules. A source with no categories attaches to all rules (backward compatible).

## Taint Safe Pattern Suppression (B11.5.3)

Safe patterns suppress taint-mode findings when the pattern matches in the same
function:

```yaml
safe_patterns:
  - pattern: "TenantAuth.requireOwner"
    reason: "Ownership validation"
```

### How it works

1. After scanning, PatchFlow reads each file with taint findings
2. It detects function boundaries (function/method definitions)
3. For each taint finding, it finds the containing function
4. It checks if any safe pattern regex matches within that function's line range
5. If a match is found, the finding is suppressed

This is intentionally conservative — it does not attempt whole-program auth
proof. It simply checks if the safe pattern appears in the same function.

## Validation

Validate your extensions before committing:

```bash
patchflow rules validate .patchflow/rules.yaml
```

Validation catches:

**Errors:**
- Missing required fields (`func`/`function`, `annotation`, `pattern`)
- Invalid regex patterns

**Warnings:**
- Sink with no CWE/category (will attach to ALL rules — potential noise)
- Duplicate source/sink/sanitizer entries
- Unknown framework name
- CWE format doesn't match `CWE-NNN`
- Missing `schema_version`

Example output:

```text
✓ 0 rules, 1 framework overrides, and 1 framework extensions validated successfully
  ⚠ framework_extensions.spring.custom_sinks[0] UnscopedSink.run: no cwe or category — will attach to ALL taint rules
  ⚠ framework_extensions.spring.custom_sources[1]: duplicate source "@TenantInput"
  ⚠ schema_version missing, assuming 1.0
```

## Config Migration

If you have an old config with `framework_overrides`, migrate it:

```bash
patchflow config migrate
```

This creates `.patchflow/rules.migrated.yaml` with:
- `schema_version: "1.0"` added
- `framework_overrides` copied to `framework_extensions` (for CWE scoping and
  safe pattern support)

Review the migrated file, then replace your old config:

```bash
mv .patchflow/rules.migrated.yaml .patchflow/rules.yaml
```

## YAML Field Aliases

Both `func` and `function` are accepted as YAML keys for sources, sinks, and
sanitizers. Use whichever feels more natural:

```yaml
# These are equivalent:
custom_sinks:
  - func: "LegacySql.run"
  - function: "LegacySql.run"
```

## Examples

### Spring with internal SQL wrapper and auth helper

```yaml
schema_version: "1.0"
framework_extensions:
  spring:
    custom_sources:
      - annotation: "@TenantInput"
    custom_sinks:
      - function: "LegacySql.run"
        cwe: "CWE-89"
        category: "sql_injection"
    custom_sanitizers:
      - function: "CompanySql.safe"
    safe_patterns:
      - pattern: "TenantAuth.requireOwner"
        reason: "Ownership validation"
```

### Express with internal database wrapper

```yaml
schema_version: "1.0"
framework_extensions:
  express:
    custom_sinks:
      - function: "db.raw"
        cwe: "CWE-89"
        category: "sql_injection"
```

### Rails with organization auth helper

```yaml
schema_version: "1.0"
framework_extensions:
  rails:
    safe_patterns:
      - pattern: "TenantAuth.require_owner"
        reason: "Ownership validation by internal auth helper"
```

### Django with custom request decorator

```yaml
schema_version: "1.0"
framework_extensions:
  django:
    custom_sources:
      - function: "tenant_request"
        categories: [sql_injection, path_traversal]
    custom_sinks:
      - function: "legacy_cursor.execute"
        cwe: "CWE-89"
```

### FastAPI with internal auth dependency

```yaml
schema_version: "1.0"
framework_extensions:
  fastapi:
    custom_sources:
      - function: "get_tenant_id"
        categories: [sql_injection, path_traversal]
    custom_sinks:
      - function: "legacy_db.execute"
        cwe: "CWE-89"
        category: "sql_injection"
    safe_patterns:
      - pattern: "require_tenant_scope"
        reason: "Tenant scope validation before data access"
```

### Gin with internal SQL wrapper

```yaml
schema_version: "1.0"
framework_extensions:
  gin:
    custom_sinks:
      - function: "legacyDB.Query"
        cwe: "CWE-89"
        category: "sql_injection"
    custom_sanitizers:
      - function: "sanitizeQuery"
```

### Angular with custom sanitizer

```yaml
schema_version: "1.0"
framework_extensions:
  angular:
    custom_sanitizers:
      - function: "CustomSanitizer.sanitize"
    safe_patterns:
      - pattern: "validateRouteData"
        reason: "Custom route data validation before rendering"
```

### Rails with organization auth helper

```yaml
schema_version: "1.0"
framework_extensions:
  rails:
    safe_patterns:
      - pattern: "TenantAuth.require_owner"
        reason: "Ownership validation by internal auth helper"
    custom_sinks:
      - function: "LegacyConnection.execute"
        cwe: "CWE-89"
        category: "sql_injection"
```

## Limitations

- Extensions only **add** — you cannot remove official sources/sinks/sanitizers
- Safe pattern suppression for taint rules uses function-scope regex matching
  (not whole-program auth proof)
- Unscoped sinks (no CWE/category) attach to all taint rules — validate warns
  about this
- Function boundary detection is regex-based (not AST-based) — may miss
  nested/anonymous functions

## See Also

- [Framework Packs Overview](../frameworks/index)
- [Custom Rules (YAML)](./custom-rules)
- [Config Validation](../reference/configuration)
- [Schema Versioning](../reference/schema-versioning)
