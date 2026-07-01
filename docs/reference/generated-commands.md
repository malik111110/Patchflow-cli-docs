# Generated Command Reference

Generated from `patchflow --help` output. Run this after changing CLI commands:

```bash
cd /Users/digitalcenter/patchflow-cli
go build -o patchflow .
# then run the generation script
```

This page is auto-generated from the CLI binary's help output. For curated
documentation, see [Commands](./commands.md).

## `patchflow`

```text
PatchFlow CLI provides change intelligence for engineering teams.

Use this tool to scan, review, and analyze code changes in your repositories.

Usage:
  patchflow [command]

Available Commands:
  auth         Manage PatchFlow authentication
  baseline     Manage finding baselines for CI noise reduction
  benchmark    Run and report scanner benchmarks against open-source repositories
  cache        Manage the PatchFlow local cache
  completion   Generate shell completion scripts
  config       Manage PatchFlow CLI configuration
  deps         Analyze dependencies
  doctor       Check the PatchFlow CLI environment
  explain      Explain a finding with evidence, fix hints, and suppression info
  fix          Generate and apply safe fixes for security findings
  help         Help about any command
  init         Initialize PatchFlow in the current repository
  login        Authenticate with PatchFlow
  logout       Log out from PatchFlow
  pr-review    Simulate a PR risk review before opening a pull request
  reachability Analyze whether vulnerable dependencies are actually used
  report       Generate a security analysis report
  review       Review code changes
  rules        Manage and inspect SAST rules
  scan         Scan code for issues
  suppress     Add a //patchflow:ignore suppression directive to a file
  version      Print the version number of PatchFlow CLI

Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
  -h, --help               help for patchflow
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow [command] --help" for more information about a command.
```

## `patchflow version`

```text
Print the version number of PatchFlow CLI

Usage:
  patchflow version [flags]

Flags:
  -h, --help   help for version

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow doctor`

```text
Performs a series of checks to verify that the PatchFlow CLI environment is correctly configured.

Usage:
  patchflow doctor [flags]

Flags:
  -h, --help   help for doctor

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow init`

```text
Create a .patchflow/ directory with configuration, cache, baselines, and reports
subdirectories. This sets up the project for local analysis with PatchFlow CLI.

Usage:
  patchflow init [flags]
  patchflow init [command]

Available Commands:
  azure-devops   Generate an Azure DevOps pipeline snippet for PatchFlow scans
  github-actions Generate a GitHub Actions workflow for PatchFlow scans
  gitlab-ci      Generate a GitLab CI job for PatchFlow scans
  jenkins        Generate a Jenkins pipeline stage for PatchFlow scans
  pre-commit     Generate a pre-commit hook for PatchFlow scans

Flags:
  -h, --help   help for init

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow init [command] --help" for more information about a command.
```

## `patchflow scan`

```text
Scan your codebase for issues, vulnerabilities, and style violations.

Usage:
  patchflow scan [command]

Available Commands:
  baseline    Manage finding baselines for CI noise reduction (legacy)
  changed     Scan changed files
  export      Export scan results with real vulnerability findings
  image       Scan a container image for vulnerabilities and misconfigurations
  local       Scan the local repository
  run         Run a full security analysis (SCA + SAST + reachability + risk)

Flags:
  -h, --help   help for scan

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow scan [command] --help" for more information about a command.
```

## `patchflow scan run`

