# Azure DevOps

Use PatchFlow in Azure DevOps pipelines to scan code changes and publish
reports.

## Generate a Pipeline

```bash
patchflow init azure-devops
```

This creates an `azure-pipelines-patchflow.yml` file (standalone) with PatchFlow
pipeline tasks that:

- Install PatchFlow
- Run a scan with JSON output
- Run a scan with SARIF output
- Publish the JSON report as a build artifact

## Generated Pipeline

```yaml
trigger:
  - main

pr:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: Bash@3
    displayName: 'PatchFlow Security Scan'
    inputs:
      targetType: 'inline'
      script: |
        go install github.com/Patchflow-security/patchflow-cli@latest
        patchflow scan run --profile standard --format json --output patchflow-report.json
        patchflow scan run --profile standard --format sarif --output patchflow.sarif
        patchflow report --format markdown --output patchflow-report.md

  - task: PublishBuildArtifacts@1
    displayName: 'Publish PatchFlow Reports'
    inputs:
      PathtoPublish: |
        patchflow-report.json
        patchflow.sarif
        patchflow-report.md
      ArtifactName: 'patchflow'
      publishLocation: 'Container'
```

## Using in an Existing Pipeline

Add the PatchFlow tasks to your existing `azure-pipelines.yml`:

```yaml
steps:
  - task: GoTool@0
    inputs:
      version: '1.25'

  - task: Bash@3
    displayName: 'PatchFlow Security Scan'
    inputs:
      targetType: 'inline'
      script: |
        go install github.com/Patchflow-security/patchflow-cli@latest
        patchflow scan run --new-only --baseline v1.0 --fail-on high \
          --format sarif --output patchflow.sarif
        patchflow report --format markdown --output patchflow-report.md

  - task: PublishBuildArtifacts@1
    displayName: 'Publish PatchFlow Reports'
    inputs:
      PathtoPublish: 'patchflow.sarif,patchflow-report.md'
      ArtifactName: 'patchflow'
      publishLocation: 'Container'
```

## With Exit Gate

```yaml
- task: Bash@3
  displayName: 'PatchFlow Security Gate'
  inputs:
    targetType: 'inline'
    script: |
      patchflow scan run --new-only --baseline v1.0 --fail-on high
```

The scan exits with code 1 if new high or critical findings are found, which
fails the Azure DevOps pipeline task.

## With Backend Authentication

```yaml
- task: Bash@3
  displayName: 'PatchFlow Review'
  env:
    PATCHFLOW_TOKEN: $(PATCHFLOW_TOKEN)
  inputs:
    targetType: 'inline'
    script: |
      patchflow login --token "$PATCHFLOW_TOKEN"
      patchflow review pr --submit
```

Store `PATCHFLOW_TOKEN` as a secret variable in Azure DevOps pipeline variables.

## SARIF in Azure DevOps

Azure DevOps supports SARIF results via the
[SARIF Viewer extension](https://marketplace.visualstudio.com/items?itemName=sariftools.sarif-viewer).
Publish the SARIF file as a build artifact and use the extension to view results
in the pipeline UI.

## Prerequisites

- Go 1.25 or later installed on the build agent (or use the `GoTool@0` task)
- Git installed on the build agent
- Internet access for OSV.dev queries (or a local OSV database for offline mode)

## Tips

- Use the `GoTool@0` task to ensure the correct Go version is installed
- Use secret variables for tokens — never hardcode tokens in pipeline scripts
- Use `continueOnError: true` during the observe phase to collect findings
  without failing the pipeline
- Publish SARIF as a build artifact for the SARIF Viewer extension

## Next Steps

- [SARIF Uploads](./sarif.md) — SARIF format details
- [Baselines](../user-guides/baselines.md) — Baseline management
- [Recommended Workflow](../workflows/recommended.md) — Team adoption
