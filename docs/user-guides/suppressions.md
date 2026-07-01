---
title: Suppressions
description: Suppress false positives with patchflow:ignore directives
---

# Suppressions

Suppress false positives with `//patchflow:ignore` directives. These comments
tell PatchFlow to skip a specific finding at a specific location.

## Suppression Syntax

The comment syntax adapts to the file type:

### `#` comment syntax

Used for Python, Ruby, Shell, YAML, TOML, Dockerfile, Terraform:

```python
# patchflow:ignore PY001 -- eval is safe here, input is sanitized
result = eval(user_input)
```

```ruby
# patchflow:ignore PF-RAILS-XSS-001 -- output is sanitized
raw(params[:content])
```

```yaml
# patchflow:ignore TF001 -- container runs in isolated network
ports:
  - "8080:8080"
```

### `//` comment syntax

Used for Go, JavaScript, TypeScript, Java, C/C++, C#, Rust, Swift, Kotlin, Scala,
PHP:

```go
//patchflow:ignore G404 -- using math/rand for non-security purpose
n := rand.Intn(100)
```

```javascript
//patchflow:ignore TS-JS004 -- validated input
element.innerHTML = userInput;
```

## Suppression Format

```
COMMENT_PREFIX patchflow:ignore RULE_ID [-- REASON]
```

- `COMMENT_PREFIX`: `#` or `//` depending on file type
- `RULE_ID`: The rule ID to suppress (e.g., `PY001`, `G404`)
- `-- REASON`: Optional but recommended justification for the suppression

The suppression directive must be on the line **before** the finding.

## Adding Suppressions Programmatically

Use the `patchflow suppress` command to add suppression directives:

```bash
patchflow suppress PY001 --file src/app.py --line 42 --reason "eval is safe, input is sanitized"
```

### Flags

| Flag | Type | Required | Description |
| --- | --- | --- | --- |
| `--file` | string | Yes | File to add suppression to |
| `--line` | int | Yes | Line number of the finding |
| `--reason` | string | Yes | Justification for suppression |

The rule ID is passed as a positional argument:

```bash
patchflow suppress RULE_ID --file PATH --line N --reason TEXT
```

### Behavior

- Inserts the suppression directive on the line before the specified line number
- Automatically detects the correct comment syntax for the file type
- Validates the file path to prevent path traversal (must be within the repo root)
- Checks for existing suppressions nearby (within 3 lines) to avoid duplicates

## Viewing Suppressed Findings

By default, suppressed findings are hidden from scan output. To include them:

```bash
patchflow scan run --show-suppressed
```

Suppressed findings are marked as suppressed in the output.

## Best Practices

1. **Always include a reason**: The `-- REASON` portion documents why the
   finding is suppressed. This helps future reviewers understand the decision.

2. **Be specific with rule IDs**: Suppress only the specific rule that is a false
   positive, not all rules at that location.

3. **Review suppressions periodically**: Use `patchflow scan run
   --show-suppressed` to review all suppressions and verify they are still
   valid.

4. **Prefer fixing over suppressing**: If a finding is real, fix it. Only
   suppress findings that are genuinely false positives or have compensating
   controls.

5. **Track suppressions in code review**: Suppression directives are code. They
   should be reviewed like any other change.

## How Suppressions Work Internally

When PatchFlow scans a file, it checks for `patchflow:ignore` directives on the
line before each finding. If a directive matches the finding's rule ID, the
finding is marked as suppressed and excluded from the default output.

The suppression matching is:

- Rule ID must match exactly (e.g., `PY001` suppresses only `PY001` findings)
- The directive must be on the line immediately before the finding
- The directive can be on the same line as the finding for single-line patterns

## Next Steps

- [Explain](./explain.md) — Understand findings before suppressing
- [Custom Rules](./custom-rules.md) — Reduce false positives with sanitizers
- [Scan Your Project](./scan.md) — Run scans with `--show-suppressed`
