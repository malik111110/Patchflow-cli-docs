# Recommended Workflow

Use PatchFlow in layers. Start with local scans, add baselines, then enforce CI
gates once the signal is stable.

## Developer Loop

Run before opening a pull request:

```bash
patchflow init
patchflow scan run --profile quick
patchflow explain --rule <rule-id>
```

Developers should treat local scans as fast feedback. Do not block local work
on every medium or low finding.

## Pull Request Loop

Run standard scans in CI:

```bash
patchflow scan run --profile standard --format sarif --output patchflow.sarif
```

Upload SARIF and publish a Markdown report as an artifact.

## Baseline Adoption

For existing repositories:

```bash
patchflow scan run --profile deep
patchflow baseline create default
patchflow scan run --baseline default --new-only --fail-on high
```

This avoids blocking teams on historical findings while still preventing new
critical issues from landing.

## Framework Pack Workflow

1. Run `patchflow rules list-frameworks` to inspect detected packs.
2. Let auto-detection run first.
3. Add explicit `frameworks.enabled` only for monorepos or unusual layouts.
4. Add custom sanitizers after triage, not before.
5. Promote framework rules through maturity only after fixtures and real-repo benchmarks pass.

## CI Gate Policy

| Stage | Command Shape | Gate |
| --- | --- | --- |
| Local | `patchflow scan run --profile quick` | No hard fail |
| Pull request | `patchflow scan run --profile standard --new-only` | Fail on critical |
| Protected branch | `patchflow scan run --profile standard --new-only --fail-on high` | Fail on high |
| Scheduled audit | `patchflow scan run --profile deep` | No hard fail, review report |

## Triage Policy

Use the same labels in issues and pull requests:

| Label | Meaning |
| --- | --- |
| `fix-now` | Real exploitable path or exposed secret |
| `needs-owner` | Code owner must confirm data flow or business impact |
| `baseline` | Existing accepted finding tracked outside PR scope |
| `false-positive` | Rule needs sanitizer, exclusion, or pack improvement |
