---
title: SARIF Uploads
description: Generate and upload SARIF to GitHub Code Scanning and security dashboards
---

# SARIF Uploads

SARIF (Static Analysis Results Interchange Format) is the industry standard
format for security tool results. PatchFlow generates SARIF 2.1.0 output
compatible with GitHub Code Scanning, Azure DevOps, and other security
dashboards.

## Generate SARIF

```bash
patchflow scan run --format sarif --output patchflow.sarif
```

Or use the `report` command:

```bash
patchflow report --format sarif --output patchflow.sarif
```

Or use `scan export` for additional format options:

```bash
patchflow scan export --format sarif --output patchflow.sarif
```

## Upload to GitHub Code Scanning

### Using the GitHub CodeQL Action

```yaml
- uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: patchflow.sarif
```

The workflow needs:

```yaml
permissions:
  contents: read
  security-events: write
```

### Using PatchFlow's Built-in Upload

```bash
patchflow scan export --format sarif --upload-github
```

This uploads SARIF directly to GitHub Code Scanning via the GitHub API. Requires
the `GITHUB_TOKEN` environment variable. PatchFlow resolves the repository,
commit SHA, and ref automatically.

## Validate SARIF in CI

Keep the SARIF file as an artifact even if the scan fails:

```yaml
- run: patchflow scan run --format sarif --output patchflow.sarif
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: patchflow-sarif
    path: patchflow.sarif
```

## SARIF Content

PatchFlow's SARIF output includes:

- **Tool information**: PatchFlow CLI name and version
- **Rules**: All rules that generated findings, with metadata (CWE, OWASP,
  severity, confidence)
- **Results**: Each finding with location (file, line), message, rule ID, and
  severity
- **Run metadata**: Scan profile, governance profile, scan duration

## Recommended Artifacts

Keep these outputs together in CI:

| File | Purpose |
| --- | --- |
| `patchflow.sarif` | Code scanning upload |
| `patchflow-report.md` | Human-readable review artifact |
| `patchflow-report.json` | Automation and dashboards |

## Other SARIF Consumers

SARIF is a standard format supported by many tools:

- **GitHub Code Scanning**: Native SARIF upload via CodeQL action
- **Azure DevOps**: SARIF viewer extension
- **SonarQube**: SARIF importer
- **DefectDojo**: SARIF import for vulnerability management
- **OWASP Dependency-Track**: SARIF import for dependency scanning

## Next Steps

- [GitHub Actions](./github-actions.md) — Full CI integration
- [Reports](../user-guides/reports.md) — All report formats
- [Scan Your Project](../user-guides/scan.md) — Scan command reference
