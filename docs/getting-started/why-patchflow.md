---
title: Why PatchFlow
description: Why PatchFlow replaces stacking Trivy, Semgrep, Gitleaks, and scripts
---

# Why PatchFlow

Most security scanning setups are a stack of independent tools held together
with shell scripts and hope. PatchFlow replaces that stack with a single binary
that covers SCA, SAST, secret detection, reachability, license scanning,
framework-aware rules, reports, baselines, fixes, and PR review — all local,
all from one command.

## The Problem: Tool Stacking

A typical security pipeline looks like this:

```text
┌─────────────┐   ┌───────────┐   ┌───────────┐   ┌──────────┐   ┌─────────┐
│   Trivy     │ + │  Semgrep  │ + │ Gitleaks  │ + │ OSV-Scan │ + │ Scripts │
│  (SCA+IaC)  │   │   (SAST)  │   │ (Secrets) │   │   (SCA)  │   │ (Glue)  │
└─────────────┘   └───────────┘   └───────────┘   └──────────┘   └─────────┘
      │                 │               │               │              │
      └─────────────────┴───────────────┴───────────────┴──────────────┘
                                    │
                          Different output formats
                          Different severity scales
                          Different rule IDs
                          Different update cycles
                          No deduplication
                          No unified risk score
                          No reachability
                          No fix suggestions
                          No baseline comparison
```

Each tool has its own:

- **Output format**: JSON, SARIF, text, custom
- **Severity scale**: Critical/High/Medium/Low vs Error/Warning/Info
- **Rule IDs**: `AVD-DS-0002` vs `python.lang.security.audit.dangerous-eval`
- **Update cycle**: Weekly, monthly, never
- **Configuration**: YAML, TOML, INI, env vars
- **CI integration**: Different action/workflow for each

And none of them provide:

- **Reachability analysis** (is the vulnerable dependency actually used?)
- **Unified risk score** (one number for the whole project)
- **Baseline comparison** (only fail on new findings)
- **Fix suggestions** (here is how to fix it, not just "you have a problem")
- **Framework-aware rules** (Rails, Express, Django, Spring-specific sources/sinks)
- **PR review** (risk score for this specific change)

## The PatchFlow Alternative

```text
┌─────────────────────────────────────────────────────────────┐
│                      patchflow scan run                      │
├─────────────────────────────────────────────────────────────┤
│  SCA (OSV.dev)  │  SAST (6 engines)  │  Secrets (40+ patterns) │
│  License scan   │  Framework packs   │  Reachability analysis  │
│  Risk scoring   │  Baseline diff     │  Fix suggestions        │
│  PR review      │  Report generation │  SARIF export           │
└─────────────────────────────────────────────────────────────┘
                              │
                    One binary, one output
                    One severity scale
                    One rule ID format
                    One configuration
                    One CI integration
```

## Feature Comparison

| Feature | PatchFlow | Trivy | Semgrep | Gitleaks | OSV-Scanner |
| --- | --- | --- | --- | --- | --- |
| SCA (dependency vulnerabilities) | Yes | Yes | No | No | Yes |
| SAST (code analysis) | Yes | No | Yes | No | No |
| Secret detection | Yes | Limited | No | Yes | No |
| Reachability analysis | Yes | No | No | No | No |
| License scanning | Yes | Yes | No | No | No |
| Framework-aware rules | Yes (17 packs) | No | Limited | No | No |
| Container image scanning | Yes (via Trivy) | Yes | No | No | No |
| Unified risk score | Yes | No | No | No | No |
| Baseline comparison | Yes | No | No | No | No |
| Fix suggestions | Yes | No | Limited | No | No |
| PR review | Yes | No | No | No | No |
| SARIF output | Yes | Yes | Yes | Yes | Yes |
| SBOM generation | Yes (CycloneDX, SPDX) | Yes | No | No | No |
| Offline mode | Yes | Limited | Yes | Yes | No |
| Single binary | Yes | Yes | No (Python) | Yes | Yes |
| No backend required | Yes | Yes | Yes | Yes | Yes |

## What You Get From Consolidating

### 1. One Output Format

Instead of parsing Trivy JSON + Semgrep JSON + Gitleaks JSON and merging them,
PatchFlow produces one unified output:

```bash
patchflow scan run --format json --output patchflow-report.json
```

