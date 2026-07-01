---
title: Changelog
description: All notable changes to PatchFlow CLI
---

# Changelog

All notable changes to PatchFlow CLI will be documented in this page.

## v0.1.2 - 2025-07-01

### Added
- Non-blocking update check notification
- GitHub Actions workflow for self-hosted PatchFlow scans
- CI workflow with comprehensive test gates
- Release workflow with GoReleaser, Cosign signing, and SBOM generation

### Security
- Pinned all GitHub Actions to immutable SHA commits
- Hardened Docker runtime: non-root user, read-only filesystem
- Token retrieval now uses OS keychain with file fallback
- Report file permissions set to 0600

### Fixed
- pnpm workspaces under packages/ were skipped during manifest detection
- Bare return err wrapped with context in critical paths

## v0.1.1 - 2025-07-01

### Security
- Pinned all GitHub Actions to immutable SHA commits
- Hardened Docker runtime: non-root user, read-only filesystem
- Token retrieval now uses OS keychain with file fallback

### Fixed
- pnpm workspaces under packages/ were skipped during manifest detection

## v0.1.0 - 2025-06-27

### Added
- Embedded SAST engine: pattern scanner, taint analysis, AST rules
- Framework pack system (Rails reference pack)
- Governance registry with maturity levels
- Incremental scanning with mtime fast-path
- SCA via OSV.dev with offline database support
- Secret detection with 40+ curated regex patterns
- Reachability analysis for Python, Go, JavaScript/TypeScript
- Risk scoring engine (0-100)
- Report generation: Markdown, JSON, SARIF
- SBOM generation: CycloneDX, SPDX, VEX
- Baseline management with stable semantic fingerprints
- PR review simulation with reviewer suggestions
- Fix proposal generation and application
- Container image scanning via Trivy integration
- 17 official framework packs

## How to Update

### Homebrew

```bash
brew upgrade patchflow
```

### Scoop

```bash
scoop update patchflow
```

### Go Install

```bash
go install github.com/Patchflow-security/patchflow-cli@latest
```

### Verify Your Version

```bash
patchflow version
```

## Next Steps

- [Installation](../getting-started/installation) - Install or upgrade PatchFlow
- [Quickstart](../getting-started/quickstart) - Get started in 5 minutes
