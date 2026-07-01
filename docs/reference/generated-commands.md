# Generated Command Reference

Generated from `/Users/digitalcenter/patchflow-cli/patchflow`.

Run `npm run generate:commands` after changing CLI commands.

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

