---
title: Recommended Workflow for Solo Developers
description: Security scanning workflow for individual developers
---

# Recommended Workflow for Solo Developers

As a solo developer, you need security scanning that is fast, does not block
your flow, and catches real issues without noise. This workflow is designed for
one-person projects and small teams without dedicated security engineers.

## The Solo Dev Loop

```text
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │  1. patchflow scan run --profile quick           │
  │     (before commit, fast feedback)               │
  │                                                  │
  │  2. patchflow explain --rule RULE_ID           │
  │     (understand any unexpected finding)          │
  │                                                  │
  │  3. patchflow fix apply --all --auto-only --yes  │
  │     (auto-fix safe issues)                       │
  │                                                  │
  │  4. patchflow suppress RULE_ID ...             │
  │     (suppress false positives)                   │
  │                                                  │
  │  5. patchflow pr-review                           │
  │     (before opening a PR)                        │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

## Step 1: Quick Scan Before Commit

Run a fast scan before committing. Use the `quick` profile for fast feedback:

```bash
patchflow scan run --profile quick
```

The `quick` profile:

- Uses the `dev` governance profile (stable rules only, no experimental)
- Skips reachability analysis (faster)
- Scans all files but with lighter rules

Expected output:

```text
PatchFlow Security Scan (quick)
================================
Files scanned:  142
SAST findings:  3
  High:    1  (SQL injection in src/db.py:28)
  Medium:  1  (Weak crypto in src/auth.py:15)
  Low:     1  (Missing input validation)

Risk Score: 34/100 (Caution)
```

If the risk score is low and findings are manageable, proceed. If there are
high or critical findings, investigate before committing.

## Step 2: Explain Unexpected Findings

When a finding does not make sense, explain it:

```bash
patchflow explain --rule PY001
```

Or explain a finding at a specific location:

```bash
patchflow explain --file src/db.py --line 28
```

This shows what the rule detects, why it is dangerous, and how to fix it. If the
finding is a false positive, suppress it (Step 4).

## Step 3: Auto-Fix Safe Issues

Many findings have safe, deterministic fixes. Apply them in one command:

```bash
# Preview fixes first
patchflow fix suggest --auto-only

# Apply all auto-applicable fixes
patchflow fix apply --all --auto-only --yes --backup
```

The `--backup` flag creates `.bak` files before modifying source. The
`--auto-only` flag only applies fixes that are safe (high confidence,
deterministic transformation).

Review the changes with `git diff` before committing:

```bash
git diff
git add -A
git commit -m "fix: apply PatchFlow auto-fixes for security findings"
```

## Step 4: Suppress False Positives

For findings that are genuinely false positives, add suppression directives:

```bash
patchflow suppress PY001 --file src/app.py --line 42 --reason "eval is safe, input is from trusted config"
```

This adds a `# patchflow:ignore PY001 -- eval is safe, input is from trusted config`
comment above the finding. Future scans will skip this finding.

Only suppress findings you have investigated. Do not suppress blindly.

## Step 5: PR Review Before Opening

Before opening a pull request, run a local PR review:

```bash
patchflow pr-review
```

This gives you a risk score scoped to your changes, plus recommendations. If the
risk level is "Warning" or "BLOCKING", fix the issues before opening the PR.

For richer output:

```bash
patchflow pr-review --suggest-fixes --format pr-summary
```

## CI For Solo Developers

Even solo developers benefit from CI scanning. A minimal GitHub Actions
workflow:

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
```

This uploads findings to GitHub code scanning without blocking CI. Review
findings in the GitHub Security tab.

## When To Add Baselines

Add a baseline when your project has historical findings that you cannot fix
immediately:

```bash
patchflow scan run --profile deep
patchflow baseline create --name v1.0
```

Then gate CI on new findings only:

```bash
patchflow scan run --new-only --baseline v1.0 --fail-on high
```

For solo developers, this is optional. If your project is clean (no historical
findings), you do not need a baseline.

## Weekly Deep Scan

Once a week, run a deep scan with all rules (including experimental):

```bash
patchflow scan run --profile deep --governance-profile audit
```

This catches issues that the `quick` and `standard` profiles miss. Review the
output and decide which findings to fix, suppress, or track for later.

## Solo Dev Command Summary

| When | Command | Purpose |
| --- | --- | --- |
| Before commit | `patchflow scan run --profile quick` | Fast feedback |
| Investigate | `patchflow explain --rule RULE_ID` | Understand a finding |
| Fix | `patchflow fix apply --all --auto-only --yes --backup` | Auto-fix safe issues |
| Suppress FP | `patchflow suppress RULE_ID --file PATH --line N --reason "..."` | Suppress false positives |
| Before PR | `patchflow pr-review` | Risk assessment for changes |
| In CI | `patchflow scan run --profile standard --format sarif --output patchflow.sarif` | CI scanning |
| Weekly | `patchflow scan run --profile deep --governance-profile audit` | Deep audit |

## Next Steps

- [Quickstart](../getting-started/quickstart.md) — Get started in 30 seconds
- [Fixes](../user-guides/fixes.md) — Auto-fix security findings
- [Suppressions](../user-guides/suppressions.md) — Suppress false positives
- [Team Workflow](./teams.md) — Workflow for larger teams
