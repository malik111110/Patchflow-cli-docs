# Scanner Reference

This page documents all embedded and external scanners in PatchFlow CLI.

## Embedded Scanners

Embedded scanners require zero installation. They are built into the PatchFlow
binary and always available.

### gosast-embedded

Go AST-based security analysis, ported from gosec.

| Attribute | Value |
| --- | --- |
| Engine name | `gosast-embedded` |
| Language | Go |
| Rule count | ~30 rules (G101–G601) |
| Maturity | Stable |
| Package | `internal/sast/gosast/` |

**Coverage:**

- SQL injection (G201, G202)
- Command injection (G204)
- Weak crypto (G401, G402, G501, G502, G503, G505)
- Unsafe pointers (G103)
- Hardcoded credentials (G101, G102)
- Bad file permissions (G306, G307)
- Path traversal (G304)
- TLS misconfiguration (G402)
- Slowloris (G602)
- Decompression bombs (G110)
- Trojan Source (G117)
- Error handling (G104)
- Integer overflow (G109)
- Implicit aliasing (G601)
- Secret serialization (G104)
- Slice bounds (G603)
- Blocklist imports (G301, G302, G303)
- Templates (G303)
- SSH host key (G106)
- Pprof endpoints (G108)
- Random number generation (G404)

**Implementation:** Uses `golang.org/x/tools/go/packages` for package loading.
Supports `go.work` workspaces. Monorepo support loads sub-modules individually.

### secrets-embedded

Regex-based secret detection with Shannon entropy analysis.

| Attribute | Value |
| --- | --- |
| Engine name | `secrets-embedded` |
| Language | Multi-language |
| Pattern count | ~40 patterns |
| Maturity | Stable |
| Package | `internal/sast/secrets/` |

**Detected secret types:**

- Cloud provider keys: AWS (access key, secret key), Google (API key, OAuth),
  Azure (access key, connection string)
- Version control tokens: GitHub, GitLab
- SaaS tokens: Slack, Stripe, Twilio, Square, Heroku, Mailgun, MailChimp,
  Telegram
- Private keys: RSA, EC, DSA, OpenSSH, PGP
- Database connection strings: PostgreSQL, MySQL, MongoDB, Redis
- API keys and tokens: generic API key patterns
- JWT tokens
- SSH keys
- High-entropy strings: Shannon entropy > 4.5, length > 20

**File filtering:**

- Skips binary extensions (.png, .jpg, .pdf, .zip, etc.)
- Skips lockfiles (.lock, .sum)
- Skips minified files (.min.js, .min.css)
- Max file size: 2MB

### patterns-embedded

Multi-language regex patterns for OWASP Top 10 coverage.

| Attribute | Value |
| --- | --- |
| Engine name | `patterns-embedded` |
| Language | Multi-language |
| Rule count | ~100+ rules |
| Maturity | Beta |
| Package | `internal/sast/patterns/` |

**Languages and rule prefixes:**

| Language | Prefix | Example Rules |
| --- | --- | --- |
| Python | PY | PY001 (eval), PY002 (exec), PY003 (os.system), PY004 (subprocess shell=True), PY005 (pickle), PY006 (yaml.load) |
| JavaScript/TypeScript | JS/TS | JS001 (eval), JS002 (Function), JS003 (innerHTML), JS004 (document.write), JS005 (execSync) |
| Ruby | RB | RB001 (eval), RB002 (system), RB003 (YAML.load) |
| PHP | PH | PH001 (eval), PH002 (exec), PH003 (shell_exec) |
| Java | JV | JV001 (deserialization), JV002 (weak crypto) |
| C# | CS | CS001 (deserialization), CS002 (weak crypto) |
| Rust | RS | RS001 (unsafe), RS002 (unwrap) |
| Go | GO | GO001 (exec.Command), GO002 (unsafe) |
| Dockerfile | TF | TF001 (no USER), TF002 (no HEALTHCHECK) |
| YAML/Terraform | TF | TF003 (S3 public), TF004 (no encryption) |

**Coverage areas:**

- Code injection (eval, exec, subprocess)
- SQL injection
- Command injection
- Path traversal
- SSRF
- Weak crypto (MD5, SHA1, weak random)
- Hardcoded secrets
- Debug mode enabled
- TLS verification disabled
- Unsafe deserialization
- Logging sensitive data
- CORS misconfiguration
- IaC security (S3 posture, encryption, public access)

**Features:**

- SkipQuoteFilter: Detects patterns inside string literals (for injection rules)
- SuppressFunc: Post-match validation to reduce false positives
- Language-specific patterns

### treesitter-ast

AST-based rules via tree-sitter for structural matching.

| Attribute | Value |
| --- | --- |
| Engine name | `treesitter-ast` |
| Language | Multi-language |
| Rule count | ~50+ rules |
| Maturity | Beta |
| Package | `internal/sast/treesitter/` |

**Languages:** Python, JavaScript, TypeScript, Ruby, PHP, Java, C#, Rust

**Coverage by language:**