```text
Run a comprehensive local security analysis on the current repository.
This performs Software Composition Analysis (SCA) via OSV.dev, Static Analysis
Security Testing (SAST) via embedded scanners (Go SAST, secret scanner, multi-language
pattern scanner) plus external tools when available (gosec, bandit, semgrep, gitleaks),
reachability analysis, and computes a risk score.

No backend connection required — all analysis runs locally.
Embedded scanners require zero installation; external tools supplement when installed.

Usage:
  patchflow scan run [flags]

Flags:
      --baseline string                  Baseline name to compare against (used with --new-only)
      --changed-only                     Only analyze changed files
      --disable-framework strings        Disable specific framework rule packs (can be repeated; takes precedence over --framework)
      --fail-on string                   Fail (exit code 1) if findings at or above this severity: low, medium, high, critical
      --format string                    Output format for report file: markdown, json, sarif
      --framework strings                Enable specific framework rule packs (can be repeated, or 'auto' for detection-based activation)
      --governance-profile string        Rule governance profile: dev, pr, ci, audit (filters findings by rule maturity)
  -h, --help                             help for run
      --include-tests                    Include test files in SAST analysis
      --incremental                      Only re-scan files changed since last scan (uses global cache sast_state.json)
      --license-policy string            License policy: fail on restricted licenses (e.g., 'gpl,agpl,proprietary,unknown')
      --new-only                         Only report findings not in the baseline (requires --baseline)
      --no-gitignore                     Do not respect .gitignore patterns (scan all files)
      --no-licenses                      Skip license scanning
      --no-reachability                  Skip reachability analysis
      --no-sast                          Skip SAST analysis
      --no-secrets                       Skip secret detection
      --offline patchflow cache update   Offline mode: only use local OSV DB and cache, no API calls (requires patchflow cache update first)
      --output string                    Write report to file (stdout if omitted)
      --path strings                     Specific paths to scan (can be repeated)
      --profile string                   Scan profile: quick, standard, deep (default "standard")
      --rules string                     Path to custom rules YAML file (default: .patchflow/rules.yaml)
      --show-suppressed                  Show findings suppressed by //patchflow:ignore comments
      --since string                     Scan files changed since the given branch/commit (e.g., --since main)
      --staged                           Only scan staged files (git staged changes)
      --suggest-fixes                    Generate safe fix proposals for detected vulnerabilities
      --taint-depth int                  Maximum inter-procedural call-hop depth for taint analysis (0 disables) (default 3)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow scan local`

```text
Scan the local repository

Usage:
  patchflow scan local [flags]

Flags:
  -h, --help   help for local

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow scan changed`

```text
Scan changed files

Usage:
  patchflow scan changed [flags]

Flags:
  -h, --help   help for changed

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow scan export`

```text
Run a full security analysis and export results in SARIF or JSON format.
This performs SCA (OSV.dev), SAST (local tools), reachability analysis, and risk scoring,
then exports the findings in the specified format.

Usage:
  patchflow scan export [flags]

Flags:
      --format string   Export format (json, sarif, cyclonedx-json, spdx-json, vex-json, dep-tree, dep-dot) (default "json")
  -h, --help            help for export
      --include-vex     Include VEX statements in CycloneDX SBOM (vulnerability exploitability)
      --no-gitignore    Do not respect .gitignore patterns (scan all files)
      --output string   Output file path (stdout if omitted)
      --upload-github   Upload SARIF to GitHub Code Scanning (requires --format sarif and GITHUB_TOKEN)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow scan image`

```text
Scan a container image for OS package vulnerabilities, language dependency
vulnerabilities, and misconfigurations.

This command uses Trivy as an external analyzer. Trivy must be installed
and available in PATH.

Supported formats: json, markdown.

Examples:
  patchflow scan image nginx:1.21
  patchflow scan image myapp:latest --format json --output report.json
  patchflow scan image alpine:3.18 --timeout 5m --severities CRITICAL,HIGH

Usage:
  patchflow scan image [IMAGE] [flags]

Flags:
      --format string       Output format: json, markdown
  -h, --help                help for image
      --output string       Write report to file (stdout if omitted)
      --severities string   Comma-separated severities to include (CRITICAL,HIGH,MEDIUM,LOW,INFO)
      --timeout duration    Scan timeout (default 10m0s)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow scan baseline`

