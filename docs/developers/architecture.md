# Architecture

This page provides a detailed architectural deep-dive into PatchFlow CLI's
internal design.

## SAST Runner

The SAST runner (`internal/sast/runner.go`) orchestrates all embedded and
external scanners. It is the central component that ties together file
collection, scanner dispatch, finding deduplication, and governance filtering.

### Scan Flow

```
1. Collect files    filecollector.go (single-pass parallel walk)
2. Dispatch         Each file sent to applicable scanners
3. Scan             Embedded + external scanners run in parallel
4. Merge            Findings from all scanners merged
5. Deduplicate      Findings deduplicated by fingerprint
6. Governance       Findings filtered by maturity/profile
7. Return           Findings returned to caller
```

### File Collection

The file collector (`filecollector.go`) walks the file tree once and dispatches
each file to all applicable scanners. This eliminates redundant tree traversals
and reduces I/O by approximately 4x compared to per-scanner walks.

**Key features:**

- Single-pass: walks the tree once for all scanners
- Parallel: uses a worker pool (`runtime.NumCPU()` workers)
- Smart filtering: skips ignored directories, minified files, third-party
  libraries
- Per-file categorization: each file is categorized for which scanners should
  process it

**Ignored directories:** `.git`, `node_modules`, `vendor`, `dist`, `build`,
`__pycache__`, `.next`, `.nuxt`, `target`, `.gradle`, `.idea`, `.vscode`, `bin`,
`obj`, `.cache`, `.pytest_cache`, `.mypy_cache`, `.ruff_cache`, `coverage`,
`.turbo`, `.svelte-kit`, `lib`, `libs`, `wwwroot`, `third_party`, `thirdparty`,
`external`, `deps`, `bower_components`, `jspm_packages`, `webjars`, `packages`,
`Content`, `Scripts`, `testdata`, `test_data`, `fixtures`, `__fixtures__`,
`__mocks__`.

**Third-party file filtering:** Minified files (`*.min.js`, `*.min.css`,
`*_min.js`), common libraries (jquery, bootstrap, angular, react, vue), and
bundled output (`bundle.*`, `vendor.*`, `commons.*`, `main.chunk.*`).

## Embedded Scanners

### gosast-embedded (`internal/sast/gosast/`)

Go AST-based security analysis, ported from gosec.

- **Rule count**: ~30 rules (G101–G601)
- **Coverage**: SQL injection, command injection, weak crypto, unsafe pointers,
  hardcoded credentials, bad file permissions, path traversal, TLS
  misconfiguration, slowloris, decompression bombs, Trojan Source, error
  handling, integer overflow, implicit aliasing, secret serialization, slice
  bounds
- **Maturity**: Stable
- **Implementation**: Uses `golang.org/x/tools/go/packages` for package loading.
  Supports `go.work` workspaces. Monorepo support loads sub-modules individually.

### secrets-embedded (`internal/sast/secrets/`)

Regex-based secret detection with entropy analysis.

- **Pattern count**: ~40 patterns
- **Coverage**: AWS, Google, Azure, GitHub, GitLab, Slack, Stripe, Twilio, Square,
  Heroku, Mailgun, MailChimp, Telegram, private keys (RSA, EC, DSA, OpenSSH,
  PGP), database connection strings, API keys, JWTs, SSH keys, high-entropy
  strings (Shannon entropy > 4.5, length > 20)
- **Maturity**: Stable
- **File filtering**: Skips binary extensions, lockfiles, minified files. Max
  file size: 2MB.

### patterns-embedded (`internal/sast/patterns/`)

Multi-language regex patterns for OWASP Top 10 coverage.

- **Rule count**: ~100+ rules
- **Languages**: Python (PY001+), JavaScript/TypeScript (JS001+), Ruby (RB001+),
  PHP (PH001+), Java (JV001+), C# (CS001+), Rust (RS001+), Go (GO001+),
  Dockerfile (TF001+), YAML/Terraform
- **Coverage**: eval, exec, subprocess, SQL injection, command injection, path
  traversal, SSRF, weak crypto, hardcoded secrets, debug mode, TLS verification,
  unsafe deserialization, logging sensitive data, CORS, IaC posture
- **Maturity**: Beta (upgraded from experimental after SkipQuoteFilter mechanism)
- **Features**: SkipQuoteFilter for injection rules (detects patterns inside
  string literals), SuppressFunc for post-match validation

### treesitter-ast (`internal/sast/treesitter/`)

AST-based rules via tree-sitter for structural matching beyond regex.

- **Rule count**: ~50+ rules
- **Languages**: Python, JavaScript, TypeScript, Ruby, PHP, Java, C#, Rust
- **Coverage**: Language-specific dangerous APIs (eval, exec, os.system,
  subprocess, pickle, yaml.load, hashlib, Flask debug, marshal, shelve,
  innerHTML, document.write, execSync, spawn, system, shell_exec, unserialize,
  deserialization, weak crypto, SSRF)
- **Maturity**: Beta
- **Performance**: Uses `sync.Pool` per language for parser reuse

### taint-ssa (`internal/sast/taint/`)

Go SSA-based taint analysis for source-to-sink tracking.

