---
title: Reports
description: Generate security analysis reports in Markdown, JSON, and SARIF
---

# Reports

The `patchflow report` command generates security analysis reports in Markdown,
JSON, or SARIF format. Reports make findings portable for review, CI integration,
and audit documentation.

## Generate Reports

```bash
# Markdown (human-readable, default)
patchflow report --format markdown --output patchflow-report.md

# JSON (automation, dashboards)
patchflow report --format json --output patchflow-report.json

# SARIF (GitHub code scanning, security dashboards)
patchflow report --format sarif --output patchflow.sarif
```

When `--output` is omitted, the report is written to stdout.

## Report Formats

| Format | Use Case | Content |
| --- | --- | --- |
| Markdown | Human review, audit notes, PR comments | Tables with findings, dependencies, risk score, recommendations |
| JSON | Automation, internal dashboards, data pipelines | Structured JSON with all findings, dependencies, risk score, metadata |
| SARIF | GitHub code scanning, security dashboards | SARIF 2.1.0 standard format for tool integration |

## Report Content

All report formats include:

- **SCA findings**: Vulnerable dependencies with CVE ID, severity, affected
  versions, and fixed versions
- **SAST findings**: Code vulnerabilities with rule ID, file, line, severity,
  confidence, and code snippet
- **Secret findings**: Detected secrets with type, file, line, and masked value
- **Dependency list**: All dependencies with name, version, ecosystem, and
  manifest path
- **Risk score**: 0–100 score with breakdown by category
- **Recommendations**: Actionable next steps based on findings

## Auto-Saved Reports

If the `.patchflow/reports/` directory exists (created by `patchflow init`),
reports are automatically saved there in addition to the specified output path.

## SARIF for CI

SARIF is the preferred format for CI integration. See
[SARIF Uploads](../integrations/sarif) for GitHub code scanning integration.

## Export Formats (scan export)

For SBOM and dependency graph formats, use `patchflow scan export`:

```bash
# CycloneDX SBOM with VEX
patchflow scan export --format cyclonedx-json --include-vex --output sbom.json

# SPDX SBOM
patchflow scan export --format spdx-json --output spdx.json

# VEX (Vulnerability Exploitability Exchange)
patchflow scan export --format vex-json --output vex.json

# Dependency tree (text)
patchflow scan export --format dep-tree

# Dependency graph (DOT format)
patchflow scan export --format dep-dot
```

| Format | Description |
| --- | --- |
| `json` | Full analysis result as JSON |
| `sarif` | SARIF 2.1.0 for code scanning |
| `cyclonedx-json` | CycloneDX SBOM with optional VEX statements |
| `spdx-json` | SPDX SBOM format |
| `vex-json` | VEX (Vulnerability Exploitability Exchange) format |
| `dep-tree` | Dependency tree as text |
| `dep-dot` | Dependency graph as DOT format |

## GitHub SARIF Upload

Upload SARIF directly to GitHub Code Scanning:

```bash
patchflow scan export --format sarif --upload-github
```

Requires `GITHUB_TOKEN` environment variable and `--format sarif`. PatchFlow
resolves the GitHub repository, commit SHA, and uploads the SARIF file via the
GitHub API.

## Recommended Report Artifacts

Keep these outputs together in CI:

| File | Purpose |
| --- | --- |
| `patchflow.sarif` | Code scanning upload |
| `patchflow-report.md` | Human-readable review artifact |
| `patchflow-report.json` | Automation and dashboards |

## Next Steps

- [Baselines](./baselines) — Focus on new findings
- [SARIF Uploads](../integrations/sarif) — CI integration
- [Scan Your Project](./scan) — Scan command reference
