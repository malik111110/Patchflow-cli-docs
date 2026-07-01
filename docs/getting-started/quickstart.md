# Quickstart

This guide takes a repository from zero to its first PatchFlow scan, report, and
baseline.

## Prerequisites

- PatchFlow CLI installed as `patchflow` on your `PATH` (see
  [Installation](./installation.md))
- Git installed and available on `PATH`
- A Git repository as your working directory

## Step 1: Initialize PatchFlow

Run `init` inside the repository you want to scan:

```bash
cd your-repo
patchflow init
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
`.patchflow/rules.yaml` (see [Custom Rules](../user-guides/custom-rules.md)).

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

The terminal output shows a summary with:

- Total findings by severity (critical, high, medium, low)
- Vulnerable dependencies
- Secrets detected
- SAST findings by scanner
- Reachability summary
- Risk score and risk level

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

See [Reports](../user-guides/reports.md) for format details and
[SARIF Uploads](../integrations/sarif.md) for CI integration.

## Step 4: Explain a Finding

When a finding needs investigation, use `explain` to get the full context:

```bash
patchflow explain --rule PY001
```

Or explain a specific finding by its ID (shown in scan output):

```bash
patchflow explain <finding-id>
```

Or explain a finding at a specific file and line:

```bash
patchflow explain --file src/app.py --line 42
```

`explain` shows:

- What the issue is and why it is dangerous
- Where the evidence is (file, line, code snippet)
- How to fix it (with example code)
- How to suppress it if it is a false positive
- Whether it would block a PR (based on `--fail-on` thresholds)

For framework rules, `explain` also shows sources, sinks, sanitizers, safe
patterns, and exclusions. See [Explain](../user-guides/explain.md) for details.

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

Baselines use stable semantic fingerprints (rule ID + scanner + normalized path +
normalized snippet) so findings survive line-number shifts from unrelated edits.
See [Baselines](../user-guides/baselines.md) for the full workflow.

## Step 6: Simulate a PR Review

Before opening a pull request, run `pr-review` to get a local risk assessment:

```bash
patchflow pr-review
```

This analyzes your current branch changes and produces:

- A risk score and risk level
- Vulnerability findings (scoped to changed files for SAST, full repo for SCA)
- Reachability data
- Recommendations
- Optional reviewer suggestions (`--suggest-reviewers`)
- Optional inline annotations (`--annotations`)
- Optional fix proposals (`--suggest-fixes`)

See [PR Review](../user-guides/pr-review.md) for details.

## Step 7: Authenticate (Optional)

Authentication is only required for backend-connected review commands
(`review pr --submit`, `review status`). All local analysis works without
authentication.

```bash
patchflow login --token "$PATCHFLOW_TOKEN"
patchflow auth status
```

In CI, provide the token via an environment variable. Never store tokens in
repository configuration files. See
[Authentication](../user-guides/authentication.md) for details.

## Verify Your Environment

If scans behave unexpectedly, run `doctor`:

```bash
patchflow doctor
```

`doctor` checks Git installation, repository detection, embedded scanners, and
external tool availability.

## Next Steps

- [Core Concepts](./concepts.md) — Understand the analysis pipeline
- [Scan Your Project](../user-guides/scan.md) — All scan flags and profiles
- [Recommended Workflow](../workflows/recommended.md) — Team adoption strategy
