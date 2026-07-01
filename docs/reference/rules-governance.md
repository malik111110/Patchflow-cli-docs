---
title: Rule Governance
description: Rule maturity levels, governance profiles, and blocking eligibility
---

# Rule Governance

PatchFlow's governance system controls which rules are visible, which findings
can block CI, and how rules mature over time. This page documents the governance
model in detail.

## Maturity Levels

Every rule carries a maturity level that reflects its test coverage,
false-positive rate, and production readiness.

| Maturity | Value | Description | Typical Visibility |
| --- | --- | --- | --- |
| `experimental` | 0 | New rule, few tests, may have false positives | Audit profile only |
| `beta` | 1 | Has tests, enabled in audit/standard, non-blocking | CI and audit, non-blocking |
| `stable` | 2 | Has false-positive tests, CWE/OWASP mapping, can block | Developer, PR, CI, audit |
| `enterprise` | 3 | Large regression corpus, low false-positive rate | All profiles, blocking |

### Promotion Criteria

Rules are promoted through maturity levels based on:

| From | To | Requirements |
| --- | --- | --- |
| (new) | `experimental` | Rule implemented, basic positive fixture |
| `experimental` | `beta` | Positive fixtures, safe fixtures, normal no-noise fixtures |
| `beta` | `stable` | All fixture types, CWE/OWASP mapping, real-repo benchmark coverage, clean-repo noise checks |
| `stable` | `enterprise` | Large regression corpus, very low false-positive rate, production-validated |

## Governance Profiles

Profiles filter findings by rule maturity. They control which rules are visible
and which findings can block CI.

| Profile | Purpose | Rules Included | Blocking |
| --- | --- | --- | --- |
| `dev` | Local development, fast feedback | Stable only | No |
| `pr` | Pull request review signal | Stable | Stable+ can block |
| `ci` | CI warnings and blocking gates | Stable + beta | Stable+ can block |
| `audit` | Full rule surface for security audits | All (including experimental) | Configurable |

### Default Profile Mapping

When `--governance-profile` is not specified, the governance profile defaults
based on the scan profile:

| Scan Profile | Default Governance Profile |
| --- | --- |
| `quick` | `dev` |
| `standard` | `ci` |
| `deep` | `audit` |

Override with `--governance-profile`:

```bash
patchflow scan run --governance-profile audit
patchflow scan run --profile standard --governance-profile pr
```

## Engine Types

Rules are registered under engine types in the governance registry:

| Engine | Description | Default Maturity |
| --- | --- | --- |
| `gosast-embedded` | Go AST-based rules (ported from gosec) | Stable |
| `taint-ssa` | Go SSA-based taint analysis | Stable |
| `secrets-embedded` | Regex-based secret detection | Stable |
| `treesitter-ast` | Tree-sitter AST rules | Beta |
| `taint-patterns` | Tree-sitter source-to-sink taint patterns | Beta |
| `patterns-embedded` | Multi-language regex patterns | Beta |
| `framework-packs` | Framework-specific rule packs | Experimental |

## Blocking Eligibility

A finding is "blocking-eligible" in CI when all of the following are true:

1. The rule maturity is `stable` or `enterprise`
2. The governance profile includes the rule
3. The finding severity meets the `--fail-on` threshold

### Example

```bash
patchflow scan run --governance-profile ci --fail-on high
```

With `ci` profile:
- `stable` and `enterprise` rules: findings at or above `high` severity block CI
- `beta` rules: findings are visible but do not block (warnings only)
- `experimental` rules: findings are not visible (audit profile only)

## Governance Commands

### Maturity Coverage Report

```bash
patchflow rules maturity
```

Shows:

- Maturity level distribution (experimental, beta, stable, enterprise counts)
- Blocking-eligible vs excluded rule counts
- CWE and OWASP mapping coverage
- Profile activation counts (dev, pr, ci, audit)
- Per-engine rule counts

```bash
patchflow rules maturity --json
patchflow rules maturity --all
```

`--all` lists every rule with its maturity and CWE mapping.

### List Rules

```bash
# Summary of all rules
patchflow rules list

# All rules with full details
patchflow rules list --all

# Filter by framework pack
patchflow rules list --framework express

# JSON output
patchflow rules list --json
```

### List Framework Packs

```bash
patchflow rules list-frameworks
patchflow rules list-frameworks --json
```

Shows each pack with name, language, file/template extensions, rule count, and
auto-detection status.

### Generate Rule Documentation

```bash
patchflow rules docs
patchflow rules docs --output rules.md
patchflow rules docs --engine gosast-embedded
```

Generates Markdown documentation for all registered rules, including:

- Rule ID, title, severity, confidence
- Maturity level and blocking eligibility
- CWE and OWASP mapping
- Security category
- Fix recommendation
- Active scan profiles

## CWE and OWASP Mapping

Rules map to CWE (Common Weakness Enumeration) and OWASP Top 10 categories:

| OWASP | CWE Examples | Description |
| --- | --- | --- |
| A01 | CWE-200, CWE-201, CWE-359 | Broken Access Control |
| A02 | CWE-327, CWE-328, CWE-916 | Cryptographic Failures |
| A03 | CWE-79, CWE-89, CWE-95 | Injection |
| A04 | CWE-502, CWE-915 | Insecure Design |
| A05 | CWE-276, CWE-732 | Security Misconfiguration |
| A06 | CWE-1104, CWE-1357 | Vulnerable and Outdated Components |
| A07 | CWE-287, CWE-306 | Identification and Authentication Failures |
| A08 | CWE-829, CWE-937 | Software and Data Integrity Failures |
| A09 | CWE-778, CWE-532 | Security Logging and Monitoring Failures |
| A10 | CWE-79, CWE-116 | Server-Side Request Forgery |

Use `patchflow rules maturity --all` to see the CWE and OWASP mapping for each
rule.

## Registry Architecture

The governance registry (`internal/rules/`) provides:

- `NewRegistry()`: Creates empty registry
- `Register(meta RuleMetadata)`: Adds or updates rule metadata
- `RegisterEngineRule(...)`: Convenience for scanner engines
- `Get(id string)`: Returns metadata for a rule
- `All()`: Returns all rules sorted by engine then ID
- `ByEngine(engine)`: Returns rules from a specific engine
- `MaturityCounts()`: Returns a map of maturity level → count
- `Coverage()`: Computes governance coverage report
- `IsRuleActiveInProfile(id, profile)`: Checks if a rule is active in a profile
- `ShouldBlock(id, profile)`: Checks if a finding should block CI

Framework rules are added to the governance registry via
`packs.RegisterFrameworkRules(reg)`, which bridges `frameworks.Maturity` to
`rules.Maturity`.

## Next Steps

- [Scanner Reference](./scanners) — Embedded scanner details
- [Framework Packs](../user-guides/framework-packs) — Official pack reference
- [Custom Rules](../user-guides/custom-rules) — Custom rules and overrides
