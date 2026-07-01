---
title: YAML Policy Reference
description: Full YAML policy format for custom rules and framework overrides
---

# YAML Policy Reference

This page documents the full YAML policy format for `.patchflow/rules.yaml`.

## File Location

```text
.patchflow/rules.yaml
```

Override with `--rules` flag:

```bash
patchflow scan run --rules .patchflow/custom-rules.yaml
```

## Validation

```bash
patchflow rules validate
patchflow rules validate .patchflow/rules.yaml
```

Validation checks:

1. File exists and is valid YAML
2. Each rule has required fields: `id`, `title`, `pattern`, `severity`
3. Severity is one of: `low`, `medium`, `high`, `critical`
4. Pattern is a valid regex (compiles without error)
5. Languages field is present and non-empty
6. Rule IDs are unique
7. Rule IDs match `^[A-Z][A-Z0-9]{2,}-[A-Z0-9]+$`

## Full YAML Structure

```yaml
# ──────────────────────────────────────────────
# Framework pack selection
# ──────────────────────────────────────────────
frameworks:
  auto_detect: true
  enabled:
    - express
    - django
  disabled:
    - spring-security

# ──────────────────────────────────────────────
# Framework pack overrides
# ──────────────────────────────────────────────
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
  rails:
    severity_overrides:
      PF-RAILS-XSS-001: medium
    custom_sanitizers:
      - func: sanitize_html
    custom_sources:
      - func: current_tenant_param
    custom_sinks:
      - func: render_raw_html

# ──────────────────────────────────────────────
# Custom regex rules
# ──────────────────────────────────────────────
rules:
  - id: ACME-001
    title: Dangerous eval usage
    pattern: "eval\\("
    severity: high
    languages: [python, javascript]
    confidence: medium
    cwe: "CWE-95"
    fix_hint: "Avoid eval(); use ast.literal_eval or JSON.parse"
  - id: ACME-002
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
  auto_detect: true        # Auto-detect frameworks (default: true)
  enabled:                 # Force-enable packs
    - express
    - django
  disabled:                # Disable packs even if detected
    - spring-security
```

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `auto_detect` | bool | true | Auto-detect frameworks from filesystem signals |
| `enabled` | list | (none) | Force-enable specific packs |
| `disabled` | list | (none) | Disable packs even if detected |

## Framework Overrides

Framework overrides extend official packs with application-specific sources,
sinks, sanitizers, and severity policy.

### Severity Overrides

```yaml
framework_overrides:
  PACK_NAME:
    severity_overrides:
      RULE_ID: SEVERITY
```

Example:

```yaml
framework_overrides:
  django:
    severity_overrides:
      PF-DJANGO-XSS-002: medium
      PF-DJANGO-SQLI-001: critical
```

### Custom Sanitizers

```yaml
framework_overrides:
  PACK_NAME:
    custom_sanitizers:
      - func: FUNCTION_NAME
      - regex: REGEX_PATTERN
```

Example:

```yaml
framework_overrides:
  express:
    custom_sanitizers:
      - func: validateRedirectTarget
      - regex: "allowlistedHost\\("
```

When a sanitizer is detected in the code path, the finding is suppressed.
Sanitizers can be specified by function name (`func`) or regex pattern (`regex`).

### Custom Sources

```yaml
framework_overrides:
  PACK_NAME:
    custom_sources:
      - func: FUNCTION_NAME
```

Example:

```yaml
framework_overrides:
  rails:
    custom_sources:
      - func: current_tenant_param
```

Custom sources extend taint rules with application-specific entry points for
tainted data.

### Custom Sinks

```yaml
framework_overrides:
  PACK_NAME:
    custom_sinks:
      - func: FUNCTION_NAME
```

Example:

```yaml
framework_overrides:
  rails:
    custom_sinks:
      - func: render_raw_html
```

Custom sinks extend taint rules with application-specific dangerous endpoints.

## Custom Rules

Custom rules are regex-based patterns that run alongside the embedded
`patterns-embedded` scanner.

### Rule Fields

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

| Language | Identifier |
| --- | --- |
| Python | `python` |
| JavaScript | `javascript` |
| TypeScript | `typescript` |
| Ruby | `ruby` |
| PHP | `php` |
| Java | `java` |
| C# | `csharp` |
| Go | `go` |
| Rust | `rust` |
| YAML | `yaml` |
| Dockerfile | `dockerfile` |
| Terraform | `terraform` |

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

  - id: ACME-004
    title: Insecure SSL/TLS configuration
    pattern: "verify\\s*=\\s*False|SSL_VERIFY_PEER\\s*=\\s*false"
    severity: high
    languages: [python, javascript]
    confidence: high
    cwe: "CWE-295"
    fix_hint: "Always verify SSL/TLS certificates"
```

## Rule ID Format

Rule IDs must match the pattern `^[A-Z][A-Z0-9]{2,}-[A-Z0-9]+$`:

- Starts with an uppercase letter
- Followed by 2+ uppercase letters or digits
- A hyphen separator
- Followed by 1+ uppercase letters or digits

Examples: `ACME-001`, `MYAPP-XSS-001`, `TEAM-SEC-002`

## Next Steps

- [Custom Rules](../user-guides/custom-rules) — User guide for custom rules
- [Framework Packs](../user-guides/framework-packs) — Official pack reference
- [Rule Governance](./rules-governance) — Maturity and profiles
