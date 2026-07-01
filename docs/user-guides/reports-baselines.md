# Reports And Baselines

Reports make findings portable. Baselines make adoption practical on existing
repositories.

## Report Formats

```bash
patchflow report --format markdown --output patchflow-report.md
patchflow report --format json --output patchflow-report.json
patchflow report --format sarif --output patchflow.sarif
```

| Format | Use Case |
| --- | --- |
| Markdown | Human review, audit notes, PR comments |
| JSON | Automation, internal dashboards, data pipelines |
| SARIF | GitHub code scanning and security dashboards |

## Baseline Existing Findings

Create a baseline after the first full scan:

```bash
patchflow baseline create main
```

Then scan only new findings:

```bash
patchflow scan run --baseline main --new-only
```

## Recommended Baseline Flow

1. Run a full scan on the default branch.
2. Create a named baseline.
3. Store the baseline in CI artifacts or repository configuration as your team prefers.
4. Gate pull requests on new high or critical findings.
5. Schedule separate work to burn down old findings.

## SARIF Upload

Generate SARIF:

```bash
patchflow scan run --format sarif --output patchflow.sarif
```

Upload it from CI to your code scanning provider.
