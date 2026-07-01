# Framework Rule Packs

Framework packs are official embedded rule sets. User YAML can extend them, but
the shipped pack remains the source of truth.

## Pack Structure

Each pack follows this directory structure:

```text
internal/sast/frameworks/<pack>/
  pack.go           Implements frameworks.Pack interface
  sources.go        Source patterns (where tainted data enters)
  sinks.go          Sink patterns (where tainted data is used dangerously)
  sanitizers.go     Sanitizer patterns (functions that neutralize taint)
  templates.go      Template-specific rules (for MatchTemplate)
  rules.go          All framework rules for the pack
  pack_test.go      Test fixtures and validation
```

## Pack Interface

Every pack implements the `frameworks.Pack` interface:

```go
type Pack interface {
    Name() string
    Language() string
    FileExtensions() []string
    TemplateExtensions() []string
    Rules() []FrameworkRule
    Sources() []SourcePattern
    Sinks() []SinkPattern
    Sanitizers() []SanitizerPattern
}
```

- `Name()`: Canonical pack name (e.g., `rails`, `express`)
- `Language()`: Primary language (e.g., `ruby`, `javascript`)
- `FileExtensions()`: File extensions the pack processes (e.g., `.rb`, `.js`)
- `TemplateExtensions()`: Template extensions (e.g., `.erb`, `.ejs`)
- `Rules()`: All framework rules for the pack
- `Sources()`: Source patterns for taint rules
- `Sinks()`: Sink patterns for taint rules
- `Sanitizers()`: Sanitizer patterns that neutralize taint

## Rule Design

Use the narrowest rule that catches the intended vulnerability:

| Rule Type | Good Use | Example |
| --- | --- | --- |
| Pattern | One-line dangerous framework API usage | Rails `html_safe`, Spring `permitAll` |
| Template | Template output and escaping problems | ERB raw output, Jinja `\|safe` |
| Taint | Source-to-sink flow across variables or helper functions | `params` → `find_by_sql` |
| AST | Structured call shapes that regex cannot model safely | Reserved for complex structural matching |

## Example: Rails Pack Rules

### XSS Rules

- `PF-RAILS-XSS-001`: Unescaped output via `raw`/`html_safe` (MatchPattern)
- `PF-RAILS-XSS-002`: `render html:` with user input (MatchPattern)
- `PF-RAILS-XSS-003`: Raw/html_safe in ERB templates (MatchTemplate)

### SQL Injection Rules

- `PF-RAILS-SQLI-001`: `find_by_sql` with interpolated string (MatchPattern)
- `PF-RAILS-SQLI-002`: `where` with string interpolation (MatchPattern)
- `PF-RAILS-SQLI-003`: Taint rule: `params` → `find_by_sql`/`where` (MatchTaint)

### Other Rules

- `PF-RAILS-REDIRECT-001`: Open redirect via `redirect_to` (MatchPattern)
- `PF-RAILS-FILE-001`: File disclosure via `send_file` (MatchPattern)
- `PF-RAILS-DESER-001`: Unsafe deserialization via `YAML.load`/`Marshal.load`
  (MatchPattern)
- `PF-RAILS-MASS-001`: Mass assignment via `permit!` (MatchPattern)
- `PF-RAILS-AUTH-001`: Auth bypass via `skip_before_action` (MatchPattern)

## Noise Controls

When a rule produces false positives, apply these controls in order:

1. **Make the regex more specific**: Narrow the pattern to reduce matches
2. **Restrict `FileTypes` or `TemplateTypes`**: Limit which files the rule
   processes
3. **Add `SafePatterns`**: Patterns that indicate safe usage and should not
   trigger the rule
4. **Add `Sanitizers`**: Functions or patterns that neutralize the taint
5. **Add `Exclusions`**: Paths or patterns to exclude (framework source, docs,
   generated files, tests)

## Benchmark Rule

A pack is not ready for promotion until it has:

- **Positive fixtures**: Code that should trigger the rule
- **Safe fixtures**: Code that should not trigger the rule (safe patterns,
  sanitized inputs)
- **Normal no-noise fixtures**: Real-world code that should not trigger the rule
- **Real benchmark repo coverage**: Tested against real repositories
- **Framework-specific recall metrics**: Measured recall on known vulnerabilities
- **Clean-repo noise checks**: No false positives on clean repositories

## Pack Registration

Packs are registered in `internal/sast/frameworks/packs/default_registry.go`:

```go
func BuildDefaultRegistry() *Registry {
    reg := NewRegistry()
    reg.Register(rails.New())
    reg.Register(express.New())
    reg.Register(django.New())
    // ... all 17 packs
    return reg
}
```

## Framework Detection

Framework detection signals are defined in
`internal/frameworks/signatures.go`:

```go
{
    Name:       NameRails,
    Language:   "ruby",
    MinSignals: 2,
    Signals: []Signal{
        {Kind: SignalFileContains, Path: "Gemfile", Contains: "rails"},
        {Kind: SignalFileContains, Path: "Gemfile.lock", Contains: "rails"},
        {Kind: SignalFilePresent, Path: "config/routes.rb"},
        {Kind: SignalFilePresent, Path: "app/controllers"},
        {Kind: SignalGlobMatch, Glob: "app/views/**/*.erb"},
    },
}
```

Each framework has a `MinSignals` threshold (typically 2) to avoid false
positives. Multiple signal types are supported:

- `SignalFileContains`: File exists and contains specific text
- `SignalFilePresent`: File or directory exists
- `SignalGlobMatch`: Glob pattern matches at least one file

## Maturity Bridge

The `packs` subpackage bridges `frameworks.Maturity` → `rules.Maturity` via
`ToRulesMaturity`:

```go
func ToRulesMaturity(m frameworks.Maturity) rules.Maturity {
    switch m {
    case frameworks.MaturityExperimental:
        return rules.MaturityExperimental
    case frameworks.MaturityBeta:
        return rules.MaturityBeta
    case frameworks.MaturityStable:
        return rules.MaturityStable
    case frameworks.MaturityEnterprise:
        return rules.MaturityEnterprise
    default:
        return rules.MaturityExperimental
    }
}
```

This bridge exists to avoid an import cycle between `internal/sast/frameworks`
and `internal/rules`.

## Next Steps

- [Adding a Framework Pack](./adding-packs.md) — Step-by-step pack creation
- [Architecture](./architecture.md) — Detailed architecture
- [Rule Governance](../reference/rules-governance.md) — Maturity and profiles
