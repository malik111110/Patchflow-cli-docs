# Developer CI/CD

This page documents CI for the PatchFlow CLI and the docs site.

## CLI Test Gate

Recommended CLI checks before merging:

```bash
# Run framework foundation tests
go test ./internal/frameworks/ ./internal/sast/frameworks/... ./internal/rules -count=1

# Run all SAST + config + command tests
go test ./internal/sast/... ./internal/config/... ./cmd/... -count=1

# Run all tests
go test ./internal/... ./cmd/... -count=1

# Vet
go vet ./...

# Build
go build ./...
```

### Focused Framework Tests

Run the focused framework set before touching packs:

```bash
go test ./internal/frameworks/... ./internal/sast/frameworks/... -count=1
```

### Specific Scanner Tests

```bash
# Pattern scanner tests
go test ./internal/sast/patterns/ -v

# Secret scanner tests
go test ./internal/sast/secrets/ -v

# Taint pattern tests
go test ./internal/sast/taintpatterns/ -v
```

## Benchmark Gate

Use the real benchmark suite for framework pack changes:

```bash
go build -o patchflow .
./patchflow benchmark run benchmarks/framework-pack-validation.yaml --no-tools
```

The benchmark results are stored under:

```text
results/<YYYY-MM>/<repo-name>/patchflow.json
results/<YYYY-MM>/<repo-name>/patchflow.sarif
results/<YYYY-MM>/summary.json
```

See [Benchmarks](./benchmarks.md) for the benchmark suite documentation.

## Build the CLI Binary

```bash
go build -o patchflow .
```

The binary is placed in the project root. Use it for local testing and
benchmarking.

## Release Process

### 1. Build the CLI Binary

```bash
go build -o patchflow .
```

### 2. Run Full Test Suite

```bash
go test ./internal/... ./cmd/... -count=1
go vet ./...
```

### 3. Run Benchmark Suites

Run targeted benchmark suites for changed scanner areas:

```bash
./patchflow benchmark run benchmarks/framework-pack-validation.yaml --no-tools
```

### 4. Generate Release

PatchFlow uses GoReleaser for cross-platform builds:

```bash
# Dry run
goreleaser release --snapshot --clean

# Full release (requires git tag)
git tag v0.x.x
goreleaser release --clean
```

The `.goreleaser.yml` configuration defines build targets for Linux, macOS, and
Windows (amd64 and arm64).

### 5. Publish

- CLI release: GitHub Releases (via GoReleaser)
- Homebrew tap: Updated automatically by GoReleaser
- Scoop bucket: Updated manually
- Docker/Podman image: Built and pushed via CI

## CI Workflows

The CLI repository includes GitHub Actions workflows for:

- **CI**: Runs tests, vet, and build on every push and pull request
- **Release**: Builds and publishes releases on tag push (via GoReleaser)

### CI Workflow

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.25'
      - run: go vet ./...
      - run: go test ./internal/... ./cmd/... -count=1
      - run: go build ./...
```

### Release Workflow

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    tags: ['v*']
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.25'
      - uses: goreleaser/goreleaser-action@v6
        with:
          version: latest
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Pre-commit Hooks

The CLI repository includes a pre-commit hook configuration
(`.pre-commit-hooks.yaml`) that allows other repositories to use PatchFlow as a
pre-commit hook:

```yaml
- id: patchflow
  name: PatchFlow Security Scan
  entry: patchflow scan run --profile dev --no-reachability
  language: system
  stages: [commit]
```

## Release Checklist

1. Build the CLI binary
2. Run full test suite
3. Run targeted benchmark suites for changed scanner areas
4. Create a git tag (`git tag v0.x.x`)
5. Push the tag (`git push origin v0.x.x`)
6. GoReleaser builds and publishes the release
7. Verify the release on GitHub Releases
8. Verify Homebrew tap update
9. Update documentation if needed

## Next Steps

- [Benchmarks](./benchmarks.md) — Benchmark suite
- [Architecture](./architecture.md) — Internal architecture
- [Overview](./overview.md) — Package overview
