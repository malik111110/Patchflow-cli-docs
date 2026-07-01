# GitHub Actions

Use PatchFlow in GitHub Actions to scan pull requests and upload SARIF to code
scanning.

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
      - run: go install github.com/patchflow/patchflow-cli@latest
      - run: patchflow scan run --profile standard --format sarif --output patchflow.sarif
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: patchflow.sarif
```

## With Baseline And Exit Gate

```yaml
- run: patchflow scan run --baseline default --new-only --fail-on high --format sarif --output patchflow.sarif
```

Use this after a baseline has been created and committed or restored in the CI
environment.

## With Backend Authentication

```yaml
- run: patchflow login --token "$PATCHFLOW_TOKEN"
  env:
    PATCHFLOW_TOKEN: ${{ secrets.PATCHFLOW_TOKEN }}
- run: patchflow review pr --submit
```

Keep the token in GitHub Secrets. Do not store tokens in repository YAML files.

## Recommended Artifacts

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
