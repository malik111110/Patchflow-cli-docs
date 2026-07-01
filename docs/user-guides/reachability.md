---
title: Reachability Analysis
description: Determine whether vulnerable dependencies are actually used
---

# Reachability Analysis

Reachability analysis determines whether a vulnerable dependency is actually
imported and used in your codebase. This helps prioritize which vulnerabilities
to fix first — a critical vulnerability in an unused dependency is less urgent
than a high vulnerability in a directly imported package.

## How Reachability Works

1. PatchFlow parses import statements for Python, Go, and JavaScript/TypeScript
   source files.
2. It builds an import graph mapping source files to imported packages.
3. For each vulnerable dependency, it checks whether the package name appears in
   the import graph.
4. It assigns a confidence level based on how directly the package is used.

## Confidence Levels

| Level | Meaning | Action |
| --- | --- | --- |
| `HIGH` | Directly imported or invoked | Fix immediately |
| `MEDIUM` | Direct dependency, possible runtime usage | Fix soon |
| `LOW` | Transitive dependency, no direct usage found | Fix when possible |
| `NONE` | Not present in dependency graph | Low priority |
| `UNKNOWN` | Analysis incomplete | Investigate manually |

## Check a Specific Package

```bash
patchflow reachability --package lodash
```

Checks whether the specified package is reachable in the codebase. Shows the
confidence level and a summary of the assessment.

Expected output:

```text
Reachability Analysis
=====================

Package:     lodash
Version:     4.17.20
Ecosystem:   npm
Vulnerabilities: 1 (CVE-2021-23337, HIGH)

Reachability: REACHABLE (HIGH confidence)

Evidence:
  src/utils/format.js:3   const _ = require('lodash');
  src/utils/format.js:7   _.template(userInput)
  src/api/handler.js:12   import { merge } from 'lodash';

Imported in 2 files. Direct usage detected.
```

## Check by CVE

```bash
patchflow reachability --cve CVE-2021-23337
```

Finds the package associated with the specified CVE (by running SCA first), then
checks its reachability.

Expected output:

```text
Reachability Analysis
=====================

CVE:         CVE-2021-23337
Package:     lodash
Version:     4.17.20
Severity:    HIGH

Reachability: REACHABLE (HIGH confidence)

Evidence:
  src/utils/format.js:3   const _ = require('lodash');
  src/utils/format.js:7   _.template(userInput)
```

## Show Evidence

```bash
patchflow reachability --package lodash --explain
```

Shows the import evidence — which files import the package and how it is used.
This helps you verify that the reachability assessment is correct.

The `--explain` flag shows the evidence behind the reachability assessment:

- Which source files import the package
- Import statements that were found
- Why the confidence level was assigned

This is essential for validating reachability assessments and understanding
false negatives.

## JSON Output

```bash
patchflow reachability --package lodash --json
```

Outputs the reachability assessment as structured JSON for automation and
dashboards.

## Reachability in Scans

Reachability analysis runs automatically as part of `patchflow scan run` and
`patchflow pr-review`. The results are included in all report formats and
contribute to the risk score.

To skip reachability analysis:

```bash
patchflow scan run --no-reachability
patchflow pr-review --no-reachability
```

## Language Support

Import graph parsing is supported for:

| Language | Import Syntax |
| --- | --- |
| Python | `import`, `from ... import` |
| Go | `import` |
| JavaScript/TypeScript | `import`, `require()` |

Other languages do not have import graph parsing. Vulnerable dependencies in
unsupported languages will receive `UNKNOWN` reachability.

## Next Steps

- [Dependencies](./dependencies.md) — List and analyze dependencies
- [Scan Your Project](./scan.md) — Full security analysis with reachability
- [Reports](./reports.md) — Include reachability in reports
