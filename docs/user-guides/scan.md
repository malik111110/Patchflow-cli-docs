# Scan Your Project

`patchflow scan run` is the primary local security command. It runs a full
analysis pipeline: SCA, license scanning, SAST, secrets, framework packs,
reachability, and risk scoring.

## Standard Scan

```bash
patchflow scan run
```

Runs all analysis stages with the `standard` profile and `ci` governance profile.
Output is a terminal summary with findings grouped by severity and scanner.

## Scan Profiles

Profiles control scan depth and which scanners run:

| Profile | Max Depth | Timeout | Scanners | Governance | Use Case |
| --- | --- | --- | --- | --- | --- |
| `quick` | 1 | 60s | Embedded + gosec/gitleaks only | `dev` | Developer feedback loop |
| `standard` | 3 | 120s | All embedded + all external | `ci` | Pull request and default CI |
| `deep` | 5 | 10min | All + Maven transitive resolution | `audit` | Security audit and scheduled review |

```bash
patchflow scan run --profile quick
patchflow scan run --profile standard
patchflow scan run --profile deep
```

## Governance Profiles

Governance profiles filter findings by rule maturity:

| Profile | Rules Included | Blocking | Use Case |
| --- | --- | --- | --- |
| `dev` | Stable only | No | Local development |
| `pr` | Stable | Stable+ can block | Pull request review |
| `ci` | Stable + beta | Stable+ can block | CI pipelines |
| `audit` | All (including experimental) | Configurable | Security audits |

```bash
patchflow scan run --governance-profile audit
```

When not specified, the governance profile defaults based on the scan profile:
`quick` → `dev`, `standard` → `ci`, `deep` → `audit`.

## Output Formats

### Terminal Summary (default)

```bash
patchflow scan run
```

Prints a human-readable summary to stdout with findings grouped by severity,
scanner, and type.

### JSON

```bash
patchflow scan run --format json --output patchflow-report.json
```

Structured JSON with all findings, dependencies, risk score, and metadata. Use
for CI parsing, dashboards, and automation.

### Markdown

```bash
patchflow scan run --format markdown --output patchflow-report.md
```

Human-readable report with tables and sections. Use for PR comments and audit
notes.

### SARIF

```bash
patchflow scan run --format sarif --output patchflow.sarif
```

SARIF 2.1.0 format for GitHub code scanning and other security dashboards. See
[SARIF Uploads](../integrations/sarif.md).

## Exit Gates

Use `--fail-on` to make CI fail when findings meet or exceed a severity
threshold:

```bash
patchflow scan run --fail-on high
```

The process exits with code 1 if any finding is at or above the specified
severity. Supported values: `low`, `medium`, `high`, `critical`.

**Recommended thresholds:**

| Environment | Threshold | Rationale |
| --- | --- | --- |
| Developer machine | none | Fast feedback, no blocking |
| Pull request | `critical` | Block only critical issues |
| Protected branch CI | `high` (after baselining) | Block high and critical |
| Audit | none | Review full report |

## Skipping Analysis Stages

Each stage can be skipped independently:

```bash
# Skip SAST
patchflow scan run --no-sast

# Skip secret detection
patchflow scan run --no-secrets

# Skip reachability analysis
patchflow scan run --no-reachability

# Skip license scanning
patchflow scan run --no-licenses

# Skip SAST and reachability for fastest SCA-only scan
patchflow scan run --no-sast --no-reachability --no-secrets
```

## Scoping the Scan

### Changed Files Only

```bash
patchflow scan run --changed-only
```

Only analyzes files changed in the current git diff. Automatically enables
incremental scanning for 5–50x speedup on large repos.

### Since a Branch or Commit

```bash
patchflow scan run --since main
```

Scans files changed since the given branch or commit. Useful for PR-style scans
against a base branch.

### Staged Files Only

```bash
patchflow scan run --staged
```

Scans only git staged files. Useful for pre-commit hooks.

### Specific Paths

```bash
patchflow scan run --path src/ --path tests/
```

