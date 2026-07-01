---
title: pre-commit Hook
description: Use PatchFlow as a pre-commit hook to catch issues before commit
---

# pre-commit Hook

Use PatchFlow as a pre-commit hook to catch security issues before they reach
your repository.

## Generate a pre-commit Configuration

```bash
patchflow init pre-commit
```

This appends a PatchFlow hook to `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/Patchflow-security/patchflow-cli
    rev: v0.1.0
    hooks:
      - id: patchflow
        entry: patchflow scan run --profile dev --no-reachability
        language: system
        stages: [commit]
```

## Install the Hook

After generating the configuration:

```bash
pre-commit install
```

Ensure `patchflow` is on your `PATH`. The hook runs `patchflow scan run` with the
`dev` governance profile (fast, stable rules only) and skips reachability
analysis for speed.

## Customizing the Hook

Edit `.pre-commit-config.yaml` to customize the scan:

```yaml
repos:
  - repo: https://github.com/Patchflow-security/patchflow-cli
    rev: v0.1.0
    hooks:
      - id: patchflow
        entry: patchflow scan run --profile quick --no-reachability --no-licenses
        language: system
        stages: [commit]
        # Only run on changed files
        pass_filenames: false
```

### Recommended pre-commit Configuration

For fast pre-commit feedback:

```yaml
repos:
  - repo: https://github.com/Patchflow-security/patchflow-cli
    rev: v0.1.0
    hooks:
      - id: patchflow
        entry: patchflow scan run --profile quick --changed-only --no-reachability --no-licenses
        language: system
        stages: [commit]
        pass_filenames: false
```

This configuration:

- Uses `quick` profile for fast feedback
- Only scans changed files (`--changed-only`)
- Skips reachability and license scanning for speed
- Uses `dev` governance profile (stable rules only)

## Running Manually

You can also run PatchFlow as a staged-files scan without pre-commit:

```bash
patchflow scan run --staged --profile quick --no-reachability
```

The `--staged` flag scans only git staged files, which is equivalent to what a
pre-commit hook would check.

## Tips

- Keep the pre-commit scan fast: use `--profile quick` and skip slow stages
- Do not use `--fail-on` in pre-commit — use it in CI instead
- Use `--changed-only` or `--staged` to limit scope
- Ensure `patchflow` is installed and on `PATH` for all team members

## Next Steps

- [GitHub Actions](./github-actions) — CI integration
- [Recommended Workflow](../workflows/recommended) — Team adoption
- [Scan Your Project](../user-guides/scan) — All scan flags
