---
title: Configuration
description: Configuration files, profiles, and environment variables
---

# Configuration

PatchFlow CLI stores configuration at two levels: global user configuration and
project-level configuration. This page covers the `config` command, configuration
profiles, and the config file format.

## Global Configuration

The global config file is located at:

```text
~/.patchflow/config.yaml
```

### Config File Format

```yaml
apiurl: "https://api.patchflow.dev"  # PatchFlow API URL
token: ""                             # Never written to config (stored in keychain)
org: ""                               # Organization name
loglevel: ""                          # Logging level
```

### Environment Variables

Environment variables override config file values:

| Variable | Overrides | Description |
| --- | --- | --- |
| `PATCHFLOW_API_URL` | `apiurl` | PatchFlow API URL |
| `PATCHFLOW_TOKEN` | `token` | API token (for migration; prefer `patchflow login`) |
| `PATCHFLOW_ORG` | `org` | Organization name |
| `PATCHFLOW_LOG_LEVEL` | `loglevel` | Logging level |
| `PATCHFLOW_CACHE_DIR` | (cache dir) | Override cache directory |

## Config Command

### Show Current Configuration

```bash
patchflow config show
```

Displays current config values. The token is masked as `***` if present.

```bash
patchflow config show --json
```

Outputs configuration as structured JSON.

### Set a Configuration Value

```bash
patchflow config set api_url https://api.patchflow.dev
patchflow config set org my-organization
patchflow config set log_level info
```

Valid keys: `api_url`, `org`, `log_level`. The `token` key is rejected — use
`patchflow login --token` instead.

## Configuration Profiles

Profiles allow switching between different org/workspace contexts (e.g.,
development, staging, production). Profiles are stored in:

```text
~/.patchflow/profiles.yaml
```

### Profile File Format

```yaml
active: "default"
items:
  default:
    api_url: ""
    org: ""
    log_level: ""
  production:
    api_url: "https://api.patchflow.dev"
    org: "my-org"
    log_level: "info"
```

### Create a Profile

```bash
patchflow config profile create production \
  --api-url https://api.patchflow.dev \
  --org my-org \
  --log-level info
```

Flags fall back to current config values when not provided. Tokens are
intentionally excluded from profiles (stored in keychain).

### Switch Active Profile

```bash
patchflow config profile use production
```

Sets the active profile. Active profile values override base config on load.

### List Profiles

```bash
patchflow config profile list
```

Lists all profiles with an active indicator (`*`). Shows name, API URL, org, and
log level.

### Show Profile Details

```bash
patchflow config profile show production
```

### Delete a Profile

```bash
patchflow config profile delete production
```

Cannot delete the `default` profile or the currently active profile.

## Project-Level Configuration

Project-level configuration is stored in:

```text
.patchflow/config.yml
```

Created by `patchflow init`, this file contains project-specific settings. The
`.patchflow/` directory also contains:

- `baselines/` — Baseline snapshots
- `reports/` — Generated reports
- `rules.yaml` — Custom rules and framework overrides (see
  [Custom Rules](./custom-rules.md))

## Global CLI Flags

These flags are available on all commands and override configuration:

| Flag | Description |
| --- | --- |
| `--config <path>` | Config file path |
| `--api-url <string>` | PatchFlow API URL (overrides config) |
| `--cache-dir <string>` | Override cache directory |
| `--json` | Output in JSON format |
| `-v, --verbose` | Enable verbose logging |
| `--no-color` | Disable colored output |
| `-q, --quiet` | Suppress non-essential output (for CI scripting) |

## Cache Directory Resolution

The cache directory is resolved in the following order (highest priority first):

1. `--cache-dir` CLI flag
2. `PATCHFLOW_CACHE_DIR` environment variable
3. `$XDG_CACHE_HOME/patchflow/<project-hash>/`
4. `~/.cache/patchflow/<project-hash>/` (XDG default)

The project hash is a SHA256 of the absolute project root path (first 16 hex
characters). See [Cache Management](./cache.md) for details.

## Next Steps

- [Authentication](./authentication.md) — Login and token management
- [Cache Management](./cache.md) — Cache directory and local OSV database
- [Custom Rules](./custom-rules.md) — Project-level rules YAML
