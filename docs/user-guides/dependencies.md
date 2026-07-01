---
title: Dependencies
description: Analyze project dependencies with the deps command
---

# Dependencies

The `patchflow deps` command analyzes project dependencies: list them, find
vulnerable ones, show changes against a base branch, display the dependency
tree, and check licenses.

## List All Dependencies

```bash
patchflow deps list
```

Lists all dependencies with name, version, ecosystem, direct/transitive status,
and manifest path. Dependencies are grouped by ecosystem (Go, npm, PyPI, Maven,
RubyGems, Packagist, Cargo).

Expected output (abbreviated):

```text
Dependencies (247 total)
========================

npm (142)
  Name                    Version    Type      Manifest
  express                 4.18.2     direct    package.json
  lodash                  4.17.20    direct    package.json
  minimist                1.2.5      transitive package-lock.json
  ...

PyPI (78)
  Name                    Version    Type      Manifest
  requests                2.25.0     direct    requirements.txt
  flask                   2.0.1      direct    requirements.txt
  ...

Go (27)
  Name                    Version    Type      Manifest
  github.com/gin-gonic/gin v1.7.0    direct    go.mod
  ...
```

Use `--json` for machine-readable output:

```bash
patchflow deps list --json
```

## Find Vulnerable Dependencies

```bash
patchflow deps vulnerable
```

Queries OSV.dev for each dependency and lists vulnerabilities with:

- Dependency name and version
- Vulnerability ID (CVE or OSV ID)
- Severity
- Affected version range
- Fixed version

Expected output (abbreviated):

```text
Vulnerable Dependencies (12 found)
===================================

1. lodash@4.17.20 (npm)
   CVE-2021-23337    HIGH      Command Injection
   Affected: <4.17.21
   Fixed:    4.17.21

2. minimist@1.2.5 (npm)
   CVE-2021-44906    CRITICAL  Prototype Pollution
   Affected: <1.2.6
   Fixed:    1.2.6

3. requests@2.25.0 (PyPI)
   CVE-2021-33503    HIGH      Denial of Service
   Affected: <2.26.0
   Fixed:    2.26.0
   ...
```

Use `--json` for machine-readable output:

```bash
patchflow deps vulnerable --json
```

## Show Dependency Changes

```bash
patchflow deps diff
```

Shows dependency changes against the base branch by identifying changed manifest
files via git diff. Useful for PR review to see what dependencies were added,
removed, or updated.

## Dependency Tree

```bash
patchflow deps tree
```

Shows the dependency tree grouped by ecosystem and manifest. Direct dependencies
are marked with `*`. The tree shows the hierarchy of dependencies within each
ecosystem.

Use `--json` for machine-readable output:

```bash
patchflow deps tree --json
```

## Check Licenses

```bash
patchflow deps licenses
```

Extracts and classifies license information from project dependencies. Licenses
are fetched from package registries (npm, PyPI, Maven, RubyGems, Packagist) and
categorized:

| Category | Risk Level | Examples |
| --- | --- | --- |
| Permissive | Low | MIT, Apache-2.0, BSD-3-Clause, ISC |
| Weak copyleft | Medium | LGPL-2.1, MPL-2.0, EPL-2.0 |
| Copyleft | High | GPL-2.0, GPL-3.0, AGPL-3.0 |
| Proprietary | High | Commercial licenses |
| Unknown | Critical | License could not be determined |

The output shows a summary by category and risk level, with high and critical
risk licenses highlighted.

Use `--json` for machine-readable output:

```bash
patchflow deps licenses --json
```

## Supported Ecosystems

PatchFlow parses dependency manifests from the following ecosystems:

| Ecosystem | Manifest Files |
| --- | --- |
| Go | `go.mod`, `go.sum` |
| npm | `package.json`, `package-lock.json` |
| PyPI | `requirements.txt`, `pyproject.toml`, `Pipfile`, `poetry.lock`, `setup.py` |
| Maven | `pom.xml` |
| RubyGems | `Gemfile`, `Gemfile.lock` |
| Packagist | `composer.json`, `composer.lock` |
| Cargo | `Cargo.toml`, `Cargo.lock` |

## Next Steps

- [Reachability](./reachability) — Check if vulnerable dependencies are used
- [Scan Your Project](./scan) — Full security analysis
- [Container Scanning](./container-scanning) — Scan container images
