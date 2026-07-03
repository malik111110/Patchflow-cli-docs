---
title: The Problem We Solve
description: Why PatchFlow exists and what problem it solves for engineering teams
---

# The Problem We Solve

## The Security Tooling Problem

Engineering teams today face a fragmented security tooling landscape. A typical
project needs:

- **Trivy** for dependency vulnerabilities (SCA)
- **Semgrep** for multi-language SAST
- **gosec** for Go-specific analysis
- **bandit** for Python analysis
- **gitleaks** for secret detection
- **A SARIF aggregator** to merge results for CI
- **A custom script** to deduplicate overlapping findings

Each tool has its own config format, output format, severity scale, and update
cadence. Each tool produces false positives that someone has to triage. Each
tool needs to be installed, maintained, and kept in sync across developer
machines and CI environments.

The result is that most teams either:

1. **Don't scan at all** because the setup cost is too high
2. **Scan only in CI** where findings are discovered late, after code is merged
3. **Stack 5+ tools** and spend more time maintaining scanners than fixing
   vulnerabilities
4. **Use a cloud SaaS** that sends source code to a third party

## What PatchFlow Does Differently

PatchFlow CLI is a **single binary** that replaces the stack:

| Instead of | PatchFlow does |
| --- | --- |
| Trivy (SCA) | Embedded OSV.dev query with offline DB support |
| Semgrep (SAST) | 822 embedded rules across 24 scanners + 18 framework packs |
| gosec / bandit | Go AST + tree-sitter multi-language analysis |
| gitleaks | 40+ curated secret patterns with false-positive filtering |
| Custom dedup scripts | Semantic fingerprinting + function-boundary grouping |
| SARIF aggregator | Native SARIF 2.1.0 output with GitHub Code Scanning upload |

### Local-first, no backend required

All core analysis runs on your machine. Source code never leaves the local
environment unless you explicitly use a backend-connected review command. This
matters for:

- **Compliance**: GDPR, HIPAA, SOC 2, and internal data classification policies
- **Speed**: No network round-trips for SAST, secrets, or reachability
- **Privacy**: Source code stays on developer machines and CI runners
- **Offline**: `patchflow scan run --offline` works with a local OSV database

### Framework-aware, not just regex

Generic SAST tools flag `db.query("SELECT ...")` the same way whether you're
using Express, Django, or Spring. PatchFlow's framework packs understand:

- **Sources**: `req.query` (Express), `request.GET` (Django), `@RequestParam` (Spring)
- **Sinks**: `db.query` (Express), `cursor.execute` (Django), `jdbcTemplate.query` (Spring)
- **Sanitizers**: `escapeHtml` (Express), `escape` (Django), `HtmlUtils.htmlEscape` (Spring)
- **Template engines**: EJS, Jinja, Blade, ERB, Thymeleaf, Razor, JSX

This means fewer false positives and more accurate taint tracking.

### Custom framework extensions

Every team has internal libraries that generic scanners don't know about.
PatchFlow lets you extend official framework packs with your own sources, sinks,
sanitizers, and safe patterns — scoped by CWE to prevent cross-rule noise:

```yaml
framework_extensions:
  spring:
    custom_sinks:
      - function: "LegacySql.run"
        cwe: "CWE-89"
        category: "sql_injection"
    safe_patterns:
      - pattern: "TenantAuth.requireOwner"
        reason: "Ownership validation"
```

### Governance, not just detection

Rules carry maturity levels (experimental, beta, stable, enterprise) and map to
CWE and OWASP categories. Governance profiles control which rules are visible:

| Profile | Rules visible | Blocking |
| --- | --- | --- |
| `dev` | Stable + beta | High/critical only |
| `pr` | Stable + beta | High/critical only |
| `ci` | Stable only | High/critical |
| `audit` | All rules (including experimental) | None |

This means you can adopt PatchFlow incrementally — start with audit mode, verify
the findings, then switch to CI-blocking when you're ready.

## Who PatchFlow Is For

| Audience | Use case |
| --- | --- |
| **Solo developers** | Run `patchflow scan run` before pushing. Catch vulnerabilities locally. |
| **Small teams** | Add PatchFlow to CI. Use SARIF upload for inline GitHub annotations. |
| **Security engineers** | Use `--profile audit` to assess a codebase. Use custom extensions for internal libraries. |
| **Enterprise teams** | Use governance profiles to enforce blocking in CI. Use baselines to suppress known findings. |
| **Framework developers** | Use framework packs to understand framework-specific vulnerability patterns. |

## What PatchFlow Is Not

- **Not a cloud SaaS**: No backend required for core analysis. No source code
  leaves your machine.
- **Not a replacement for manual review**: PatchFlow finds patterns, not logic
  errors. Human review is still necessary for business-logic vulnerabilities.
- **Not a dependency scanner only**: PatchFlow does SCA + SAST + secrets +
  reachability + risk scoring in one tool.
- **Not a Semgrep clone**: PatchFlow has framework-aware taint tracking, not
  just regex patterns. But it can run Semgrep alongside it as an external tool.

## The Design Philosophy

1. **Local-first**: Source code stays on your machine
2. **Zero-install scanners**: Embedded SAST requires no external installation
3. **Framework-aware**: Official packs understand framework-specific patterns
4. **Governance-driven**: Maturity levels and profiles control rule visibility
5. **CI-ready**: SARIF, exit gates, baselines, incremental scanning
6. **Privacy-preserving**: Only OSV.dev API calls (replaceable with local DB)
7. **Extensible**: Custom framework extensions for internal libraries
8. **Reproducible**: Version metadata, schema versioning, config migration

## Next Steps

- [Why PatchFlow](./why-patchflow) — Detailed comparison with Trivy, Semgrep, Gitleaks
- [Quickstart](./quickstart) — Get running in under 5 minutes
- [Framework Packs](../frameworks/index) — Browse all 18 framework packs
- [Custom Framework Extensions](../user-guides/custom-framework-extensions) — Extend packs with your own rules
