# Commands

This page provides the curated command map. The generated command help is in
[Generated Commands](/reference/generated-commands).

## Core Commands

| Command | Purpose |
| --- | --- |
| `patchflow version` | Print CLI version |
| `patchflow doctor` | Check Git and local environment |
| `patchflow init` | Initialize PatchFlow project files |
| `patchflow scan run` | Run the main security scan |
| `patchflow scan export` | Export scan artifacts |
| `patchflow report` | Generate reports from latest scan data |
| `patchflow explain --rule <id>` | Explain a rule or finding |

## Rules

```bash
patchflow rules list
patchflow rules list --framework express
patchflow rules list-frameworks
patchflow rules validate --rules .patchflow/rules.yaml
patchflow rules maturity
```

## Framework Packs

```bash
patchflow scan run --framework auto
patchflow scan run --framework express --framework react
patchflow scan run --disable-framework spring-security
```

## Baselines

```bash
patchflow baseline create default
patchflow scan run --baseline default --new-only
```

## Dependencies

```bash
patchflow deps list
patchflow deps vulnerable
patchflow reachability --package <name> --explain
```

## Review

```bash
patchflow pr-review
patchflow review context
patchflow review pr --submit
patchflow review status <job-id>
```

## Authentication And Config

```bash
patchflow login --token <token>
patchflow logout
patchflow auth status
patchflow config show
patchflow config set org <org>
```

## JSON

Most commands support:

```bash
patchflow <command> --json
```

Use JSON in CI and scripts.
