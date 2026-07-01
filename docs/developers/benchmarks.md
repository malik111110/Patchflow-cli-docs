---
title: Benchmarks
description: Real-repo benchmark suite for scanner accuracy and performance
---

# Benchmarks

PatchFlow includes a real-repo benchmark suite for measuring scanner accuracy,
performance, and false-positive rates against open-source repositories.

## Benchmark Command

### Run a Benchmark Suite

```bash
patchflow benchmark run benchmarks/framework-pack-validation.yaml
```

Executes a benchmark suite from a YAML config file. PatchFlow is run as a real
subprocess to measure actual CLI behavior.

**Flags:**

| Flag | Type | Default | Description |
| --- | --- | --- | --- |
| `--config` | string | (positional) | Benchmark config file path |
| `--no-tools` | bool | false | Skip comparison tools (only run PatchFlow) |
| `--no-warm` | bool | false | Skip the warm (cached) second run |
| `--timeout` | duration | 15m | Per-scan timeout |
| `--results-dir` | string | `results/<YYYY-MM>/` | Override results directory |

### Compare Benchmark Runs

```bash
patchflow benchmark compare results/
```

Compares `summary.json` results across multiple runs. Shows metrics comparison
sorted by run directory name (YYYY-MM).

### Regenerate a Report

```bash
patchflow benchmark report results/2026-01/
patchflow benchmark report results/2026-01/ --format json
```

Regenerates a summary report from existing benchmark results. Markdown writes to
`<path>/summary.md`, JSON outputs to stdout.

## Benchmark YAML Format

```yaml
suite: "patchflow-cli-benchmark"
date: "2026-01-15"
patchflow:
  binary: "patchflow"
  profile: "standard"
  fail_on: ""
  no_reachability: false
  extra_args: []
tools: ["semgrep", "trivy", "gitleaks", "osv-scanner"]
repos:
  - name: "juice-shop"
    type: "intentionally-vulnerable"
    language: "javascript"
    expected: "high-vulnerability-density"
    url: "https://github.com/bkimminich/juice-shop.git"
    ref: "v14.0.0"
    expected_findings: ["CVE-2020-1234", "PF-JS-XSS-001"]
    expected_framework_findings: ["PF-EXPRESS-XSS-001"]
    publish_detail: true
  - name: "clean-repo"
    type: "clean-real-world"
    language: "go"
    expected: "low-false-positive-rate"
    path: "/path/to/local/repo"
results_dir: ""
work_dir: ""
```

### Configuration Fields

| Field | Description |
| --- | --- |
| `suite` | Benchmark suite name |
| `date` | Benchmark date (default: today) |
| `patchflow.binary` | PatchFlow binary path (default: `patchflow`) |
| `patchflow.profile` | Scan profile: `quick`, `standard`, `deep` |
| `patchflow.fail_on` | Severity threshold for exit code 1 |
| `patchflow.no_reachability` | Skip reachability analysis |
| `patchflow.extra_args` | Additional CLI arguments |
| `tools` | Comparison tools to run (semgrep, trivy, gitleaks, osv-scanner) |
| `repos` | List of repositories to benchmark |
| `results_dir` | Results directory (default: `results/<YYYY-MM>/`) |
| `work_dir` | Working directory (default: `.bench-work`) |

### Repository Configuration

| Field | Description |
| --- | --- |
| `name` | Repository name |
| `type` | `intentionally-vulnerable`, `historical-vulnerable`, or `clean-real-world` |
| `language` | Primary language |
| `expected` | Expected outcome |
| `url` | Git URL (for cloning) |
| `ref` | Git ref (tag, branch, or commit) |
| `path` | Local path (alternative to `url`) |
| `expected_findings` | Findings expected to be detected |
| `expected_framework_findings` | Framework findings expected |
| `publish_detail` | Whether findings can be published in detail |

### Repository Types

| Type | Description | Publishing |
| --- | --- | --- |
| `intentionally-vulnerable` | Safe, designed-for-testing projects (Juice Shop, WebGoat, DVWA) | All findings may be published |
| `historical-vulnerable` | Old tagged releases with known CVEs | Known/historical issues may be published |
| `clean-real-world` | Reputable, active, mature projects | Findings anonymized/responsibly disclosed, never published in detail |

## Metrics Collected

| Metric | Description |
| --- | --- |
| LOC | Lines of code scanned |
| Files scanned | Number of files processed |
| Scan duration (cold/warm) | Time for cold and cached runs |
| Cache speedup | Warm vs cold speedup ratio |
| Engines used | Which scanner engines ran |
| Total findings | All findings by count |
| Findings by severity | Critical, high, medium, low counts |
| Findings by confidence | High, medium, low counts |
| Findings by type | SCA, SAST, secrets counts |
| Framework findings | Findings by framework pack |
| SARIF generated/valid | Whether SARIF was produced and valid |
| Exit code | Scan exit code |
| Memory MB | Peak memory usage |
| True positives | Correctly identified vulnerabilities |
| False positives | Incorrectly flagged issues |
| Unknown | Unverified findings |
| Precision | True positives / (true + false positives) |
| Recall | True positives / expected findings |
| Noise rate | False positives / total findings |
| Tool comparison | Finding counts per comparison tool |
| Expected findings detected/missed | Which expected findings were found |
| Framework recall | Framework-specific recall metric |

## Results Storage

Benchmark results are stored at:

```text
results/<YYYY-MM>/<repo-name>/patchflow.json
results/<YYYY-MM>/<repo-name>/patchflow.sarif
results/<YYYY-MM>/summary.json
results/<YYYY-MM>/summary.md  (when using benchmark report)
```

## Running Benchmarks for Framework Pack Changes

When modifying framework packs, run the benchmark suite to validate:

```bash
# Build the CLI with your changes
go build -o patchflow .

# Run the framework pack validation benchmark
./patchflow benchmark run benchmarks/framework-pack-validation.yaml --no-tools

# Compare with previous results
./patchflow benchmark compare results/
```

The benchmark suite validates:

- **Recall**: Are known vulnerabilities detected?
- **Precision**: Are false positives minimized?
- **Noise**: Do clean repos produce no findings?
- **Framework recall**: Are framework-specific rules effective?

## Promotion Criteria

A framework pack is not ready for promotion from `experimental` to `beta` or
`stable` until benchmarks show:

- Positive fixtures pass (vulnerable code triggers rules)
- Safe fixtures pass (safe code does not trigger rules)
- Normal no-noise fixtures pass (real-world code does not trigger rules)
- Real benchmark repo coverage
- Framework-specific recall metrics are acceptable
- Clean-repo noise checks pass (no false positives on clean repos)

## Next Steps

- [Framework Rule Packs](./rule-packs) — Pack development
- [Adding a Framework Pack](./adding-packs) — Pack creation guide
- [Developer CI/CD](./ci-cd) — CI for the CLI
