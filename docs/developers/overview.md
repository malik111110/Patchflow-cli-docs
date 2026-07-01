# Developer Overview

PatchFlow CLI is a Go application organized around scanner engines, rule
governance, and CLI commands.

## Main Areas

| Area | Purpose |
| --- | --- |
| `cmd/` | Cobra command definitions |
| `internal/sast/` | SAST runner and embedded scanners |
| `internal/sast/frameworks/` | Framework pack model, matcher, overrides |
| `internal/frameworks/` | Framework detection signatures |
| `internal/rules/` | Rule maturity, profiles, and governance registry |
| `internal/benchmark/` | Real-repo benchmark runner and reports |
| `internal/baseline/` | Baseline creation and comparison |
| `internal/reachability/` | Dependency reachability analysis |

## Build

```bash
go build ./...
go build -o patchflow .
```

## Test

```bash
go test ./internal/benchmark ./internal/sast/frameworks/... ./internal/rules -count=1
go test ./internal/sast/... ./internal/config/... ./cmd/... -count=1
```

## Documentation Workflow

When command behavior changes:

```bash
cd /Users/digitalcenter/patchflow-cli
go build -o patchflow .
cd /Users/digitalcenter/patchflow-cli-docs
PATCHFLOW_BIN=/Users/digitalcenter/patchflow-cli/patchflow npm run generate:commands
```

When source docs change:

```bash
PATCHFLOW_CLI_REPO=/Users/digitalcenter/patchflow-cli npm run sync:cli
```

## Rule Governance

Rules move through maturity levels:

| Maturity | Typical Visibility |
| --- | --- |
| `experimental` | Audit only |
| `beta` | CI and audit, non-blocking unless explicitly configured |
| `stable` | Developer, PR, CI, audit |
| `enterprise` | Large regression corpus and low-noise default behavior |

Framework rules are promoted only when fixtures and real-repo benchmark results
support the change.