- **Maturity**: Stable
- **Implementation**: Uses Go SSA representation for precise taint tracking
- **Inter-procedural**: Configurable call-hop depth (default: 3)

### taint-patterns (`internal/sast/taintpatterns/`)

Tree-sitter-based source-to-sink taint analysis for non-Go languages.

- **Rule count**: ~30 taint rules
- **Languages**: Python, JavaScript, TypeScript, Ruby, PHP, Java, C#
- **Coverage**: SQL injection, command injection, path traversal, SSRF, XSS
- **Maturity**: Beta
- **Features**: Intra-procedural and inter-procedural taint analysis,
  source-sink tracking via tree-sitter AST, sanitizer detection
- **Configuration**: `--taint-depth <n>` sets inter-procedural depth (0 =
  disabled, default: 3)

## Framework Pack System

Framework packs are official embedded rule sets that understand framework-specific
sources, sinks, sanitizers, and template engines.

### Pack Flow

```
1. Detect frameworks    internal/frameworks.Detector via filesystem signals
2. Select packs         frameworks.Loader based on detection + CLI config
3. Pattern/template     frameworks.Matcher (line-oriented, sanitizer-aware)
4. Taint rules          Registered into taintpatterns.Analyzer (source→sink)
5. Deduplicate          SAST runner deduplicates findings
```

### FrameworkRule Model

```go
type FrameworkRule struct {
    ID          string
    Framework   string
    Language    string
    CWE         string
    Title       string
    Severity    analysis.Severity
    Confidence  analysis.Confidence
    Maturity    Maturity
    FileTypes       []string
    TemplateTypes   []string
    MatchMode       MatchMode
    Pattern         *regexp.Regexp
    Sources         []SourcePattern
    Sinks           []SinkPattern
    Sanitizers      []SanitizerPattern
    SafePatterns    []SafePattern
    Exclusions      []PathPattern
    Recommendation  string
}
```

### MatchMode Values

| MatchMode | Description | Use Case |
| --- | --- | --- |
| `MatchPattern` | Regex against source lines | One-line dangerous API usage |
| `MatchAST` | Framework-specific call structures (tree-sitter) | Structural matching beyond regex |
| `MatchTaint` | Source-to-sink taint tracking | Data flow across variables or helpers |
| `MatchTemplate` | Template engine output issues | ERB, Jinja, Razor, Blade, JSX output |

See [Framework Rule Packs](./rule-packs.md) and [Adding a Pack](./adding-packs.md)
for pack development details.

## SCA Pipeline

The SCA pipeline (`internal/sca/`) identifies vulnerable dependencies:

1. **Manifest parsing** (`internal/manifest/`): Parses dependency manifests from
   Go, npm, PyPI, Maven, RubyGems, Packagist, Cargo.
2. **OSV.dev query** (`internal/osv/`): Batch API calls to OSV.dev for
   vulnerability data. Uses local cache to avoid repeated queries.
3. **Offline mode** (`internal/osvdb/`): Local OSV database for air-gapped
   scanning. Downloaded via `patchflow cache update`.
4. **License analysis** (`internal/registry/`): Fetches license info from package
   registries and classifies by category and risk.

## Reachability Analysis

The reachability analyzer (`internal/reachability/`) builds an import graph to
determine if vulnerable dependencies are actually used:

1. Parses import statements for Python, Go, JavaScript/TypeScript
2. Builds an import graph mapping source files to imported packages
3. Matches vulnerable dependency names against the import graph
4. Assigns confidence levels: HIGH, MEDIUM, LOW, NONE, UNKNOWN

## Risk Scoring

The risk engine (`internal/risk/`) computes a 0–100 score from:

- Vulnerability points (SCA findings weighted by severity and reachability)
- SAST points (SAST findings weighted by severity)
- Secret points (secret detection findings)
- Change points (files changed, lines added/deleted)
- Sensitivity points (auth files, CI workflows, dependency files changed)
- Reachability bonus (reachable vulnerable dependencies score higher)

## Governance System

The governance registry (`internal/rules/`) manages rule metadata:

- **Maturity levels**: experimental, beta, stable, enterprise
- **Profiles**: dev, pr, ci, audit
- **Engine types**: patterns-embedded, treesitter-ast, gosast-embedded,
  secrets-embedded, taint-ssa, taint-patterns, framework-packs
- **CWE/OWASP mapping**: Each rule maps to CWE and OWASP categories
- **Blocking eligibility**: Rules can block CI based on maturity and profile

See [Rule Governance](../reference/rules-governance.md) for details.

## Report Generation

The report generator (`internal/report/`) produces:

- **Markdown**: Human-readable report with tables and sections
- **JSON**: Structured JSON with all findings and metadata
- **SARIF**: SARIF 2.1.0 for tool integration

The SBOM generator (`internal/sbom/`) produces:

- **CycloneDX**: CycloneDX SBOM with optional VEX statements
- **SPDX**: SPDX SBOM format
- **VEX**: Vulnerability Exploitability Exchange format
- **Dependency graph**: Tree and DOT formats

## Next Steps

- [Framework Rule Packs](./rule-packs.md) — Pack development guide
- [Adding a Framework Pack](./adding-packs.md) — Step-by-step pack creation
- [Rule Governance](../reference/rules-governance.md) — Maturity and profiles