Restricts the scan to specific paths. Can be repeated for multiple paths.

## Incremental Scanning

```bash
patchflow scan run --incremental
```

Only re-scans files that changed since the last scan. Uses the global cache
(`~/.cache/patchflow/<project-hash>/sast_state.json`) to track file hashes
between scans. This provides 5–50x speedup on large repositories.

`--changed-only` automatically enables incremental scanning.

## Offline Mode

```bash
patchflow scan run --offline
```

Skips all API calls and uses only the local OSV database and cache. Requires
`patchflow cache update` to have been run beforehand to download the local OSV
database.

Offline mode enables air-gapped scanning in regulated environments where
outbound network calls are prohibited.

See [Cache Management](./cache.md) for setting up the local OSV database.

## License Policy

```bash
patchflow scan run --license-policy "gpl,agpl,proprietary,unknown"
```

Fails (exit code 1) if any dependency has a restricted license. Licenses are
classified as:

| Category | Risk Level | Examples |
| --- | --- | --- |
| Permissive | Low | MIT, Apache-2.0, BSD-3-Clause |
| Weak copyleft | Medium | LGPL-2.1, MPL-2.0, EPL-2.0 |
| Copyleft | High | GPL-2.0, GPL-3.0, AGPL-3.0 |
| Proprietary | High | Commercial licenses |
| Unknown | Critical | License could not be determined |

## Framework Pack Selection

```bash
# Auto-detect frameworks (default)
patchflow scan run --framework auto

# Force-enable specific packs
patchflow scan run --framework express --framework react

# Disable a pack even if detected
patchflow scan run --disable-framework spring-security
```

`--disable-framework` takes precedence over `--framework`. See
[Framework Packs](./framework-packs.md) for the full list of packs.

## Custom Rules

```bash
patchflow scan run --rules .patchflow/rules.yaml
```

Loads custom rules from the specified YAML file. Defaults to
`.patchflow/rules.yaml` if it exists. See [Custom Rules](./custom-rules.md).

## Showing Suppressed Findings

```bash
patchflow scan run --show-suppressed
```

By default, findings suppressed by `//patchflow:ignore` comments are hidden. Use
`--show-suppressed` to include them in the output (marked as suppressed).

## Including Test Files

```bash
patchflow scan run --include-tests
```

By default, test files are excluded from SAST analysis. Use `--include-tests` to
include them.

## Taint Analysis Depth

```bash
patchflow scan run --taint-depth 5
```

Controls the maximum inter-procedural call-hop depth for taint analysis. Default
is 3. Set to 0 to disable taint analysis entirely. Higher values catch more
flows but increase scan time.

## Suggesting Fixes

```bash
patchflow scan run --suggest-fixes
```

Generates safe fix proposals for detected vulnerabilities. Fix proposals include
the original code, fixed code, patch, confidence level, and whether the fix is
auto-applicable. See [Fixes](./fixes.md).

## Baseline Comparison

```bash
patchflow scan run --new-only --baseline v1.0
```

Only reports findings not in the named baseline. Requires `--baseline`. See
[Baselines](./baselines.md).

## Not Respecting .gitignore

```bash
patchflow scan run --no-gitignore
```

By default, PatchFlow respects `.gitignore` patterns. Use `--no-gitignore` to
scan all files, including those ignored by git.

## Complete Flag Reference

