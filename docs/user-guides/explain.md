# Explain

The `patchflow explain` command explains a security finding in detail. It shows
what the issue is, where the evidence is, how to fix it, and how to suppress it
if it is a false positive.

## Usage

### Explain by Rule ID

```bash
patchflow explain --rule PY001
patchflow explain --rule PF-EXPRESS-REDIRECT-001
```

Shows the rule definition, including severity, confidence, CWE/OWASP mapping,
fix hints, and suppression format. For framework rules, it also shows sources,
sinks, sanitizers, safe patterns, and exclusions.

### Explain by Finding ID

```bash
patchflow explain <finding-id>
```

The finding ID is shown in `patchflow scan run` output. This runs a scan to
locate the finding and shows:

- Finding details (rule ID, severity, file, line)
- Evidence (code snippet at the finding location)
- Fix hints and fix snippets (if available)
- Suppression information
- Whether the finding would block a PR (based on `--fail-on` thresholds)

### Explain by Location

```bash
patchflow explain --file src/app.py --line 42
```

Explains a finding at a specific file and line. Useful when you know where a
finding is but want the full explanation.

## What Explain Shows

### For Framework Rules

- Rule ID, title, severity, confidence
- Framework and language
- CWE and OWASP mapping
- Match mode (pattern, AST, taint, template)
- Maturity level
- Recommendation
- **Sources** (for taint rules): Where tainted data enters
- **Sinks** (for taint rules): Where tainted data is used dangerously
- **Sanitizers**: Functions or patterns that neutralize the taint
- **Safe patterns**: Patterns that indicate safe usage
- **Exclusions**: Paths or patterns excluded from the rule
- Suppression format

### For Core Scanner Rules

- Rule ID, title, severity
- Scanner engine and language
- Fix hint
- Fix snippet (if available) — shows vulnerable code and fixed code side by side
- Suppression format

### For Findings (by ID or location)

- Finding details
- Evidence (file, line, code snippet)
- Fix hints
- Fix snippet (if available)
- Suppression information
- Whether it would block a PR

## JSON Output

```bash
patchflow explain --rule PY001 --json
```

Outputs the explanation as structured JSON for automation and tooling.

## Complete Flag Reference

| Flag | Type | Default | Description |
| --- | --- | --- | --- |
| `--rule` | string | (none) | Rule ID to explain (e.g., `PY001`, `PF-EXPRESS-REDIRECT-001`) |
| `--file` | string | (none) | Explain finding at this file path |
| `--line` | int | 0 | Line number (use with `--file`) |

The finding ID can also be passed as a positional argument:

```bash
patchflow explain <finding-id>
```

## Examples

### Explain a Python eval finding

```bash
patchflow explain --rule PY001
```

Output includes:

```
Rule:       PY001
Title:      Dangerous eval() usage
Severity:   high
Confidence: high
Scanner:    patterns-embedded
Language:   python
CWE:        CWE-95 (Improper Evaluation of Code with Untrusted Input)
OWASP:      A03:2021 - Injection

Description:
  eval() executes arbitrary Python code. If the input comes from an untrusted
  source, this can lead to arbitrary code execution.

Fix hint:
  Use ast.literal_eval() for safe literal evaluation, or parse input with
  json.loads() if the input is JSON.

Fix snippet:
  # Vulnerable
  result = eval(user_input)

  # Fixed
  import ast
  result = ast.literal_eval(user_input)

Suppression:
  # patchflow:ignore PY001 -- eval is safe here, input is sanitized
  result = eval(user_input)
```

### Explain a framework taint rule

```bash
patchflow explain --rule PF-RAILS-SQLI-003
```

Output includes sources, sinks, and sanitizers for the taint rule, showing how
data flows from request parameters to SQL queries.

## Next Steps

- [Suppressions](./suppressions.md) — Suppress false positives
- [Fixes](./fixes.md) — Apply fixes to findings
- [Framework Packs](./framework-packs.md) — Framework rule reference
