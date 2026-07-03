---
title: Version and Doctor
description: patchflow version and patchflow doctor commands for diagnostics and bug reports
---

# Version and Doctor

## patchflow version

Print version information. Useful for bug reports, CI logs, and verifying
installations.

### Text output

```bash
patchflow version
```

```text
patchflow version 0.1.3 (commit: 85ca3f3, built: 2026-07-03T21:30:00Z)
```

### JSON output

```bash
patchflow version --json
```

```json
{
  "version": "0.1.3",
  "commit": "85ca3f3",
  "built_at": "2026-07-03T21:30:00Z",
  "go_version": "go1.26.4",
  "ruleset_version": "framework-rules-v1",
  "schema_version": "1.0",
  "sarif_version": "2.1.0",
  "osv_db_version": "runtime"
}
```

### Fields

| Field | Description |
| --- | --- |
| `version` | Semantic version (e.g., `0.1.3`) |
| `commit` | Git commit hash at build time |
| `built_at` | Build timestamp (ISO 8601) |
| `go_version` | Go runtime version used to build the binary |
| `ruleset_version` | Embedded rule collection version (e.g., `framework-rules-v1`) |
| `schema_version` | Config schema version (e.g., `1.0`) |
| `sarif_version` | SARIF output schema version (e.g., `2.1.0`) |
| `osv_db_version` | OSV database version (`runtime` = determined at scan time) |

### Use in CI

Include version info in CI logs for reproducibility:

```bash
patchflow version --json > patchflow-version.json
patchflow scan run --json --output results.json
```

## patchflow doctor

Run environment diagnostics. Checks that PatchFlow is correctly configured and
all dependencies are available.

### Text output

```bash
patchflow doctor
```

```text
PatchFlow Doctor
================
[OK] Version: 0.1.3 (commit: 85ca3f3, go: go1.26.4)
[OK] Git installed: git version 2.51.2
[OK] Inside a git repository: /path/to/repo
[OK] Remote configured: git@github.com:org/repo.git

Configuration:
[OK] Config found and valid: .patchflow/rules.yaml

Cache:
[OK] Cache directory writable: /home/user/.cache/patchflow

Output:
[OK] SARIF output writable

Embedded SAST Scanners (always available, zero installation):
[OK] gosast-embedded      (go) — 35 rules
[OK] secrets-embedded     (secrets) — 35 rules
[OK] patterns-embedded    (multi) — 472 rules
[OK] taint-ssa            (go) — 9 rules
[OK] treesitter-ast       (multi) — 92 rules
[OK] taint-patterns       (multi) — 47 rules

External SAST Tools (optional supplements):
[OK] gosec                (go) — installed
[--] bandit               (python) — not installed (optional)
[--] semgrep              (multi) — not installed (optional)
[--] gitleaks             (secrets) — not installed (optional)

Overall status: ok
```

### JSON output

```bash
patchflow doctor --json
```

```json
{
  "version": "0.1.3",
  "commit": "85ca3f3",
  "go_version": "go1.26.4",
  "built_at": "2026-07-03T21:30:00Z",
  "status": "ok",
  "is_git_repo": true,
  "git_version": "git version 2.51.2",
  "repo_root": "/path/to/repo",
  "remote_url": "git@github.com:org/repo.git",
  "config_found": true,
  "config_path": ".patchflow/rules.yaml",
  "config_valid": true,
  "cache_dir": "/home/user/.cache/patchflow",
  "cache_writable": true,
  "sarif_writable": true,
  "embedded_scanners": [...],
  "external_tools": [...]
}
```

### Checks

| Check | Description |
| --- | --- |
| Version | CLI version, commit, Go version |
| Git | Git installation, repository detection, remote URL |
| Config | `.patchflow/rules.yaml` found and valid |
| Cache | Cache directory exists and is writable |
| SARIF | SARIF output file is writable |
| Embedded scanners | All embedded SAST scanners are available (always) |
| External tools | Optional tools (gosec, bandit, semgrep, gitleaks) detected |

### Status values

| Status | Meaning |
| --- | --- |
| `ok` | All checks passed |
| `warning` | Some checks failed (e.g., config not found, external tools missing) |
| `error` | Critical checks failed (e.g., git not installed) |

### Use in CI

Run `doctor` before scanning to verify the environment:

```bash
patchflow doctor --json || true
patchflow scan run --config .patchflow/rules.yaml
```

## See Also

- [CI Templates](../user-guides/ci-templates)
- [Configuration](./configuration)
- [Global Flags](./global-flags)