```text
Create and compare finding baselines to reduce CI noise.

A baseline stores a snapshot of known findings. Subsequent scans can compare
against the baseline to only report NEW findings, dramatically reducing
CI noise on existing codebases.

Baselines are stored in .patchflow/baselines/ as JSON files.

Prefer the top-level 'patchflow baseline' command for new scripts:
  patchflow baseline create --name v1.0
  patchflow baseline diff --from v1.0

Legacy usage:
  patchflow scan baseline create v1.0
  patchflow scan baseline compare v1.0
  patchflow scan baseline list
  patchflow scan baseline delete v1.0

Usage:
  patchflow scan baseline [command]

Available Commands:
  compare     Compare current scan findings against a baseline
  create      Create a baseline from current scan findings
  delete      Delete a saved baseline
  list        List all saved baselines

Flags:
  -h, --help   help for baseline

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow scan baseline [command] --help" for more information about a command.
```

## `patchflow deps`

```text
Analyze project dependencies: list, diff against a base branch, find vulnerable packages, or check licenses.

Usage:
  patchflow deps [command]

Available Commands:
  diff        Show dependency changes against base branch
  licenses    Check dependency licenses and classify by risk
  list        List all dependencies
  tree        Show dependency tree by ecosystem
  vulnerable  List vulnerable dependencies

Flags:
  -h, --help   help for deps

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow deps [command] --help" for more information about a command.
```

## `patchflow deps list`

```text
List all dependencies

Usage:
  patchflow deps list [flags]

Flags:
  -h, --help   help for list

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow deps vulnerable`

```text
List vulnerable dependencies

Usage:
  patchflow deps vulnerable [flags]

Flags:
  -h, --help   help for vulnerable

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow deps diff`

```text
Show dependency changes against base branch

Usage:
  patchflow deps diff [flags]

Flags:
  -h, --help   help for diff

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow deps tree`

```text
Show dependency tree by ecosystem

Usage:
  patchflow deps tree [flags]

Flags:
  -h, --help   help for tree

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow deps licenses`

```text
Extract and classify license information from project dependencies.
Licenses are categorized as permissive, weak copyleft, copyleft, proprietary, or unknown,
with risk levels (low, medium, high, critical) for policy enforcement.

Usage:
  patchflow deps licenses [flags]

Flags:
  -h, --help   help for licenses

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow reachability`

```text
Determine whether a vulnerable dependency is reachable — i.e., actually
imported and used in the codebase. This helps prioritize which vulnerabilities
to fix first.

Reachability confidence levels:
  HIGH       directly imported or invoked
  MEDIUM     direct dependency, possible runtime usage
  LOW        transitive dependency, no direct usage found
  NONE       not present in dependency graph
  UNKNOWN    analysis incomplete

Usage:
  patchflow reachability [flags]

Flags:
      --cve string       Check reachability for a specific CVE (finds the package first)
      --explain          Show evidence for the reachability assessment
  -h, --help             help for reachability
      --package string   Check reachability for a specific package

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow report`

```text
Run a full analysis and generate a report in the specified format.
Supported formats: markdown, json, sarif.

The report includes all findings (SCA, SAST, secrets), dependency list,
risk score breakdown, and recommendations.

Usage:
  patchflow report [flags]

Flags:
      --format string   Report format: markdown, json, sarif (default "markdown")
  -h, --help            help for report
      --output string   Output file path (stdout if omitted)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow baseline`

```text
Create, compare, and manage finding baselines to reduce CI noise.

A baseline stores a snapshot of known findings. Subsequent scans can compare
against the baseline to only report NEW findings, dramatically reducing
CI noise on existing codebases.

Baselines are stored under .patchflow/baselines/<name>.json and compared
using stable semantic fingerprints (rule id + scanner + normalized path +
normalized snippet) so that findings survive line-number shifts from
unrelated edits.

Examples:
  patchflow baseline create --name v1.0
  patchflow baseline list
  patchflow baseline diff --from v1.0
  patchflow baseline delete --name v1.0
  patchflow scan run --new-only --baseline v1.0

Usage:
  patchflow baseline [command]

Available Commands:
  create      Create a baseline from current scan findings
  delete      Delete a saved baseline
  diff        Diff current scan findings against a baseline
  list        List all saved baselines

Flags:
  -h, --help   help for baseline

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow baseline [command] --help" for more information about a command.
```

