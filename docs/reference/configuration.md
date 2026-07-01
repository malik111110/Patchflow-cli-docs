# Configuration

PatchFlow reads configuration from project files, user config, environment
variables, and CLI flags.

## Project Config

Project-level configuration lives under `.patchflow/`.

```text
.patchflow/
  config.yml
  rules.yaml
  reports/
  cache/
```

## User Config

User-level config is typically stored in:

```text
~/.patchflow/config.yaml
```

Authentication tokens should be stored with `patchflow login` and the platform
keychain when available.

## Precedence

From lowest to highest:

| Layer | Example |
| --- | --- |
| Built-in defaults | Standard scan profile |
| User config | `~/.patchflow/config.yaml` |
| Project config | `.patchflow/config.yml` |
| YAML rule policy | `.patchflow/rules.yaml` |
| Environment | `PATCHFLOW_TOKEN` |
| CLI flags | `--framework express` |

## Common Settings

```yaml
api_url: https://api.patchflow.dev
org: engineering
log_level: info

frameworks:
  auto_detect: true
  enabled:
    - express
    - django
```

## Configuration Commands

```bash
patchflow config show
patchflow config set org engineering
patchflow config profile list
patchflow config profile use production
```
