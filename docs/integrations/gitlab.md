# GitLab CI

Use PatchFlow in GitLab CI to scan merge requests and publish artifacts.

## Minimal Job

```yaml
patchflow:
  image: golang:1.25
  stage: test
  script:
    - go install github.com/patchflow/patchflow-cli@latest
    - patchflow scan run --profile standard --format sarif --output patchflow.sarif
    - patchflow report --format markdown --output patchflow-report.md
  artifacts:
    when: always
    paths:
      - patchflow.sarif
      - patchflow-report.md
```

## Exit Gate

```yaml
patchflow:
  image: golang:1.25
  stage: test
  script:
    - go install github.com/patchflow/patchflow-cli@latest
    - patchflow scan run --baseline default --new-only --fail-on high
```

Adopt the gate after creating a baseline. For first rollout, run without
`--fail-on` and review the report.

## Authenticated Review

```yaml
patchflow-review:
  image: golang:1.25
  stage: test
  script:
    - go install github.com/patchflow/patchflow-cli@latest
    - patchflow login --token "$PATCHFLOW_TOKEN"
    - patchflow review pr --submit
  variables:
    PATCHFLOW_TOKEN: $PATCHFLOW_TOKEN
```

Store `PATCHFLOW_TOKEN` as a protected masked CI/CD variable.
