---
title: Recommended Workflow
description: Choose the right PatchFlow workflow for your team size
---

# Recommended Workflow

PatchFlow supports two primary workflows depending on your team size and
security maturity. Choose the one that fits your situation.

## Which Workflow Should I Use?

| Workflow | Best For | CI Blocking | Baselines | Time Investment |
| --- | --- | --- | --- | --- |
| [Solo Developer](./solo-dev.md) | Individual developers, small projects | Optional | Optional | Low |
| [Team Workflow](./teams.md) | Teams of 2+, shared codebase | Phased | Required for gating | Medium |

## Solo Developer Workflow

For individual developers and small projects. Focus on fast feedback, auto-fixes,
and suppressing false positives. CI blocking is optional.

```bash
# The solo dev loop
patchflow scan run --profile quick           # Fast scan before commit
patchflow explain --rule PY001               # Understand findings
patchflow fix apply --all --auto-only --yes  # Auto-fix safe issues
patchflow suppress PY001 --file ... --reason # Suppress false positives
patchflow pr-review                          # Risk assessment before PR
```

See [Solo Developer Workflow](./solo-dev.md) for the complete guide.

## Team Workflow

For teams sharing a codebase. Uses phased adoption: observe, baseline, gate,
enforce. Baselines are required to avoid blocking on historical findings.

```text
Phase 1: Observe (no block) → Phase 2: Baseline → Phase 3: Gate on new → Phase 4: Enforce
```

See [Team Workflow](./teams.md) for the complete guide.

## CI Adoption Strategy

For a detailed phased adoption plan with metrics, tuning, and false positive
management, see [CI Adoption Strategy](./ci-adoption.md).

## Common Patterns

Regardless of team size, these patterns apply everywhere:

### Start Local, Then CI

Always start with local scans before adding CI gates. This gives you time to
understand the output, suppress false positives, and tune the configuration.

### Use Baselines For Existing Projects

If your project already has security findings, create a baseline before enabling
CI gates. This prevents blocking on historical issues while still catching new
ones.

```bash
patchflow scan run --profile deep
patchflow baseline create --name v1.0
patchflow scan run --new-only --baseline v1.0 --fail-on high
```

### Fix Before Suppressing

Prefer fixing findings over suppressing them. Use `patchflow fix suggest` to
see if a safe fix is available. Only suppress findings that are genuinely false
positives or have compensating controls.

### Use The Right Profile

| Profile | When To Use | Speed |
| --- | --- | --- |
| `quick` | Before commit, fast feedback | Fast |
| `standard` | CI, PR review | Medium |
| `deep` | Weekly audit, baseline creation | Slow |

### Upload SARIF To GitHub Code Scanning

Even without CI blocking, uploading SARIF to GitHub code scanning gives you a
persistent security dashboard:

```yaml
- uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: patchflow.sarif
```

## Next Steps

- [Solo Developer Workflow](./solo-dev.md) — Workflow for individuals
- [Team Workflow](./teams.md) — Workflow for teams
- [CI Adoption Strategy](./ci-adoption.md) — Detailed adoption guide
- [GitHub Actions](../integrations/github-actions.md) — CI integration
