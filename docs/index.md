# PatchFlow CLI Documentation

PatchFlow CLI is a local-first security scanner for modern engineering teams. It
performs Software Composition Analysis (SCA), Static Application Security
Testing (SAST), secret detection, reachability analysis, framework-aware rule
packs, license scanning, container image scanning, and risk scoring — all from a
single binary, with no backend connection required for core analysis.

## What PatchFlow Does

PatchFlow runs entirely on your machine or in your CI pipeline. Source code
never leaves the local environment unless you explicitly use a backend-connected
review command.

| Capability | Description |
| --- | --- |
| **SCA** | Parses dependency manifests (Go, npm, PyPI, Maven, RubyGems, Packagist, Cargo, Composer) and queries the OSV.dev vulnerability database. Supports offline mode via a local OSV database. |
| **SAST** | Embedded scanners (Go AST, multi-language regex, tree-sitter AST, taint analysis) require zero installation. External tools (gosec, bandit, semgrep, gitleaks, checkov) supplement when installed. |
| **Secrets** | 40+ curated regex patterns for cloud provider keys, VCS tokens, SaaS tokens, private keys, database URLs, JWTs, and high-entropy strings, with false-positive filtering. |
| **Framework Packs** | Official embedded rule sets for 17 web frameworks: Rails, Express, Next.js, React, Spring, Spring Security, ASP.NET, Razor, Django, Laravel, FastAPI, Gin, Flask, Symfony, Angular, NestJS, and Echo. |
| **Reachability** | Determines whether vulnerable dependencies are actually imported and used in the codebase, with confidence levels (HIGH, MEDIUM, LOW, NONE, UNKNOWN). |
| **Risk Scoring** | Computes a 0–100 risk score from findings, change size, sensitivity (auth files, CI workflows, dependency changes), and reachability data. |
| **Reports** | Generates Markdown, JSON, SARIF 2.1.0, CycloneDX SBOM, SPDX SBOM, VEX, and dependency tree/graph exports. |
| **Baselines** | Snapshots known findings so CI can focus on new issues, using stable semantic fingerprints that survive line-number shifts. |
| **Fixes** | Generates and applies safe fix proposals for common vulnerability patterns (eval, SQL injection, command injection, weak crypto, hardcoded secrets, TLS, path traversal). |
| **Container Scanning** | Scans container images for OS package and language dependency vulnerabilities via Trivy integration. |
| **PR Review** | Simulates a pre-PR risk review locally, with reviewer suggestions, inline annotations, and fix proposals. |

## Documentation Sections

| Section | Audience | Content |
| --- | --- | --- |
| [Getting Started](./getting-started/installation.md) | All users | Installation, first scan, core concepts |
| [User Guides](./user-guides/scan.md) | Developers, security engineers | Detailed command usage for every feature |
| [Workflows](./workflows/recommended.md) | Teams | Recommended team adoption strategies |
| [Integrations](./integrations/github-actions.md) | CI/CD engineers | GitHub Actions, GitLab CI, SARIF, pre-commit, Jenkins, Azure DevOps |
| [Developers](./developers/overview.md) | Contributors | Architecture, framework pack development, CI/CD, benchmarks |
| [Reference](./reference/commands.md) | All users | Command reference, configuration, YAML policy, rule governance, scanner reference |

## Quick Start

```bash
# Install
go install github.com/Patchflow-security/patchflow-cli@latest

# Initialize in your repository
cd your-repo
patchflow init

# Run a full security analysis
patchflow scan run

# Generate a SARIF report for CI
patchflow report --format sarif --output patchflow.sarif
```

## Design Principles

- **Local-first**: All core analysis runs on your machine. No source code is
  sent to a backend unless you explicitly use a review submission command.
- **Zero-install scanners**: Embedded SAST scanners (Go AST, regex patterns,
  tree-sitter, taint analysis) require no external installation. External tools
  supplement when available.
- **Framework-aware**: Official rule packs understand framework-specific sources,
  sinks, sanitizers, and template engines — not just generic language patterns.
- **Governance-driven**: Rules carry maturity levels (experimental, beta, stable,
  enterprise) and map to CWE and OWASP. Governance profiles (dev, pr, ci, audit)
  control which rules are visible and blocking.
- **CI-ready**: SARIF output, exit gates, baselines, and incremental scanning
  make PatchFlow practical for pull request and CI pipelines.
- **Privacy-preserving**: Secret detection, SAST, and reachability all run
  locally. The only network calls are to the public OSV.dev API for vulnerability
  data (which can be replaced with a local database for offline mode).
