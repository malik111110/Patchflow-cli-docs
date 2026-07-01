# Framework Rule Packs

Framework packs are official embedded rule sets. User YAML can extend them,
but the shipped pack remains the source of truth.

## Pack Structure

Each pack follows this shape:

```text
internal/sast/frameworks/<pack>/
  pack.go
  sources.go
  sinks.go
  sanitizers.go
  rules.go
  pack_test.go
```

## Add A Pack

1. Create the pack directory.
2. Implement `frameworks.Pack`.
3. Add source, sink, and sanitizer catalogs.
4. Add pattern, template, or taint rules.
5. Register the pack in `internal/sast/frameworks/packs/default_registry.go`.
6. Add framework detection in `internal/frameworks/signatures.go`.
7. Add vulnerable, safe, and normal fixture tests.
8. Run the benchmark suite against real repos.

## Rule Design

Use the narrowest rule that catches the intended vulnerability:

| Rule Type | Good Use |
| --- | --- |
| Pattern | One-line dangerous framework API usage |
| Template | Template output and escaping problems |
| Taint | Source-to-sink flow across variables or helper functions |
| AST | Structured call shapes that regex cannot model safely |

## Noise Controls

Use these in order:

1. Make the regex more specific.
2. Restrict `FileTypes` or `TemplateTypes`.
3. Add `SafePatterns`.
4. Add `Sanitizers`.
5. Add `Exclusions` for framework source, docs, generated files, or tests.

## Benchmark Rule

A pack is not ready for promotion until it has:

- Positive fixtures
- Safe fixtures
- Normal no-noise fixtures
- Real benchmark repo coverage
- Framework-specific recall metrics
- Clean-repo noise checks
