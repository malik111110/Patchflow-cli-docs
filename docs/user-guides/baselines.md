---
title: Baselines
description: Snapshot known findings so CI can focus on new issues
---

# Baselines

Baselines store a snapshot of known findings so CI can focus on new issues. This
is the preferred path for adopting PatchFlow on existing repositories with
historical findings.

## How Baselines Work

1. Run a full scan on your default branch to populate findings.
2. Create a named baseline: `patchflow baseline create --name v1.0`
3. Future scans compare against the baseline: `patchflow scan run --new-only
   --baseline v1.0`
4. Only findings not in the baseline are reported.

Baselines use **stable semantic fingerprints** (rule ID + scanner + normalized
path + normalized snippet) so findings survive line-number shifts from unrelated
edits. A finding at line 42 that moves to line 50 due to an unrelated edit is
still recognized as the same finding.

## Baseline Storage

Baselines are stored under:

```text
.patchflow/baselines/<name>.json
```

Each baseline includes:

- Finding fingerprints (rule ID, scanner, normalized path, normalized snippet)
- Finding metadata (severity, file, line)
- Commit SHA (if in a git repository)
- Creation timestamp

Baselines are preserved by `patchflow cache clean` — they are project artifacts,
not cache.

## Create a Baseline

```bash
# Run a full scan first to populate findings
patchflow scan run --profile deep

# Create a named baseline
patchflow baseline create --name v1.0
```

The baseline captures all current SAST findings. The command runs a full SAST
scan (120s timeout) and stores the results with stable fingerprints.

Expected output:

```text
Running full SAST scan...
Scan complete: 23 findings detected.
Baseline 'v1.0' created with 23 findings.
Stored at: .patchflow/baselines/v1.0.json
```

## List Baselines

```bash
patchflow baseline list
```

Lists all saved baselines with finding counts and creation dates, sorted
alphabetically by name.

Expected output:

```text
Baselines
=========

  Name      Findings    Created
  v1.0      23          2026-01-15 09:30:12
  v2.0      18          2026-02-01 14:22:45
```

## Diff Against a Baseline

```bash
patchflow baseline diff --from v1.0
```

Runs a full SAST scan and reports new, resolved, and unchanged findings relative
to the named baseline. Exits with code 1 if new findings are found (for CI
blocking).

Expected output:

```text
Baseline Diff: v1.0 → current
==============================

New findings (3):
  PF-SCAN-024  HIGH    src/api/auth.py:28    SQL injection
  PF-SCAN-025  MEDIUM  src/utils/crypto.py:15  Weak crypto
  PF-SCAN-026  LOW     src/config.js:8       Hardcoded API key

Resolved findings (5):
  PF-SCAN-012  HIGH    src/old/auth.py:42    Command injection
  PF-SCAN-015  MEDIUM  src/old/utils.py:20   Path traversal
  ...

Unchanged findings (18)

Summary: 3 new, 5 resolved, 18 unchanged
Exit code: 1 (new findings detected)
```

The output shows:

- **New findings**: Present in current scan but not in baseline
- **Resolved findings**: Present in baseline but not in current scan
- **Unchanged findings**: Present in both

## Delete a Baseline

```bash
patchflow baseline delete --name v1.0
```

Removes the baseline file. This operation cannot be undone.

## Using Baselines in Scans

```bash
# Only report new findings
patchflow scan run --new-only --baseline v1.0

# Block CI on new high or critical findings
patchflow scan run --new-only --baseline v1.0 --fail-on high
```

`--new-only` requires `--baseline`. When both are specified, only findings not
in the baseline are reported. Combined with `--fail-on`, this creates a
practical CI gate that blocks on new issues without blocking on historical
findings.

## Recommended Baseline Workflow

1. **Run a full scan** on the default branch:
   ```bash
   patchflow scan run --profile deep
   ```

2. **Create a baseline**:
   ```bash
   patchflow baseline create --name v1.0
   ```

3. **Commit the baseline** to the repository (or store as a CI artifact):
   ```bash
   git add .patchflow/baselines/v1.0.json
   git commit -m "chore: add PatchFlow baseline v1.0"
   ```

4. **Gate pull requests** on new high or critical findings:
   ```bash
   patchflow scan run --new-only --baseline v1.0 --fail-on high
   ```

5. **Schedule separate work** to burn down old findings in the baseline.

6. **Refresh the baseline** periodically (e.g., after fixing historical
   findings):
   ```bash
   patchflow baseline create --name v2.0
   ```

## Legacy Baseline Commands

For backward compatibility, the legacy subcommand is also available:

```bash
# Legacy usage (still supported)
patchflow scan baseline create v1.0
patchflow scan baseline compare v1.0
patchflow scan baseline list
patchflow scan baseline delete v1.0
```

The modern `patchflow baseline` commands are preferred.

## CI Integration

See [GitHub Actions](../integrations/github-actions.md) and
[GitLab CI](../integrations/gitlab.md) for baseline-aware CI pipeline examples.

## Next Steps

- [Reports](./reports.md) — Generate reports from scans
- [Recommended Workflow](../workflows/recommended.md) — Team adoption strategy
- [GitHub Actions](../integrations/github-actions.md) — CI integration
