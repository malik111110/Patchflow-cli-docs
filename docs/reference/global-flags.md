# Global Flags

These flags are available on all PatchFlow commands. They are inherited from the
root command via Cobra's persistent flag mechanism.

## Flag Reference

| Flag | Type | Description |
| --- | --- | --- |
| `--config` | string | Config file path (default: `~/.patchflow/config.yaml`) |
| `--api-url` | string | PatchFlow API URL (overrides config file) |
| `--cache-dir` | string | Override cache directory (default: `~/.cache/patchflow/` or `$XDG_CACHE_HOME/patchflow/`) |
| `--json` | bool | Output in JSON format |
| `-v, --verbose` | bool | Enable verbose logging |
| `--no-color` | bool | Disable colored output |
| `-q, --quiet` | bool | Suppress non-essential output (for CI scripting) |
| `-h, --help` | bool | Help for the current command |

## Usage Examples

### JSON Output

```bash
patchflow scan run --json
patchflow deps list --json
patchflow rules list --json
patchflow cache status --json
```

JSON mode outputs structured JSON suitable for automation, dashboards, and
piping to tools like `jq`.

### Verbose Logging

```bash
patchflow scan run -v
patchflow scan run --verbose
```

Verbose mode enables detailed logging, showing which scanners run, which files
are processed, and timing information.

### Quiet Mode

```bash
patchflow scan run -q
patchflow scan run --quiet
```

Quiet mode suppresses non-essential output, keeping only findings and errors.
Useful for CI scripting where only the result matters.

### No Color

```bash
patchflow scan run --no-color
```

Disables ANSI color codes in terminal output. Useful when output is piped to a
file or when the terminal does not support colors.

### Custom Config File

```bash
patchflow scan run --config /path/to/custom-config.yaml
```

Loads configuration from the specified file instead of the default
`~/.patchflow/config.yaml`.

### Custom API URL

```bash
patchflow review pr --submit --api-url https://custom-api.patchflow.dev
```

Overrides the API URL for backend-connected commands. Useful for self-hosted
PatchFlow instances.

### Custom Cache Directory

```bash
patchflow scan run --cache-dir /fast-ssd/patchflow-cache
```

Overrides the cache directory. Useful for directing cache to faster storage or
to a shared location in CI.

## Environment Variables

Environment variables override config file values but are overridden by CLI
flags:

| Variable | Overrides |
| --- | --- |
| `PATCHFLOW_API_URL` | `apiurl` config key |
| `PATCHFLOW_TOKEN` | `token` config key (for migration; prefer `patchflow login`) |
| `PATCHFLOW_ORG` | `org` config key |
| `PATCHFLOW_LOG_LEVEL` | `loglevel` config key |
| `PATCHFLOW_CACHE_DIR` | Cache directory |
| `XDG_CACHE_HOME` | Base cache directory (cache is at `$XDG_CACHE_HOME/patchflow/`) |
| `GITHUB_TOKEN` | Used by `scan export --upload-github` for SARIF upload |

## Precedence

Configuration values are resolved in this order (highest priority first):

1. CLI flags (`--api-url`, `--cache-dir`, etc.)
2. Environment variables (`PATCHFLOW_API_URL`, etc.)
3. Active configuration profile
4. Config file (`~/.patchflow/config.yaml`)

## Next Steps

- [Configuration](./configuration.md) — Config file format and profiles
- [Commands](./commands.md) — Command reference
- [Generated Commands](./generated-commands.md) — Full command help output
