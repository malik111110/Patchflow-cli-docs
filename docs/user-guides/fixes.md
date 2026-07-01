---
title: Fixes
description: Generate and apply safe fixes for security findings
---

# Fixes

The `patchflow fix` command generates and applies safe fixes for security
findings detected by PatchFlow. Fixes are classified by confidence and strategy,
with safety controls to prevent unintended changes.

## Fix Subcommands

| Subcommand | Purpose |
| --- | --- |
| `patchflow fix suggest` | Generate fix proposals for current findings |
| `patchflow fix apply` | Apply fix proposals to source files |
| `patchflow fix show` | Show the fix proposal for a specific finding |

## Suggest Fixes

```bash
patchflow fix suggest
```

Runs a scan and generates fix proposals for all detected findings. Each proposal
includes:

- Finding ID and rule ID
- Severity
- File path and line range
- Original code
- Fixed code
- Unified patch
- Confidence level
- Fix strategy
- Rationale
- Whether the fix is auto-applicable

Expected output (abbreviated):

```text
Fix Proposals (5 found)
========================

1. PF-SCAN-001  PY001  HIGH  (auto-applicable)
   File:    src/app.py:42
   Strategy: replace
   Confidence: high

   Original:
     result = eval(user_input)

   Fixed:
     import ast
     result = ast.literal_eval(user_input)

2. PF-SCAN-002  PY004  MEDIUM  (auto-applicable)
   File:    src/subprocess.py:15
   Strategy: wrap
   Confidence: high

   Original:
     subprocess.call(cmd, shell=True)

   Fixed:
     subprocess.call(cmd, shell=False)

3. PF-SCAN-003  JS003  MEDIUM  (needs review)
   File:    src/render.js:28
   Strategy: replace
   Confidence: medium

   Original:
     element.innerHTML = userInput;

   Fixed:
     element.textContent = userInput;

Summary: 3 auto-applicable, 2 needs review
```

### Filter by Severity

```bash
patchflow fix suggest --severity high
```

Only suggests fixes for findings at or above the specified severity. Supported
values: `low`, `medium`, `high`, `critical`.

### Auto-Applicable Only

```bash
patchflow fix suggest --auto-only
```

Only shows fixes that are safe to apply without human review. These are
deterministic transformations with high confidence.

### Write to File

```bash
patchflow fix suggest --output fixes.json
```

Writes proposals to a JSON file for automation and CI pipelines.

## Apply Fixes

```bash
patchflow fix apply
```

Applies fix proposals to source files. By default, shows a preview and asks for
confirmation before applying.

### Apply Specific Fixes

```bash
# Apply fix for a specific finding ID
patchflow fix apply --finding FINDING_ID

# Apply fixes for a specific rule ID
patchflow fix apply --rule PY001

# Apply fixes in a specific file
patchflow fix apply --file src/app.py

# Apply fix at a specific line
patchflow fix apply --file src/app.py --line 42
```

### Apply All Auto-Applicable Fixes

```bash
patchflow fix apply --all
```

Applies all fixes marked as auto-applicable. Use with `--yes` for CI pipelines.

### Dry Run

```bash
patchflow fix apply --dry-run
```

Previews the changes without applying them. Shows the original code, fixed code,
and patch for each proposal.

### Create Backups

```bash
patchflow fix apply --backup
```

Creates backup files before applying fixes. Backups are saved with a `.bak`
extension.

### Skip Confirmation

```bash
patchflow fix apply --yes
```

Skips the confirmation prompt. Use in CI pipelines or when applying fixes
programmatically. Always use with `--dry-run` first to review changes.

### Filter by Severity

```bash
patchflow fix apply --severity high
```

Only applies fixes for findings at or above the specified severity.

## Show a Fix

```bash
patchflow fix show FINDING_ID
```

Shows the fix proposal for a specific finding without applying it.

```bash
# Show fix at a specific location
patchflow fix show --file src/app.py --line 42

# Show fix for a specific rule
patchflow fix show --rule PY001
```

## Fix Confidence Levels

| Confidence | Meaning | Action |
| --- | --- | --- |
| `high` | Deterministic transformation | Safe to auto-apply |
| `medium` | Pattern-based, may need review | Review before applying |
| `low` | Suggestion only | Requires manual review and adaptation |

## Fix Strategies

| Strategy | Description | Example |
| --- | --- | --- |
| `replace` | Replace vulnerable code with safe alternative | `eval(x)` → `ast.literal_eval(x)` |
| `wrap` | Wrap existing code with validation | Add input validation before SQL query |
| `remove` | Remove the vulnerable code | Remove hardcoded secret |
| `upgrade` | Upgrade dependency to fixed version | Bump package version |
| `configure` | Change configuration | Disable debug mode |
| `add_validation` | Add input validation | Add allowlist check before redirect |

## Covered Vulnerability Patterns

Fix templates cover common vulnerability patterns:

- **eval/exec with user input** (Python, JS)
- **SQL injection** (string formatting → parameterized queries)
- **Command injection** (`shell=True` → `shell=False` with argument lists)
- **Weak crypto** (MD5/SHA1 → SHA-256 or bcrypt)
- **Hardcoded secrets** (literal → environment variable)
- **Debug mode enabled** (`debug=True` → `debug=False`)
- **TLS verification disabled** (`verify=False` → `verify=True`)
- **Path traversal** (unvalidated paths → validated paths)

## Safety Model

- **Never applies without confirmation** unless `--yes` is specified
- **Always shows a preview** before applying (unless `--yes`)
- **Creates backups** with `--backup`
- **Dry-run mode** for CI pipelines (`--dry-run`)
- **Auto-applicable flag** indicates safe-to-apply fixes (high confidence,
  deterministic transformation)

## Using Fixes in CI

```bash
# Generate fix proposals as an artifact
patchflow fix suggest --output fixes.json --severity high

# Apply auto-applicable fixes in CI (with dry-run first)
patchflow fix apply --all --auto-only --dry-run
patchflow fix apply --all --auto-only --yes --backup
```

## Using Fixes with Scans

`patchflow scan run --suggest-fixes` and `patchflow pr-review --suggest-fixes`
include fix proposals in the scan output. Use the `fix` command to apply them.

## Next Steps

- [Explain](./explain) — Understand findings before fixing
- [Suppressions](./suppressions) — Suppress false positives
- [Scan Your Project](./scan) — Run scans with `--suggest-fixes`
