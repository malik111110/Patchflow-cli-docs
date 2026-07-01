# Developer CI/CD

This page documents CI for the CLI and the docs site.

## CLI Test Gate

Recommended CLI checks:

```bash
go test ./internal/benchmark ./internal/sast/frameworks/... ./internal/rules -count=1
go test ./internal/sast/... ./internal/config/... ./cmd/... -count=1
go build ./...
```

Run the focused framework set before touching packs:

```bash
go test ./internal/frameworks/... ./internal/sast/frameworks/... -count=1
```

## Benchmark Gate

Use the real benchmark suite for framework pack changes:

```bash
go build -o patchflow .
cd /Users/digitalcenter/patchflow-cli
./patchflow benchmark run benchmarks/framework-pack-validation.yaml --no-tools
```

The current docs point at:

```text
/Users/digitalcenter/patchflow-benchmarks/.bench-work
```

## Docs Site Gate

```bash
cd /Users/digitalcenter/patchflow-cli-docs
npm ci
npm run check
npm run build
```

The docs build regenerates the command reference from `PATCHFLOW_BIN` before
VitePress builds the static site.

## Release Checklist

1. Build the CLI binary.
2. Generate command reference in docs.
3. Run docs checks and static build.
4. Run targeted benchmark suites for changed scanner areas.
5. Publish the CLI release.
6. Publish docs site from `docs/.vitepress/dist`.
