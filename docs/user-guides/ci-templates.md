---
title: CI Templates
description: Generate CI/CD integration templates with patchflow ci init
---

# CI Templates

PatchFlow can generate CI/CD integration templates for popular platforms. The
generated files include PatchFlow installation, scan execution, SARIF upload, and
artifact publishing.

## Quick Start

```bash
# Generate a GitHub Actions workflow (audit profile, non-blocking)
patchflow ci init github

# Generate a GitLab CI pipeline (CI-blocking profile)
patchflow ci init gitlab --profile ci-blocking

# Generate a pre-commit hook
patchflow ci init pre-commit
```

## Supported Platforms

| Platform | Command | Output File |
| --- | --- | --- |
| GitHub Actions | `patchflow ci init github` | `.github/workflows/patchflow.yml` |
| GitLab CI | `patchflow ci init gitlab` | `.gitlab-ci-patchflow.yml` |
| CircleCI | `patchflow ci init circleci` | `.circleci/patchflow-config.yml` |
| Azure Pipelines | `patchflow ci init azure` | `azure-patchflow.yml` |
| pre-commit | `patchflow ci init pre-commit` | `.pre-commit-config.yaml` |

## Profiles

Three profiles control how aggressive the CI integration is:

### audit (default)

- **Findings reported**: Yes
- **CI blocking**: No (`continue-on-error: true`)
- **Recommended for**: First-time adoption, assessing the codebase

```bash
patchflow ci init github --profile audit
```

### starter

- **Findings reported**: Yes
- **CI blocking**: High severity only
- **Recommended for**: Teams that have triaged initial findings

```bash
patchflow ci init github --profile starter
```

### ci-blocking

- **Findings reported**: Yes
- **CI blocking**: High and critical severity
- **Recommended for**: Mature teams ready to enforce security gates

```bash
patchflow ci init github --profile ci-blocking
```

## Generated Workflow Example (GitHub Actions)

```yaml
name: PatchFlow Security Scan

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

permissions:
  contents: read
  security-events: write

jobs:
  patchflow:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install PatchFlow
        run: |
          curl -sSL https://github.com/Patchflow-security/patchflow-cli/releases/latest/download/patchflow_Linux_x86_64.tar.gz | tar xz
          sudo mv patchflow /usr/local/bin/

      - name: Run PatchFlow scan
        run: patchflow scan run --config .patchflow/rules.yaml --format sarif --output patchflow.sarif --fail-on never
        continue-on-error: true

      - name: Upload SARIF to GitHub Code Scanning
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: patchflow.sarif
```

## Custom Output Path

Use `--output` to write to a custom path:

```bash
patchflow ci init github --output .github/workflows/security.yml
patchflow ci init gitlab --output .gitlab-ci-security.yml
```

## Adoption Workflow

1. **Start with audit mode**: `patchflow ci init github --profile audit`
2. **Review findings**: Check the SARIF results in GitHub Code Scanning
3. **Triage and suppress**: Add `//patchflow:ignore` for false positives
4. **Create a baseline**: `patchflow baseline save initial` to suppress known
   findings
5. **Switch to starter**: `patchflow ci init github --profile starter`
6. **Eventually go ci-blocking**: When all high/critical findings are resolved

## See Also

- [GitHub Actions Integration](../integrations/github-actions)
- [GitLab CI Integration](../integrations/gitlab)
- [SARIF Output](../integrations/sarif)
- [Baselines](./baselines)
- [Suppressions](./suppressions)
