---
title: Framework Packs
description: Official embedded framework rule packs for PatchFlow CLI
---

# Framework Packs

Framework packs are official embedded rule sets that understand framework-specific
sources, sinks, sanitizers, and template engines. They run alongside the general
SAST engines and provide framework-aware vulnerability detection.

## How Framework Packs Work

```
1. Detect frameworks    internal/frameworks.Detector via filesystem signals
2. Select packs         frameworks.Loader based on detection + CLI config
3. Pattern/template     frameworks.Matcher (line-oriented, sanitizer-aware)
4. Taint rules          Registered into taintpatterns.Analyzer (source→sink)
5. Deduplicate          SAST runner deduplicates findings across engines
```

1. **Detection**: PatchFlow scans the filesystem for framework signals (config
   files, directory structures, manifest contents). Each framework has a
   `MinSignals` threshold (typically 2) to avoid false positives.
2. **Selection**: Based on detection results and CLI flags, PatchFlow activates
   the appropriate packs.
3. **Matching**: Pattern and template rules are matched line-by-line with
   sanitizer awareness. Taint rules are registered into the taint analysis
   engine for source-to-sink tracking.
4. **Deduplication**: The SAST runner deduplicates findings across all engines.

## Enabling Packs

### Auto-detection (default)

```bash
patchflow scan run --framework auto
```

PatchFlow auto-detects frameworks based on filesystem signals and activates the
appropriate packs. This is the default behavior when no `--framework` flag is
provided.

### Force-enable specific packs

```bash
patchflow scan run --framework express
patchflow scan run --framework express --framework react --framework nextjs
```

### Disable a pack

```bash
patchflow scan run --disable-framework spring-security
```

`--disable-framework` takes precedence over `--framework`. Use this when
auto-detection is too broad or when a pack is not relevant to your project.

## Official Packs

| Pack | Language | File Extensions | Template Extensions | Coverage |
| --- | --- | --- | --- | --- |
| `rails` | Ruby | `.rb` | `.erb`, `.rhtml`, `.haml`, `.slim` | Controllers, ERB, redirects, SQL, XSS, mass assignment, deserialization |
| `express` | JavaScript | `.js`, `.mjs`, `.cjs`, `.ts` | `.ejs`, `.hbs`, `.pug` | Request sources, redirects, response XSS, SQL, file access |
| `nextjs` | JavaScript | `.js`, `.jsx`, `.ts`, `.tsx` | `.jsx`, `.tsx` | Server-side fetch, redirects, JSX XSS |
| `react` | JavaScript | `.jsx`, `.tsx` | `.jsx`, `.tsx` | JSX XSS, client-side navigation |
| `spring` | Java | `.java` | `.jsp`, `.jspx`, `.ftl`, `.vm`, `.html`, `.thymeleaf.html` | SQL, SSRF, redirects, XXE, templates, command execution |
| `spring-security` | Java | `.java` | — | CSRF disabled, public sensitive routes |
| `aspnet` | C# | `.cs` | `.cshtml`, `.razor` | SQL, redirects, raw HTML responses |
| `razor` | C# | `.cshtml`, `.razor` | `.cshtml`, `.razor` | Html.Raw, MarkupString |
| `django` | Python | `.py` | `.html`, `.jinja`, `.jinja2` | SQL, redirects, template safe filter, unsafe deserialization |
| `laravel` | PHP | `.php` | `.blade.php`, `.twig`, `.phtml` | Raw SQL, redirects, Blade unescaped output, mass assignment |
| `fastapi` | Python | `.py` | `.html`, `.jinja`, `.jinja2` | Request sources, SQL, redirects, templates |
| `gin` | JavaScript | `.js`, `.mjs`, `.cjs`, `.ts` | `.html` | Request sources, SQL, redirects, command execution |
| `flask` | Python | `.py` | `.html`, `.jinja`, `.jinja2` | SQL, redirects, Jinja safe filter, dynamic template strings |
| `symfony` | PHP | `.php` | `.twig`, `.php` | SQL, redirects, Twig unsafe output |
| `angular` | TypeScript | `.ts` | `.html` | Template and DOM sink patterns |
| `nestjs` | TypeScript | `.ts` | — | Controller sources, redirects, command patterns |
| `echo` | JavaScript | `.js`, `.mjs`, `.cjs`, `.ts` | `.html` | Request sources, redirects, response output |

