---
title: PatchFlow CLI Documentation
description: Local-first security scanner for modern engineering teams
---

# PatchFlow CLI Documentation

<Badge type="version" text="v0.1.3" /> <Badge type="color" text="Latest" color="green" />

PatchFlow CLI is a **single-binary security scanner** that finds vulnerable
dependencies, code flaws, hardcoded secrets, and container image issues — all
locally, no backend required.

## Start Here

If you're new to PatchFlow, read the [Overview](./getting-started/overview) first.
It covers what you get, a 2-minute quickstart, and what PatchFlow replaces.

<Columns>
  <Column>
    ### I want to try it

    [Overview →](./getting-started/overview) — What it is, 2-minute start

    [Quickstart →](./getting-started/quickstart) — Full walkthrough

    [Install →](./getting-started/installation) — All install methods
  </Column>
  <Column>
    ### I want to understand it

    [Why PatchFlow →](./getting-started/why-patchflow) — vs Trivy + Semgrep + Gitleaks

    [Core Concepts →](./getting-started/concepts) — SCA, SAST, reachability

    [Privacy Model →](./getting-started/local-vs-backend) — What data leaves your machine
  </Column>
  <Column>
    ### I want to use it in CI

    [GitHub Actions →](./integrations/github-actions)

    [GitLab CI →](./integrations/gitlab)

    [SARIF Uploads →](./integrations/sarif)
  </Column>
</Columns>

## Get Running Now

<Tabs>
  <Tab title="macOS / Linux">
    ```bash
    curl -fsSL https://github.com/Patchflow-security/patchflow-cli/raw/main/scripts/install.sh | bash
    cd your-repo
    patchflow scan run
    ```
  </Tab>
  <Tab title="Homebrew">
    ```bash
    brew tap patchflow-security/patchflow
    brew install patchflow
    cd your-repo
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
    ./patchflow scan run
    ```
  </Tab>
</Tabs>

## What PatchFlow Scans

| What | How | Example finding |
| --- | --- | --- |
| **Dependencies (SCA)** | Parses manifests, queries OSV.dev | `lodash@4.17.20 has CVE-2021-23337 (high)` |
| **Code (SAST)** | 800+ embedded rules + 18 framework packs | `SQL injection in src/db.py:18` |
| **Secrets** | 40+ curated patterns | `AWS key in config/settings.py:10` |
| **Container images** | Embedded OCI scanner | `openssl 1.1.1k has CVE-2021-3711` |
| **Licenses** | Registry metadata + classification | `GPL-3.0 found in dependency tree` |

All scanners are embedded — zero external installs required. External tools
(Semgrep, gosec, bandit, gitleaks, trivy) are auto-detected and supplement
the embedded scanners when available.

## Documentation Index

| Section | What's in it |
| --- | --- |
| [Getting Started](./getting-started/overview) | Overview, installation, quickstart, concepts, why, vision, errors |
| [Framework Packs](./frameworks/index) | 18 framework pages with full rule tables and code examples |
| [User Guides](./user-guides/scan) | Detailed command usage for every feature |
| [Workflows](./workflows/recommended) | Recommended workflows for solo devs and teams |
| [Integrations](./integrations/github-actions) | GitHub Actions, GitLab, SARIF, pre-commit, Jenkins, Azure DevOps |
| [Developers](./developers/overview) | Architecture, framework pack development, benchmarks |
| [Reference](./reference/commands) | Command reference, configuration, scanner reference |
