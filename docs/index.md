---
title: PatchFlow CLI Documentation
description: Local-first security scanner for modern engineering teams
---

# PatchFlow CLI Documentation

<Badge type="version" text="v0.1.3" /> <Badge type="color" text="Latest" color="green" />

PatchFlow CLI is a local-first security scanner for modern engineering teams. It
performs Software Composition Analysis (SCA), Static Application Security
Testing (SAST), secret detection, reachability analysis, framework-aware rule
packs, license scanning, container image scanning, and risk scoring — all from a
single binary, with no backend connection required for core analysis.

> **New in v0.1.3:** Install script now verifies the binary after installation
> and prints clearer PATH instructions. Added containerized install test matrix
> for Ubuntu, Alpine, and non-root environments. [View changelog →](./getting-started/release-notes)
> [View changelog →](./getting-started/release-notes)

## What PatchFlow Does

PatchFlow runs entirely on your machine or in your CI pipeline. Source code
never leaves the local environment unless you explicitly use a backend-connected
review command.

| Capability | Description |
| --- | --- |
| **SCA** | Parses dependency manifests (Go, npm, PyPI, Maven, RubyGems, Packagist, Cargo, Composer) and queries the OSV.dev vulnerability database. Supports offline mode via a local OSV database. |
| **SAST** | Embedded scanners (Go AST, multi-language regex, tree-sitter AST, taint analysis) require zero installation. External tools (gosec, bandit, semgrep, gitleaks, checkov) supplement when installed. |
| **Secrets** | 40+ curated regex patterns for cloud provider keys, VCS tokens, SaaS tokens, private keys, database URLs, JWTs, and high-entropy strings, with false-positive filtering. |
| **Framework Packs** | Official embedded rule sets for 18 web frameworks: Rails, Express, Next.js, React, Spring, Spring Security, ASP.NET, Razor, Django, Laravel, FastAPI, Gin, Flask, Symfony, Angular, NestJS, GraphQL, and Echo. Each pack has a dedicated page with full rule tables, sources, sinks, sanitizers, and code examples. |
| **Reachability** | Determines whether vulnerable dependencies are actually imported and used in the codebase, with confidence levels (HIGH, MEDIUM, LOW, NONE, UNKNOWN). |
| **Risk Scoring** | Computes a 0–100 risk score from findings, change size, sensitivity (auth files, CI workflows, dependency changes), and reachability data. |
| **Reports** | Generates Markdown, JSON, SARIF 2.1.0, CycloneDX SBOM, SPDX SBOM, VEX, and dependency tree/graph exports. |
| **Baselines** | Snapshots known findings so CI can focus on new issues, using stable semantic fingerprints that survive line-number shifts. |
| **Fixes** | Generates and applies safe fix proposals for common vulnerability patterns (eval, SQL injection, command injection, weak crypto, hardcoded secrets, TLS, path traversal). |
| **Container Scanning** | Scans container images for OS package and language dependency vulnerabilities via the embedded OCI image scanner (no external tools required). |
| **PR Review** | Simulates a pre-PR risk review locally, with reviewer suggestions, inline annotations, and fix proposals. |

## Documentation Sections

| Section | Audience | Content |
| --- | --- | --- |
| [Getting Started](./getting-started/installation) | All users | Installation, quickstart, core concepts, why PatchFlow, vision, trust model, common errors |
| [Framework Packs](./frameworks/index) | All users | 18 dedicated framework pages with full rule tables, sources, sinks, sanitizers, and code examples |
| [User Guides](./user-guides/scan) | Developers, security engineers | Detailed command usage for every feature |
| [Workflows](./workflows/recommended) | Solo devs and teams | Recommended workflows for solo developers and teams, CI adoption |
| [Integrations](./integrations/github-actions) | CI/CD engineers | GitHub Actions, GitLab CI, SARIF, pre-commit, Jenkins, Azure DevOps |
| [Developers](./developers/overview) | Contributors | Architecture, framework pack development, CI/CD, benchmarks |
| [Reference](./reference/commands) | All users | Command reference, configuration, YAML policy, rule governance, scanner reference |

## New to PatchFlow?

Start here:

1. [Why PatchFlow](./getting-started/why-patchflow) — Why use PatchFlow instead of stacking Trivy + Semgrep + Gitleaks
2. [The Problem We Solve](./getting-started/vision) — The fragmented security tooling problem and how PatchFlow solves it
3. [Quickstart](./getting-started/quickstart) — Get running in under 5 minutes
4. [Framework Packs](./frameworks/index) — Browse all 18 framework packs with dedicated pages
5. [Custom Framework Extensions](./user-guides/custom-framework-extensions) — Extend packs with your own sources, sinks, sanitizers
6. [Local vs Backend](./getting-started/local-vs-backend) — What data leaves your machine (trust model)
7. [Common Errors](./getting-started/common-errors) — Troubleshooting guide

## Get Started

<Tabs>
  <Tab title="macOS / Linux">
    ```bash
    curl -fsSL https://github.com/Patchflow-security/patchflow-cli/raw/main/scripts/install.sh | bash
    cd your-repo
    patchflow init
    patchflow scan run
    ```
  </Tab>
  <Tab title="Windows">
    ```powershell
    scoop bucket add patchflow https://github.com/Patchflow-security/scoop-bucket
    scoop install patchflow
    cd your-repo
    patchflow init
    patchflow scan run
    ```
  </Tab>
  <Tab title="Docker / Podman">
    ```bash
    podman pull ghcr.io/patchflow-security/cli:latest
    podman run --rm -v "$PWD:/repo" ghcr.io/patchflow-security/cli:latest scan run --path /repo
    ```
  </Tab>
  <Tab title="Build from Source">
    ```bash
    git clone https://github.com/Patchflow-security/patchflow-cli.git
    cd patchflow-cli
    go build -o patchflow .
    cd your-repo
    ./patchflow init
    ./patchflow scan run
    ```
  </Tab>
</Tabs>

See [Installation](./getting-started/installation) for all methods, including Homebrew, WinGet, and CI setup.

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