## Rule Match Modes

Each framework rule declares a `MatchMode` that determines how it is evaluated:

| MatchMode | Description | Example |
| --- | --- | --- |
| `MatchPattern` | Regex against source lines (fast, line-oriented) | Rails `html_safe`, Spring `permitAll` |
| `MatchAST` | Framework-specific call structures via tree-sitter | Reserved for structural matching |
| `MatchTaint` | Source-to-sink taint tracking with sanitizers | `params` → `find_by_sql` flow |
| `MatchTemplate` | Template engine output issues | ERB raw output, Jinja `\|safe`, Razor `@Html.Raw`, Blade `{!! !!}`, JSX `dangerouslySetInnerHTML` |

## Inspecting Packs

### List all packs and detection status

```bash
patchflow rules list-frameworks
```

Shows each pack with its name, language, file/template extensions, rule count,
and whether it would be auto-detected in the current project.

### List rules in a specific pack

```bash
patchflow rules list --framework express
```

### Explain a framework rule

```bash
patchflow explain --rule PF-EXPRESS-REDIRECT-001
```

Shows the rule's sources, sinks, sanitizers, safe patterns, exclusions, and
suppression format. See [Explain](./explain) for details.

## Configuring Packs in YAML

`.patchflow/rules.yaml` can extend official packs with application-specific
sources, sinks, sanitizers, and severity policy:

```yaml
frameworks:
  auto_detect: true
  enabled:
    - express
    - react
  disabled:
    - spring-security

framework_overrides:
  express:
    severity_overrides:
      PF-EXPRESS-REDIRECT-001: high
    custom_sanitizers:
      - func: isSafeRedirect
      - regex: "allowlistedHost\\("
  rails:
    custom_sources:
      - func: current_tenant_param
    custom_sinks:
      - func: render_raw_html
```

### Framework Selection

```yaml
frameworks:
  auto_detect: true        # Auto-detect frameworks (default: true)
  enabled:                 # Force-enable packs
    - express
    - django
  disabled:                # Disable packs even if detected
    - spring-security
```

Use explicit `enabled` packs for monorepos or generated layouts where
auto-detection is incomplete.

### Severity Overrides

```yaml
framework_overrides:
  django:
    severity_overrides:
      PF-DJANGO-XSS-002: medium
```

Severity overrides are useful when a team has compensating controls or a
stricter internal policy than the default pack.

### Custom Sanitizers

```yaml
framework_overrides:
  express:
    custom_sanitizers:
      - func: validateRedirectTarget
      - regex: "allowlistedHost\\("
```

Use sanitizers to reduce false positives for established internal helpers. When
a sanitizer is detected in the code path, the finding is suppressed.

### Custom Sources and Sinks

```yaml
framework_overrides:
  rails:
    custom_sources:
      - func: current_tenant_param
    custom_sinks:
      - func: render_raw_html
```

Add custom sources and sinks when an application wraps framework APIs behind
internal helpers. This enables taint rules to track data flow through
application-specific functions.

## Governance

Framework rules register under the `framework-packs` engine in the governance
registry. Default maturity is `experimental` (audit profile only, non-blocking).
Rules are promoted to `beta` or `stable` as they gain test fixtures and
regression coverage, which activates them in PR/CI profiles and makes high and
critical findings blocking-eligible.

See [Rule Governance](../reference/rules-governance) for maturity levels and
profiles.

## Recommended Policy

Start with official packs and only add overrides after a finding has been
triaged. Keep each override small, named, and reviewed like production code.

## Next Steps

- [Custom Rules](./custom-rules) — Full YAML policy reference
- [Explain](./explain) — Inspect framework rules in detail
- [Rule Governance](../reference/rules-governance) — Maturity and profiles
