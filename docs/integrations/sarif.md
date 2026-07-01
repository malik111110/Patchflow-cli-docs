# SARIF Uploads

SARIF is the preferred format for security dashboards and code scanning
systems.

## Generate SARIF

```bash
patchflow scan run --format sarif --output patchflow.sarif
```

## Validate In CI

PatchFlow validates SARIF during benchmark runs. In normal CI, keep the SARIF
file as an artifact even if the scan fails:

```yaml
- run: patchflow scan run --format sarif --output patchflow.sarif
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: patchflow-sarif
    path: patchflow.sarif
```

## GitHub Code Scanning

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

## Recommended Metadata

Keep these outputs together:

| File | Purpose |
| --- | --- |
| `patchflow.sarif` | Code scanning upload |
| `patchflow-report.md` | Human-readable review artifact |
| `patchflow-report.json` | Automation and dashboards |
