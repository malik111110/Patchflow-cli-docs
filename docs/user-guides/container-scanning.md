---
title: Container Scanning
description: Scan container images for OS package vulnerabilities and misconfigurations
---

# Container Scanning

The `patchflow scan image` command scans container images for OS package
vulnerabilities, language dependency vulnerabilities, and misconfigurations.

## Prerequisites

Container scanning uses the embedded OCI image scanner (`internal/imagescan/`).
No external tools are required — the scanner is built into the PatchFlow CLI
binary and performs vulnerability matching against NVD, OSV, Alpine, Debian,
and Ubuntu advisory databases.

> **Note**: Prior to v0.1.4, container scanning required [Trivy](https://trivy.dev)
> as an external dependency. The embedded scanner replaces this requirement.
> Trivy remains supported as an optional external analyzer when available on
> `PATH` (the CLI auto-detects it for supplementary coverage).

## Basic Usage

```bash
patchflow scan image nginx:1.21
```

Scans the specified image and prints a terminal summary with the top 20 findings
by severity.

## Output Formats

```bash
# JSON report
patchflow scan image myapp:latest --format json --output report.json

# Markdown report
patchflow scan image myapp:latest --format markdown --output report.md
```

## Severity Filtering

```bash
patchflow scan image alpine:3.18 --severities CRITICAL,HIGH
```

Filters findings to only the specified severities. Supported values
(comma-separated): `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `INFO`.

## Timeout

```bash
patchflow scan image large-image:latest --timeout 15m
```

Sets the scan timeout. Default is 10 minutes. Large images may require a longer
timeout.

## Complete Flag Reference

| Flag | Type | Default | Description |
| --- | --- | --- | --- |
| `--format` | string | (terminal) | `json`, `markdown` |
| `--output` | string | (stdout) | Output file path |
| `--timeout` | duration | 10m | Scan timeout |
| `--severities` | string | (all) | Comma-separated: `CRITICAL,HIGH,MEDIUM,LOW,INFO` |

The image name is passed as a positional argument:

```bash
patchflow scan image IMAGE_NAME [flags]
```

## What Container Scanning Detects

| Category | Description |
| --- | --- |
| OS package vulnerabilities | Vulnerabilities in packages installed via the OS package manager (apt, apk, yum, dnf) |
| Language dependencies | Vulnerabilities in application dependencies (npm, PyPI, Maven, RubyGems, etc.) |
| Misconfigurations | Configuration issues in IaC files, Dockerfiles, Kubernetes manifests |

## Using with Podman/Docker

PatchFlow scans images available in the local image store. Pull the image first:

```bash
# Pull with Podman
podman pull nginx:1.21

# Scan
patchflow scan image nginx:1.21

# Pull with Docker
docker pull myapp:latest

# Scan
patchflow scan image myapp:latest
```

## CI Integration

Scan images in CI before pushing to a registry:

```yaml
# GitHub Actions example
- name: Build image
  run: podman build -t myapp:${{ github.sha }} .

- name: Scan image
  run: patchflow scan image myapp:${{ github.sha }} --format json --output image-scan.json

- name: Upload scan results
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: image-scan
    path: image-scan.json
```

## Next Steps

- [Scan Your Project](./scan) — Source code scanning
- [Dependencies](./dependencies) — Dependency analysis
- [Reports](./reports) — Report generation
