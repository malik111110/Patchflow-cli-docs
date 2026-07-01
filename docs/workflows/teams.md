---
title: Recommended Workflow for Teams
description: Phased team adoption with baselines, CI gates, and enforcement
---

# Recommended Workflow for Teams

For teams, PatchFlow needs to enforce security quality without blocking
development. This workflow uses baselines, CI gates, and phased adoption to
balance security with velocity.

## The Team Adoption Curve

```text
  Phase 1: Observe          Phase 2: Baseline         Phase 3: Gate
  (Week 1–2)                (Week 2–3)                (Week 3–4)

  ┌──────────┐              ┌──────────┐              ┌──────────┐
  │ Scan CI  │              │ Create   │              │ Gate on  │
  │ No block │ ──────────►  │ baseline │ ──────────►  │ new high │
  │ Collect  │              │ Snapshot │              │ Block CI │
  │ findings │              │ history  │              │ on new   │
  └──────────┘              └──────────┘              └──────────┘
                                                          │
                                                          ▼
                                                    ┌──────────┐
                                                    │ Phase 4: │
                                                    │ Enforce  │
                                                    │ on main  │
                                                    └──────────┘
```

## Phase 1: Observe (Week 1–2)

Add PatchFlow to CI without blocking. Collect findings and triage false
positives.

### Developer Workflow

Each developer runs PatchFlow locally before opening a PR:

```bash
# Quick scan before commit
patchflow scan run --profile quick

# PR review before opening
patchflow pr-review --suggest-fixes
```

### CI Workflow

```yaml
# .github/workflows/patchflow.yml
name: PatchFlow
on: [pull_request]
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
      - run: patchflow report --format markdown --output patchflow-report.md
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: patchflow-report
          path: |
            patchflow.sarif
            patchflow-report.md
```

### Actions During This Phase

- Review all findings in GitHub code scanning
- Suppress false positives with `//patchflow:ignore` directives
- Categorize real findings by priority
- Document the triage process for the team
- Communicate that CI is not blocking yet

## Phase 2: Baseline (Week 2–3)

Create a baseline from the default branch to snapshot historical findings.

```bash
# On the default branch
patchflow scan run --profile deep
patchflow baseline create --name v1.0

# Commit the baseline
git add .patchflow/baselines/v1.0.json
git commit -m "chore: add PatchFlow baseline v1.0"
```

### What the Baseline Does

- Snapshots all current SAST findings with stable fingerprints
- Enables `--new-only` mode to focus on new findings
- Preserves historical findings for separate burn-down
- Survives line-number shifts (findings are fingerprinted by rule ID + normalized
  path + normalized snippet, not line number)

## Phase 3: Gate on New Findings (Week 3–4)

Start blocking CI on new high or critical findings relative to the baseline.

```yaml
- run: patchflow scan run --new-only --baseline v1.0 --fail-on high --format sarif --output patchflow.sarif
```

### Exit Gate Behavior

| Condition | Exit Code | CI Result |
| --- | --- | --- |
| No new findings at or above threshold | 0 | Pass |
| New high or critical findings | 1 | Fail |
| Scan error or timeout | 2 | Fail |

### Actions During This Phase

- Monitor CI for false positives and adjust suppressions
- Communicate the gating policy to the team
- Provide a clear process for disputing findings
- Fix new high/critical findings before merging

## Phase 4: Enforce on Protected Branches (Week 4+)

Apply the same gate to protected branch pushes:

```yaml
on:
  pull_request:
  push:
    branches: [main]
```

## Phase 5: Periodic Deep Audit (Ongoing)

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
      - uses: actions/setup-go@v5
        with:
          go-version: '1.25'
      - run: go install github.com/Patchflow-security/patchflow-cli@latest
      - run: patchflow scan run --profile deep --governance-profile audit --format json --output audit.json
      - uses: actions/upload-artifact@v4
        with:
          name: patchflow-audit
          path: audit.json
```

## Team Roles

### Developer

```bash
# Before commit
patchflow scan run --profile quick

# Before opening a PR
patchflow pr-review --suggest-fixes

# Fix safe issues
patchflow fix apply --all --auto-only --yes --backup

# Suppress false positives (with reason)
patchflow suppress RULE_ID --file PATH --line N --reason "explanation"
```

### Reviewer

```bash
# Review PR risk
patchflow pr-review --suggest-reviewers --annotations

# Check for new findings vs baseline
patchflow scan run --new-only --baseline v1.0
```

### Security Lead

```bash
# Weekly deep audit
patchflow scan run --profile deep --governance-profile audit

# Review suppressions
patchflow scan run --show-suppressed

# Refresh baseline after burn-down
patchflow baseline create --name v2.0
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

## Managing False Positives At Scale

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

## Metrics To Track

| Metric | How to Measure | Goal |
| --- | --- | --- |
| New findings per PR | Count from `--new-only` output | Trend toward zero |
| False positive rate | Triage findings, track suppressions | < 10% |
| Time to fix | Time from finding to resolution | < 1 sprint for high/critical |
| Baseline burn-down | Compare baseline sizes over time | Decreasing |
| Scan time | CI scan duration | < 5 minutes for standard profile |

## Complete Production Workflow

```yaml
name: PatchFlow Security

on:
  pull_request:
  push:
    branches: [main]
  schedule:
    - cron: '0 9 * * 1'

jobs:
  patchflow:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-go@v5
        with:
          go-version: '1.25'

      - run: go install github.com/Patchflow-security/patchflow-cli@latest

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

## Next Steps

- [Solo Developer Workflow](./solo-dev) — Simpler workflow for individuals
- [CI Adoption Strategy](./ci-adoption) — Detailed adoption guide
- [Baselines](../user-guides/baselines) — Baseline management
- [GitHub Actions](../integrations/github-actions) — CI integration
