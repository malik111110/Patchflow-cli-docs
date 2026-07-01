---
title: Quickstart
description: Get started with PatchFlow CLI in under five minutes
---

# Quickstart

This guide takes a repository from zero to its first PatchFlow scan, report, and
baseline in under five minutes.

## 30-Second Path

If you already have PatchFlow installed and a Git repository open, run these
four commands:

```bash
patchflow init
patchflow scan run
patchflow report --format sarif --output patchflow.sarif
patchflow pr-review
```

That is the minimum loop: initialize, scan, export, and review. Everything
below goes deeper, but if you just want to see the tool work, start there.

## Prerequisites

- PatchFlow CLI installed as `patchflow` on your `PATH` (see
  [Installation](./installation))
- Git installed and available on `PATH`
- A Git repository as your working directory

## Try It on a Demo Repo

If you do not want to run PatchFlow on your real project yet, clone a small
intentionally-vulnerable repository and scan it:

```bash
git clone https://github.com/juice-shop/juice-shop.git
cd juice-shop
patchflow init
patchflow scan run
```

Juice Shop is a well-known intentionally-vulnerable Node.js application. It
produces realistic findings without risking your production code. Use it to
explore PatchFlow output, test SARIF export, and practice with baselines before
running scans on your own repository.

## Step 1: Initialize PatchFlow

Run `init` inside the repository you want to scan:

```bash
cd your-repo
patchflow init
```

Expected output:

```text
Initializing PatchFlow...
Created .patchflow/config.yml
Created .patchflow/baselines/
Created .patchflow/reports/
PatchFlow initialized successfully.
```

This creates a `.patchflow/` directory with:

```
.patchflow/
  config.yml      Project configuration
  baselines/      Baseline snapshots
  reports/        Generated reports
```

The `.patchflow/` directory is the project-level home for PatchFlow artifacts.
Baselines and reports are stored here. Custom rules go in
`.patchflow/rules.yaml` (see [Custom Rules](../user-guides/custom-rules)).

## Step 2: Run Your First Scan

```bash
patchflow scan run
```

The `scan run` command performs a full security analysis:

1. **SCA** — Parses dependency manifests and queries OSV.dev for known
   vulnerabilities
2. **License scanning** — Fetches and classifies license information from package
   registries
3. **SAST** — Runs embedded scanners (Go AST, multi-language patterns, tree-sitter
   AST, taint analysis) and external tools when available
4. **Secret detection** — Scans for hardcoded secrets using 40+ regex patterns
5. **Framework packs** — Auto-detects and activates framework-specific rule packs
6. **Reachability** — Determines whether vulnerable dependencies are actually
   imported and used
7. **Risk scoring** — Computes a 0–100 risk score from all findings

Expected output (abbreviated):

```text
PatchFlow Security Scan
========================

Repository:       /home/user/my-project
Branch:           main
Scan profile:     standard
Governance:       ci

SCA Analysis
------------
Dependencies scanned:    247
Vulnerable dependencies: 12
  Critical: 1
  High:      4
  Medium:    5
  Low:       2

SAST Analysis
-------------
Files scanned:     1,832
SAST findings:     23
  Critical: 0
  High:      6
  Medium:   12
  Low:       5

Secret Detection
----------------
Secrets found:     2
  High: 1 (AWS access key in src/config.js:42)
  Medium: 1 (Generic API key in .env.local:8)

Reachability
------------
Reachable vulnerabilities: 7 of 12
  High confidence: 4
  Medium confidence: 2
  Low confidence: 1

Risk Score: 67/100 (Warning)

Report generated: .patchflow/reports/patchflow-report-20260115-093012.md
SARIF generated:  .patchflow/reports/patchflow-report-20260115-093012.sarif
```

Your output will vary based on your project's dependencies, code, and
frameworks. The key sections to look for:

- **SCA Analysis**: How many dependencies have known vulnerabilities
- **SAST Analysis**: Code-level findings from embedded scanners
- **Secret Detection**: Hardcoded credentials or API keys
- **Reachability**: Which vulnerable dependencies are actually used (not just
  present)
- **Risk Score**: A single 0–100 number summarizing your project's risk

If the risk score is high, do not panic. Use `explain` to investigate findings
and create a baseline to focus on new issues first.

## Step 3: Export Results

Generate a report for review or CI integration:

```bash
# Markdown report (human-readable)
patchflow report --format markdown --output patchflow-report.md

# JSON report (automation, dashboards)
patchflow report --format json --output patchflow-report.json

# SARIF report (GitHub code scanning, security dashboards)
patchflow report --format sarif --output patchflow.sarif
```

Expected output:

```text
Generating markdown report...
Report written to patchflow-report.md (14.2 KB)
```

See [Reports](../user-guides/reports) for format details and
[SARIF Uploads](../integrations/sarif) for CI integration.

## Step 4: Explain a Finding

When a finding needs investigation, use `explain` to get the full context:

```bash
patchflow explain --rule PY001
```

Or explain a specific finding by its ID (shown in scan output):

```bash
patchflow explain PF-SCAN-001
```

Or explain a finding at a specific file and line:

```bash
patchflow explain --file src/app.py --line 42
```

Expected output (abbreviated):

```text
Rule:       PY001
Title:      Dangerous eval() usage
Severity:   high
Confidence: high
Scanner:    patterns-embedded
Language:   python
CWE:        CWE-95 (Improper Evaluation of Code with Untrusted Input)

Description:
  eval() executes arbitrary Python code. If the input comes from an untrusted
  source, this can lead to arbitrary code execution.

Fix hint:
  Use ast.literal_eval() for safe literal evaluation, or parse input with
  json.loads() if the input is JSON.

Fix snippet:
  # Vulnerable
  result = eval(user_input)

  # Fixed
  import ast
  result = ast.literal_eval(user_input)

Suppression:
  # patchflow:ignore PY001 -- eval is safe here, input is sanitized
  result = eval(user_input)
```

