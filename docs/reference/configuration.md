---
title: Configuration Reference
description: Config file format, profiles, and all configurable options
---

# Configuration Reference

This page documents the PatchFlow CLI configuration file format, profiles, and
all configurable options.

## Config File Location

```text
~/.patchflow/config.yaml
```

Override with `--config` flag or `PATCHFLOW_CONFIG` environment variable.

## Config File Format

```yaml
# PatchFlow API URL (default: https://api.patchflow.dev)
apiurl: "https://api.patchflow.dev"

# API token (never written to config file; stored in OS keychain)
# Use `patchflow login --token <token>` to set the token
token: ""

# Organization name (for backend-connected commands)
org: ""

# Log level: debug, info, warn, error (default: info)
loglevel: ""
```

## Configuration Options

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `apiurl` | string | `https://api.patchflow.dev` | PatchFlow API URL |
| `token` | string | (empty) | API token (stored in keychain, not config file) |
| `org` | string | (empty) | Organization name |
| `loglevel` | string | (empty) | Logging level: `debug`, `info`, `warn`, `error` |

## Environment Variables

| Variable | Overrides | Description |
| --- | --- | --- |
| `PATCHFLOW_API_URL` | `apiurl` | PatchFlow API URL |
| `PATCHFLOW_TOKEN` | `token` | API token (for migration; prefer `patchflow login`) |
| `PATCHFLOW_ORG` | `org` | Organization name |
| `PATCHFLOW_LOG_LEVEL` | `loglevel` | Logging level |
| `PATCHFLOW_CACHE_DIR` | (cache dir) | Override cache directory |
| `XDG_CACHE_HOME` | (cache base) | XDG cache home directory |

## Configuration Profiles

Profiles allow switching between different org/workspace contexts. Stored in:

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
  staging:
    api_url: "https://staging-api.patchflow.dev"
    org: "my-org-staging"
    log_level: "debug"
```

### Profile Commands

```bash
# Create a profile
patchflow config profile create production \
  --api-url https://api.patchflow.dev \
  --org my-org \
  --log-level info

# Switch active profile
patchflow config profile use production

# List profiles
patchflow config profile list

# Show profile details
patchflow config profile show production

# Delete a profile (cannot delete "default" or active profile)
patchflow config profile delete staging
```

### Profile Merge Behavior

- Active profile values override base config on load
- Profile values include: `api_url`, `org`, `log_level` (no token — tokens are
  stored in keychain)
- The `default` profile cannot be deleted

## Project-Level Configuration

Project-level configuration is stored in `.patchflow/`:

```text
.patchflow/
  config.yml      Project configuration
  rules.yaml      Custom rules and framework overrides
  baselines/      Baseline snapshots
  reports/        Generated reports
```

### Custom Rules File

The custom rules file (`.patchflow/rules.yaml`) contains:

- Framework pack selection and overrides
- Custom regex rules

See [YAML Policy Reference](./yaml-policy.md) for the full format.

## Config Command Reference

### `patchflow config show`

Show current configuration. Token is masked.

```bash
patchflow config show
patchflow config show --json
```

### `patchflow config set <key> <value>`

Set a configuration value. Valid keys: `api_url`, `org`, `log_level`. The
`token` key is rejected — use `patchflow login --token` instead.

```bash
patchflow config set api_url https://api.patchflow.dev
patchflow config set org my-organization
patchflow config set log_level info
```

## Cache Directory Resolution

The cache directory is resolved in this order (highest priority first):

1. `--cache-dir` CLI flag
2. `PATCHFLOW_CACHE_DIR` environment variable
3. `$XDG_CACHE_HOME/patchflow/<project-hash>/`
4. `~/.cache/patchflow/<project-hash>/` (XDG default)

The project hash is a SHA256 of the absolute project root path (first 16 hex
characters).

See [Cache Management](../user-guides/cache.md) for details.

## Next Steps

- [YAML Policy Reference](./yaml-policy.md) — Custom rules YAML format
- [Global Flags](./global-flags.md) — CLI flags reference
- [Cache Management](../user-guides/cache.md) — Cache configuration