| Language | Dangerous APIs Detected |
| --- | --- |
| Python | eval, exec, os.system, subprocess, pickle, yaml.load, hashlib, Flask debug, marshal, shelve |
| JavaScript | eval, Function, innerHTML, document.write, execSync, spawn |
| TypeScript | Same as JavaScript |
| Ruby | eval, system, exec, YAML.load, Marshal.load |
| PHP | eval, exec, shell_exec, system, unserialize |
| Java | deserialization, weak crypto, SSRF |
| C# | deserialization, weak crypto |

**Performance:** Uses `sync.Pool` per language for parser reuse.

### taint-ssa

Go SSA-based taint analysis for source-to-sink tracking.

| Attribute | Value |
| --- | --- |
| Engine name | `taint-ssa` |
| Language | Go |
| Maturity | Stable |
| Package | `internal/sast/taint/` |

**Implementation:**

- Uses Go SSA (Static Single Assignment) representation
- Precise taint tracking through Go code
- Inter-procedural analysis with configurable call-hop depth
- Source and sink detection

### taint-patterns

Tree-sitter-based source-to-sink taint analysis for non-Go languages.

| Attribute | Value |
| --- | --- |
| Engine name | `taint-patterns` |
| Language | Multi-language |
| Rule count | ~30 taint rules |
| Maturity | Beta |
| Package | `internal/sast/taintpatterns/` |

**Languages:** Python, JavaScript, TypeScript, Ruby, PHP, Java, C#

**Coverage:**

| Category | Rule Examples |
| --- | --- |
| SQL injection | TP-PY001, TP-JS001, TP-RB001, TP-PH001 |
| Command injection | TP-PY002, TP-JS002, TP-RB002 |
| Path traversal | TP-PY003, TP-JS004 |
| SSRF | TP-PY004, TP-JS005 |
| XSS | TP-PY006, TP-JS006 |

**Features:**

- Intra-procedural taint analysis (within function scope)
- Inter-procedural taint analysis (configurable depth, default 3)
- Source-sink tracking via tree-sitter AST
- Sanitizer detection

**Configuration:**

- `--taint-depth <n>`: Sets inter-procedural depth (0 = disabled, default: 3)

### framework-packs

Framework-specific rule packs.

| Attribute | Value |
| --- | --- |
| Engine name | `framework-packs` |
| Language | Multi-language |
| Pack count | 17 official packs |
| Maturity | Experimental (default; promoted per-pack) |
| Package | `internal/sast/frameworks/` |

See [Framework Packs](../user-guides/framework-packs.md) for the full pack
reference.

## External Tools

External tools supplement the embedded scanners when installed. They are
optional and run automatically when detected on `PATH`.

| Tool | Language | Coverage | Maturity |
| --- | --- | --- | --- |
| `gosec` | Go | Go AST security analysis | N/A (external) |
| `bandit` | Python | Python security linting | N/A (external) |
| `semgrep` | Multi-language | Pattern-based SAST | N/A (external) |
| `gitleaks` | Secrets | Secret detection | N/A (external) |
| `trivy` | Containers, IaC | Container image and IaC scanning | N/A (external) |
| `checkov` | IaC | Infrastructure-as-code scanning | N/A (external) |

External tool findings are merged with embedded scanner findings and
deduplicated by the SAST runner.

## File Collection

The file collector (`internal/sast/filecollector.go`) uses a single-pass
parallel dispatch:

- Walks the file tree once for all scanners
- Dispatches each file to applicable scanners
- Uses a worker pool (`runtime.NumCPU()` workers)
- Eliminates redundant tree traversals (4x I/O reduction)

**Ignored directories:** `.git`, `node_modules`, `vendor`, `dist`, `build`,
`__pycache__`, `.next`, `.nuxt`, `target`, `.gradle`, `.idea`, `.vscode`, `bin`,
`obj`, `.cache`, `.pytest_cache`, `.mypy_cache`, `.ruff_cache`, `coverage`,
`.turbo`, `.svelte-kit`, `lib`, `libs`, `wwwroot`, `third_party`, `thirdparty`,
`external`, `deps`, `bower_components`, `jspm_packages`, `webjars`, `packages`,
`Content`, `Scripts`, `testdata`, `test_data`, `fixtures`, `__fixtures__`,
`__mocks__`.

**Third-party file filtering:** Minified files, common libraries, bundled
output.

## Fix Snippets

The fix snippet library (`internal/fixsnippet/`) provides vulnerable code
patterns and their fixed equivalents. Used by `patchflow explain` and
`--suggest-fixes` output.

**Snippet count:** ~30 code snippets

**Coverage:**

- Python taint patterns (TP-PY001 through TP-PY006)
- JavaScript/TypeScript taint patterns (TP-JS001 through TP-JS006)
- Ruby, PHP, Java, C# patterns

## Next Steps

- [Rule Governance](./rules-governance.md) — Maturity and profiles
- [Framework Packs](../user-guides/framework-packs.md) — Official pack reference
- [Architecture](../developers/architecture.md) — Internal architecture