## `patchflow baseline create`

```text
Run a full SAST scan and store the findings as a named baseline.

Usage:
  patchflow baseline create [flags]

Flags:
  -h, --help          help for create
      --name string   Name of the baseline to create (required)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow baseline list`

```text
List all saved baselines

Usage:
  patchflow baseline list [flags]

Flags:
  -h, --help   help for list

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow baseline diff`

```text
Run a full SAST scan and report new, resolved, and unchanged findings relative to the named baseline.

Usage:
  patchflow baseline diff [flags]

Flags:
      --from string   Baseline name to diff against (required)
  -h, --help          help for diff

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow baseline delete`

```text
Delete a saved baseline

Usage:
  patchflow baseline delete [flags]

Flags:
  -h, --help          help for delete
      --name string   Name of the baseline to delete (required)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow rules`

```text
Manage and inspect security rules from the embedded SAST scanners.

Usage:
  patchflow rules [command]

Available Commands:
  docs            Generate rule documentation
  list            List all registered SAST rules
  list-frameworks List all official embedded framework rule packs
  maturity        Show rule governance maturity coverage report
  validate        Validate a custom rules YAML file

Flags:
  -h, --help   help for rules

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow rules [command] --help" for more information about a command.
```

## `patchflow rules list`

```text
List all security rules from the embedded SAST scanners.

Shows rules from:
  - gosast-embedded (Go AST-based rules ported from gosec)
  - secrets-embedded (regex-based secret detection patterns)
  - patterns-embedded (multi-language regex patterns for Python, JS/TS, Ruby, PHP)

Custom rules from .patchflow/rules.yaml are included if present.

Use --rules <path> to load custom rules from a specific file.
Use --json for machine-readable output.

Usage:
  patchflow rules list [flags]

Flags:
      --all                 Show all rules (default: summary only)
      --framework strings   Filter to one or more framework packs (repeat flag or use comma-separated values)
  -h, --help                help for list
      --json                Output in JSON format
      --rules string        Path to custom rules YAML file

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow rules list-frameworks`

```text
List all official embedded framework rule packs and their detection status.

Shows:
  - Pack name and primary language
  - File and template extensions owned by the pack
  - Number of rules in the pack
  - Whether the pack would be auto-detected in the current project

Use --json for machine-readable output.

Usage:
  patchflow rules list-frameworks [flags]

Flags:
  -h, --help   help for list-frameworks
      --json   Output in JSON format

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow rules maturity`

```text
Show a governance coverage report for all registered rules.

The report includes:
  - Maturity level distribution (experimental, beta, stable, enterprise)
  - Blocking-eligible vs excluded rule counts
  - CWE and OWASP mapping coverage
  - Profile activation counts (dev, pr, ci, audit)
  - Per-engine rule counts

Use --json for machine-readable output.
Use --all to list every rule with its maturity, CWE, and blocking status.

Usage:
  patchflow rules maturity [flags]

Flags:
      --all    List every rule with maturity and CWE
  -h, --help   help for maturity
      --json   Output in JSON format

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow rules validate`

```text
Validate the structure and contents of a custom rules YAML file.

If a path argument is provided, that file is validated. Otherwise the command
looks for .patchflow/rules.yaml in the project root.

Validation checks:
  1. File exists and is valid YAML
  2. Each rule has required fields: id, title, pattern, severity
  3. Severity is one of: low, medium, high, critical
  4. Pattern is a valid regex (compiles without error)
  5. Languages field is present and non-empty
  6. Rule IDs are unique
  7. Rule IDs match pattern ^[A-Z][A-Z0-9]{2,}-[A-Z0-9]+$

Exit code is 0 if valid, 1 if invalid.

Usage:
  patchflow rules validate [path] [flags]

Flags:
  -h, --help   help for validate

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow rules docs`

