---
title: GitLab CI
description: Use PatchFlow in GitLab CI for merge request scanning
---

# GitLab CI

Use PatchFlow in GitLab CI to scan merge requests, publish artifacts, and enforce
security gates.

## Generate a Job

PatchFlow can generate a GitLab CI job for you:

```bash
patchflow init gitlab-ci
```

This appends a PatchFlow job to `.gitlab-ci.yml` that:

- Runs on merge requests
- Installs PatchFlow
- Runs a scan with JSON and SARIF output
- Publishes artifacts (SARIF file, JSON report)

## Minimal Job

```yaml
patchflow:
  image: golang:1.25
  stage: test
  script:
    - go install github.com/Patchflow-security/patchflow-cli@latest
    - patchflow scan run --profile standard --format sarif --output patchflow.sarif
    - patchflow report --format markdown --output patchflow-report.md
  artifacts:
    when: always
    paths:
      - patchflow.sarif
      - patchflow-report.md
```

## With Exit Gate

After creating a baseline (see [Baselines](../user-guides/baselines)):

```yaml
patchflow:
  image: golang:1.25
  stage: test
  script:
    - go install github.com/Patchflow-security/patchflow-cli@latest
    - patchflow scan run --new-only --baseline v1.0 --fail-on high
```

Adopt the gate after creating a baseline. For first rollout, run without
`--fail-on` and review the report.

## With SARIF and JSON Reports

```yaml
patchflow:
  image: golang:1.25
  stage: test
  script:
    - go install github.com/Patchflow-security/patchflow-cli@latest
    - patchflow scan run --profile standard --format json --output patchflow-report.json
    - patchflow scan run --profile standard --format sarif --output patchflow.sarif
    - patchflow report --format markdown --output patchflow-report.md
  artifacts:
    when: always
    reports:
      dotenv: patchflow-report.json
    paths:
      - patchflow.sarif
      - patchflow-report.md
      - patchflow-report.json
    expire_in: 1 week
```

## Authenticated Review

For backend-connected review commands:

```yaml
patchflow-review:
  image: golang:1.25
  stage: test
  script:
    - go install github.com/Patchflow-security/patchflow-cli@latest
    - patchflow login --token "$PATCHFLOW_TOKEN"
    - patchflow review pr --submit
  variables:
    PATCHFLOW_TOKEN: $PATCHFLOW_TOKEN
```

Store `PATCHFLOW_TOKEN` as a protected, masked CI/CD variable in GitLab project
settings.

## Complete Production Job

```yaml
patchflow:
  image: golang:1.25
  stage: test
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
  script:
    - go install github.com/Patchflow-security/patchflow-cli@latest
    - |
      if [ "$CI_PIPELINE_SOURCE" = "merge_request_event" ]; then
        patchflow scan run \
          --new-only --baseline v1.0 \
          --fail-on high \
          --format sarif --output patchflow.sarif
      else
        patchflow scan run \
          --profile deep --governance-profile audit \
          --format sarif --output patchflow.sarif
      fi
    - patchflow report --format markdown --output patchflow-report.md
  artifacts:
    when: always
    paths:
      - patchflow.sarif
      - patchflow-report.md
    expire_in: 1 week
```

## Tips

- Use `when: always` on artifacts so results are preserved even when the scan
  fails
- Use GitLab CI/CD variables for tokens — never commit tokens to the repository
- Set `expire_in` on artifacts to control storage costs
- Use rules to run different scan profiles for MRs vs default branch pushes

## Next Steps

- [SARIF Uploads](./sarif) — SARIF format details
- [Baselines](../user-guides/baselines) — Baseline management
- [Recommended Workflow](../workflows/recommended) — Team adoption
