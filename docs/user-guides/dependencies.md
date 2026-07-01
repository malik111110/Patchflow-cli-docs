# Dependencies

The `patchflow deps` command analyzes project dependencies: list them, find
vulnerable ones, show changes against a base branch, display the dependency
tree, and check licenses.

## List All Dependencies

```bash
patchflow deps list
```

Lists all dependencies with name, version, ecosystem, direct/transitive status,
and manifest path. Dependencies are grouped by ecosystem (Go, npm, PyPI, Maven,
RubyGems, Packagist, Cargo).

Use `--json` for machine-readable output:

```bash
patchflow deps list --json
```

## Find Vulnerable Dependencies

```bash
patchflow deps vulnerable
```

Queries OSV.dev for each dependency and lists vulnerabilities with:

- Dependency name and version
- Vulnerability ID (CVE or OSV ID)
- Severity
- Affected version range
- Fixed version

Use `--json` for machine-readable output:

```bash
patchflow deps vulnerable --json
```

## Show Dependency Changes

```bash
patchflow deps diff
```

Shows dependency changes against the base branch by identifying changed manifest
files via git diff. Useful for PR review to see what dependencies were added,
removed, or updated.

## Dependency Tree

```bash
patchflow deps tree
```

Shows the dependency tree grouped by ecosystem and manifest. Direct dependencies
are marked with `*`. The tree shows the hierarchy of dependencies within each
ecosystem.

Use `--json` for machine-readable output:

```bash
patchflow deps tree --json
```

## Check Licenses

```bash
patchflow deps licenses
```

Extracts and classifies license information from project dependencies. Licenses
are fetched from package registries (npm, PyPI, Maven, RubyGems, Packagist) and
categorized:

| Category | Risk Level | Examples |
| --- | --- | --- |
| Permissive | Low | MIT, Apache-2.0, BSD-3-Clause, ISC |
| Weak copyleft | Medium | LGPL-2.1, MPL-2.0, EPL-2.0 |
| Copyleft | High | GPL-2.0, GPL-3.0, AGPL-3.0 |
| Proprietary | High | Commercial licenses |
| Unknown | Critical | License could not be determined |

The output shows a summary by category and risk level, with high and critical
risk licenses highlighted.

Use `--json` for machine-readable output:

```bash
patchflow deps licenses --json
```

## Supported Ecosystems

PatchFlow parses dependency manifests from the following ecosystems:

| Ecosystem | Manifest Files |
| --- | --- |
| Go | `go.mod`, `go.sum` |
| npm | `package.json`, `package-lock.json` |
| PyPI | `requirements.txt`, `pyproject.toml`, `Pipfile`, `poetry.lock`, `setup.py` |
| Maven | `pom.xml` |
| RubyGems | `Gemfile`, `Gemfile.lock` |
| Packagist | `composer.json`, `composer.lock` |
| Cargo | `Cargo.toml`, `Cargo.lock` |

## Next Steps

- [Reachability](./reachability.md) — Check if vulnerable dependencies are used
- [Scan Your Project](./scan.md) — Full security analysis
- [Container Scanning](./container-scanning.md) — Scan container images
