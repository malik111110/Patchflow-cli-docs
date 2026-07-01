---
title: Local vs Backend: What Data Leaves Your Machine
description: What data PatchFlow sends where, for trust and compliance verification
---

# Local vs Backend: What Data Leaves Your Machine

PatchFlow CLI is designed local-first. For the vast majority of workflows, no
source code, no scan results, and no telemetry leave your machine. This page
documents exactly what is sent where, so you can verify the trust model before
adopting PatchFlow in regulated or air-gapped environments.

## Summary

| Activity | Data Sent | Destination | Required? |
| --- | --- | --- | --- |
| SCA vulnerability lookup | Dependency names and versions | `api.osv.dev` (OSV.dev) | Yes (unless offline mode) |
| License metadata fetch | Package names and versions | Package registries (npm, PyPI, Maven, RubyGems, Packagist, Cargo) | Yes (unless `--no-licenses`) |
| Local SAST scan | Nothing | Nothing | N/A — runs entirely locally |
| Secret detection | Nothing | Nothing | N/A — runs entirely locally |
| Reachability analysis | Nothing | Nothing | N/A — runs entirely locally |
| Report generation | Nothing | Nothing | N/A — runs entirely locally |
| Baseline management | Nothing | Nothing | N/A — runs entirely locally |
| Fix generation | Nothing | Nothing | N/A — runs entirely locally |
| PR review (local) | Nothing | Nothing | N/A — runs entirely locally |
| Backend review submission | Git context, diff, findings | PatchFlow API | No — only with `review pr --submit` |
| SARIF upload to GitHub | SARIF file | GitHub API | No — only with `scan export --upload-github` |

**The key takeaway**: source code never leaves your machine. PatchFlow reads
files locally, runs all scanners locally, and generates all reports locally. The
only external network calls during a standard scan are OSV.dev vulnerability
lookups (dependency names + versions) and optional license metadata fetches.

## What Is Sent During a Standard Scan

### OSV.dev Vulnerability Lookups

When PatchFlow scans dependencies (SCA), it sends dependency names and versions
to `api.osv.dev` to check for known vulnerabilities.

**What is sent:**

```json
{
  "queries": [
    {"package": {"name": "lodash", "ecosystem": "npm"}, "version": "4.17.20"},
    {"package": {"name": "requests", "ecosystem": "PyPI"}, "version": "2.25.0"}
  ]
}
```

**What is NOT sent:**

- Source code
- File contents
- File paths
- Repository URL or name
- Author or organization information
- Scan results or findings

OSV.dev is a free, open-source vulnerability database operated by Google. See
[osv.dev](https://osv.dev) for their privacy policy.

### License Metadata Fetches

When license scanning is enabled (default), PatchFlow fetches license
information from package registries:

- **npm**: `registry.npmjs.org`
- **PyPI**: `pypi.org`
- **Maven**: `repo1.maven.org`
- **RubyGems**: `rubygems.org`
- **Packagist**: `packagist.org`
- **Cargo**: `crates.io`

**What is sent:** Package names and versions (e.g., `GET /lodash/4.17.20`)

**What is NOT sent:** Source code, file contents, repository information

Disable license scanning with `--no-licenses`:

```bash
patchflow scan run --no-licenses
```

## What Never Leaves Your Machine

### Source Code

PatchFlow reads source files locally for SAST, secret detection, and
reachability analysis. No source code is transmitted to any external service.

### Scan Results and Findings

All findings are generated locally and stored in `.patchflow/reports/` or
written to the path you specify with `--output`. Findings are not transmitted
unless you explicitly upload them (SARIF to GitHub) or submit them (backend
review).

### Baselines

Baselines are stored locally in `.patchflow/baselines/`. They never leave your
machine.

### Configuration and Tokens

Configuration is stored in `~/.patchflow/config.yaml`. Tokens are stored in the
OS keychain (macOS Keychain, Linux Secret Service, Windows Credential Manager).
Neither is transmitted to any service during local analysis.

## Offline Mode (Air-Gapped Scanning)

For air-gapped or regulated environments, PatchFlow supports fully offline
scanning with zero network calls:

```bash
# Download the OSV database (once, on a connected machine)
patchflow cache update

# Transfer the database to the air-gapped machine:
# ~/.patchflow/osv-db/

# Run scans with no network calls
patchflow scan run --offline
```

Offline mode:

- Skips all API calls to OSV.dev
- Uses only the local OSV database
- Skips license metadata fetches
- Enables air-gapped scanning in regulated environments

See [Cache Management](../user-guides/cache.md) for details.

## Backend-Connected Features (Optional)

The following features require authentication and send data to the PatchFlow
API. They are entirely optional — all local analysis works without
authentication.

### `review pr --submit`

Submits review context to the PatchFlow backend for AI-assisted review.

**What is sent:**

- Git context (repository URL, branch names, commit SHAs)
- Changed file paths and diff content
- PatchFlow findings (SAST, SCA, secrets)
- Reachability data

**What is NOT sent:**

- Unchanged source files
- Baselines
- Configuration or tokens

### `review status`

Checks the status of a submitted review job. Sends only the job ID.

### `scan export --upload-github`

Uploads a SARIF file to GitHub Code Scanning.

**What is sent:** SARIF file (findings, rule metadata, locations)

**Destination:** GitHub API (`api.github.com`)

**Required:** `GITHUB_TOKEN` environment variable

## Disabling All Network Access

To run PatchFlow with zero network calls:

```bash
# Skip OSV.dev lookups (use local database)
patchflow scan run --offline

# Skip license scanning
patchflow scan run --no-licenses

# Combined
patchflow scan run --offline --no-licenses
```

## Verifying Network Behavior

You can verify PatchFlow's network behavior using standard tools:

```bash
# Monitor network calls during a scan (macOS/Linux)
sudo tcpdump -i any dst port 443 -w patchflow-traffic.pcap &
patchflow scan run
sudo kill %1
# Analyze: tcpdump -r patchflow-traffic.pcap -n

# Or use mitmproxy for detailed inspection
mitmproxy --mode regular &
HTTPS_PROXY=http://localhost:8080 patchflow scan run
```

You will see connections to `api.osv.dev` and optionally to package registries.
You will not see any connections transmitting source code or scan results.

## Data Retention

| Data | Stored Where | Retention |
| --- | --- | --- |
| Scan results | `.patchflow/reports/` (local) | Until you delete them |
| Baselines | `.patchflow/baselines/` (local) | Until you delete them |
| OSV cache | `~/.cache/patchflow/` (local) | Until `patchflow cache clean` |
| SAST state | `~/.cache/patchflow/` (local) | Until `patchflow cache clean` |
| Config | `~/.patchflow/config.yaml` (local) | Until you delete it |
| Token | OS keychain (local) | Until `patchflow logout` |
| Backend review | PatchFlow API (if submitted) | Per backend retention policy |

PatchFlow does not collect telemetry, analytics, or usage data. No information
about your scans, findings, or repository is sent to PatchFlow unless you
explicitly use backend-connected features.

## Next Steps

- [Quickstart](./quickstart.md) — Get started with local scanning
- [Cache Management](../user-guides/cache.md) — Offline mode setup
- [Authentication](../user-guides/authentication.md) — When authentication is needed
