---
title: Cache Management
description: Local cache management and offline OSV database
---

# Cache Management

PatchFlow maintains a local cache for OSV vulnerability responses, incremental
SAST state, and package registry metadata. The `patchflow cache` command manages
this cache.

## Cache Location

The cache lives in a global XDG-compliant location:

```text
~/.cache/patchflow/<project-hash>/
```

Or, if `XDG_CACHE_HOME` is set:

```text
$XDG_CACHE_HOME/patchflow/<project-hash>/
```

The project hash is a SHA256 of the absolute project root path (first 16 hex
characters). This ensures each project has an isolated cache.

### Override the Cache Directory

```bash
# CLI flag
patchflow scan run --cache-dir /custom/cache/path

# Environment variable
export PATCHFLOW_CACHE_DIR=/custom/cache/path
```

Resolution order (highest priority first):

1. `--cache-dir` CLI flag
2. `PATCHFLOW_CACHE_DIR` environment variable
3. `$XDG_CACHE_HOME/patchflow/<project-hash>/`
4. `~/.cache/patchflow/<project-hash>/` (XDG default)

## Cache Contents

| Path | Description |
| --- | --- |
| `osv/` | OSV vulnerability response cache (JSON files keyed by dependency hash) |
| `sast_state.json` | Incremental SAST scan state (file hashes between scans) |
| `registry/` | Package registry metadata cache (npm, PyPI, Maven licenses) |
| `maven/` | Maven POM file cache for transitive resolution |

**Not in cache:** Baselines (`.patchflow/baselines/`) and reports
(`.patchflow/reports/`) are project artifacts and are preserved by
`cache clean`.

## Cache Status

```bash
patchflow cache status
```

Shows:

- Cache directory path
- Total cache size (human-readable + bytes)
- OSV cache: entry count, size, directory
- SAST state: exists, last modified time, path
- Baselines: count (reported for context, not part of cache)

```bash
patchflow cache status --json
```

Outputs the status as structured JSON.

## Cache Clean

```bash
patchflow cache clean
```

Removes:

- `osv/` directory (OSV response cache)
- `sast_state.json` (incremental SAST state)

Preserves:

- Baselines (`.patchflow/baselines/`)
- Reports (`.patchflow/reports/`)

Shows what will be removed and the freed space. Requires confirmation unless
`--force` is set:

```bash
patchflow cache clean --force
```

JSON mode requires `--force` (returns an error otherwise, to prevent accidental
clean in automation).

## Local OSV Database (Offline Mode)

```bash
patchflow cache update
```

Downloads the local OSV vulnerability database for offline scanning. This
enables `patchflow scan run --offline` without any API calls to OSV.dev.

### How It Works

1. Downloads OSV bulk exports from Google Cloud Storage:
   `https://osv-vulnerabilities.storage.googleapis.com/{ecosystem}/all.zip`
2. Stores at: `~/.patchflow/osv-db/{ecosystem}/`
3. Creates a consolidated `.db.bin` file (gob-encoded for fast loading)
4. Builds a package→vulnIDs index (`.index.bin`) for quick lookups
5. Refresh interval: 24 hours (skips download if up-to-date unless `--force`)

### Supported Ecosystems

| Ecosystem | Description |
| --- | --- |
| `PyPI` | Python |
| `npm` | Node.js |
| `Maven` | Java |
| `Go` | Golang |
| `RubyGems` | Ruby |
| `Packagist` | PHP |
| `crates.io` | Rust |

### Flags

```bash
# Force refresh even if DB is fresh (< 24h old)
patchflow cache update --force

# Download specific ecosystems only
patchflow cache update --ecosystems pypi,npm,go
```

| Flag | Type | Default | Description |
| --- | --- | --- | --- |
| `--force` | bool | false | Force refresh even if DB is fresh |
| `--ecosystems` | string | (all) | Comma-separated ecosystems to download |

### Default Ecosystems

If `--ecosystems` is not specified, PatchFlow downloads: PyPI, npm, Maven, Go,
RubyGems, Packagist.

### Output

The command reports vulnerability counts per ecosystem after download:

```
Downloaded OSV database:
  PyPI:      45,231 vulnerabilities
  npm:       32,104 vulnerabilities
  Maven:     12,876 vulnerabilities
  Go:         3,421 vulnerabilities
  RubyGems:   2,891 vulnerabilities
  Packagist:  1,234 vulnerabilities
Total: 97,757 vulnerabilities
```

## Using Offline Mode

After downloading the local OSV database:

```bash
patchflow scan run --offline
```

Offline mode:

- Skips all API calls to OSV.dev
- Uses only the local OSV database and cache
- Enables air-gapped scanning in regulated environments
- Requires `patchflow cache update` to have been run beforehand

## Cache Migration

PatchFlow automatically migrates legacy `.patchflow/cache/` directories to the
global cache location. This happens transparently on the first scan after
upgrading.

## Next Steps

- [Scan Your Project](./scan) — Use `--offline` for air-gapped scans
- [Configuration](./configuration) — Cache directory configuration
- [Dependencies](./dependencies) — Dependency analysis with OSV data
