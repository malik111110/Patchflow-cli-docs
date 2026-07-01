# Adding a Framework Pack

This guide walks through creating a new framework pack for PatchFlow CLI.

## Overview

Framework packs are official embedded rule sets. The flow from creation to
activation:

```
1. Create pack directory and implement Pack interface
2. Add sources, sinks, sanitizers, and rules
3. Register the pack in the default registry
4. Add framework detection signals
5. Add test fixtures (vulnerable, safe, normal)
6. Set maturity to experimental
7. Run tests and benchmark suite
8. Promote to beta/stable as coverage grows
```

## Step 1: Create the Pack Directory

Create a directory under `internal/sast/frameworks/<name>/`:

```bash
mkdir internal/sast/frameworks/myframework
```

## Step 2: Implement the Pack Interface

Create `pack.go`:

```go
package myframework

import "github.com/Patchflow-security/patchflow-cli/internal/sast/frameworks"

type Pack struct{}

func New() *Pack { return &Pack{} }

func (p *Pack) Name() string             { return "myframework" }
func (p *Pack) Language() string         { return "python" }
func (p *Pack) FileExtensions() []string { return []string{".py"} }
func (p *Pack) TemplateExtensions() []string { return []string{".html"} }

func (p *Pack) Rules() []frameworks.FrameworkRule {
    return allRules
}

func (p *Pack) Sources() []frameworks.SourcePattern {
    return allSources
}

func (p *Pack) Sinks() []frameworks.SinkPattern {
    return allSinks
}

func (p *Pack) Sanitizers() []frameworks.SanitizerPattern {
    return allSanitizers
}
```

## Step 3: Add Sources

Create `sources.go` with source patterns (where tainted data enters):

```go
package myframework

import "github.com/Patchflow-security/patchflow-cli/internal/sast/frameworks"

var allSources = []frameworks.SourcePattern{
    {Func: "request.GET"},
    {Func: "request.POST"},
    {Func: "request.args"},
    {Func: "request.form"},
    {Func: "request.json"},
}
```

## Step 4: Add Sinks

Create `sinks.go` with sink patterns (where tainted data is used dangerously):

```go
package myframework

import "github.com/Patchflow-security/patchflow-cli/internal/sast/frameworks"

var allSinks = []frameworks.SinkPattern{
    {Func: "db.execute", Category: "sql-injection"},
    {Func: "db.query", Category: "sql-injection"},
    {Func: "subprocess.run", Category: "command-injection"},
    {Func: "os.system", Category: "command-injection"},
    {Func: "redirect", Category: "open-redirect"},
}
```

## Step 5: Add Sanitizers

Create `sanitizers.go` with sanitizer patterns (functions that neutralize taint):

```go
package myframework

import "github.com/Patchflow-security/patchflow-cli/internal/sast/frameworks"

var allSanitizers = []frameworks.SanitizerPattern{
    {Func: "escape"},
    {Func: "sanitize_input"},
    {Func: "validate_input"},
    {Regex: "parameterized_query\\("},
}
```

## Step 6: Add Rules

Create `rules.go` with all framework rules:

```go
package myframework

import (
    "regexp"

    "github.com/Patchflow-security/patchflow-cli/internal/analysis"
    "github.com/Patchflow-security/patchflow-cli/internal/sast/frameworks"
)

var allRules = []frameworks.FrameworkRule{
    {
        ID:          "PF-MYFW-SQLI-001",
        Framework:   "myframework",
        Language:    "python",
        CWE:         "CWE-89",
        Title:       "SQL injection via string interpolation in db.execute",
        Severity:    analysis.SeverityHigh,
        Confidence:  analysis.ConfidenceHigh,
        Maturity:    frameworks.MaturityExperimental,
        FileTypes:   []string{".py"},
        MatchMode:   frameworks.MatchPattern,
        Pattern:     regexp.MustCompile(`db\.execute\(.*%[sd].*request\.`),
        Recommendation: "Use parameterized queries instead of string interpolation",
    },
    {
        ID:          "PF-MYFW-SQLI-002",
        Framework:   "myframework",
        Language:    "python",
        CWE:         "CWE-89",
        Title:       "SQL injection: request data flows to db.execute",
        Severity:    analysis.SeverityHigh,
        Confidence:  analysis.ConfidenceMedium,
        Maturity:    frameworks.MaturityExperimental,
        FileTypes:   []string{".py"},
        MatchMode:   frameworks.MatchTaint,
        Sources:     []frameworks.SourcePattern{{Func: "request.GET"}, {Func: "request.POST"}},
        Sinks:       []frameworks.SinkPattern{{Func: "db.execute"}},
        Sanitizers:  []frameworks.SanitizerPattern{{Func: "escape"}},
        Recommendation: "Use parameterized queries and validate input",
    },
    {
        ID:          "PF-MYFW-XSS-001",
        Framework:   "myframework",
        Language:    "python",
        CWE:         "CWE-79",
        Title:       "Unescaped output in template",
        Severity:    analysis.SeverityMedium,
        Confidence:  analysis.ConfidenceMedium,
        Maturity:    frameworks.MaturityExperimental,
        FileTypes:   []string{".html"},
        TemplateTypes: []string{".html"},
        MatchMode:   frameworks.MatchTemplate,
        Pattern:     regexp.MustCompile(`\{\{.*\|safe.*\}\}`),
        Recommendation: "Avoid the |safe filter unless input is sanitized",
    },
}
```

## Step 7: Register the Pack

Add the pack to `internal/sast/frameworks/packs/default_registry.go`:

```go
func BuildDefaultRegistry() *Registry {
    reg := NewRegistry()
    reg.Register(rails.New())
    reg.Register(express.New())
    // ...
    reg.Register(myframework.New())  // Add your pack
    return reg
}
```

## Step 8: Add Detection Signals

Add framework detection in `internal/frameworks/signatures.go`:

```go
{
    Name:       "myframework",
    Language:   "python",
    MinSignals: 2,
    Signals: []Signal{
        {Kind: SignalFileContains, Path: "requirements.txt", Contains: "myframework"},
        {Kind: SignalFileContains, Path: "pyproject.toml", Contains: "myframework"},
        {Kind: SignalFilePresent, Path: "myframework_settings.py"},
        {Kind: SignalGlobMatch, Glob: "templates/**/*.html"},
    },
},
```

Also add the framework name constant:

```go
const NameMyFramework = "myframework"
```

## Step 9: Add Test Fixtures

Create `pack_test.go` with test fixtures:

### Vulnerable Fixture

```go
func TestSQLInjectionVulnerable(t *testing.T) {
    code := `
import myframework

@app.route("/")
def handler(request):
    name = request.GET["name"]
    db.execute("SELECT * FROM users WHERE name = '%s'" % name)
`
    findings := runScan(t, code, "test.py")
    assertFinding(t, findings, "PF-MYFW-SQLI-001")
}
```

### Safe Fixture

```go
func TestSQLInjectionSafe(t *testing.T) {
    code := `
import myframework

@app.route("/")
def handler(request):
    name = request.GET["name"]
    db.execute("SELECT * FROM users WHERE name = ?", (name,))
`
    findings := runScan(t, code, "test.py")
    assertNoFinding(t, findings, "PF-MYFW-SQLI-001")
}
```

### Normal No-Noise Fixture

```go
func TestNoFalsePositives(t *testing.T) {
    code := `
import myframework

@app.route("/")
def handler(request):
    # Normal framework usage that should not trigger rules
    users = db.query("SELECT * FROM users")
    return render_template("users.html", users=users)
`
    findings := runScan(t, code, "test.py")
    assertNoFindings(t, findings)
}
```

## Step 10: Run Tests

```bash
# Run pack tests
go test ./internal/sast/frameworks/myframework/ -v

# Run framework foundation tests
go test ./internal/frameworks/ ./internal/sast/frameworks/ ./internal/sast/frameworks/packs/ -v

# Run all SAST tests
go test ./internal/sast/... -count=1
```

## Step 11: Run Benchmarks

Test against real repositories:

```bash
go build -o patchflow .
./patchflow benchmark run benchmarks/framework-pack-validation.yaml --no-tools
```

## Step 12: Set Maturity

Set `Maturity: frameworks.MaturityExperimental` for all rules. Experimental rules
are only visible in the `audit` governance profile and are non-blocking.

Promote rules to `beta` or `stable` as they gain:

- Positive fixtures (vulnerable code that triggers the rule)
- Safe fixtures (safe code that does not trigger the rule)
- Normal no-noise fixtures (real-world code that does not trigger the rule)
- Real benchmark repo coverage
- Framework-specific recall metrics
- Clean-repo noise checks

## Checklist

- [ ] Pack directory created with `pack.go`, `sources.go`, `sinks.go`,
      `sanitizers.go`, `rules.go`
- [ ] Pack interface implemented (Name, Language, FileExtensions,
      TemplateExtensions, Rules, Sources, Sinks, Sanitizers)
- [ ] Pack registered in `default_registry.go`
- [ ] Detection signals added in `signatures.go`
- [ ] Vulnerable fixtures added
- [ ] Safe fixtures added
- [ ] Normal no-noise fixtures added
- [ ] Tests pass (`go test ./internal/sast/frameworks/<name>/ -v`)
- [ ] All SAST tests pass (`go test ./internal/sast/... -count=1`)
- [ ] Benchmark suite run against real repos
- [ ] Maturity set to `experimental`
- [ ] No false positives on clean repos

## Next Steps

- [Framework Rule Packs](./rule-packs.md) — Pack design guide
- [Benchmarks](./benchmarks.md) — Benchmark suite
- [Rule Governance](../reference/rules-governance.md) — Maturity levels