```text
Generate documentation for all registered rules.

Outputs a Markdown document with one section per rule, including:
  - Rule ID, title, severity, confidence
  - Maturity level and blocking eligibility
  - CWE and OWASP mapping
  - Security category
  - Fix recommendation
  - Active scan profiles

Use --output <file> to write to a file (stdout if omitted).
Use --engine <name> to filter to a specific engine.

Usage:
  patchflow rules docs [flags]

Flags:
      --engine string   Filter to a specific engine
  -h, --help            help for docs
      --output string   Write docs to file (stdout if omitted)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow explain`

```text
Explain a security finding in detail.

Shows:
  - What the issue is and why it's dangerous
  - Where the evidence is (file, line, code snippet)
  - How to fix it (with example code)
  - How to suppress it if it's a false positive
  - Whether it would block a PR (--fail-on)

The finding ID can be obtained from `patchflow scan run` output.
You can also use --file and --line to explain a finding at a specific location.

Usage:
  patchflow explain [finding-id] [flags]

Flags:
      --file string   Explain finding at this file path
  -h, --help          help for explain
      --line int      Line number (use with --file)
      --rule string   Rule ID to explain (e.g., PY001, TS-JS004)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow fix`

```text
Generate and apply safe fixes for security findings detected by PatchFlow.

The fix command can:
  1. Suggest fixes for findings (patchflow fix suggest)
  2. Apply fixes with dry-run preview and confirmation (patchflow fix apply)
  3. Show a specific fix proposal (patchflow fix show <id>)

Fixes are generated from built-in templates that target common vulnerability
patterns (eval, command injection, SQL injection, weak crypto, etc.).
Each fix includes a confidence score, rationale, and unified diff patch.

Safe by design:
  - Never applies without confirmation (unless --yes)
  - Always shows a preview before applying
  - Creates backups with --backup
  - Dry-run mode for CI pipelines

Usage:
  patchflow fix [command]

Available Commands:
  apply       Apply fix proposals to source files
  show        Show the fix proposal for a specific finding
  suggest     Generate fix proposals for current findings

Flags:
  -h, --help   help for fix

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow fix [command] --help" for more information about a command.
```

## `patchflow fix suggest`

```text
Run a scan and generate fix proposals for all findings that have
matching fix templates. Outputs a summary with confidence levels and
auto-applicability flags.

Usage:
  patchflow fix suggest [flags]

Flags:
      --auto-only         Only show auto-applicable fixes
  -h, --help              help for suggest
      --output string     Write proposals to file (JSON format)
      --severity string   Only suggest fixes for findings at or above this severity (low, medium, high, critical)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow fix apply`

```text
Apply fix proposals to source files. By default, shows a preview
and asks for confirmation before applying. Use --dry-run to preview
without applying, or --yes to skip confirmation (for CI).

Usage:
  patchflow fix apply [flags]

Flags:
      --all               Apply all auto-applicable fixes
      --backup            Create backups before applying fixes
      --dry-run           Preview changes without applying
      --file string       Apply fixes in a specific file
      --finding string    Apply fix for a specific finding ID
  -h, --help              help for apply
      --line int          Apply fix at a specific line (use with --file)
      --rule string       Apply fixes for a specific rule ID
      --severity string   Only apply fixes at or above this severity
      --yes               Skip confirmation prompt (for CI use)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow fix show`

```text
Show the fix proposal for a specific finding. Runs a scan to locate
the finding, then generates and displays the proposed fix with diff.

Usage:
  patchflow fix show [finding-id] [flags]

Flags:
      --file string   Show fix for finding at this file
  -h, --help          help for show
      --line int      Line number (use with --file)
      --rule string   Rule ID to show fix for

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow suppress`

```text
Add a suppression directive to ignore a specific finding.

This inserts a // patchflow:ignore comment at the specified line,
which tells the scanner to skip that finding in future scans.

Examples:
  patchflow suppress PY001 --file app.py --line 42 --reason "safe eval of trusted config"
  patchflow suppress TS-JS004 --file src/app.tsx --line 100

Usage:
  patchflow suppress [rule-id] --file [path] --line [n] [flags]

Flags:
      --file string     File to add suppression to (required)
  -h, --help            help for suppress
      --line int        Line number of the finding (required)
      --reason string   Justification for suppression (required)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow pr-review`

