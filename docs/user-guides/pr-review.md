---
title: PR Review
description: Simulate a PR risk review locally before opening a pull request
---

# PR Review

The `patchflow pr-review` command simulates a PR risk review locally, before you
open a pull request. It analyzes your current branch changes and computes a risk
score, vulnerability findings, reachability data, and recommendations — all
without a backend connection.

## Basic Usage

```bash
patchflow pr-review
```

This analyzes changes between your current branch and the auto-detected base
branch, then prints a terminal summary with:

- Risk score (0–100) and risk level
- Severity breakdown (critical, high, medium, low)
- Top findings
- Reachability summary
- Recommendations
- Status: "Ready for review", "Caution", "Warning", or "BLOCKING"

## How It Differs from `scan run`

| Aspect | `scan run` | `pr-review` |
| --- | --- | --- |
| Scope | Full repository | Changed files (SAST) + full repo (SCA) |
| Purpose | Full security analysis | Pre-PR risk assessment |
| Risk score | Computed from all findings | Computed from change-scoped findings |
| Reviewer suggestions | No | Yes (`--suggest-reviewers`) |
| Inline annotations | No | Yes (`--annotations`) |
| Fix proposals | Yes (`--suggest-fixes`) | Yes (`--suggest-fixes`) |
| Output formats | markdown, json, sarif | markdown, json, sarif, pr-summary, annotations |

## Specifying Base and Head Branches

```bash
# Auto-detect base branch (default)
patchflow pr-review

# Explicit base branch
patchflow pr-review --base main

# Explicit head branch
patchflow pr-review --base main --head feature/my-branch
```

## Output Formats

```bash
# Terminal summary (default)
patchflow pr-review

# Markdown report
patchflow pr-review --format markdown --output pr-review.md

# JSON report
patchflow pr-review --format json --output pr-review.json

# SARIF report
patchflow pr-review --format sarif --output pr-review.sarif

# PR summary (compact, for PR comments)
patchflow pr-review --format pr-summary

# Inline annotations
patchflow pr-review --format annotations
```

## Reviewer Suggestions

```bash
patchflow pr-review --suggest-reviewers
```

Suggests reviewers based on:

- **CODEOWNERS**: Parses `.github/CODEOWNERS` or `CODEOWNERS` files to find
  owners of changed files
- **Git blame**: Identifies developers who recently touched the changed code

Returns a maximum of 5 reviewer suggestions.

## Inline Annotations

```bash
patchflow pr-review --annotations
```

Generates inline code annotations for the PR diff. Each annotation shows the
finding at the specific file and line, making it easy to review issues in
context.

## Fix Proposals

```bash
patchflow pr-review --suggest-fixes
```

Generates safe fix proposals for detected vulnerabilities. Proposals are
classified as:

- **Auto-applicable**: Safe to apply without review
- **Needs review**: Requires human judgment before applying

See [Fixes](./fixes) for the fix command.

## Skipping Analysis Stages

```bash
# Skip SAST
patchflow pr-review --no-sast

# Skip secret detection
patchflow pr-review --no-secrets

# Skip reachability
patchflow pr-review --no-reachability
```

## Analysis Details

`pr-review` runs the following analysis:

1. **Git detection**: Detects repository, changed files, and diff stats
2. **SCA analysis**: Full dependency scan via OSV.dev (max depth 3, not scoped to
   changed files — dependency changes affect the entire project)
3. **SAST analysis**: Only on changed files (`ChangedOnly: true`,
   `IncludeTests: false`)
4. **Reachability**: If SCA findings exist and not disabled
5. **Risk scoring**: Computes risk score from findings, change metadata, and
   sensitivity (auth files, CI workflows, dependency changes)

## Risk Levels

| Score Range | Status | Meaning |
| --- | --- | --- |
| 0–24 | Ready for review | Minimal risk |
| 25–49 | Caution | Some risk, review findings |
| 50–74 | Warning | Significant risk, review required |
| 75–100 | BLOCKING | High risk, do not merge without review |

## Complete Flag Reference

| Flag | Type | Default | Description |
| --- | --- | --- | --- |
| `--base` | string | (auto-detected) | Base branch |
| `--head` | string | (current branch) | Head branch |
| `--format` | string | (terminal) | `markdown`, `json`, `sarif`, `pr-summary`, `annotations` |
| `--output` | string | (stdout) | Write report to file |
| `--no-sast` | bool | false | Skip SAST analysis |
| `--no-secrets` | bool | false | Skip secret detection |
| `--no-reachability` | bool | false | Skip reachability analysis |
| `--suggest-reviewers` | bool | false | Suggest reviewers (CODEOWNERS + git blame) |
| `--annotations` | bool | false | Generate inline annotations |
| `--suggest-fixes` | bool | false | Generate fix proposals |

## Backend Review Commands

For backend-connected review (submitting to the PatchFlow platform), use the
`review` command family:

```bash
# Show review context
patchflow review context

# Submit review to backend (requires authentication)
patchflow review pr --submit

# Check review job status
patchflow review status JOB_ID

# Watch review job status
patchflow review status JOB_ID --watch
```

These commands require authentication (`patchflow login`) and a backend
connection. See [Authentication](./authentication).

### Submitting PR Review Results Directly

You can also submit `pr-review` results directly to the PatchFlow backend
using the `--submit` flag:

```bash
patchflow pr-review --json --submit \
  --project-id 42 \
  --repository owner/repo \
  --pr-number 123 \
  --pr-title "Add login feature" \
  --pr-author alice \
  --pr-url https://github.com/owner/repo/pull/123
```

This posts the pr-review JSON to `POST /api/v1/cli/pr-review-results`.
Requires authentication (`patchflow login`) and a configured backend URL.

## Next Steps

- [Fixes](./fixes) — Apply fix proposals
- [Explain](./explain) — Investigate findings
- [Recommended Workflow](../workflows/recommended) — Team adoption
