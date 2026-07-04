---
title: Overview
description: What PatchFlow CLI is, what it gives you, and how to start using it in 2 minutes
---

# Overview

PatchFlow CLI is a **single-binary security scanner** that runs on your machine
or in CI. You point it at a code repository and it tells you what's vulnerable —
in your dependencies, your code, your secrets, and your container images.

No backend. No cloud. No source code leaving your machine.

## What You Get

Run one command, get a complete security picture:

```bash
patchflow scan run
```

That single command gives you:

| What | Example | Why it matters |
| --- | --- | --- |
| **Vulnerable dependencies** | `lodash@4.17.20 → CVE-2021-23337 (high)` | Know which packages to upgrade and why |
| **Code vulnerabilities** | `SQL injection in app.py:42` | Catch injection, XSS, SSRF, and 800+ other patterns |
| **Hardcoded secrets** | `AWS key found in config/settings.py:10` | Prevent credential leaks before they ship |
| **Reachability** | `lodash is imported but the vulnerable function is not called` | Stop wasting time on unreachable CVEs |
| **Risk score** | `Risk: 34/100 (LOW)` | One number to track security posture over time |
| **Fix suggestions** | `Use parameterized queries: cursor.execute("SELECT ... WHERE name = ?", (name,))` | Know how to fix it, not just that it's broken |

## The 2-Minute Version

### 1. Install

```bash
# macOS / Linux
curl -fsSL https://github.com/Patchflow-security/patchflow-cli/raw/main/scripts/install.sh | bash

# Or with Homebrew
brew tap patchflow-security/patchflow
brew install patchflow
```

### 2. Scan

```bash
cd your-project
patchflow scan run
```

You'll see a terminal summary like this:

```text
PatchFlow Analysis Report
=========================

Repository:  /home/you/your-project
Branch:      main
Scan:        scan=20260101-abc  v0.1.5  profile=standard

Dependencies:  42
Findings:      7 (2 high, 3 medium, 2 low)

Risk Score: 48/100 (MODERATE)

Top findings:
  1. [HIGH]   CVE-2021-23337 — lodash@4.17.20
     Package: lodash → 4.17.21
  2. [HIGH]   SQL query with string formatting
     File:    src/db.py:18
  3. [MEDIUM] Hardcoded AWS access key
     File:    config/settings.py:10
```

### 3. Get a report

```bash
# JSON (for CI / programmatic use)
patchflow scan run --json --output report.json

# SARIF (for GitHub Code Scanning)
patchflow scan run --format sarif --output patchflow.sarif

# Markdown (for sharing)
patchflow scan run --format markdown --output report.md
```

### 4. Review before opening a PR

```bash
patchflow pr-review
```

Tells you the risk of your current branch changes *before* you open a pull
request — with reviewer suggestions and inline annotations.

That's it. You now have vulnerability scanning, secret detection, and PR risk
review running locally.

## What PatchFlow Replaces

If you're currently using a stack of tools, PatchFlow consolidates them:

| Instead of | PatchFlow does |
| --- | --- |
| Trivy (dependency scanning) | Embedded OSV.dev query with offline support |
| Semgrep (code scanning) | 800+ embedded rules across 24 scanners + 18 framework packs |
| gosec / bandit | Go AST + tree-sitter multi-language analysis |
| gitleaks (secret scanning) | 40+ curated secret patterns with false-positive filtering |
| Custom dedup scripts | Semantic fingerprinting + function-boundary grouping |
| SARIF aggregator | Native SARIF 2.1.0 output |

One tool. One output format. One severity scale. No glue scripts.

## Where to Go Next

<Columns>
  <Column>
    ### Get started

    - [Installation](./getting-started/installation) — All install methods
    - [Quickstart](./getting-started/quickstart) — Full 5-minute walkthrough
    - [Common Errors](./getting-started/common-errors) — Troubleshooting
  </Column>
  <Column>
    ### Understand

    - [Why PatchFlow](./getting-started/why-patchflow) — Detailed comparison
    - [Core Concepts](./getting-started/concepts) — SCA, SAST, reachability, risk
    - [Local vs Backend](./getting-started/local-vs-backend) — Privacy model
  </Column>
  <Column>
    ### Go deeper

    - [User Guides](./user-guides/scan) — Every command explained
    - [Framework Packs](./frameworks/index) — 18 framework-specific rule sets
    - [CI Integrations](./integrations/github-actions) — GitHub, GitLab, Jenkins
  </Column>
</Columns>
