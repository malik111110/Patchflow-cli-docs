---
title: Framework Packs
description: 18 official embedded framework rule packs for framework-aware SAST analysis
---

# Framework Packs

Framework packs are official embedded rule sets that understand framework-specific
sources, sinks, sanitizers, and template engines. They run alongside the general
SAST engines and provide framework-aware vulnerability detection.

## How It Works

```
1. Detect frameworks    Filesystem signals (config files, manifests, directories)
2. Select packs         Detection results + CLI flags (--framework / --disable-framework)
3. Match patterns       Line-oriented matching with sanitizer awareness
4. Track taint          Source → sink taint analysis through the taint engine
5. Check safe patterns  Suppress findings when safe patterns are present
6. Deduplicate          SAST runner deduplicates findings across all engines
```

## Enabling Packs

### Auto-detection (default)

```bash
patchflow scan run
```

PatchFlow auto-detects frameworks based on filesystem signals and activates the
appropriate packs automatically.

### Force-enable specific packs

```bash
patchflow scan run --framework express
patchflow scan run --framework express --framework react --framework nextjs
```

### Disable a pack

```bash
patchflow scan run --disable-framework spring-security
```

`--disable-framework` takes precedence over `--framework`.

### List all packs and detection status

```bash
patchflow rules list-frameworks
```

## All 18 Framework Packs

| Pack | Language | Rules | File Extensions | Key Coverage |
| --- | --- | --- | --- | --- |
| [Spring](./spring) | Java | 31 | `.java` | SQLi, SSRF, redirect, XXE, deserialization, XSS, command injection, path traversal, auth bypass |
| [Spring Security](./spring-security) | Java | 4 | `.java` | CSRF disabled, permitAll on sensitive routes, @PermitAll |
| [Express](./express) | JavaScript | 9 | `.js`, `.mjs`, `.cjs`, `.ts` | SQLi, XSS, redirect, command injection, path traversal, NoSQL injection, SSRF |
| [Django](./django) | Python | 7 | `.py` | SQLi, XSS, redirect, deserialization, CSRF, SSRF |
| [Flask](./flask) | Python | 9 | `.py` | SQLi, SSRF, redirect, XSS, SSTI, path traversal, debug mode |
| [FastAPI](./fastapi) | Python | 9 | `.py` | SQLi, SSRF, redirect, command injection, path traversal, XSS, auth |
| [Rails](./rails) | Ruby | 15 | `.rb` | XSS, SQLi, redirect, file disclosure, deserialization, mass assignment, auth bypass |
| [Laravel](./laravel) | PHP | 6 | `.php` | SQLi, redirect, XSS, mass assignment, deserialization, auth |
| [Symfony](./symfony) | PHP | 3 | `.php` | SQLi, redirect, Twig XSS |
| [React](./react) | JavaScript | 4 | `.jsx`, `.tsx` | XSS (dangerouslySetInnerHTML), redirect, DOM injection, insecure storage |
| [Next.js](./nextjs) | JavaScript | 4 | `.js`, `.jsx`, `.ts`, `.tsx` | SSRF, redirect, XSS, secret exposure |
| [Angular](./angular) | TypeScript | 5 | `.ts` | XSS (bypassSecurityTrust), redirect, template XSS |
| [NestJS](./nestjs) | TypeScript | 4 | `.ts` | SQLi, SSRF, redirect, auth |
| [GraphQL](./graphql) | Python | 5 | `.py` | SQLi, SSRF, path traversal, IDOR, DoS |
| [ASP.NET](./aspnet) | C# | 8 | `.cs` | SQLi, redirect, XSS, deserialization, command injection, path traversal |
| [Razor](./razor) | C# | 2 | `.cshtml`, `.razor` | XSS (Html.Raw, MarkupString) |
| [Gin](./gin) | Go | 4 | `.go` | SQLi, redirect, XSS, command injection |
| [Echo](./echo) | Go | 3 | `.go` | SQLi, redirect, XSS |

## Rule Match Modes

Each rule uses one of four match modes:

| Mode | Description | Example |
| --- | --- | --- |
| `pattern` | Regex against source lines (simple dangerous APIs) | `JdbcTemplate.query` with string concatenation |
| `template` | Template engine output issues (ERB, Jinja, Blade, JSX) | `th:utext` with user input in Thymeleaf |
| `taint` | Source → sink taint tracking through the taint engine | `@RequestParam` → `jdbcTemplate.query` |
| `AST` | Framework-specific call structures (tree-sitter, reserved) | Reserved for future use |

## Rule Maturity

| Level | Meaning | Default in profiles |
| --- | --- | --- |
| `experimental` | New rule, limited test coverage | audit only |
| `beta` | Tested against fixtures, some regression coverage | dev, pr, audit |
| `stable` | Full test coverage, regression tested, low false-positive rate | all profiles |
| `enterprise` | Validated against enterprise codebases | all profiles |

## CWE Coverage

Framework packs cover 17 CWE categories:

| CWE | Category | Rules |
| --- | --- | --- |
| CWE-89 | SQL Injection | 22 |
| CWE-79 | Cross-Site Scripting (XSS) | 18 |
| CWE-601 | Open Redirect | 15 |
| CWE-918 | Server-Side Request Forgery (SSRF) | 10 |
| CWE-78 | Command Injection | 7 |
| CWE-502 | Unsafe Deserialization | 6 |
| CWE-22 | Path Traversal | 6 |
| CWE-306 | Missing Authentication | 5 |
| CWE-352 | CSRF | 3 |
| CWE-611 | XXE | 2 |
| CWE-639 | IDOR | 1 |
| CWE-915 | Mass Assignment | 2 |
| CWE-400 | DoS | 1 |
| CWE-94 | SSTI | 1 |
| CWE-489 | Debug Mode | 1 |
| CWE-200 | Secret Exposure | 1 |
| CWE-922 | Insecure Storage | 1 |
| CWE-943 | NoSQL Injection | 1 |

## Custom Framework Extensions

You can extend any official pack with organization-specific sources, sinks,
sanitizers, and safe patterns. See the
[Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for details.

## Explaining Rules

Get detailed information about any rule, including its sources, sinks,
sanitizers, and safe patterns:

```bash
patchflow explain --rule PF-SPRING-SQLI-001
```

## Listing Rules

List rules for a specific framework:

```bash
patchflow rules list --framework spring
```

List all rules across all scanners:

```bash
patchflow rules list --all
```
