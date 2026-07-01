---
title: Custom Rules and YAML
description: Project-level YAML for custom regex rules and framework pack overrides
---

# Custom Rules and YAML

PatchFlow supports project-level YAML for custom regex rules and framework pack
overrides. The YAML file extends the official embedded rules — it does not
replace them.

## Rule File Location

By convention, PatchFlow looks for:

```text
.patchflow/rules.yaml
```

You can specify a different path with `--rules`:

```bash
patchflow scan run --rules .patchflow/custom-rules.yaml
patchflow rules validate --rules .patchflow/custom-rules.yaml
```

## Validation

Validate the rule file before committing:

```bash
patchflow rules validate
```

Or validate a specific file:

```bash
patchflow rules validate .patchflow/rules.yaml
```

Validation checks:

1. File exists and is valid YAML
2. Each rule has required fields: `id`, `title`, `pattern`, `severity`
3. Severity is one of: `low`, `medium`, `high`, `critical`
4. Pattern is a valid regex (compiles without error)
5. Languages field is present and non-empty
6. Rule IDs are unique
7. Rule IDs match the pattern `^[A-Z][A-Z0-9]{2,}-[A-Z0-9]+$`

Exit code is 0 if valid, 1 if invalid.

## Full YAML Structure

```yaml
# Framework pack selection
frameworks:
  auto_detect: true
  enabled:
    - express
    - django
  disabled:
    - spring-security

# Framework pack overrides
framework_overrides:
  express:
    severity_overrides:
      PF-EXPRESS-REDIRECT-001: high
    custom_sanitizers:
      - func: validateRedirectTarget
      - regex: "allowlistedHost\\("
    custom_sources:
      - func: getQueryParam
    custom_sinks:
      - func: rawResponse

# Custom regex rules
rules:
  - id: CUSTOM-001
    title: Dangerous eval usage
    pattern: "eval\\("
    severity: high
    languages: [python, javascript]
    confidence: medium
    cwe: "CWE-95"
    fix_hint: "Avoid eval(); use ast.literal_eval or JSON.parse"
  - id: CUSTOM-002
    title: Hardcoded API key
    pattern: "API_KEY\\s*=\\s*['\"][A-Za-z0-9]{32}['\"]"
    severity: critical
    languages: [python, javascript, go]
    confidence: high
    cwe: "CWE-798"
    fix_hint: "Load API keys from environment variables"
```

## Framework Selection

```yaml
frameworks:
  auto_detect: true
  enabled:
    - express
    - django
  disabled:
    - spring-security
```

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `auto_detect` | bool | true | Auto-detect frameworks from filesystem signals |
| `enabled` | list | (none) | Force-enable specific packs |
| `disabled` | list | (none) | Disable packs even if detected |

Use explicit `enabled` packs for monorepos or generated layouts where
auto-detection is incomplete.

## Framework Overrides

### Severity Overrides

```yaml
framework_overrides:
  django:
    severity_overrides:
      PF-DJANGO-XSS-002: medium
```

Severity overrides are useful when a team has compensating controls or a stricter
internal policy than the default pack.

### Custom Sanitizers

```yaml
framework_overrides:
  express:
    custom_sanitizers:
      - func: validateRedirectTarget
      - regex: "allowlistedHost\\("
```

Sanitizers can be specified by function name (`func`) or regex pattern (`regex`).
When a sanitizer is detected in the code path, the finding is suppressed.

### Custom Sources and Sinks

```yaml
framework_overrides:
  rails:
    custom_sources:
      - func: current_tenant_param
    custom_sinks:
      - func: render_raw_html
```

Add custom sources and sinks when an application wraps framework APIs behind
internal helpers. This enables taint rules to track data flow through
application-specific functions.

## Custom Rules

Custom rules are regex-based patterns that run alongside the embedded
`patterns-embedded` scanner. Each rule has the following fields:

| Field | Required | Type | Description |
| --- | --- | --- | --- |
| `id` | Yes | string | Unique rule ID matching `^[A-Z][A-Z0-9]{2,}-[A-Z0-9]+$` |
| `title` | Yes | string | Human-readable rule title |
| `pattern` | Yes | string | Valid regex pattern |
| `severity` | Yes | string | `low`, `medium`, `high`, `critical` |
| `languages` | Yes | list | Languages the rule applies to |
| `confidence` | No | string | `low`, `medium`, `high` (default: medium) |
| `cwe` | No | string | CWE identifier (e.g., `CWE-79`) |
| `fix_hint` | No | string | Short fix recommendation |

### Supported Languages

Custom rules support the following language identifiers:

- `python`
- `javascript`
- `typescript`
- `ruby`
- `php`
- `java`
- `csharp`
- `go`
- `rust`
- `yaml`
- `dockerfile`
- `terraform`

### Example Custom Rules

```yaml
rules:
  - id: ACME-001
    title: Internal admin endpoint exposed
    pattern: "admin_/[a-z_]+"
    severity: high
    languages: [python, javascript]
    confidence: medium
    cwe: "CWE-200"
    fix_hint: "Ensure admin endpoints require authentication"

  - id: ACME-002
    title: Weak password hashing
    pattern: "md5\\(|sha1\\("
    severity: high
    languages: [python, javascript, php, ruby]
    confidence: high
    cwe: "CWE-327"
    fix_hint: "Use bcrypt, scrypt, or Argon2 for password hashing"

  - id: ACME-003
    title: Debug mode in production
    pattern: "DEBUG\\s*=\\s*True|debug\\s*:\\s*true"
    severity: medium
    languages: [python, javascript, yaml]
    confidence: medium
    cwe: "CWE-489"
    fix_hint: "Disable debug mode in production configurations"
```

## Listing Custom Rules

Custom rules are included in `patchflow rules list` output when the rules file
is present:

```bash
patchflow rules list
patchflow rules list --all
patchflow rules list --json
```

## Recommended Policy

- Start with official packs and only add overrides after a finding has been
  triaged.
- Keep each override small, named, and reviewed like production code.
- Validate the rules file in CI with `patchflow rules validate`.
- Use severity overrides sparingly — prefer adding sanitizers to reduce false
  positives.
- Document the rationale for each custom rule and override in the rule file
  comments.

## Next Steps

- [Framework Packs](./framework-packs.md) — Official pack reference
- [Rule Governance](../reference/rules-governance.md) — Maturity and profiles
- [Suppressions](./suppressions.md) — Suppress false positives in code
