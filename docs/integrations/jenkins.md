# Jenkins

Use PatchFlow in Jenkins pipelines to scan code changes and archive reports.

## Generate a Pipeline Stage

```bash
patchflow init jenkins
```

This creates a `Jenkinsfile.patchflow` file (standalone, does not modify an
existing Jenkinsfile) with a PatchFlow pipeline stage that:

- Installs PatchFlow
- Runs a scan with JSON output
- Runs a scan with SARIF output
- Archives artifacts

## Generated Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('PatchFlow Security Scan') {
            steps {
                sh '''
                    go install github.com/Patchflow-security/patchflow-cli@latest
                    patchflow scan run --profile standard --format json --output patchflow-report.json
                    patchflow scan run --profile standard --format sarif --output patchflow.sarif
                    patchflow report --format markdown --output patchflow-report.md
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'patchflow.sarif,patchflow-report.md,patchflow-report.json', allowEmptyArchive: true
                }
            }
        }
    }
}
```

## Using in an Existing Pipeline

Reference the generated file from your main Jenkinsfile:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        stage('PatchFlow Security Scan') {
            steps {
                sh '''
                    go install github.com/Patchflow-security/patchflow-cli@latest
                    patchflow scan run --new-only --baseline v1.0 --fail-on high \
                        --format sarif --output patchflow.sarif
                    patchflow report --format markdown --output patchflow-report.md
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'patchflow.sarif,patchflow-report.md', allowEmptyArchive: true
                }
            }
        }
    }
}
```

## With Exit Gate

```groovy
stage('PatchFlow Security Gate') {
    steps {
        sh '''
            patchflow scan run --new-only --baseline v1.0 --fail-on high
        '''
    }
}
```

The scan exits with code 1 if new high or critical findings are found, which
fails the Jenkins stage.

## With Backend Authentication

```groovy
stage('PatchFlow Review') {
    steps {
        withCredentials([string(credentialsId: 'patchflow-token', variable: 'PATCHFLOW_TOKEN')]) {
            sh '''
                patchflow login --token "$PATCHFLOW_TOKEN"
                patchflow review pr --submit
            '''
        }
    }
}
```

Store the PatchFlow token as a Jenkins credential (secret text type).

## Prerequisites

- Go 1.25 or later installed on the Jenkins agent
- Git installed on the Jenkins agent
- Internet access for OSV.dev queries (or a local OSV database for offline mode)

## Tips

- Use `post { always { ... } }` to archive artifacts even when the scan fails
- Use Jenkins credentials for tokens — never hardcode tokens in pipeline scripts
- Use `agent { label 'go' }` to run on agents with Go pre-installed
- Consider using a custom Docker agent image with PatchFlow pre-installed for
  faster pipeline execution

## Next Steps

- [SARIF Uploads](./sarif.md) — SARIF format details
- [Baselines](../user-guides/baselines.md) — Baseline management
- [Recommended Workflow](../workflows/recommended.md) — Team adoption
