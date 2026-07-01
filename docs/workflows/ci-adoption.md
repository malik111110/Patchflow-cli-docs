# CI Adoption Strategy

Adopting PatchFlow in CI requires a phased approach to avoid blocking teams on
historical findings. This page outlines a practical adoption strategy.

## Step 1: Observe (Week 1–2)

Add PatchFlow to CI without blocking. Collect findings and triage false
positives.

```yaml
# GitHub Actions — observe mode
- run: patchflow scan run --profile standard --format sarif --output patchflow.sarif
- uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: patchflow.sarif
- run: patchflow report --format markdown --output patchflow-report.md
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: patchflow-report
    path: |
      patchflow.sarif
      patchflow-report.md
```

**Actions during this phase:**

- Review all findings and identify false positives
- Suppress false positives with `//patchflow:ignore` directives
- Categorize real findings by priority
- Document the triage process for the team

## Step 2: Baseline (Week 2–3)

Create a baseline from the default branch to snapshot historical findings.

```bash
# On the default branch
patchflow scan run --profile deep
patchflow baseline create --name v1.0

# Commit the baseline
git add .patchflow/baselines/v1.0.json
git commit -m "chore: add PatchFlow baseline v1.0"
```

## Step 3: Gate on New Findings (Week 3–4)

Start blocking CI on new high or critical findings relative to the baseline.

```yaml
- run: patchflow scan run --new-only --baseline v1.0 --fail-on high --format sarif --output patchflow.sarif
```

**Actions during this phase:**

- Monitor CI for false positives and adjust suppressions
- Communicate the gating policy to the team
- Provide a clear process for disputing findings

## Step 4: Enforce on Protected Branches (Week 4+)

Apply the same gate to protected branch pushes.

```yaml
on:
  pull_request:
  push:
    branches: [main]
```

## Step 5: Periodic Deep Audit (Ongoing)

Schedule weekly deep audits to review the full finding surface:

```yaml
on:
  schedule:
    - cron: '0 9 * * 1'  # Monday 09:00 UTC

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: patchflow scan run --profile deep --governance-profile audit --format json --output audit.json
      - uses: actions/upload-artifact@v4
        with:
          name: patchflow-audit
          path: audit.json
```

## Tuning Governance Profiles

| CI Stage | Governance Profile | Rationale |
| --- | --- | --- |
| Developer (local) | `dev` | Fast, stable rules only |
| Pull request | `ci` | Stable + beta as warnings |
| Protected branch | `ci` | Stable + beta, blocking on stable+ |
| Audit | `audit` | All rules including experimental |

```bash
patchflow scan run --governance-profile ci
```

## Managing False Positives

### In-Code Suppression

For findings that are genuinely false positives:

```python
# patchflow:ignore PY001 -- eval is safe here, input is from trusted config
result = eval(config_expression)
```

### Custom Sanitizers (Framework Packs)

For framework rules, add custom sanitizers in `.patchflow/rules.yaml`:

```yaml
framework_overrides:
  express:
    custom_sanitizers:
      - func: validateRedirectTarget
```

### Severity Overrides

For findings that are real but lower priority than the default severity:

```yaml
framework_overrides:
  django:
    severity_overrides:
      PF-DJANGO-XSS-002: medium
```

## Refreshing Baselines

After fixing historical findings, refresh the baseline:

```bash
patchflow scan run --profile deep
patchflow baseline create --name v2.0
git add .patchflow/baselines/v2.0.json
git commit -m "chore: refresh PatchFlow baseline to v2.0"
```

Update CI to use the new baseline:

```yaml
- run: patchflow scan run --new-only --baseline v2.0 --fail-on high
```

## Metrics to Track

| Metric | How to Measure | Goal |
| --- | --- | --- |
| New findings per PR | Count from `--new-only` output | Trend toward zero |
| False positive rate | Triage findings, track suppressions | < 10% |
| Time to fix | Time from finding to resolution | < 1 sprint for high/critical |
| Baseline burn-down | Compare baseline sizes over time | Decreasing |
| Scan time | CI scan duration | < 5 minutes for standard profile |

## Next Steps

- [Recommended Workflow](./recommended.md) — Full workflow guide
- [GitHub Actions](../integrations/github-actions.md) — CI integration
- [Baselines](../user-guides/baselines.md) — Baseline management