| Flag | Type | Default | Description |
| --- | --- | --- | --- |
| `--profile` | string | `standard` | Scan profile: `quick`, `standard`, `deep` |
| `--governance-profile` | string | (from profile) | `dev`, `pr`, `ci`, `audit` |
| `--format` | string | (terminal) | `markdown`, `json`, `sarif` |
| `--output` | string | (stdout) | Write report to file |
| `--path` | string (repeatable) | (all) | Specific paths to scan |
| `--changed-only` | bool | false | Only analyze changed files |
| `--since` | string | (none) | Scan files changed since branch/commit |
| `--staged` | bool | false | Only scan staged files |
| `--incremental` | bool | false | Only re-scan changed files |
| `--no-sast` | bool | false | Skip SAST analysis |
| `--no-secrets` | bool | false | Skip secret detection |
| `--no-reachability` | bool | false | Skip reachability analysis |
| `--no-licenses` | bool | false | Skip license scanning |
| `--no-gitignore` | bool | false | Do not respect .gitignore |
| `--include-tests` | bool | false | Include test files in SAST |
| `--show-suppressed` | bool | false | Show suppressed findings |
| `--framework` | string (repeatable) | `auto` | Enable framework packs |
| `--disable-framework` | string (repeatable) | (none) | Disable framework packs |
| `--rules` | string | `.patchflow/rules.yaml` | Custom rules YAML file |
| `--taint-depth` | int | 3 | Taint analysis call-hop depth (0 disables) |
| `--fail-on` | string | (none) | Exit 1 if findings ≥ severity |
| `--baseline` | string | (none) | Baseline name for comparison |
| `--new-only` | bool | false | Only report new findings (requires `--baseline`) |
| `--license-policy` | string | (none) | Fail on restricted licenses |
| `--offline` | bool | false | Use local OSV DB only |
| `--suggest-fixes` | bool | false | Generate fix proposals |

## Other Scan Subcommands

### `patchflow scan local`

Scans the local repository for manifests and dependencies. Returns manifest
types and paths without running vulnerability queries.

```bash
patchflow scan local
```

### `patchflow scan changed`

Scans changed files in the repository. Identifies changed files via git diff
and parses manifests from them.

```bash
patchflow scan changed
```

### `patchflow scan export`

Runs a full analysis and exports results in various formats. Supports SBOM and
dependency graph formats in addition to JSON and SARIF.

```bash
# JSON export
patchflow scan export --format json --output report.json

# SARIF export with GitHub upload
patchflow scan export --format sarif --upload-github

# CycloneDX SBOM with VEX
patchflow scan export --format cyclonedx-json --include-vex --output sbom.json

# SPDX SBOM
patchflow scan export --format spdx-json --output spdx.json

# VEX (Vulnerability Exploitability Exchange)
patchflow scan export --format vex-json --output vex.json

# Dependency tree
patchflow scan export --format dep-tree

# Dependency graph (DOT format)
patchflow scan export --format dep-dot
```

**`scan export` flags:**

| Flag | Type | Default | Description |
| --- | --- | --- | --- |
| `--format` | string | `json` | `json`, `sarif`, `cyclonedx-json`, `spdx-json`, `vex-json`, `dep-tree`, `dep-dot` |
| `--output` | string | (stdout) | Output file path |
| `--upload-github` | bool | false | Upload SARIF to GitHub Code Scanning (requires `GITHUB_TOKEN`) |
| `--no-gitignore` | bool | false | Do not respect .gitignore |
| `--include-vex` | bool | false | Include VEX statements in CycloneDX SBOM |

### `patchflow scan image`

Scans a container image for OS package and language dependency vulnerabilities.
Uses Trivy as an external analyzer (must be installed).

```bash
patchflow scan image nginx:1.21
patchflow scan image myapp:latest --format json --output report.json
patchflow scan image alpine:3.18 --timeout 5m --severities CRITICAL,HIGH
```

**`scan image` flags:**

| Flag | Type | Default | Description |
| --- | --- | --- | --- |
| `--format` | string | (terminal) | `json`, `markdown` |
| `--output` | string | (stdout) | Output file path |
| `--timeout` | duration | 10m | Scan timeout |
| `--severities` | string | (all) | Comma-separated: `CRITICAL,HIGH,MEDIUM,LOW,INFO` |

See [Container Scanning](./container-scanning.md) for details.

## Next Steps

- [Dependencies](./dependencies.md) — `deps` command for dependency analysis
- [Reachability](./reachability.md) — Vulnerable dependency reachability
- [Reports](./reports.md) — Report generation
- [Baselines](./baselines.md) — Baseline management
