---
title: CI Init Command
description: Generate CI/CD integration templates with patchflow ci init
---

# patchflow ci init

Generate CI/CD integration templates for integrating PatchFlow into your
pipeline.

## Syntax

```bash
patchflow ci init [platform] [flags]
```

## Platforms

| Platform | Output File |
| --- | --- |
| `github` | `.github/workflows/patchflow.yml` |
| `gitlab` | `.gitlab-ci-patchflow.yml` |
| `circleci` | `.circleci/patchflow-config.yml` |
| `azure` | `azure-patchflow.yml` |
| `pre-commit` | `.pre-commit-config.yaml` |

## Flags

| Flag | Default | Description |
| --- | --- | --- |
| `--profile` | `audit` | CI profile: `audit`, `starter`, `ci-blocking` |
| `--output` | platform-specific | Output file path |

## Profiles

| Profile | Blocking behavior | Recommended for |
| --- | --- | --- |
| `audit` | Non-blocking (`continue-on-error: true`) | First-time adoption |
| `starter` | High severity blocks | Teams that have triaged findings |
| `ci-blocking` | High and critical block | Mature teams enforcing security gates |

## Examples

```bash
# GitHub Actions, audit profile (default)
patchflow ci init github

# GitLab CI, blocking profile
patchflow ci init gitlab --profile ci-blocking

# GitHub Actions, custom output path
patchflow ci init github --output .github/workflows/security.yml

# pre-commit hook
patchflow ci init pre-commit
```

## Generated Content

Each generated file includes:

1. **Install step**: Downloads and installs PatchFlow from GitHub releases
2. **Scan step**: Runs `patchflow scan run --config .patchflow/rules.yaml`
3. **SARIF output**: Generates SARIF for GitHub Code Scanning (where supported)
4. **Artifact upload**: Publishes scan results as CI artifacts
5. **Non-blocking by default**: `continue-on-error: true` in audit profile

## See Also

- [CI Templates Guide](../user-guides/ci-templates)
- [GitHub Actions Integration](../integrations/github-actions)
- [GitLab CI Integration](../integrations/gitlab)