```text
Analyze your current branch changes and compute a risk score, vulnerability
findings, reachability data, and recommendations — all locally, before you open a PR.

This is the PatchFlow "is this change safe?" command. It runs SCA (OSV.dev),
SAST (local tools), reachability analysis, and risk scoring, then produces a
terminal summary or a report file (markdown/json/sarif).

Use --suggest-reviewers to get CODEOWNERS and git blame based reviewer suggestions.
Use --annotations to generate inline code annotations for the PR diff.
Use --suggest-fixes to generate safe fix proposals for detected vulnerabilities.

Usage:
  patchflow pr-review [flags]

Flags:
      --annotations         Generate inline code annotations for the PR diff
      --base string         Base branch (auto-detected if omitted)
      --format string       Report format: markdown, json, sarif, pr-summary, annotations
      --head string         Head branch (current branch if omitted)
  -h, --help                help for pr-review
      --no-reachability     Skip reachability analysis
      --no-sast             Skip SAST analysis
      --no-secrets          Skip secret detection
      --output string       Write report to file (stdout if omitted)
      --suggest-fixes       Generate safe fix proposals for detected vulnerabilities
      --suggest-reviewers   Suggest reviewers based on CODEOWNERS and git blame

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow review`

```text
Review code changes with context, PR analysis, or diff inspection.

Usage:
  patchflow review [command]

Available Commands:
  context     Show review context for the current repository
  diff        Review a diff
  pr          Review a pull request
  status      Check the status of a submitted review

Flags:
  -h, --help   help for review

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow review [command] --help" for more information about a command.
```

## `patchflow review context`

```text
Show review context for the current repository

Usage:
  patchflow review context [flags]

Flags:
  -h, --help   help for context

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow review pr`

```text
Review a pull request

Usage:
  patchflow review pr [flags]

Flags:
  -h, --help     help for pr
      --submit   Submit review payload to PatchFlow backend

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow review diff`

```text
Review a diff

Usage:
  patchflow review diff [flags]

Flags:
      --full-diff   Include full diff content (not yet implemented)
  -h, --help        help for diff

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow review status`

```text
Check the status of a submitted review

Usage:
  patchflow review status <job-id> [flags]

Flags:
  -h, --help    help for status
      --watch   Poll every 5 seconds until the job completes or fails

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow auth`

```text
Check authentication status and manage credentials for the PatchFlow platform.

Usage:
  patchflow auth [command]

Available Commands:
  status      Show authentication status

Flags:
  -h, --help   help for auth

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow auth [command] --help" for more information about a command.
```

## `patchflow auth status`

```text
Show authentication status

Usage:
  patchflow auth status [flags]

Flags:
  -h, --help   help for status

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow login`

```text
Authenticate with the PatchFlow platform using an API token or GitHub OAuth device flow.

Usage:
  patchflow login [flags]

Flags:
      --client-id string   GitHub OAuth app client ID (required with --device)
      --device             Use GitHub OAuth device flow
  -h, --help               help for login
      --token string       API token

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow logout`

```text
Remove stored credentials and log out from the PatchFlow platform.

Usage:
  patchflow logout [flags]

Flags:
  -h, --help   help for logout

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow config`

```text
View and modify PatchFlow CLI configuration settings.

Usage:
  patchflow config [command]

Available Commands:
  profile     Manage configuration profiles
  set         Set a configuration value
  show        Show current configuration

Flags:
  -h, --help   help for config

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow config [command] --help" for more information about a command.
```

## `patchflow config show`

```text
Show current configuration

Usage:
  patchflow config show [flags]

Flags:
  -h, --help   help for show

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow config set`

```text
Set a configuration value

Usage:
  patchflow config set <key> <value> [flags]

Flags:
  -h, --help   help for set

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow config profile`

