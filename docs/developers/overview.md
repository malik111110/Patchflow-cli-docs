---
title: Developer Overview
description: Architecture and package map for PatchFlow CLI contributors
---

# Developer Overview

PatchFlow CLI is a Go application organized around scanner engines, rule
governance, and CLI commands. This page provides an architectural overview for
contributors.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      cmd/ (Cobra CLI)                        │
│   scan · deps · reachability · report · baseline · rules    │
│   explain · fix · suppress · pr-review · review · auth      │
│   config · cache · benchmark · init · doctor · version      │
├─────────────────────────────────────────────────────────────┤
│                    internal/sast/                            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │  gosast     │ │  secrets    │ │  patterns           │   │
│  │  (Go AST)   │ │  (regex)    │ │  (multi-lang regex) │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │  treesitter │ │  taint-ssa  │ │  taint-patterns     │   │
│  │  (AST)      │ │  (Go SSA)   │ │  (tree-sitter)      │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  frameworks/ (17 official packs)                     │   │
│  │  rails · express · django · spring · react · ...     │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  filecollector.go (single-pass parallel dispatch)   │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  internal/sca/    internal/reachability/   internal/risk/   │
│  (OSV.dev SCA)   (import graph)          (0-100 scoring)   │
├─────────────────────────────────────────────────────────────┤
│  internal/rules/     internal/baseline/   internal/report/  │
│  (governance)       (snapshots)         (md/json/sarif)    │
├─────────────────────────────────────────────────────────────┤
│  internal/fix/   internal/sbom/   internal/benchmark/       │
│  (fix engine)   (SBOM/VEX)       (real-repo benchmarks)    │
├─────────────────────────────────────────────────────────────┤
│  internal/auth/  internal/config/  internal/cacheutil/      │
│  (keychain)     (profiles)       (XDG cache)               │
└─────────────────────────────────────────────────────────────┘
```

## Main Packages

| Package | Purpose |
| --- | --- |
| `cmd/` | Cobra command definitions for all CLI commands |
| `internal/sast/` | SAST runner and embedded scanners |
| `internal/sast/gosast/` | Go AST-based rules (ported from gosec) |
| `internal/sast/secrets/` | Regex-based secret detection |
| `internal/sast/patterns/` | Multi-language regex patterns |
| `internal/sast/treesitter/` | Tree-sitter AST rules |
| `internal/sast/taint/` | Go SSA-based taint analysis |
| `internal/sast/taintpatterns/` | Tree-sitter source-to-sink taint engine |
| `internal/sast/frameworks/` | Framework pack model, matcher, overrides |
| `internal/sast/frameworks/<pack>/` | Individual framework packs |
| `internal/sast/frameworks/packs/` | Pack registration and registry |
| `internal/sast/filecollector.go` | Single-pass parallel file dispatch |
| `internal/frameworks/` | Framework detection signatures and detector |
| `internal/rules/` | Rule maturity, profiles, and governance registry |
| `internal/sca/` | Software Composition Analysis (OSV.dev) |
| `internal/reachability/` | Dependency reachability analysis |
| `internal/risk/` | Risk scoring engine (0–100) |
| `internal/baseline/` | Baseline creation and comparison |
| `internal/report/` | Report generation (Markdown, JSON, SARIF) |
| `internal/sbom/` | SBOM generation (CycloneDX, SPDX, VEX) |
| `internal/fix/` | Fix proposal generation and application |
| `internal/fixsnippet/` | Fix code snippets for explain output |
| `internal/benchmark/` | Real-repo benchmark runner and reports |
| `internal/auth/` | Token storage (keychain), OAuth device flow |
| `internal/config/` | Config loading and profile management |
| `internal/cacheutil/` | Cache directory resolution (XDG-compliant) |
| `internal/osvdb/` | Local OSV database management |
| `internal/git/` | Git repository detection and diff analysis |
| `internal/manifest/` | Dependency manifest parsing |
| `internal/osv/` | OSV.dev API client |
| `internal/registry/` | Package registry metadata (licenses) |
| `internal/container/` | Container image scanning (embedded OCI scanner; Trivy optional) |
| `internal/imagescan/` | Embedded OCI image scanner — native Go, NVD/OSV/Alpine/Debian/Ubuntu matching |
| `internal/pr/` | PR diff parsing and finding placement |
| `internal/reviewers/` | Reviewer suggestions (CODEOWNERS, git blame) |
| `internal/review/` | Review context collection |
| `internal/api/` | PatchFlow API client (review + scan-results + pr-review-results submission) |
| `internal/doctor/` | Environment diagnostics |
| `internal/templates/` | CI/CD template generation |
| `internal/analysis/` | Core types (Finding, Severity, ReachabilityStatus) |
| `internal/cwe/` | CWE metadata |
| `internal/exitcode/` | Exit code constants |
| `pkg/version/` | Version information |

## Build

```bash
# Build all packages
go build ./...

# Build the CLI binary
go build -o patchflow .
```

## Test

```bash
# Run framework foundation tests
go test ./internal/frameworks/ ./internal/sast/frameworks/ ./internal/sast/frameworks/rails/ ./internal/sast/frameworks/packs/ -v

# Run all SAST + rules tests
go test ./internal/sast/... ./internal/rules/...

# Run all tests
go test ./internal/... ./cmd/... -count=1

# Run specific test modules
go test ./internal/sast/patterns/ ./internal/sast/secrets/ -v
```

## Vet

```bash
go vet ./...
```

## Rule Governance

Rules move through maturity levels:

| Maturity | Description | Typical Visibility |
| --- | --- | --- |
| `experimental` | New rule, few tests, may have false positives | Audit profile only |
| `beta` | Has tests, enabled in audit/standard, non-blocking | CI and audit, non-blocking |
| `stable` | Has false-positive tests, CWE/OWASP mapping, can block | Developer, PR, CI, audit |
| `enterprise` | Large regression corpus, low false-positive rate | All profiles, blocking |

Framework rules are promoted only when fixtures and real-repo benchmark results
support the change. See [Rule Governance](../reference/rules-governance) for
details.

## Import Cycle Notes

- `internal/sast/frameworks` defines its own `Maturity` type (not
  `rules.Maturity`) to avoid an import cycle with `internal/rules` (which imports
  `internal/sast`).
- The `packs` subpackage bridges `frameworks.Maturity` → `rules.Maturity` via
  `ToRulesMaturity`.
- `rules.BuildRegistryFromRunner` was removed (it was unused and created a
  cycle). Use `packs.RegisterFrameworkRules(reg)` to add framework rules to a
  governance registry.

## Next Steps

- [Architecture](./architecture) — Detailed architecture deep-dive
- [Framework Rule Packs](./rule-packs) — Pack development guide
- [Adding a Framework Pack](./adding-packs) — Step-by-step pack creation
- [Developer CI/CD](./ci-cd) — CI for the CLI and docs
- [Benchmarks](./benchmarks) — Real-repo benchmark suite