`explain` shows:

- What the issue is and why it is dangerous
- Where the evidence is (file, line, code snippet)
- How to fix it (with example code)
- How to suppress it if it is a false positive
- Whether it would block a PR (based on `--fail-on` thresholds)

For framework rules, `explain` also shows sources, sinks, sanitizers, safe
patterns, and exclusions. See [Explain](../user-guides/explain) for details.

## Step 5: Create a Baseline

For existing repositories with known findings, create a baseline so future scans
focus only on new issues:

```bash
# Run a full scan first to populate findings
patchflow scan run --profile deep

# Create a named baseline
patchflow baseline create --name v1.0

# Future scans: only show new findings
patchflow scan run --new-only --baseline v1.0
```

Expected output:

```text
Running full SAST scan...
Scan complete: 23 findings detected.
Baseline 'v1.0' created with 23 findings.
Stored at: .patchflow/baselines/v1.0.json
```

Baselines use stable semantic fingerprints (rule ID + scanner + normalized path +
normalized snippet) so findings survive line-number shifts from unrelated edits.
See [Baselines](../user-guides/baselines) for the full workflow.

## Step 6: Simulate a PR Review

Before opening a pull request, run `pr-review` to get a local risk assessment:

```bash
patchflow pr-review
```

Expected output (abbreviated):

```text
PatchFlow PR Review
====================

Base: main
Head: feature/add-login

Changed files: 8
Lines added:   142
Lines deleted:  23

Risk Score: 42/100 (Caution)

Findings:
  High:     1  (SQL injection in src/auth.py:28)
  Medium:   2  (Weak crypto, debug mode)
  Low:      1  (Missing input validation)

Reachability:
  1 reachable vulnerable dependency (lodash@4.17.20)

Recommendations:
  1. Fix SQL injection in src/auth.py:28 — use parameterized queries
  2. Replace MD5 with bcrypt for password hashing
  3. Disable debug mode in production configuration

Status: Caution — review findings before merging
```

This analyzes your current branch changes and produces:

- A risk score and risk level
- Vulnerability findings (scoped to changed files for SAST, full repo for SCA)
- Reachability data
- Recommendations
- Optional reviewer suggestions (`--suggest-reviewers`)
- Optional inline annotations (`--annotations`)
- Optional fix proposals (`--suggest-fixes`)

See [PR Review](../user-guides/pr-review) for details.

## Step 7: Set Up CI (GitHub Actions)

Once you are comfortable with local scans, add PatchFlow to your pull request
pipeline. This is the fastest path to GitHub code scanning integration:

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

After creating a baseline (Step 5), add an exit gate to block CI on new high or
critical findings:

```yaml
- run: patchflow scan run --new-only --baseline v1.0 --fail-on high --format sarif --output patchflow.sarif
```

See [GitHub Actions](../integrations/github-actions) for the complete
workflow and [CI Adoption Strategy](../workflows/ci-adoption) for the phased
adoption plan.

## Step 8: Authenticate (Optional)

Authentication is only required for backend-connected review commands
(`review pr --submit`, `review status`). All local analysis — scanning, SAST,
SCA, secrets, reachability, reports, baselines, fixes — works without
authentication. No source code or scan results leave your machine for local
analysis.

```bash
patchflow login --token "$PATCHFLOW_TOKEN"
patchflow auth status
```

In CI, provide the token via an environment variable. Never store tokens in
repository configuration files. See
[Authentication](../user-guides/authentication) and
[Local vs Backend](./local-vs-backend) for details.

## Verify Your Environment

If scans behave unexpectedly, run `doctor`:

```bash
patchflow doctor
```

Expected output:

```text
PatchFlow Environment Check
============================

Git:              installed (2.43.0)
Repository:       detected (/home/user/my-project)
Embedded scanners: all available
  gosast:         ready
  patterns:       ready
  secrets:        ready
  treesitter:     ready
  taint-ssa:      ready
  taint-patterns: ready

External tools:
  gosec:    not installed (optional)
  semgrep:  not installed (optional)
  bandit:   not installed (optional)
  gitleaks: not installed (optional)
  trivy:    not installed (optional)

Cache:            ~/.cache/patchflow/a1b2c3d4e5f6a7b8/
OSV database:     not downloaded (run 'patchflow cache update' for offline mode)

All checks passed.
```

`doctor` checks Git installation, repository detection, embedded scanners, and
external tool availability. External tools are optional — embedded scanners
work without them.

## Exit Codes

PatchFlow uses standard exit codes so CI pipelines can gate on scan results:

| Code | Meaning |
| --- | --- |
| 0 | Scan completed, no findings above `--fail-on` threshold |
| 1 | Scan completed, findings at or above `--fail-on` threshold |
| 2 | Scan failed (error, timeout, invalid configuration) |

```bash
# Fail CI on high or critical findings
patchflow scan run --fail-on high

# Fail CI only on critical findings
patchflow scan run --fail-on critical

# Combine with baseline: fail on new high+ findings only
patchflow scan run --new-only --baseline v1.0 --fail-on high
```

## Next Steps

- [Core Concepts](./concepts) — Understand the analysis pipeline
- [Scan Your Project](../user-guides/scan) — All scan flags and profiles
- [Why PatchFlow](./why-patchflow) — PatchFlow vs stacking Trivy + Semgrep + Gitleaks
- [Local vs Backend](./local-vs-backend) — What data leaves your machine
- [Common Errors](./common-errors) — Troubleshooting guide
- [Recommended Workflow](../workflows/recommended) — Team adoption strategy
