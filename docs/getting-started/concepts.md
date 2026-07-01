---
title: Core Concepts
description: Understand SCA, SAST, reachability, risk scoring, and governance
---

# Core Concepts

PatchFlow CLI combines multiple analysis engines into a single local-first
security scanner. This page explains the core concepts that underpin every
command.

## Analysis Pipeline

When you run `patchflow scan run`, the following stages execute in sequence:

```
┌─────────────────────────────────────────────────────────────┐
│                    patchflow scan run                        │
├─────────────────────────────────────────────────────────────┤
│  1. SCA           Parse manifests → Query OSV.dev           │
│  2. Licenses      Fetch registry metadata → Classify        │
│  3. SAST          Embedded scanners + External tools        │
│  4. Secrets       Regex patterns + Entropy detection        │
│  5. Frameworks    Auto-detect → Activate rule packs         │
│  6. Reachability  Build import graph → Assess usage         │
│  7. Risk          Compute 0–100 score from all findings     │
│  8. Governance    Filter by maturity/profile                │
│  9. Output        Terminal summary + Optional report file   │
└─────────────────────────────────────────────────────────────┘
```

Each stage can be skipped with flags like `--no-sast`, `--no-secrets`,
`--no-reachability`, and `--no-licenses`.

## SCA (Software Composition Analysis)

SCA identifies vulnerable dependencies in your project.

**How it works:**

1. PatchFlow parses dependency manifests from supported ecosystems:
   - Go (`go.mod`)
   - npm (`package.json`, `package-lock.json`)
   - PyPI (`requirements.txt`, `pyproject.toml`, `Pipfile`, `poetry.lock`)
   - Maven (`pom.xml`)
   - RubyGems (`Gemfile`, `Gemfile.lock`)
   - Packagist (`composer.json`, `composer.lock`)
   - Cargo (`Cargo.toml`, `Cargo.lock`)
