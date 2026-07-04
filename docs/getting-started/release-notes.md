---
title: Changelog
description: All notable changes to PatchFlow CLI
---

# Changelog

All notable changes to PatchFlow CLI will be documented in this page.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## v0.1.5 - 2026-07-04

### Added
- **LICENSE file crawling**: When all registry API lookups fail or return
  NOASSERTION/Other/UNKNOWN, the CLI now fetches the raw LICENSE file directly
  from the source repository via `raw.githubusercontent.com`. Tries multiple
  file names (LICENSE, LICENSE.md, COPYING, UNLICENSE, NOTICE) and multiple
  branches (main, master, HEAD, develop, trunk).
- **Text-based license detection**: New `internal/licensedetect` package that
  identifies licenses from file content using n-gram similarity matching
  (Dice's coefficient). Supports 35+ license types (MIT, Apache-2.0, BSD,
  GPL, LGPL, AGPL, MPL, EPL, ISC, Unlicense, BSL, CC0, Zlib, and more).
- **PyPI repository URL extraction**: Extracts GitHub repo URL from PyPI
  `project_urls` (Homepage, Source, Repository) when license is missing.
- **RubyGems repository URL extraction**: Extracts GitHub repo URL from
  `source_code_uri`, `homepage_uri`, `repository_uri` when license is missing.
- **GitHub API LICENSE crawl fallback**: When the GitHub API returns
  NOASSERTION, falls back to fetching and analyzing the raw LICENSE file.

### Fixed
- **License coverage**: 89.6% to 95.6% (+6.0pp) across 10 real-world projects.
  Unknown licenses reduced by 55.8% (353 to 156). Total improvement from
  original: 78.2% to 95.6% (+17.4pp), unknowns reduced by 78.7%.

## v0.1.4 - 2026-07-04

### Added
- **Embedded OCI Image Scanner**: The image scanner is now embedded directly in the
  CLI as `internal/imagescan/` — no external Trivy dependency required. Native Go,
  OCI-first scanner with vulnerability matching (NVD, OSV, Alpine, Debian, Ubuntu).
- **New scan_image flags**: `--input` (SBOM/tarball input), `--platform`
  (multi-arch selection), `--no-vulns` (skip vulnerability scan), `--severities`
  (filter by severity), `--format` (json/sarif/cyclonedx), `--output` (write to file).
- **Go license detection**: `golang.org/x/*` (googlesource mirror mapping),
  `gopkg.in/*` (40+ repo mappings), `google.golang.org/*` (protobuf, grpc, etc.),
  Go module proxy VCS URL fallback.
- **Maven license detection**: POM file parsing from Maven Central, SCM URL
  extraction, version resolution via search API (3 query formats), 130+ groupId
  to GitHub repo mappings.
- **npm license detection**: Repository URL extraction from registry metadata,
  full package metadata fallback, homepage URL fallback.
- **Python license detection**: PyPI classifier-based fallback (40+ mappings),
  empty version handling, `pyproject.toml` inline table license parsing.
- **Ruby license detection**: `source_code_uri`/`homepage_uri`/`repository_uri`
  fallback for GitHub license lookup, version constraint cleanup.
- **SPDX mapping tables**: Expanded from ~25 to 300+ entries. Added AFL licenses,
  short GPL forms (GPL-2, GPL-3, GPLV2, etc.), compound license expressions.
