# YAML Policy Reference

Use `.patchflow/rules.yaml` to configure custom rules and framework pack
extensions.

## Framework Selection

```yaml
frameworks:
  auto_detect: true
  enabled:
    - express
    - react
  disabled:
    - spring-security
```

## Framework Overrides

```yaml
framework_overrides:
  express:
    severity_overrides:
      PF-EXPRESS-REDIRECT-001: high
    custom_sources:
      - func: tenantParam
    custom_sinks:
      - func: unsafeRedirect
    custom_sanitizers:
      - func: validateRedirectTarget
      - regex: "isAllowedUrl\\("
```

## Validation

```bash
patchflow rules validate --rules .patchflow/rules.yaml
```

Validation should run in CI when teams maintain shared policy files.

## Policy Rules

- Keep overrides close to the project that needs them.
- Prefer named helper functions over broad regex sanitizers.
- Review severity overrides with security owners.
- Add comments in pull requests explaining why an override is needed.
- Revisit overrides when framework packs are upgraded.