```text
Create, switch, and manage multiple configuration profiles for different org/workspace contexts.

Usage:
  patchflow config profile [command]

Available Commands:
  create      Create a new configuration profile
  delete      Delete a configuration profile
  list        List all configuration profiles
  show        Show configuration profile details
  use         Switch active configuration profile

Flags:
  -h, --help   help for profile

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow config profile [command] --help" for more information about a command.
```

## `patchflow config profile create`

```text
Create a new configuration profile

Usage:
  patchflow config profile create <name> [flags]

Flags:
      --api-url string     API URL for the profile
  -h, --help               help for create
      --log-level string   Log level for the profile
      --org string         Organization for the profile

Global Flags:
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow config profile use`

```text
Switch active configuration profile

Usage:
  patchflow config profile use <name> [flags]

Flags:
  -h, --help   help for use

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow config profile list`

```text
List all configuration profiles

Usage:
  patchflow config profile list [flags]

Flags:
  -h, --help   help for list

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow config profile show`

```text
Show configuration profile details

Usage:
  patchflow config profile show <name> [flags]

Flags:
  -h, --help   help for show

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow config profile delete`

```text
Delete a configuration profile

Usage:
  patchflow config profile delete <name> [flags]

Flags:
  -h, --help   help for delete

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow cache`

```text
Inspect and clean the PatchFlow local cache.

The cache lives in a global XDG-compliant location:
  ~/.cache/patchflow/<project-hash>/
  (or $XDG_CACHE_HOME/patchflow/<project-hash>/)

It contains:
  - osv/            OSV vulnerability response cache (JSON files keyed by dependency hash)
  - sast_state.json Incremental SAST scan state (file hashes between scans)
  - registry/       Package registry metadata cache (npm, PyPI, Maven licenses)
  - maven/          Maven POM file cache for transitive resolution

Override the cache location with:
  --cache-dir DIR   CLI flag
  PATCHFLOW_CACHE_DIR  environment variable

Baselines (.patchflow/baselines/) and reports (.patchflow/reports/) are NOT
part of the cache and are preserved by 'cache clean'.

Usage:
  patchflow cache [command]

Available Commands:
  clean       Remove OSV and incremental SAST cache contents
  status      Show cache directory location, size, and entry counts
  update      Download the local OSV vulnerability database for offline scanning

Flags:
  -h, --help   help for cache

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow cache [command] --help" for more information about a command.
```

## `patchflow cache status`

```text
Show cache directory location, size, and entry counts

Usage:
  patchflow cache status [flags]

Flags:
  -h, --help   help for status

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow cache clean`

```text
Remove all cache contents (OSV responses and incremental SAST state).

Baselines and reports are NOT removed.

Use --force to skip the confirmation prompt.

Usage:
  patchflow cache clean [flags]

Flags:
      --force   Skip confirmation prompt
  -h, --help    help for clean

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow cache update`

```text
Download OSV vulnerability data from the OSV.dev bulk export and cache
it locally at ~/.patchflow/osv-db/. This enables fast offline scanning without
API calls to OSV.dev.

The database is refreshed automatically when stale (>24h old), but you can
force a refresh with --force.

Supported ecosystems: PyPI, npm, Maven, Go, RubyGems, Packagist, crates.io.
Use --ecosystems to select specific ones (default: all relevant to detected manifests).

Usage:
  patchflow cache update [flags]

Flags:
      --ecosystems strings   Ecosystems to download (pypi,npm,maven,go,rubygems,packagist,crates.io)
      --force                Force refresh even if DB is fresh
  -h, --help                 help for update

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow benchmark`

```text
Benchmark PatchFlow (and optional comparison tools) against a declared suite
of open-source repositories to measure detection, false positives, performance,
SARIF quality, and CI behavior.

Subcommands:
  run      Execute a benchmark suite from a benchmark.yaml config.
  compare  Compare summary.json results across multiple runs.
  report   Regenerate a summary report from existing results.

The benchmark invokes PatchFlow as a real subprocess so it measures actual CLI
behavior (startup, exit codes, SARIF). Comparison tools (semgrep, trivy,
gitleaks, osv-scanner) run when installed.

Responsible disclosure: findings from active/maintained repos are never
published in detail. Only intentionally-vulnerable and historical repos expose
per-finding detail.

Usage:
  patchflow benchmark [command]

Available Commands:
  compare     Compare summary results across multiple benchmark runs
  report      Regenerate a summary report from existing benchmark results
  run         Execute a benchmark suite

Flags:
  -h, --help   help for benchmark

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging

Use "patchflow benchmark [command] --help" for more information about a command.
```