2. For each dependency, PatchFlow queries the [OSV.dev](https://osv.dev) public
   vulnerability database via batch API calls.
3. Vulnerability results include severity, CVE ID, affected versions, and fixed
   versions.
4. In `deep` profile, Maven transitive dependencies are resolved for deeper
   coverage.

**Offline mode:** Run `patchflow cache update` to download a local OSV database.
Then use `--offline` to scan without any API calls. See
[Cache Management](../user-guides/cache.md).

**Max depth by profile:**

| Profile | Max Depth | Behavior |
| --- | --- | --- |
| `quick` | 1 | Fast, direct dependencies only |
| `standard` | 3 | Direct + transitive (limited) |
| `deep` | 5 | Full transitive resolution including Maven |

## SAST (Static Analysis Security Testing)

SAST finds security vulnerabilities in your source code. PatchFlow uses a
multi-engine approach:

### Embedded Scanners (zero installation)

| Engine | Maturity | Description |
| --- | --- | --- |
| `gosast-embedded` | Stable | ~30 Go AST rules ported from gosec: SQL injection, command injection, weak crypto, unsafe pointers, hardcoded credentials, bad file permissions, path traversal, TLS misconfiguration, slowloris, decompression bombs, and more. |
| `secrets-embedded` | Stable | ~40 regex patterns for AWS, Google, Azure, GitHub, GitLab, Slack, Stripe, Twilio, private keys, database URLs, JWTs, SSH keys, plus Shannon entropy detection (>4.5, length >20). |
| `patterns-embedded` | Beta | ~100+ multi-language regex rules for Python, JS/TS, Ruby, PHP, Java, C#, Rust, Go, Dockerfile, YAML/Terraform. Covers OWASP Top 10: eval, exec, shell=True, pickle, yaml.load, SQL injection, weak crypto, SSL verification, debug mode, dangerouslySetInnerHTML, CORS, IaC posture. |
| `treesitter-ast` | Beta | ~50+ AST rules via tree-sitter for Python, JS/TS, Ruby, PHP, Java, C#, Rust. Structural matching beyond regex. |
| `taint-ssa` | Stable | Go SSA-based taint analysis. Source-to-sink tracking with inter-procedural call-hop depth. |
| `taint-patterns` | Beta | ~30 tree-sitter taint rules for Python, JS/TS, Ruby, PHP, Java, C#. Source-to-sink tracking with sanitizer detection. |
| `framework-packs` | Experimental | 17 official framework rule packs. See [Framework Packs](../user-guides/framework-packs.md). |

### External Tools (optional supplements)

| Tool | Coverage | When Used |
| --- | --- | --- |
| `gosec` | Go | Always when installed (all profiles except quick skips some) |
| `bandit` | Python | Always when installed |
| `semgrep` | Multi-language | Always when installed |
| `gitleaks` | Secrets | Always when installed |
| `checkov` | IaC | Always when installed |

External tool findings are merged with embedded scanner findings and
deduplicated by the SAST runner.

### File Collection

PatchFlow uses a single-pass parallel file collector (`filecollector.go`) that
walks the file tree once and dispatches each file to all applicable scanners.
This eliminates redundant tree traversals and reduces I/O by approximately 4x.

**Ignored directories:** `.git`, `node_modules`, `vendor`, `dist`, `build`,
`__pycache__`, `.next`, `.nuxt`, `target`, `.gradle`, `.idea`, `.vscode`, `bin`,
`obj`, `.cache`, `.pytest_cache`, `.mypy_cache`, `.ruff_cache`, `coverage`,
`.turbo`, `.svelte-kit`, `lib`, `libs`, `wwwroot`, `third_party`, `thirdparty`,
`external`, `deps`, `bower_components`, `jspm_packages`, `webjars`, `packages`,
`Content`, `Scripts`, `testdata`, `test_data`, `fixtures`, `__fixtures__`,
`__mocks__`.

**Third-party file filtering:** Minified files (`*.min.js`, `*.min.css`), common
libraries (jquery, bootstrap, angular, react, vue), and bundled output
(`bundle.*`, `vendor.*`, `commons.*`, `main.chunk.*`) are skipped.

## Reachability Analysis

Reachability determines whether a vulnerable dependency is actually used in your
codebase, helping you prioritize which vulnerabilities to fix first.

**How it works:**

1. PatchFlow parses import statements for Python, Go, and JavaScript/TypeScript.
2. It builds an import graph mapping source files to imported packages.
3. For each vulnerable dependency, it checks whether the package name appears in
   the import graph.
4. It assigns a confidence level:

| Level | Meaning |
| --- | --- |
| `HIGH` | Directly imported or invoked |
| `MEDIUM` | Direct dependency, possible runtime usage |
| `LOW` | Transitive dependency, no direct usage found |
| `NONE` | Not present in dependency graph |
| `UNKNOWN` | Analysis incomplete |

Use `patchflow reachability --package <name> --explain` to see the evidence
behind a reachability assessment. See
[Reachability](../user-guides/reachability.md) for details.

## Risk Scoring

PatchFlow computes a 0–100 risk score from all findings and change metadata.

**Inputs:**

- Vulnerability points (from SCA findings, weighted by severity and reachability)
- SAST points (from SAST findings, weighted by severity)
- Secret points (from secret detection findings)
- Change points (files changed, lines added/deleted)
- Sensitivity points (auth files changed, CI workflows changed, dependency files
  changed)
- Reachability bonus (vulnerable dependencies that are actually reachable score
  higher)

**Risk levels:**

| Score Range | Level | Meaning |
| --- | --- | --- |
| 0–24 | Low | Minimal risk, safe to proceed |
| 25–49 | Moderate | Some risk, review findings |
| 50–74 | High | Significant risk, review required |
| 75–100 | Critical | High risk, blocking recommended |

The risk score is included in all report formats and in `pr-review` output.

## Governance

Governance controls which rules are visible and which findings can block CI.

### Rule Maturity

Every rule carries a maturity level:

| Maturity | Description | Typical Visibility |
| --- | --- | --- |
| `experimental` | New rule, few tests, may have false positives | Audit profile only |
| `beta` | Has tests, enabled in audit/standard, non-blocking | CI and audit, non-blocking |
| `stable` | Has false-positive tests, CWE/OWASP mapping, can block | Developer, PR, CI, audit |
| `enterprise` | Large regression corpus, low false-positive rate | All profiles, blocking |

### Governance Profiles

Profiles filter findings by rule maturity:

| Profile | Purpose | Rules Included |
| --- | --- | --- |
| `dev` | Local development, fast feedback | Stable only |
| `pr` | Pull request review signal | Stable |
| `ci` | CI warnings and blocking gates | Stable + beta |
| `audit` | Full rule surface for security audits | All (including experimental) |

**Default profile mapping:**

| Scan Profile | Default Governance Profile |
| --- | --- |
| `quick` | `dev` |
| `standard` | `ci` |
| `deep` | `audit` |

Override with `--governance-profile`:

```bash
patchflow scan run --governance-profile audit
```

### Blocking Eligibility

A finding is "blocking-eligible" in CI when:

1. The rule maturity is `stable` or `enterprise`
2. The governance profile includes the rule (e.g., `ci` includes stable + beta,
   but only stable+ can block)
3. The finding severity meets the `--fail-on` threshold

Use `patchflow rules maturity` to see the governance coverage report. See
[Rule Governance](../reference/rules-governance.md) for details.

## Framework Packs

Framework packs are official embedded rule sets that understand framework-specific
sources, sinks, sanitizers, and template engines. They run alongside the general
SAST engines.

**17 official packs:**

| Pack | Language | Coverage |
| --- | --- | --- |
| `rails` | Ruby | Controllers, ERB, redirects, SQL, XSS, mass assignment, deserialization |
| `express` | JavaScript | Request sources, redirects, response XSS, SQL, file access |
| `nextjs` | JavaScript | Server-side fetch, redirects, JSX XSS |
| `react` | JavaScript | JSX XSS, client-side navigation |
| `spring` | Java | SQL, SSRF, redirects, XXE, templates, command execution |
| `spring-security` | Java | CSRF disabled, public sensitive routes |
| `aspnet` | C# | SQL, redirects, raw HTML responses |
| `razor` | C# | Html.Raw, MarkupString |
| `django` | Python | SQL, redirects, template safe filter, unsafe deserialization |
| `laravel` | PHP | Raw SQL, redirects, Blade unescaped output, mass assignment |
| `fastapi` | Python | Request sources, SQL, redirects, templates |
| `gin` | JavaScript | Request sources, SQL, redirects, command execution |
| `flask` | Python | SQL, redirects, Jinja safe filter, dynamic template strings |
| `symfony` | PHP | SQL, redirects, Twig unsafe output |
| `angular` | TypeScript | Template and DOM sink patterns |
| `nestjs` | TypeScript | Controller sources, redirects, command patterns |
| `echo` | JavaScript | Request sources, redirects, response output |

Auto-detection is the default. Override with `--framework` and
`--disable-framework`. See [Framework Packs](../user-guides/framework-packs.md)
for details.

## Suppression Directives

Suppress false positives with `//patchflow:ignore` comments:

```go
//patchflow:ignore G404 -- using math/rand for non-security purpose
n := rand.Intn(100)
```

```python
# patchflow:ignore PY001 -- eval is safe here, input is sanitized
result = eval(user_input)
```

The comment syntax adapts to the file type:

- `#` for Python, Ruby, Shell, YAML, TOML, Dockerfile, Terraform
- `//` for Go, JS/TS, Java, C/C++, C#, Rust, Swift, Kotlin, Scala, PHP

Use `patchflow suppress` to add directives programmatically. See
[Suppressions](../user-guides/suppressions.md) for details.

## Baselines

Baselines store a snapshot of known findings so CI can focus on new issues.

**How they work:**

1. Run a full scan on your default branch.
2. Create a named baseline: `patchflow baseline create --name v1.0`
3. Future scans compare against the baseline: `patchflow scan run --new-only
   --baseline v1.0`
4. Only findings not in the baseline are reported.

Baselines use stable semantic fingerprints (rule ID + scanner + normalized path +
normalized snippet) so findings survive line-number shifts from unrelated edits.
See [Baselines](../user-guides/baselines.md) for the full workflow.

## Next Steps

- [Scan Your Project](../user-guides/scan.md) — All scan flags and profiles
- [Framework Packs](../user-guides/framework-packs.md) — Framework-specific rules
- [Recommended Workflow](../workflows/recommended.md) — Team adoption strategy
