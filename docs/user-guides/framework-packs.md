---
title: Framework Packs
description: Official embedded framework rule packs for PatchFlow CLI
---

# Framework Packs

Framework packs are official embedded rule sets that understand framework-specific
sources, sinks, sanitizers, and template engines.

See the [Framework Packs reference](../frameworks/index) for the complete list of
all 18 packs with full rule tables, sources, sinks, and sanitizers organized by
language.

## Quick Reference

| Pack | Language | Rules | Key Coverage |
| --- | --- | --- | --- |
| Spring | Java | 31 | SQLi, SSRF, redirect, XXE, deserialization, XSS, command injection, path traversal, auth bypass |
| Spring Security | Java | 4 | CSRF disabled, permitAll, @PermitAll |
| Express | JavaScript | 9 | SQLi, XSS, redirect, command injection, path traversal, NoSQL injection, SSRF |
| React | JavaScript | 4 | XSS, redirect, DOM injection, insecure storage |
| Next.js | JavaScript | 4 | SSRF, redirect, XSS, secret exposure |
| Angular | TypeScript | 5 | XSS, redirect, template XSS |
| NestJS | TypeScript | 4 | SQLi, SSRF, redirect, auth |
| Django | Python | 7 | SQLi, XSS, redirect, deserialization, CSRF, SSRF |
| Flask | Python | 9 | SQLi, SSRF, redirect, XSS, SSTI, path traversal, debug mode |
| FastAPI | Python | 9 | SQLi, SSRF, redirect, command injection, path traversal, XSS, auth |
| GraphQL | Python | 5 | SQLi, SSRF, path traversal, IDOR, DoS |
| Rails | Ruby | 15 | XSS, SQLi, redirect, file disclosure, deserialization, mass assignment, auth bypass |
| Laravel | PHP | 6 | SQLi, redirect, XSS, mass assignment, deserialization, auth |
| Symfony | PHP | 3 | SQLi, redirect, Twig XSS |
| ASP.NET | C# | 8 | SQLi, redirect, XSS, deserialization, command injection, path traversal |
| Razor | C# | 2 | XSS (Html.Raw, MarkupString) |
| Gin | Go | 4 | SQLi, redirect, XSS, command injection |
| Echo | Go | 3 | SQLi, redirect, XSS |

## Using Framework Packs

```bash
# Auto-detect and scan (default)
patchflow scan run

# Force-enable a specific pack
patchflow scan run --framework express

# Combine multiple packs
patchflow scan run --framework express --framework react --framework nextjs

# Disable a pack
patchflow scan run --disable-framework spring-security

# List all packs and detection status
patchflow rules list-frameworks
```

## Extending Packs

You can extend any official pack with organization-specific sources, sinks,
sanitizers, and safe patterns. See
[Custom Framework Extensions](./custom-framework-extensions) for the full guide.