## `patchflow benchmark run`

```text
Execute a benchmark suite

Usage:
  patchflow benchmark run [benchmark.yaml] [flags]

Flags:
  -h, --help                 help for run
      --no-tools             Skip comparison tools (only run PatchFlow)
      --no-warm              Skip the warm (cached) second run
      --results-dir string   Override results directory (default: results/<YYYY-MM>/)
      --timeout duration     Per-scan timeout (default 15m0s)

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow benchmark compare`

```text
Compare summary results across multiple benchmark runs

Usage:
  patchflow benchmark compare [results-dir] [flags]

Flags:
  -h, --help   help for compare

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow benchmark report`

```text
Regenerate a summary report from existing benchmark results

Usage:
  patchflow benchmark report [results-dir] [flags]

Flags:
      --format string   Report format: markdown, json (default "markdown")
  -h, --help            help for report

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow completion`

```text
Generate shell completion scripts for patchflow.

To load completions:

Bash:
  source <(patchflow completion bash)

  # To load completions for each session, add the above line to your ~/.bashrc

Zsh:
  # If shell completion is not already enabled in your environment,
  # enable it by running:
  echo "autoload -U compinit; compinit" >> ~/.zshrc

  source <(patchflow completion zsh)

  # To load completions for each session, add the above line to your ~/.zshrc

Fish:
  patchflow completion fish | source

  # To load completions for each session, add the above line to your ~/.config/fish/config.fish

PowerShell:
  patchflow completion powershell | Out-String | Invoke-Expression

  # To load completions for every new session, add the output of the above
  # command to your PowerShell profile.

Usage:
  patchflow completion [bash|zsh|fish|powershell]

Flags:
  -h, --help   help for completion

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow init github-actions`

```text
Create .github/workflows/patchflow-scan.yml in the current repository.
The workflow runs patchflow scan run --profile ci --format sarif and uploads
SARIF results to GitHub Code Scanning.

Usage:
  patchflow init github-actions [flags]

Flags:
  -h, --help   help for github-actions

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow init gitlab-ci`

```text
Create or append a patchflow:scan job to .gitlab-ci.yml in the current
repository. The job runs patchflow scan run --profile ci and publishes JSON and
SARIF reports as artifacts.

Usage:
  patchflow init gitlab-ci [flags]

Flags:
  -h, --help   help for gitlab-ci

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow init jenkins`

```text
Create Jenkinsfile.patchflow in the current repository containing a
PatchFlow Security Scan stage. Existing Jenkinsfiles are not modified.

Usage:
  patchflow init jenkins [flags]

Flags:
  -h, --help   help for jenkins

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow init azure-devops`

```text
Create azure-pipelines-patchflow.yml in the current repository with a
PatchFlow Security Scan task and artifact publishing.

Usage:
  patchflow init azure-devops [flags]

Flags:
  -h, --help   help for azure-devops

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

## `patchflow init pre-commit`

```text
Create or append a patchflow hook to .pre-commit-config.yaml in the
current repository. The hook runs patchflow scan run --profile dev on commit.

Usage:
  patchflow init pre-commit [flags]

Flags:
  -h, --help   help for pre-commit

Global Flags:
      --api-url string     PatchFlow API URL
      --cache-dir string   Override cache directory (default: ~/.cache/patchflow/ or $XDG_CACHE_HOME/patchflow/)
      --config string      config file path
      --json               output in JSON format
      --no-color           disable colored output
  -q, --quiet              suppress non-essential output (for CI scripting)
  -v, --verbose            enable verbose logging
```

