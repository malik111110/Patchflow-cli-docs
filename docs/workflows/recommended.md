# Recommended Workflow

Use PatchFlow in layers. Start with local scans, add baselines, then enforce CI
gates once the signal is stable. This phased approach avoids blocking teams on
historical findings while building toward a strong security posture.

## Phase 1: Developer Loop

Run PatchFlow locally before opening a pull request. Treat local scans as fast
feedback — do not block local work on every medium or low finding.

```bash
# Initialize (once per repository)
patchflow init

# Quick scan before committing
patchflow scan run --profile quick

# Explain any unexpected findings
patchflow explain --rule <rule-id>

# Suppress false positives
patchflow suppress <rule-id> --file <path> --line <n> --reason "explanation"
```

**Goal:** Developers get fast feedback and learn to read PatchFlow output. No CI
blocking yet.

## Phase 2: Pull Request Loop

Add PatchFlow to CI on pull requests. Generate SARIF for code scanning and a
Markdown report as an artifact. Do not fail CI yet — just collect signal.

```bash
# In CI, on pull requests
patchflow scan run --profile standard --format sarif --output patchflow.sarif
patchflow report --format markdown --output patchflow-report.md
```

Upload SARIF to GitHub code scanning and publish the Markdown report as an
artifact. Review the findings and triage false positives.

**Goal:** CI produces consistent PatchFlow output. Teams review findings without
CI blocking.

## Phase 3: Baseline Adoption

For existing repositories with historical findings, create a baseline so CI
focuses on new issues only.

```bash
# On the default branch, run a full scan
patchflow scan run --profile deep

# Create a baseline
patchflow baseline create --name v1.0

# Commit the baseline
git add .patchflow/baselines/v1.0.json
git commit -m "chore: add PatchFlow baseline v1.0"
```

Now CI can compare against the baseline:

```bash
patchflow scan run --new-only --baseline v1.0 --fail-on high
```

This blocks CI only on new high or critical findings, avoiding blocking on
historical issues.

**Goal:** CI blocks on new high/critical findings. Historical findings are
tracked separately for burn-down.

## Phase 4: Enforce CI Gates

Once the baseline is stable and false positives are suppressed, enforce CI gates
on protected branches.

```bash
# Protected branch CI
patchflow scan run --new-only --baseline v1.0 --fail-on high --format sarif --output patchflow.sarif
```

**Goal:** CI enforces security quality on new changes. Old findings are
burned down over time.

## Phase 5: Deep Audit

Schedule periodic deep audits (e.g., weekly) to review the full finding surface,
including experimental rules.

```bash
# Weekly audit scan
patchflow scan run --profile deep --governance-profile audit --format json --output audit-$(date +%Y-%m-%d).json
```

**Goal:** Comprehensive security review with all rules, including experimental
ones. Identifies issues before they become critical.

## Pre-PR Review

Before opening a PR, run `pr-review` for a local risk assessment:

```bash
patchflow pr-review
```

For richer output:

```bash
patchflow pr-review --suggest-reviewers --suggest-fixes --format pr-summary
```

This gives you:

- A risk score and level
- Findings scoped to your changes
- Reviewer suggestions
- Fix proposals
- A PR-ready summary

## Team Workflow Summary

| Stage | Command | When | Blocking |
| --- | --- | --- | --- |
| Developer | `patchflow scan run --profile quick` | Before commit | No |
| Pre-PR | `patchflow pr-review` | Before opening PR | No |
| PR CI | `patchflow scan run --profile standard` | On PR | No (Phase 2) |
| PR CI + baseline | `patchflow scan run --new-only --baseline v1.0 --fail-on high` | On PR | Yes (Phase 4) |
| Protected branch | `patchflow scan run --new-only --baseline v1.0 --fail-on high` | On push to main | Yes |
| Audit | `patchflow scan run --profile deep --governance-profile audit` | Weekly | No |

## CI/CD Integration

PatchFlow integrates with major CI/CD platforms:

- [GitHub Actions](../integrations/github-actions.md)
- [GitLab CI](../integrations/gitlab.md)
- [Jenkins](../integrations/jenkins.md)
- [Azure DevOps](../integrations/azure-devops.md)
- [pre-commit](../integrations/pre-commit.md)

Use `patchflow init <platform>` to generate CI configuration:

```bash
patchflow init github-actions
patchflow init gitlab-ci
patchflow init jenkins
patchflow init azure-devops
patchflow init pre-commit
```

## Next Steps

- [GitHub Actions](../integrations/github-actions.md) — CI integration
- [Baselines](../user-guides/baselines.md) — Baseline management
- [PR Review](../user-guides/pr-review.md) — Pre-PR risk assessment