- **PyPI classifier to SPDX map**: 40+ mappings (e.g., "License :: OSI Approved ::
  MIT License" to "MIT").

### Fixed
- **License coverage**: 78.2% to 89.6% (+11.4pp) across 10 real-world projects
  with 3,270+ dependencies. Unknown licenses reduced by 51.8%.
- **"Other"/"UNKNOWN"/"NOASSERTION" handling**: These placeholder license strings
  are now treated as uninformative and trigger GitHub fallback lookups.
- **LGPL misclassification**: Weak copyleft (LGPL) is now checked before strong
  copyleft (GPL) to fix substring matching (LGPL-2.1 was matching GPL-2).
- **Python pyproject.toml extras**: `test = ["pytest"]` in
  `[project.optional-dependencies]` no longer creates a bogus `test` dependency
  with version `["pytest"]`.
- **Ruby Gemfile parser**: Fixed parsing of package entries with commas and quotes.
- **PyPI empty versions**: `pytest@` and similar now use the no-version API endpoint.
- **RubyGems version constraints**: `~>`, `>=`, etc. are stripped before API lookup.
- **Image reference validation**: Added security checks for image references.

### Changed
- `internal/container/scanner.go` rewritten as adapter for the embedded scanner.
- `cmd/scan_image.go` updated to use the embedded scanner with new flags.
- `fetchGoModuleLicense` enhanced with googlesource, gopkg.in, and google.golang.org
  mappings.
- `fetchMavenLicense` now resolves unknown versions and extracts SCM URLs.
- `fetchNPMLicense` now extracts repository URLs and falls back to GitHub.
- `fetchRubyGemsLicense` now cleans version constraints and tries source URIs.
- `fetchPyPILicense` now handles empty versions and uses classifier fallback.

## v0.1.3 - 2026-07-03

### Fixed
- **Install script PATH handling**: `scripts/install.sh` now verifies the binary
  after installation and prints clear, copy-pasteable `export PATH` instructions
  when the install directory is not on `PATH`.
- **Install script CI/Docker support**: Added `--no-verify` flag and `NO_VERIFY=1`
  environment variable for unattended installations.

### Added
- **Containerized install test matrix**: New `tests/install/` with Dockerfiles for
  Ubuntu (amd64/arm64, root/non-root), Alpine, and optional Linuxbrew. Includes
  `run-tests.sh` that auto-detects Podman/Docker and validates the install script
  plus core CLI commands in every container.

### Changed
- **README installation guidance**: Now recommends the install script for Linux,
  Docker, and CI environments; notes that the Homebrew tap does not currently
  publish Linux bottles.

## v0.1.2 - 2026-07-01

### Added — B11.5: Extension Hardening
- **Sink scoping by CWE/category**: Custom sinks can be scoped to specific CWEs
  (e.g., `CWE-89`) or categories (e.g., `sql_injection`) to prevent cross-rule
  noise. Unscoped sinks still work (backward compatible) but `rules validate`
  warns about them.
- **Source category scoping**: Custom sources can be limited to specific
  vulnerability categories (e.g., `categories: [sql_injection, path_traversal]`).
- **Taint safe pattern suppression**: Safe patterns now suppress taint-mode
  findings when the pattern matches in the same function as the finding.
- **Config UX cleanup**: Unified `--config` flag for all scan commands.
  `--rules`, `--rules-config` are now legacy aliases for `--config`.
- **Strong validation**: `rules validate` now catches duplicate entries, unknown
  framework names, invalid CWE formats, and missing `schema_version`.

### Added — B12: CLI Release Hardening
- **Enhanced version command**: `patchflow version --json` now outputs 8 fields:
  version, commit, built_at, go_version, ruleset_version, schema_version,
  sarif_version, osv_db_version
- **Enhanced doctor command**: `patchflow doctor` now checks config file
  status, cache writability, and SARIF output writability. JSON output
  includes all fields with overall status.
- **CI templates**: `patchflow ci init {github,gitlab,circleci,azure,pre-commit}`
  generates CI/CD integration files with three profiles: audit (non-blocking),
  starter (high severity blocks), ci-blocking (high+critical block).
- **Config migration**: `patchflow config migrate` adds `schema_version` and
  suggests `framework_extensions` equivalents for `framework_overrides`.
- **Schema versioning**: `schema_version` field added to rules config.
  `rules validate` warns if missing.
- **Golden release smoke test**: 5 smoke fixtures (spring, graphql, express,
  clean-go, clean-python) with `TestReleaseSmoke` in `internal/testdata/`.
- **RC1 baseline**: `PATCHFLOW_CLI_RC1_BASELINE.md` documenting frozen state.

### Added — B13: Documentation
- **18 dedicated framework pages**: Each framework pack now has its own page
  with full rule tables, sources, sinks, sanitizers, safe patterns, and code
  examples.
- **Vision page**: "The Problem We Solve" — the fragmented security tooling
  problem and how PatchFlow solves it.
- **Custom framework extensions guide**: Detailed guide for extending official
  packs with organization-specific sources, sinks, sanitizers, and safe patterns.
- **CI templates guide**: How to use `patchflow ci init` to generate CI/CD
  integration files.
- **Version/doctor reference**: Full documentation for `patchflow version --json`
  and `patchflow doctor --json`.
- **Schema versioning reference**: Documentation for `schema_version` field.

### Added — v0.1.2 Base
- Non-blocking update check notification - PatchFlow now checks for new releases
  on startup and prints a banner if a newer version is available
- GitHub Actions workflow for self-hosted PatchFlow scans
  (`.github/workflows/patchflow-scan.yml`)
- CI workflow with comprehensive test gates
  (`.github/workflows/ci.yml`)
- Release workflow with GoReleaser, Cosign signing, and SBOM generation
  (`.github/workflows/release.yml`)
- Engineering standards document (`ENGINEERING_STANDARDS.md`)
- Architecture decision records (`ARCHITECTURE_DECISION_RECORDS.md`)
- Product principles document (`PATCHFLOW_PRODUCT_PRINCIPLES.md`)
- Embedded SAST roadmap (`EMBEDDED_SAST_ROADMAP.md`)
- Quickstart guide (`QUICKSTART.md`)
- Code of Conduct (`CODE_OF_CONDUCT.md`)
- NOTICE file for Apache 2.0 license compliance
- Makefile with common build, test, and release targets
- `.gitleaks.toml` for allowlist control
- `.pre-commit-hooks.yaml` for pre-commit integration

### Security
- Pinned all GitHub Actions to immutable SHA commits (prevents mutable-action
  supply-chain attacks)
- Removed `curl | sh` syft install in release workflow - replaced with pinned
  download script
- Fixed composite-action shell injection risk in `action.yml`
- Hardened Docker runtime: non-root user, read-only filesystem, minimal base
  image
- Token retrieval now uses OS keychain with 0600-permission file fallback
- Report file permissions set to 0600 (was 0644)
- Gitleaks behavior fixed: `.gitleaks.toml` added for allowlist control
- Secret rule IDs normalized for consistent SARIF reporting

### Changed
- GoReleaser pinned to v2.15.4 (avoids `brews` deprecation as failing config in
  newer versions)
- Go version updated to 1.26.4 in all workflows
- `scripts/install.sh` hardened with better error handling and checksum
  verification
- Container image validation improved in `internal/container/scanner.go`

### Fixed
- pnpm workspaces under `packages/` were skipped during manifest detection
- Golden tests expected stale rule IDs that no longer match the registry
- PR artifact tests wrote outside the validated project path
- Bare `return err` wrapped with context in critical paths (SAST runner, report
  generator, benchmark, OSV DB)
- Hardcoded `/usr/bin/time` path in benchmark now tries PATH first, then
  absolute fallback

## v0.1.1 - 2026-07-01

### Security
- Pinned all GitHub Actions to immutable SHA commits (prevents mutable-action
  supply-chain attacks)
- Removed `curl | sh` syft install in release workflow - replaced with pinned
  download script
- Fixed composite-action shell injection risk in `action.yml`
- Hardened Docker runtime: non-root user, read-only filesystem, minimal base
  image
- Token retrieval now uses OS keychain with 0600-permission file fallback
- Report file permissions set to 0600 (was 0644)
- Gitleaks behavior fixed: `.gitleaks.toml` added for allowlist control
- Secret rule IDs normalized for consistent SARIF reporting

### Changed
- GoReleaser pinned to v2.15.4 (avoids `brews` deprecation as failing config in
  newer versions)
- Go version updated to 1.26.4 in all workflows
- `scripts/install.sh` hardened with better error handling and checksum
  verification
- Container image validation improved in `internal/container/scanner.go`

### Fixed
- pnpm workspaces under `packages/` were skipped during manifest detection
- Golden tests expected stale rule IDs that no longer match the registry
- PR artifact tests wrote outside the validated project path
- Bare `return err` wrapped with context in critical paths (SAST runner, report
  generator, benchmark, OSV DB)
- Hardcoded `/usr/bin/time` path in benchmark now tries PATH first, then
  absolute fallback

## v0.1.0 - 2026-07-01

### Added
- GoReleaser config with multi-platform builds (linux/darwin/windows,
  amd64/arm64)
- Docker images with multi-arch manifests (buildx)
- Homebrew tap and Scoop bucket auto-publish
- Cosign image and blob signing
- SPDX SBOM generation via syft
- GitHub Action (`action.yml`) for composite security scan workflow
- Pre-commit hooks for scan and secret detection
- Install script (`scripts/install.sh`) with checksum verification
- Benchmark results: 100% recall on 18 vulnerable repos, 0.094 HC/KLOC on clean
  repos
- Embedded SAST engine: pattern scanner, taint analysis (tree-sitter), AST rules
- Framework pack system (Rails reference pack)
- Governance registry with maturity levels (experimental/beta/stable)
- Incremental scanning with mtime fast-path and git diff pre-filter
- SCA via OSV.dev with offline database support
- Secret detection with 40+ curated regex patterns
- Reachability analysis for Python, Go, JavaScript/TypeScript
- Risk scoring engine (0-100)
- Report generation: Markdown, JSON, SARIF
- SBOM generation: CycloneDX, SPDX, VEX
- Baseline management with stable semantic fingerprints
- PR review simulation with reviewer suggestions and annotations
- Fix proposal generation and application
- Suppression directives (`//patchflow:ignore`)
- Container image scanning via Trivy integration
- Configuration profiles for multi-org support
- Local OSV database for offline/air-gapped scanning
- 17 official framework packs: Rails, Express, Next.js, React, Spring, Spring
  Security, ASP.NET, Razor, Django, Laravel, FastAPI, Gin, Flask, Symfony,
  Angular, NestJS, Echo
- Multi-language pattern rules (Python, JavaScript, TypeScript, Ruby, PHP, Java,
  C#, Rust, Go, Dockerfile, YAML, Terraform)
- Tree-sitter AST rules for structural matching
- Go SSA-based taint analysis
- Tree-sitter source-to-sink taint patterns
- CWE and OWASP Top 10 mapping for all rules
- Governance profiles: dev, pr, ci, audit
- Shell completion scripts (bash, zsh, fish, powershell)
- `patchflow doctor` environment diagnostics
- `patchflow benchmark` real-repo benchmark suite

### Performance
- 29x Flask speedup via tree-sitter optimizations
- Incremental scanning with mtime fast-path and git diff pre-filter
- Single-pass parallel file collection (4x I/O reduction)
- `sync.Pool` per language for tree-sitter parser reuse

### Security
- Apache 2.0 license
- Non-root Docker container with read-only filesystem
- Cosign image and blob signing for release artifacts
- Checksum verification in install script
- OS keychain token storage with file fallback

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

### Build from Source

```bash
git clone https://github.com/Patchflow-security/patchflow-cli.git
cd patchflow-cli
git checkout v0.1.3
go build -o patchflow .
```

### Verify Your Version

```bash
patchflow version
```

Expected output:

```text
patchflow version 0.1.3 (commit: 85ca3f3, built: 2026-07-03T21:30:00Z)
```

## Next Steps

- [Installation](./installation) - Install or upgrade PatchFlow
- [Quickstart](./quickstart) - Get started in 5 minutes
- [GitHub Repository](https://github.com/Patchflow-security/patchflow-cli) - Source code and releases
