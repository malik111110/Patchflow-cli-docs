---
title: Commands
description: Curated command map for all PatchFlow CLI commands
---

# Commands

This page provides a curated command map. For the full generated command help
with every flag, see [Generated Commands](./generated-commands.md).

## Command Tree

```
patchflow
├── version              Print CLI version
├── doctor               Check environment
├── init                 Initialize PatchFlow in a repository
│   ├── github-actions   Generate GitHub Actions workflow
│   ├── gitlab-ci        Generate GitLab CI job
│   ├── jenkins          Generate Jenkins pipeline stage
│   ├── azure-devops     Generate Azure DevOps pipeline
│   └── pre-commit       Generate pre-commit hook
├── scan                 Scan code for issues
│   ├── run              Full security analysis (SCA + SAST + reachability + risk)
│   ├── local            Scan local repository for manifests
│   ├── changed          Scan changed files
│   ├── export           Export results (JSON, SARIF, SBOM, dependency graph)
│   ├── image            Scan container image
│   └── baseline         Manage baselines (legacy)
├── deps                 Analyze dependencies
│   ├── list             List all dependencies
│   ├── vulnerable       List vulnerable dependencies
│   ├── diff             Show dependency changes vs base branch
│   ├── tree             Show dependency tree
│   └── licenses         Check dependency licenses
├── reachability         Analyze vulnerable dependency reachability
├── report               Generate a security report
├── baseline             Manage finding baselines
│   ├── create           Create a baseline
│   ├── list             List baselines
│   ├── diff             Diff against a baseline
│   └── delete           Delete a baseline
├── rules                Manage and inspect SAST rules
│   ├── list             List all registered rules
│   ├── list-frameworks  List all framework rule packs
│   ├── maturity         Show rule governance coverage report
│   ├── validate         Validate a custom rules YAML file
│   └── docs             Generate rule documentation
├── explain              Explain a finding or rule
├── fix                  Generate and apply fixes
│   ├── suggest          Generate fix proposals
│   ├── apply            Apply fix proposals
│   └── show             Show a fix proposal
├── suppress             Add a suppression directive
├── pr-review            Simulate a PR risk review
├── review               Review code changes
│   ├── context          Show review context
│   ├── pr               Review a pull request (--submit for backend)
│   ├── diff             Review a diff
│   └── status           Check review job status
├── auth                 Manage authentication
│   └── status           Show authentication status
├── login                Authenticate with PatchFlow
├── logout               Log out from PatchFlow
├── config               Manage configuration
│   ├── show             Show current configuration
│   ├── set              Set a configuration value
│   └── profile          Manage configuration profiles
│       ├── create       Create a profile
│       ├── use          Switch active profile
│       ├── list         List profiles
│       ├── show         Show profile details
│       └── delete       Delete a profile
├── cache                Manage local cache
│   ├── status           Show cache status
│   ├── clean            Clean cache
│   └── update           Download local OSV database
├── benchmark            Run scanner benchmarks
│   ├── run              Execute a benchmark suite
│   ├── compare          Compare benchmark results
│   └── report           Regenerate benchmark report
└── completion           Generate shell completion scripts
```

## Core Commands

| Command | Purpose |
| --- | --- |
| `patchflow version` | Print CLI version |
| `patchflow doctor` | Check Git and local environment |
| `patchflow init` | Initialize PatchFlow project files |
| `patchflow scan run` | Run the main security scan |
| `patchflow scan export` | Export scan artifacts (JSON, SARIF, SBOM) |
| `patchflow report` | Generate reports from latest scan data |
| `patchflow explain --rule <id>` | Explain a rule or finding |

## Rules

```bash
patchflow rules list
patchflow rules list --framework express
patchflow rules list-frameworks
patchflow rules maturity
patchflow rules validate
patchflow rules docs
```

## Framework Packs

```bash
patchflow scan run --framework auto
patchflow scan run --framework express --framework react
patchflow scan run --disable-framework spring-security
```

## Baselines

```bash
patchflow baseline create --name v1.0
patchflow baseline list
patchflow baseline diff --from v1.0
patchflow baseline delete --name v1.0
patchflow scan run --new-only --baseline v1.0
```

## Dependencies

```bash
patchflow deps list
patchflow deps vulnerable
patchflow deps diff
patchflow deps tree
patchflow deps licenses
```

## Reachability

```bash
patchflow reachability --package lodash
patchflow reachability --cve CVE-2021-23337
patchflow reachability --package lodash --explain
```

## Fixes

```bash
patchflow fix suggest
patchflow fix suggest --severity high --auto-only
patchflow fix apply --all --auto-only --yes
patchflow fix show FINDING_ID
```

## PR Review

```bash
patchflow pr-review
patchflow pr-review --suggest-reviewers --suggest-fixes
patchflow pr-review --format sarif --output pr-review.sarif
```

## Review (Backend-Connected)

```bash
patchflow review context
patchflow review pr --submit
patchflow review status JOB_ID
patchflow review status JOB_ID --watch
```

## Authentication

```bash
patchflow login --token "$PATCHFLOW_TOKEN"
patchflow login --device --client-id <id>
patchflow auth status
patchflow logout
```

## Configuration

```bash
patchflow config show
patchflow config set api_url https://api.patchflow.dev
patchflow config profile create production --org my-org
patchflow config profile use production
patchflow config profile list
```

## Cache

```bash
patchflow cache status
patchflow cache clean --force
patchflow cache update
patchflow cache update --ecosystems pypi,npm,go
```

## Benchmark

```bash
patchflow benchmark run benchmarks/framework-pack-validation.yaml
patchflow benchmark run --no-tools --timeout 30m
patchflow benchmark compare results/
patchflow benchmark report results/2026-01/
```

## Init Templates

```bash
patchflow init github-actions
patchflow init gitlab-ci
patchflow init jenkins
patchflow init azure-devops
patchflow init pre-commit
```

## Shell Completion

```bash
patchflow completion bash
patchflow completion zsh
patchflow completion fish
patchflow completion powershell
```

## Next Steps

- [Generated Commands](./generated-commands.md) — Full command help output
- [Global Flags](./global-flags.md) — Global CLI flags
- [Configuration](./configuration.md) — Config file reference