Every finding has the same structure: rule ID, severity, file, line, confidence,
CWE, fix hint.

### 2. One Severity Scale

PatchFlow uses a consistent severity scale across all scanners: `critical`,
`high`, `medium`, `low`. No more mapping "Error" to "High" and "Warning" to
"Medium" across tools.

### 3. One CI Integration

One GitHub Actions workflow, one SARIF upload, one exit gate:

```yaml
- run: patchflow scan run --new-only --baseline v1.0 --fail-on high --format sarif --output patchflow.sarif
- uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: patchflow.sarif
```

### 4. Reachability That Actually Helps

Trivy and OSV-Scanner tell you that `lodash@4.17.20` has a CVE. PatchFlow tells
you whether `lodash` is actually imported in your code, and with what confidence:

```text
Reachability
------------
Reachable vulnerabilities: 7 of 12
  High confidence: 4   (imported and used in source files)
  Medium confidence: 2  (imported but usage unclear)
  Low confidence: 1     (transitive dependency, no direct import)
```

This eliminates the noise of "you have 200 vulnerable dependencies" when only 7
are actually reachable.

### 5. Baselines For Incremental Adoption

PatchFlow baselines let you adopt security scanning on existing projects without
blocking on historical findings:

```bash
patchflow baseline create --name v1.0
patchflow scan run --new-only --baseline v1.0 --fail-on high
```

This blocks CI only on new high/critical findings. Historical findings are
tracked separately for burn-down.

### 6. Fix Suggestions, Not Just Findings

PatchFlow does not just tell you what is wrong — it shows you how to fix it:

```bash
patchflow fix suggest --severity high --auto-only
patchflow fix apply --all --auto-only --yes
```

For each finding, PatchFlow provides:

- The vulnerable code
- The fixed code
- A unified patch
- Confidence level
- Whether the fix is safe to auto-apply

### 7. Framework-Aware Rules

Generic SAST tools flag `eval()` the same way in every project. PatchFlow
framework packs understand framework-specific sources, sinks, and sanitizers:

- **Rails**: `params` → `find_by_sql` is SQL injection, but `params` →
  `where(name: params[:name])` is safe
- **Express**: `req.query` → `res.redirect` is open redirect, but
  `validateRedirectTarget(req.query.url)` is sanitized
- **Django**: `request.GET` → `cursor.execute` with string formatting is SQL
  injection, but parameterized queries are safe

This reduces false positives dramatically compared to generic pattern matching.

## When To Keep Using Other Tools

PatchFlow does not replace every tool in every scenario:

| Scenario | Tool | Why |
| --- | --- | --- |
| Container image scanning | Trivy (directly) | PatchFlow wraps Trivy for image scanning, but if you only need container scans, Trivy alone is simpler |
| Custom SAST rules (complex) | Semgrep | Semgrep's rule language is more expressive for highly custom rules. PatchFlow's custom rules are regex-based |
| IaC scanning (detailed) | Checkov | Checkov has deeper IaC policy coverage. PatchFlow covers basic IaC patterns |
| Dependency scanning only | OSV-Scanner | If you only need SCA, OSV-Scanner is lighter |

PatchFlow's value is consolidation: one tool, one output, one CI integration,
one severity scale, with reachability and baselines on top.

## Cost Of The Stack vs PatchFlow

| Aspect | Tool Stack | PatchFlow |
| --- | --- | --- |
| Tools to install | 3–5 | 1 |
| CI workflows to maintain | 3–5 | 1 |
| Output formats to parse | 3–5 | 1 |
| Severity scales to normalize | 2–3 | 1 |
| Rule ID formats to learn | 3–5 | 1 |
| Update cycles to track | 3–5 | 1 |
| Deduplication logic to write | Custom scripts | Built-in |
| Reachability analysis | Not available | Built-in |
| Baseline comparison | Custom scripts | Built-in |
| Fix suggestions | Not available | Built-in |
| PR review | Not available | Built-in |

## Next Steps

- [Quickstart](./quickstart.md) — Try PatchFlow in 30 seconds
- [Core Concepts](./concepts.md) — Understand the analysis pipeline
- [Local vs Backend](./local-vs-backend.md) — What data leaves your machine
- [Recommended Workflow](../workflows/recommended.md) — Adoption strategy
