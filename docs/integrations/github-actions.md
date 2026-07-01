---
title: GitHub Actions
description: Use PatchFlow in GitHub Actions for pull request scanning and code scanning
---

# GitHub Actions

Use PatchFlow in GitHub Actions to scan pull requests, upload SARIF to code
scanning, and enforce security gates.

## Generate a Workflow

PatchFlow can generate a GitHub Actions workflow for you:

```bash
patchflow init github-actions
```

This creates `.github/workflows/patchflow-scan.yml` with a complete workflow that:

- Triggers on push to main/master, pull requests, and weekly schedule
- Sets up Go 1.25
- Installs PatchFlow via `go install`
- Runs a scan with SARIF output
- Uploads SARIF to GitHub Code Scanning
- Uploads SARIF as an artifact

## Minimal Workflow

```yaml
name: PatchFlow

on:
  pull_request:
  push:
    branches: [main]

jobs:
  patchflow:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.25'
      - run: go install github.com/Patchflow-security/patchflow-cli@latest
      - run: patchflow scan run --profile standard --format sarif --output patchflow.sarif
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: patchflow.sarif
```

### Required Permissions

```yaml
permissions:
  contents: read
  security-events: write
```

The `security-events: write` permission is required for SARIF upload to GitHub
Code Scanning.

## With Baseline and Exit Gate

After creating a baseline (see [Baselines](../user-guides/baselines.md)):

```yaml
- run: patchflow scan run --new-only --baseline v1.0 --fail-on high --format sarif --output patchflow.sarif
```

This blocks CI on new high or critical findings relative to the baseline.

## With Markdown Report Artifact

```yaml
- run: patchflow report --format markdown --output patchflow-report.md
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: patchflow-report
    path: |
      patchflow.sarif
      patchflow-report.md
```

## With Backend Authentication

For backend-connected review commands:

```yaml
- run: patchflow login --token "$PATCHFLOW_TOKEN"
  env:
    PATCHFLOW_TOKEN: ${{ secrets.PATCHFLOW_TOKEN }}
- run: patchflow review pr --submit
```

Keep the token in GitHub Secrets. Never store tokens in repository YAML files.

## Complete Production Workflow

```yaml
name: PatchFlow Security

on:
  pull_request:
  push:
    branches: [main]
  schedule:
    - cron: '0 9 * * 1'  # Weekly audit on Monday

jobs:
  patchflow:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for baseline comparison

      - uses: actions/setup-go@v5
        with:
          go-version: '1.25'

      - name: Install PatchFlow
        run: go install github.com/Patchflow-security/patchflow-cli@latest

      - name: Run scan (PR)
        if: github.event_name == 'pull_request'
        run: |
          patchflow scan run \
            --new-only --baseline v1.0 \
            --fail-on high \
            --format sarif --output patchflow.sarif

      - name: Run scan (push/audit)
        if: github.event_name != 'pull_request'
        run: |
          patchflow scan run \
            --profile deep --governance-profile audit \
            --format sarif --output patchflow.sarif

      - name: Generate markdown report
        if: always()
        run: patchflow report --format markdown --output patchflow-report.md

      - name: Upload SARIF to code scanning
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: patchflow.sarif

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: patchflow-report
          path: |
            patchflow.sarif
            patchflow-report.md
```

## Using the GitHub Action

PatchFlow also provides a GitHub Action for simpler integration:

```yaml
- uses: Patchflow-security/patchflow-cli@v1
  with:
    command: scan run --profile standard --format sarif --output patchflow.sarif
```

## Tips

- Use `fetch-depth: 0` when working with baselines to ensure full git history
  is available
- Use `if: always()` on SARIF upload and artifact steps so results are preserved
  even when the scan fails
- Pin the PatchFlow version in production workflows for reproducibility
- Use GitHub Secrets for tokens — never commit tokens to the repository

## Next Steps

- [SARIF Uploads](./sarif.md) — SARIF format details
- [Baselines](../user-guides/baselines.md) — Baseline management
- [Recommended Workflow](../workflows/recommended.md) — Team adoption
